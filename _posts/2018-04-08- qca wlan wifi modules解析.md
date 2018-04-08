---
layout:     post
title:      qca wlan wifi modules解析一
subtitle:   qca wlan wifi modules
date:       2018-04-04
author:     hades
header-img: img/my-blog-picture.jpg
catalog: true
tags:
    - hades
---

# qca wlan wifi modules解析一

分析lsdk-ap121  lsdk-ap134
源码： https://github.com/hades13/lsdk_ar9531
包含wifi drivers 

另一wifi drivers版本： 
https://download.csdn.net/download/nolycjyf/3722308


目录结果：
```
apps
build 
drivers 
include 
modules 
boot  
docs   
images   
linux    
patches
rootfs
```
wifi drivers 在drivers目录下： 
```
drivers\
    firmware\  
    wlan_modules\
            adf  
            asf  
            hal  
            include  
            LicenseChoice.txt  
            lmac  
            Notice.txt  
            os  
            smartantenna  
            umac  
            wow
```

编译之后生成内核ko文件，在启动时会insmod，在文件
`rootfs/board953x/etc/rc.d/rc.wlan` 

rc.wlan启动时执行： 
```
#
# Finally, insert the modules
#
    insmod $MODULE_PATH/adf.ko
    insmod $MODULE_PATH/asf.ko
    insmod $MODULE_PATH/ath_hal.ko
    insmod $MODULE_PATH/ath_rate_atheros.ko
    insmod $MODULE_PATH/ath_spectral.ko $SPECTRAL_ARGS
    if [ "${AP_NO_A_BAND}" != "1" ]; then
        #load DFS if A band is supported,default is supported and set AP_NO_A_BAND=1 if not supported
        insmod $MODULE_PATH/ath_dfs.ko $DFS_ARGS
    fi
    insmod $MODULE_PATH/hst_tx99.ko
    insmod $MODULE_PATH/ath_dev.ko
    insmod $MODULE_PATH/umac.ko
    insmod $MODULE_PATH/wlan_me.ko
    insmod $MODULE_PATH/ath_pktlog.ko
```

查看编译后生成的文件： 
```
./asf/asf.ko
./os/linux/ath_hal/ath_hal.ko
./smartantenna/smart_antenna.ko
./adf/adf.ko
./lmac/ath_dev/ath_dev.ko
./lmac/ratectrl/ath_rate_atheros.ko
./lmac/ath_pktlog/ath_pktlog.ko
./umac/umac.ko
```
这是部分wifi驱动的内容，完整的架构包含以下内容： 
asf : Atheros Service Framework (ASF)
qdf : Qualcomm Driver Framework
lmac : LMAC (Lower Media Access Controller)
umac : UMAC (Upper Media Access Controller)
wow : The Wake on Wireless

```
1. asf.ko – Basic Framework module
2. qdf.ko – Basic Framework module
3. ath_spectral.ko – Spectral Support
4. ath_dfs.ko – DFS support
5. umac.ko – Common 802.11 protocol Management
6. ath_hal.ko – Direct-Attach HW abstraction Layer
7. ath_rate_atheros.ko – Direct-Attach Rate Control Support
8. hst_tx99.ko – Direct-Attach Tx99 Support
9. ath_dev.ko – Direct-Attach LMAC Layer
10. qca_da.ko – Direct-Attach Driver Support
11. qca_ol.ko – Offload Driver Support
12. smart_antenna.ko – Smart Antenna Support
13. ath_pktlog.ko – Direct-Attach Packet logging Support
```
驱动的层次结构如下： 

![](https://user-gold-cdn.xitu.io/2018/4/3/16289e58d999a8fb?w=885&h=444&f=png&s=81184)

驱动结构如下： 

![](https://user-gold-cdn.xitu.io/2018/4/3/16289e5e626deb31?w=572&h=381&f=png&s=23298)


