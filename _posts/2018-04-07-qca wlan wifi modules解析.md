---
layout:     post
title:      qca wlan wifi modules解析二
subtitle:   qca wlan wifi modules解析
date:       2018-04-07
author:     hades
header-img: img/my-blog-picture.jpg
catalog: true
tags:
    - hades
---

wifi驱动使用是PCI总线，首先从pci设备注册开始： 
`os/linux/src/if_ath_pci.c` 文件`init_ath_pci()`中的 :
```
if (pci_register_driver(&ath_pci_drv_id) < 0) { 此处进行的是PCI的注册。
        printk("ath_pci: No devices found, driver not installed.\n");
        pci_unregister_driver(&ath_pci_drv_id);
#ifdef ATH_AHB
        pciret = -ENODEV;
#else
        return (-ENODEV);
#endif
    }

```
其中ath_pci_drv_id定义如下
```
struct pci_driver ath_pci_drv_id = {
    .name       = "ath_pci",
    .id_table   = ath_pci_id_table,
    .probe      = ath_pci_probe,
    .remove     = ath_pci_remove,
#ifdef ATH_BUS_PM
    .suspend    = ath_pci_suspend,
    .resume     = ath_pci_resume,
#endif /* ATH_BUS_PM */
    /* Linux 2.4.6 has save_state and enable_wake that are not used here */
};
```
在PCI的probe函数`ath_pci_probe`中有如下的代码：

    dev = alloc_netdev(sizeof(struct ath_pci_softc), "wifi%d", ether_setup);  //此处分别了设备节点dev.
    if (dev == NULL) {
        printk(KERN_ERR "ath_pci: no memory for device state\n");
        goto bad2;
    }
    
    
    if (__ath_attach(id->device, dev, &bus_context, &sc->aps_osdev) != 0)
        goto bad3;
        
        
调用流程如下图： 

![](https://user-gold-cdn.xitu.io/2018/4/3/1628a2ddd069fa10?w=801&h=834&f=png&s=78121)

函数`__ath_attach()`，此函数中attach了很多操作相关的处理函数
  
    dev->netdev_ops = &athdev_net_ops;
    
`athdev_net_ops`是ath网络接口的结构体， PCI接口的wifi设备在各层中attach了很多需要处理的函数，其贯穿整个WLAN驱动框架， 定义如下： 
```
static const struct net_device_ops athdev_net_ops = {
    .ndo_open    = ath_netdev_open,
    .ndo_stop    = ath_netdev_stop,
    .ndo_start_xmit = ath_netdev_hardstart,
    .ndo_set_mac_address = ath_netdev_set_macaddr,
    .ndo_tx_timeout = ath_netdev_tx_timeout,
    .ndo_get_stats = ath_getstats,
    .ndo_change_mtu = ath_change_mtu,
#if LINUX_VERSION_CODE < KERNEL_VERSION(3,2,0)
    .ndo_set_multicast_list = ath_netdev_set_mcast_list,
#else
    .ndo_set_rx_mode = ath_netdev_set_mcast_list,
#endif
    .ndo_do_ioctl = ath_ioctl,
};
```

`ath_attach`中`ath_dev_attach`的`net80211_ops`实现了ieee80211_ops各种操作：
```
error = ath_dev_attach(devid, base_addr,
                           ic, &net80211_ops, osdev,
                           &scn->sc_dev, &scn->sc_ops,
                           scn->amem.handle,
                           ath_conf_parm, hal_conf_parm);

```
`ath_attach`中`osif_attach(dev);`初始化OSIF层的操作： 
`ath_dev_attach`实现了`ath_ops`操作，既： `ath_ar_ops`
在`ath_attach` 中, device在加载的过程中attach的各层的处理函数 :
```

    #ifdef ADF_SUPPORT
    ATH_INIT_TQUEUE(&osdev->intr_tq, (adf_os_defer_fn_t)ath_tasklet, (void*)dev);
    #else
   ATH_INIT_TQUEUE(&osdev->intr_tq, ath_tasklet, dev);
    #endif
    ...
    
    error = request_irq(dev->irq, ath_isr, IRQF_DISABLED, dev->name, dev);
    
    ...
    
    /*
     * finally register netdev and ready to go
     */
    if ((error = register_netdev(dev)) != 0) {
        printk(KERN_ERR "%s: unable to register device\n", dev->name);
        goto bad4;
    }

```
以上反操作流程如下： 

![](https://user-gold-cdn.xitu.io/2018/4/3/1628a4da6c69ef60?w=716&h=835&f=png&s=56985)




    
