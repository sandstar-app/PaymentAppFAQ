# SandStar Payment App FAQ

> This document is still writing now, if there is missing something, feel free to raise an issue or request a pull request to contribute it together. 
>
> Thanks.

## Terms Define

| term                  | description                                                  |
| --------------------- | ------------------------------------------------------------ |
| **POS Terminal**      | Point of Sale Terminal, in this article, actually is IDTech VP3300 only.<br/>It's reponsible for Swipe/Tap/Insert Card Detection and process the subsequently transactions. |
| **WorldNet**          | **WorldNet** is a payment gateway, which is popular in USA market. |
| **WorldNet SDK**      | Software Development Kit by **WorldNet**, it's responsible for POS Terminal interaction.<br/>e.g.,<br/>1. Init POS Terminal<br/>2. Setup POS Terminal<br/>3. Read Bankcard and process transaction |
| **Android Tablet**    | Android Tablet for run **Payment App**, it's just a runtime enviroment for **Payment App**.<br/>It exposed 2 port for IT stuff only<br/>1. Ethernet socket<br/>2. Power socket<br/>The port below is masked by panel<br/>1. microUSB port to IDTech VP3300<br/>2. |
| **Payment App**       | Android Tablet **Payment App**, which our developing and released.<br/>1. Payment App doesn't communicate with POS Terminal directly but use it by call **WorldNet SDK** API.<br/>2. Bank card detect ability is not our business but **WorldNet** and **IDTech** buesiness.<br/> |
| **Venus**             | Local server hardware, which run a lot of service such as **misClient**, etc...<br/>Venus usualy insatlled on roof of cabinets, is hard to see for customers. |
| **misClient**         | A local service run in every **Venus**, which is interact with cabinets hardware, **Payment App** and **VMS**. It's bind to cabinets, every cabinet has it's own **Venus**<br/>It's responsible for hardware controling such as Lock, Light, temperature sensor, etc. <br/>More importantly, It interact with payment app, response create order request, open-door request, shopping cart request and so on.<br/>It interact with VMS too, such as device info upload, forward transaction info to **VMS** and so on. |
| **App Update Server** | A standlone service run in remote server, which is responsible for app update chekcing, app downloading and so on.<br/>**AppUpdateServer**'s URI is http://app.sandstar.com:7070 |
| **VMS**               | Vending Machine System which developed by SandStar. You can login it by VMS portal URL.<br/>**VMS** interact with **misClient** only, it's not interact with **Payment App** directly. |



## User Guide

### 1. POS Initialization Screen

#### Screenshot

![Screenshot_20211025-225135](./_media/01-pos-init.png)

#### Description

>  IDTech VP3300 POS is initializing.

This screen usually does not take a long time. In our test environment  usually need 30 sec around.

In this phase, payment app only communicate with the payment gateway(actually is **WorldNet**) and will move to the next screen(Welcome Screen) as long as **POS terminal** which is IDTech VP3300 was setup successful.

> BTW: IDTech VP3300 setup process is not our code but WorldNet SDK's code, so how long time POS initializing  spend is up to WorldNet.

If the initialization is not successful within 60 seconds, the payment app will auto-restart and re-initialize.

<span style="color:red">If the payment app restarts repeatedly at this screen, check the connectivity of the tablet and the Internet. There are breif guide to checking below<span/>

1. Check the LAN cabinet is working well.

2. Check the WAN connectivity is working well.

   

### 2. Welcome Screen

#### Screenshot

![Screenshot_20211025-225153](./_media/02-welcome.png)

#### Description

> Welcome screen is waiting for touch.

In this screen, you can do nothing but touch the screen, when you touch the screen, it will move to next screen quickly.

> BTW: payment app update chekcing is run in this screen slient.

If there have a new apk was released to **AppUpdateServer**. The **payment app** will download the new apk quickly and then install it on tablet automatically. Once the newest apk was installed,tablet will auto-restart the newest app in seconds.

### 3. Prepare Transaction Screen

#### Screenshot

![Screenshot_20211028-033110](./_media/03-transaction-prepareing.png)

#### Description

> Transaction is preparing.

In this screen, payment app will request the `/sync/getPreAuthAmount`  from **misClient** server which is run inside Venus to fetch the `preOrderAmount` and `orderCode` fields, which will be used to perform next call. Once the payment app received the response from **misClient**

## Problem Resolve

### 1.  POS Initalization hangup

   
