# Payment App FAQ

>  This document is still writing now, if there is something missing, feel free to raise an issue or make a pull request to contribute this doc together.
>
> Thanks.

## Terms Define

| term                  | description                                                  |
| --------------------- | ------------------------------------------------------------ |
| **POS Terminal**      | Point of Sale Terminal, in this article, actually is IDTech VP3300 only.<br/>It's responsible for Swipe/Tap/Insert Card Detection and processing the subsequent transactions |
| **WorldNet**          | **WorldNet** is a payment gateway, which is popular in the USA market. |
| **WorldNet SDK**      | Software Development Kit by **WorldNet**, it's responsible for POS Terminal interaction.<br/>e.g.,<br/>1. Init POS Terminal<br/>2. Setup POS Terminal<br/>3. Read Bankcard and process transaction |
| **Android Tablet**    | Android Tablet for run **Payment App**, it's just a runtime environment for **Payment App**.<br/>It exposed 2 port for IT stuff only<br/>1. Ethernet socket<br/>2. Power socket<br/>The port below is masked by panel<br/>1. micro-USB port to IDTech VP3300<br/>  |
| **Payment App**       | Android Tablet **Payment App**, which are developing and released.<br/>1. Payment App doesn't communicate with POS Terminal directly but uses it by call **WorldNet SDK** API.<br/>2. Bank card detectability is not our business but **WorldNet** and **IDTech** business.<br/> |
| **Venus**             | Local server hardware, which runs a lot of services such as **misClient**, etc...<br/>Venus usually installed on the roof of cabinets, is hard to see for customers. |
| **misClient**         | A local service run in every **Venus**, which interacts with cabinets hardware, **Payment App**, and **VMS**. It's bound to cabinets, every cabinet has it's own **Venus**<br/>It's responsible for hardware controlling such as Lock, Light, temperature sensor, etc. <br/>More importantly, It interact with the payment app, response create order request, open-door request, shopping cart request and so on.<br/>It interact with VMS too, such as device info upload, forward transaction info to **VMS** and so on. |
| **App Update Server** | A standalone service run in a remote server, which is responsible for app update checking, app downloading, and so on.<br/>**AppUpdateServer**'s URI is http://app.sandstar.com:7070 |
| **VMS**               | Vending Machine System which was developed by SandStar. You can log in it by VMS portal URL.<br/>**VMS** interact with **misClient** only, it does not interact with **Payment App** directly.|



## User Guide

### 1. POS Initialization Screen

#### Screenshot

![Screenshot_20211025-225135](./_media/01-pos-init.png)

#### Description

>  IDTech VP3300 POS is initializing.

This screen usually does not take a long time. In our test environment usually need 30 sec around.

In this phase, the payment app only communicates with the payment gateway(actually is **WorldNet**) and will move to the next screen(Welcome Screen) as long as **POS terminal** which is IDTech VP3300 was set up successful.

> BTW: IDTech VP3300 setup process is not our code but WorldNet SDK's code, so how long time POS initializing spend is up to WorldNet.

If the initialization is not successful within 60 seconds, the payment app will auto-restart and re-initialize.

<span style="color:red">If the payment app restarts repeatedly at this screen, check the connectivity of the tablet and the Internet. There are a brief guide to checking below<span/>

1. Check the LAN cabinet is working well.

2. Check the WAN connectivity is working well.

   

### 2. Welcome Screen

#### Screenshot

![Screenshot_20211025-225153](./_media/02-welcome.png)

#### Description

> Welcome screen is waiting for touch.

On this screen, you can do nothing but touch the screen, when you touch the screen, it will move to the next screen quickly.

> BTW: payment app update checking is run in this screen silent.

If there has a new apk that was released to **AppUpdateServer**. The **payment app** will download the new app quickly and then install it on the tablet automatically. Once the newest apk was installed, the tablet will auto-restart the newest app in seconds.

### 3. Prepare Transaction Screen

#### Screenshot

![Screenshot_20211028-033110](./_media/03-transaction-prepareing.png)

#### Description

> Transaction is preparing.

In this screen, **payment app** will request the `/sync/getPreAuthAmount` endpoint from the **misClient** server which is run in Venus to fetch the `preOrderAmount` and `orderCode` fields, which will be used to perform the next call. 

Once the **payment app** received the response from **misClient** and get the `preOrderAmount` and `orderCode` params, it will instantly perform a `processSalce` call by **WorldNet SDK**  to active the **POS Terminal**, then the light of **POS Terminal** will be green. and the will be switched to the next screen(ScanCardScreen).

!> Know Issues

![Screenshot_20211101-174319](./_media/04-preAuthAmount-failed.png)

If there is "Count not get advance order information" occurred, it means the **misClient** not response as **Payment App** expected.

So please check below :

1. Is **misClient** Server running?
2. Is VMSClient IP in **Payment App**'s' Config Screen is configured correctly?
2. Is the tablet and Venus in same LAN?
2. Is the Ethernet Cable working?


### 4. Scan Card Screen

#### Screenshot

![Screenshot_20211025-234727](./_media/05-scancard.png)

#### Description

> Waiting customer show card.

At first, the Tap/Insert/Swipe mode is activated, so the customer can present their bank card with **POS Terminal**(IDTech VP3300) built-in card interface.

Once **POS Terminal** detected the bankcard, **payment app** will toast a message like below.

![Screenshot_20211025-234814](./_media/06-card-detected.png)

!> Know Issues

If the card is not supported by **WorldNet** or **POS Terminal**, **payment app** will toast an error message like below.

| Card tap not supported, please use insert or swipe mode.     | Card expired, please use another card.                      |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| ![Screenshot_20211025-234818](./_media/07-card-tap-not-support.png) | ![Screenshot_20211025-225901](./_media/08-card-expired.png) |

If the card detected is well, the **payment app** will request `/sync/openDoor`  from **misClient**, once **misClient** response success to **payment app**,then the screen will be switched to **Door Unlocked Screen**.

### 5. Door Unlocked Screen

#### Screenshot

![Screenshot_20211025-225211](./_media/09-door-unlocked.png)

#### Description

> Waiting customer opens the door.

The door lock will remain open for about ten seconds, and if the door is pulled within the lock open time period, then shopping can begin; if the door is opened too late, the lock will automatically fall down.

!> Know Issues

If the `/sync/openDoor` request is not working, the **payment app** will switch to the error screen. You can find it in [Opps Screen Secion](#_8-Oops-Screen)

### 6. Shopping Screen

#### Screenshot

| Empty Shopping Cart                                       | Shopping Cart                                              |
| --------------------------------------------------------- | ---------------------------------------------------------- |
| ![Screenshot_20211101-193729](./_media/11-cart-empty.png) | ![Screenshot_20211101-193814](./_media/12-cart-normal.png) |

#### Description

During open-door shopping, the shopping cart page will automatically update when you take any item out of the cabinet.

!> Potential Issues

1. If an item is incorrectly identified, such as recognizing a Coke as a cookie, please contact the vision algorithm team.
2. If the item is incorrectly priced, please contact IT support. 


### 7. Checkout Screen

#### Screenshot

![Screenshot_20211101-193833](./_media/13-purchase.png)

#### Description

When the cabinet door is closed, **payment app** automatically switches to the checkout screen and stays there for a few seconds.

The customer can check the amount of his purchase on this page and this amount will be charged to his credit card shortly afterward.

!> Potential Issues

1. If the purchase amount is incorrect, please contact IT support

### 8. Oops Screen

#### Screenshot

![Screenshot_20211101-192556](./_media/10-error-has-incomplate-order.png)

#### Description

The **payment app** will switch to the error screen like above while an exception happened during shopping. 

> The key point in this screen is the error message presented. So please record this message that IT support can locate the problem as soon as possible.

**Common Error Message Table**

| Error Message                                           | Description                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| There is an incomplete transaction(2), please try later | There is an incomplete transaction, please wait a minute.<br/>If this message repeatedly persists, please call IT support. |
| System Error(3), please try later                       | Hardware communication failed, please call IT support.       |
| System Error(4), please try later                       | Algorithm communication failed, please call IT support.      |
| System Error(5), please try later                       | Serial port equipment not ready, please call IT support.     |
| System Error(6), please try later                       | There was a problem with the scanner. Please try again later or contact our service team for help. |
| System Error(7), please try later                       | There was a problem with the temperature sensor, please call IT support. |
| System Error(8), please try later                       | The cabinet is locked, you can unlock it throught OPS app.   |
| Be Right Back...                                        | Same as System Error(8). The cabinet is locked, you can unlock it throught OPS app. |
| System Error(9), please try later                       | The service is not ready, please call IT support.            |
| Other problems.                                         | Please take a photo and call IT support.                     |




## Technician Guide

### 1.  How to login Venus

> You must connect to the cabinets VPN before login venus.

```shell
ssh sandstart@100.9.x.x
```

### 2. How to get the IP of Tablet

!> You must log in to Venus before subsequently operating.

> **Payment App** will enable the remote debug ability automatically when it was launched, the remote connection port is 5555.

Actually, the **Payment App** will auto-launch when **Tablet** power-on. But how to acquiring the IP of the tablet is required some work to do in this situation.

We can use the command below to scan the LAN in order to detect the tablet's IP.

#### Preparetion

1. Make sure the Venus has the `ip` and `nmap` and `adb` command, if the answer is not, please use `sudo apt install nmap adb` to install it directly.
2. Make sure the Venus and Tablets power on and connect to the same LAN. 
3. Use the `ip` command to fetch the LAN IP of Venus is below, the `10.81.110.217` is the IP of Venus.

```shell
# ip addr
ip addr|grep eth0 -A 10
```

![image-20211101202002244](./_media/14-show-ip.png)

4. Use the `ip` command to fetch the IP of Venus

```shell
# Get the LAN IP of Venus
ip addr show eth0|grep inet|awk 'NR==1{print $2}'
# Get the VPN IP of Venus
ip addr show tun0|grep inet|awk 'NR==1{print $2}'
```

![image-20211110133123278](./_media/14-show-ip-3.png)

5. Optional: Use `ifconfig` command to fetch the IP of Venus

!> `ifconfig` command is only available when `net-tools
was installed by `apt`

```shell
# Get the LAN IP of Venus
ifconfig eth0|grep inet|awk 'NR==1{print $2}'
# Get the VPN IP of Venus
ifconfig tun0|grep inet|awk 'NR==1{print $2}'
```

![image-20211102015229839](./_media/14-show-ip-2.png)

#### Scan Tablet's IP

Now, we have the ethernet IP(LAN) of Venus, it's a good beginning. secondly, we need to use the `nmap` command to scan the tablet's IP.

In this demonstration, we use the `10.81.110.0/24` subnet to scan it.

```shell
# for 10.81.110.0/24 subset only
nmap 10.81.110.0/24 -p5555 |grep open  -B 5
# for any LAN subset. But to be noticed the command of ifconfig is required.
nmap -p5555 $(ifconfig eth0|grep inet|awk 'NR==1{print $2}')/24|grep open -B 5
# for any LAN subset. But to be noticed the command of ip is required.
nmap -p5555 $(ip addr show eth0|grep inet|awk 'NR==1{print $2}')|grep open -B 5
```

![image-20211101202817625](./_media/15-scan-tablet-ip.png)

The figure above shows two IPs with 5555 ports opened in this subnet, which means there are two Tablets available.

### 3. How to check the log of payment app

#### Preparetion

Firstly, connect the tablet with the command below.

```shell
adb connect 10.81.110.182
```

![image-20211101203834280](./_media/16-adb-connect.png)

Secondly, check the android device with the command below.

```shell
adb devices
```

![image-20211101203945225](./_media/17-adb-devices.png)

#### Read Tablet logs

If everything is fine, you can use the command below to read the real-time logs of Tablet.

```shell
adb logcat
```

![image-20211101204226363](./_media/18-adb-logcat.png)

#### Read Tablet logs Advance

The filter is supported. You can use the `pipe` and  `grep` command to filter the log of Tablet like below.

```shell
# Read POS logs only
adb logcat |grep PosManager
# Read Exception logs only
adb logcat |grep Exception
```

#### Write Tablet logs to Venus

> This is only a temporary log write. When the Venus or Tablet restarts, or the adbd service restarts, the write will be terminated.

```shell
adb logcat >> ~/Downloads/PaymentAppLogs/logcat.log &
```

#### Read the logs in file

```shell
tail -200f ~/Downloads/PaymentAppLogs/logcat.log
```

#### Check the VmsClientIp of PaymentApp

![Show VmsClientServerIp of PaymentApp](./_media/18-show-server-ip-of-paymen-app.png)

```shell
adb root
adb connect $tabletIp
adb shell cat /data/data/com.yitunnel.creditcard/shared_prefs/app_config.xml
```

#### One more thing

Since the `adb` command is available, you can use all `adb` functions to control the **Tablet** and **Payment App**.

```shell
# ssh to Tablet
adb shell
# Kill the payment-app
adb shell am force-stop com.yitunnel.creditcard
# Start the payment-app
adb shell am start -n com.yitunnel.creditcard/com.sandstar.ui.InitPaymentActivity
# Check the version of Payment App
adb shell dumpsys package com.yitunnel.creditcard|grep version
# Check Current Top UI
adb shell dumpsys activity top |grep Activity
```

If you want to see more detail of the `adb` command, please visit this site below, which is provided by Google.

https://developer.android.com/studio/command-line/adb



