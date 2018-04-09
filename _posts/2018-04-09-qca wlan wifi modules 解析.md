---
layout:     post
title:      qca wlan wifi modules 解析四
subtitle:   qca wlan wifi modules
date:       2018-04-09
author:     hades
header-img: img/my-blog-picture.jpg
catalog: true
tags:
    - hades 
    - WiFi
    - driver
---
# qca wlan wifi modules 解析四
WiFi驱动架构的一般层次为：
- 应用层
- BSD socket层
- TCP/IP协议层
- IP层
- 网络设备层net/core
- mac8011层/ieee80211
- 设备驱动层

具体实例如下图： 

![](https://user-gold-cdn.xitu.io/2018/4/9/162a86a54b9a9db3?w=466&h=650&f=png&s=54612)

上层应用程序简历socket，对网络接口进行ioctl操作，正是通过触发，网络设备和80211层，调用底层驱动函数来实现的。

qca wlan modules中，通过创建虚拟AP来实现WiFi功能，即VAP： 
创建VAP的命令如下： 

    wlanconfig ath0 create wlandev wifi0 wlanmode ap
    
VAP网络接口是通过wifi radio网络接口的ioctl SIOC80211IFCREATE来创建的，VAP接口是wifi设备接口的子接口。

VAP创建调用了net_device中ioctl，在qca wlan modules中，`__ath_attach` 中attach了ioctl的操作： 

    dev->do_ioctl = ath_ioctl;
    
在`ath_ioctl`中，转入cmd：SIOC80211IFCREATE中执行创建： 
```
case SIOC80211IFCREATE:
	{
       	    struct ieee80211_clone_params cp;
            if (scn->sc_in_delete) {
                printk("%s: Can't create VAP, in detach\n", __func__);
                return -ENODEV;
            }
            if (__xcopy_from_user(&cp, ifr->ifr_data, sizeof(cp))) {
       	        return -EFAULT;
            }	
            error = osif_ioctl_create_vap(dev, ifr, cp, scn->sc_osdev);
	}
        break;
```
`osif_ioctl_create_vap()`调用`alloc_netdev()`,

```
dev = alloc_netdev(sizeof(osif_dev), name, ether_setup);
    if(dev == NULL) {
        adf_net_delete_wlanunit(unit);
        return EIO;
    }
```

`alloc_netdev()`函数生成一个`net_device`结构体，对其成员赋值并返回该结构体的指针。第一个参数是设备私有成员的大小，第二个参数为设备名，第三个参数为`net_device`的`setup()`函数指针。`setup()`函数接收的参数为struct `net_device`指针，用于预置`net_device`成员的值。


然后`wlan_vap_create()`函数根据创建wifi的模式来创建接口： 

    vap = wlan_vap_create(devhandle, cp.icp_opmode, scan_priority_mapping_base, cp.icp_flags, cp.icp_bssid, cp.icp_mataddr);
    
在`wlan_vap_create()`函数中实际调用的是attach关联的函数`ath_vap_create()`, 既`ath_attach`中的： 

    ic->ic_vap_create = ath_vap_create;  

在`ath_vap_create()`中设置一些列wifi参数 

- ieee80211_vap_setup
- ieee80211vap_set_macaddr
- ieee80211_vap_attach

底层的VAP创建，调用了`osif_vap_setup()`

在`osif_ioctl_create_vap()`最后在实现网络接口的注册： 

    register_netdevice(dev)
    
调用关系如下图： 


![](https://user-gold-cdn.xitu.io/2018/4/9/162a917c319fddee?w=582&h=744&f=png&s=106255)


VAR的删除操作，也通过wifi网络接口的ioctl来实现的，SIOC80211IFDESTROY

调用关系如下图： 


![](https://user-gold-cdn.xitu.io/2018/4/9/162a91a2a9e2141c?w=698&h=744&f=png&s=117515)






    


