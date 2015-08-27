---
layout: post
author: zhao
title:  Linux��ClusterShell����
date:  2014-10-23 22:14:54
categories: Linux
---

* content
{:toc}


##���

ʵ���һ����д�Ű�̨�ķ�������Ҫ����������Ҫ�Hadoop�Լ�Spark��Ⱥ�ȣ���ˣ�һ���������ļ�Ⱥ����������Ե÷ǳ��б�Ҫ�ˡ�����һ��ʱ����˽��Լ����ԣ�����ѡ����clustershell��������ԭ�����£�

- ��װ���㡣һ��ָ��������ɰ�װ��

- ���÷��㡣�ܶ༯Ⱥ�����������Ҫ�����еķ������϶���װ��������һ�Ҫ���кܶ�����Ӳ�����clustershell���൱�ķ����ˣ�������Ҫ���л����ܹ�ssh�������¼���ɣ�Ȼ��ֻ��һ̨�������ϰ�װclustershell���ɡ�����һ�����ú�Hadoop��Ⱥ�������ʹ�ã��൱���㣩

- ʹ�÷��㡣clustershell�����������˵�ǳ��򵥣�ֻ��һ����ָ���Լ����ĸ�������Ҫ�ǡ�

##��װ

###��װclustershell

��װ�ǳ��򵥣�ֻ��һ��ָ��ɣ�һ����������Ǻ�ñϵ�еģ�ʹ��yum��װ��

~~~
yum install clustershell
~~~

###����ssh�������¼

����ssh��¼��ԱȽϼ򵥣��ڴhadoop��Ⱥ��ʱ�򶼻���Ҫ��һ����

###����/etc/hosts

��hosts���ļ��н�ip����������Ӧ������ʹ�ñȽϷ��㡣

###���ùؼ��ļ�

clustershell�������ļ���/etc/clustershellĿ¼�£����е�groups����õģ���ֻ��������һ���ļ���

`all: z[1-4]`��ָ���еĽڵ㣬��ʹ�õ���ͨ��`-a`��ѡ��all

`hadoop: z[1-4]`����ָ��hadoop�������ĸ��ڵ㣬�ֱ���z1��z2��z3��z4������������Ҳ���ƣ����Լ������飬ʹ�õ�ʱ��ͨ��`-g hadoop`��ѡ��

~~~

adm: example0
oss: example4 example5
mds: example6
io: example[4-6]
compute: example[32-159]
gpu: example[156-159]
all: z[1-4]

hadoop: z[1-4]

~~~

###ʹ��

clustershell��ʹ�õ�ʱ����һ���ǳ���Ҫ��ָ�����clush��ĿǰΪֹ��Ҳֻ�õ�����һ��ָ�

clush [-option] ��������ճ���linux��ִ�е�ָ��ɣ�ûʲô���ӵģ���ʮ�ּ򵥡�

������һ��Ҫע�⣬clustershellִ�е�������һ�β�����ָ����������touchһ�����ļ������нڵ��ϣ������㲻��ͬʱ�����нڵ���vim�༭һ�����ļ���ϸ�ڻ�����ĥ��

clush�м����Ƚ���Ҫ�Ĳ�����
 
- -b : ��ͬ�������ϲ�
- -w : ָ���ڵ�
- -a : ���нڵ�
- -g : ָ����
- --copy : Ⱥ���ļ�


####������нڵ��JAVA_HOME��Ϣ

~~~
[hadoop@z1 ~]$ clush -b -a echo $JAVA_HOME
---------------
z[1-4] (4)
---------------
/mnt/zhao/soft/jdk
~~~

####ɾ��ָ���ڵ���ļ�

ɾ�� z2,z3,z4�����ڵ��ϵ�/mnt/zhao/soft/jdk�ļ���

~~~~
 clush -w z2,z3,z4 rm -rf /mnt/zhao/soft/jdk
~~~~


####��Ⱥ�ַ��ļ�

�ѱ��ص�һ��/mnt/zhao/package/jdk-7u79-linux-x64.tar.gz�ļ��ַ���hadoop�������нڵ��/mnt/zhao/package/Ŀ¼��

~~~
[hadoop@z1 ~]$ clush -b -g hadoop --copy /mnt/zhao/package/jdk-7u79-linux-x64.tar.gz --dest /mnt/zhao/package/
~~~

####��Ⱥ�鿴�ļ�

�鿴����hadoop����/mnt/zhao/package/Ŀ¼�µ��ļ����������ϲ���

~~~
[hadoop@z1 package]$ clush -b -g hadoop ls /mnt/zhao/package/
---------------
z[2-4] (3)
---------------
jdk-7u79-linux-x64.tar.gz
---------------
z1
---------------
clustershell-1.6.tar.gz
hadoop-2.7.1.tar.gz
jdk-7u79-linux-x64.tar.gz
~~~

####�ٷ��ĵ�

clustershell���кܶ๦�ܣ�������������ѧϰ��Ŀǰ�����õ��Ĺ��������������ˣ�������Ļ�����ѧϰ��һ�㡣�ϴ�һ���ٷ��ĵ���������ѧϰclustershell����������һ�¡�

http://download.csdn.net/detail/picassolovecoding/8073989
