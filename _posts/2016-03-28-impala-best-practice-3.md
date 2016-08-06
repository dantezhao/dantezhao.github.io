---
layout: post
title:  "Impala实践之三：详解invalidate metadata"
categories: 数据平台
tags: impala hive 元数据
---

* content
{:toc}

## 前言

这次主要是想通过源码找到`invalidate metadata`执行的原理，由于不太懂java和c++的互调的细节，目前只能查找到如下阶段，后面会专门看一下java的jni以及thrift的原理。




## 分析

目前主要定位到三个类，`com.cloudera.impala.service.JniCatalog `,`com.cloudera.impala.service.CatalogOpExecutor`和`com.cloudera.impala.catalog.CatalogServiceCatalog`。

大致的过程就是在JniCatalog中调用了CatalogOpExecutor的execResetMetadata()方法，然后再判断参数，如果没有表参数的时候会执行CatalogServiceCatalog的reset方法，就完成了一次`invalidate metadata`操作。

### com.cloudera.impala.service.JniCatalog

```
public class JniCatalog {
  private final CatalogOpExecutor catalogOpExecutor_;
  public byte[] resetMetadata(byte[] thriftResetMetadataReq)
      throws ImpalaException, TException {
    TResetMetadataRequest req = new TResetMetadataRequest();
    JniUtil.deserializeThrift(protocolFactory_, req, thriftResetMetadataReq);
    TSerializer serializer = new TSerializer(protocolFactory_);
    return serializer.serialize(catalogOpExecutor_.execResetMetadata(req));
  }
}
```


###  com.cloudera.impala.service.CatalogOpExecutor

这个方法的最后一部分有一句注释"Invalidate the entire catalog if no table name is provided."，可以看出，当使用`invalidate metadata`没有带参数的时候，会调用CatalogServiceCatalo的reset方法。

```
public class CatalogOpExecutor {

  //这里新建了CatalogServiceCatalog对象
  private final CatalogServiceCatalog catalog_;

/**
   * Executes a TResetMetadataRequest and returns the result as a
   * TResetMetadataResponse. Based on the request parameters, this operation
   * may do one of three things:
   * 1) invalidate the entire catalog, forcing the metadata for all catalog
   *    objects to be reloaded.
   * 2) invalidate a specific table, forcing the metadata to be reloaded
   *    on the next access.
   * 3) perform a synchronous incremental refresh of a specific table.
   *
   * For details on the specific commands see comments on their respective
   * methods in CatalogServiceCatalog.java.
   */
  public TResetMetadataResponse execResetMetadata(TResetMetadataRequest req)
      throws CatalogException {
    TResetMetadataResponse resp = new TResetMetadataResponse();
    resp.setResult(new TCatalogUpdateResult());
    resp.getResult().setCatalog_service_id(JniCatalog.getServiceId());

    if (req.isSetTable_name()) {
      // Tracks any CatalogObjects updated/added/removed as a result of
      // the invalidate metadata or refresh call. For refresh() it is only expected
      // that a table be modified, but for invalidateTable() the table's parent database
      // may have also been added if it did not previously exist in the catalog.
      Pair<Db, Table> modifiedObjects = new Pair<Db, Table>(null, null);

      boolean wasRemoved = false;
      if (req.isIs_refresh()) {
        TableName tblName = TableName.fromThrift(req.getTable_name());
        Table tbl = getExistingTable(tblName.getDb(), tblName.getTbl());
        if (tbl == null) {
          modifiedObjects.second = null;
        } else {
          modifiedObjects.second = catalog_.reloadTable(tbl);
        }
      } else {
        wasRemoved = catalog_.invalidateTable(req.getTable_name(), modifiedObjects);
      }

      if (modifiedObjects.first == null) {
        TCatalogObject thriftTable = TableToTCatalogObject(modifiedObjects.second);
        if (modifiedObjects.second != null) {
          // Return the TCatalogObject in the result to indicate this request can be
          // processed as a direct DDL operation.
          if (wasRemoved) {
            resp.getResult().setRemoved_catalog_object_DEPRECATED(thriftTable);
          } else {
            resp.getResult().setUpdated_catalog_object_DEPRECATED(thriftTable);
          }
        } else {
          // Table does not exist in the meta store and Impala catalog, throw error.
          throw new TableNotFoundException("Table not found: " +
              req.getTable_name().getDb_name() + "."
              + req.getTable_name().getTable_name());
        }
        resp.getResult().setVersion(thriftTable.getCatalog_version());
      } else {
        // If there were two catalog objects modified it indicates there was an
        // "invalidateTable()" call that added a new table AND database to the catalog.
        Preconditions.checkState(!req.isIs_refresh());
        Preconditions.checkNotNull(modifiedObjects.first);
        Preconditions.checkNotNull(modifiedObjects.second);

        // The database should always have a lower catalog version than the table because
        // it needs to be created before the table can be added.
        Preconditions.checkState(modifiedObjects.first.getCatalogVersion() <
            modifiedObjects.second.getCatalogVersion());

        // Since multiple catalog objects were modified, don't treat this as a direct DDL
        // operation. Just set the overall catalog version and the impalad will wait for
        // a statestore heartbeat that contains the update.
        resp.getResult().setVersion(modifiedObjects.second.getCatalogVersion());
      }
    } else {
      // Invalidate the entire catalog if no table name is provided.
      Preconditions.checkArgument(!req.isIs_refresh());

      //reset元数据信息
      catalog_.reset();
      resp.result.setVersion(catalog_.getCatalogVersion());
    }
    resp.getResult().setStatus(
        new TStatus(TErrorCode.OK, new ArrayList<String>()));
    return resp;
  }
}
```

### com.cloudera.impala.catalog.CatalogServiceCatalog

根据猜测，reset方法中`dbCache_`应该是impalad本地的缓存，可以看到，程序先new了一下新的`newDbCache`，`ConcurrentHashMap<String, Db> newDbCache = new ConcurrentHashMap<String, Db>();`。然后在for循环中获取所有的元数据信息并put到newDbCache中，`newDbCache.put(db.getName().toLowerCase(), db);`。最后会把`newDbCache`的内容set到`dbCache_`中。

更新元数据的方式也是通过HiveClient来获取所有的信息。

比较好奇的地方是，在这里居然没有发现和分区相关的地方，所有的partition信息是通过Catalog服务自动同步的？

```
public class CatalogServiceCatalog extends Catalog {
/**
   * Resets this catalog instance by clearing all cached table and database metadata.
   */
  public void reset() throws CatalogException {
    // First update the policy metadata.
    if (sentryProxy_ != null) {
      // Sentry Service is enabled.
      try {
        // Update the authorization policy, waiting for the result to complete.
        sentryProxy_.refresh();
      } catch (Exception e) {
        throw new CatalogException("Error updating authorization policy: ", e);
      }
    }

    catalogLock_.writeLock().lock();
    try {
      nextTableId_.set(0);

      // Not all Java UDFs are persisted to the metastore. The ones which aren't
      // should be restored once the catalog has been invalidated.
      Map<String, Db> oldDbCache = dbCache_.get();

      // Build a new DB cache, populate it, and replace the existing cache in one
      // step.
      ConcurrentHashMap<String, Db> newDbCache = new ConcurrentHashMap<String, Db>();
      List<TTableName> tblsToBackgroundLoad = Lists.newArrayList();
      MetaStoreClient msClient = metaStoreClientPool_.getClient();
      try {
        for (String dbName: msClient.getHiveClient().getAllDatabases()) {
          List<org.apache.hadoop.hive.metastore.api.Function> javaFns =
              Lists.newArrayList();
          for (String javaFn: msClient.getHiveClient().getFunctions(dbName, "*")) {
            javaFns.add(msClient.getHiveClient().getFunction(dbName, javaFn));
          }
          org.apache.hadoop.hive.metastore.api.Database msDb =
              msClient.getHiveClient().getDatabase(dbName);
          Db db = new Db(dbName, this, msDb);
          // Restore UDFs that aren't persisted.
          Db oldDb = oldDbCache.get(db.getName().toLowerCase());
          if (oldDb != null) {
            for (Function fn: oldDb.getTransientFunctions()) {
              db.addFunction(fn);
              fn.setCatalogVersion(incrementAndGetCatalogVersion());
            }
          }
          loadFunctionsFromDbParams(db, msDb);
          loadJavaFunctions(db, javaFns);
          db.setCatalogVersion(incrementAndGetCatalogVersion());
          newDbCache.put(db.getName().toLowerCase(), db);

          for (String tableName: msClient.getHiveClient().getAllTables(dbName)) {
            Table incompleteTbl = IncompleteTable.createUninitializedTable(
                getNextTableId(), db, tableName);
            incompleteTbl.setCatalogVersion(incrementAndGetCatalogVersion());
            db.addTable(incompleteTbl);
            if (loadInBackground_) {
              tblsToBackgroundLoad.add(
                  new TTableName(dbName.toLowerCase(), tableName.toLowerCase()));
            }
          }
        }
      } finally {
        msClient.release();
      }
      dbCache_.set(newDbCache);
      // Submit tables for background loading.
      for (TTableName tblName: tblsToBackgroundLoad) {
        tableLoadingMgr_.backgroundLoad(tblName);
      }
    } catch (Exception e) {
      LOG.error(e);
      throw new CatalogException("Error initializing Catalog. Catalog may be empty.", e);
    } finally {
      catalogLock_.writeLock().unlock();
    }
  }
}
```


## 总结

其实自己对于这整个流程还不是特别熟悉，下一步的方向将会从sql的query传入到解析最后到执行这一整个过程来理解和学习。

主要需要学习跨语言调用，以及在监听端口的时候的数据传输等方面。


***
2016-03-28 19:08:00 hzct
