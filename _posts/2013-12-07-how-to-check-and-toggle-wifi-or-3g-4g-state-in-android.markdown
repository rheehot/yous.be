---
layout: post
title: "How to Check and Toggle WiFi or 3G/4G State in Android"
date: 2013-12-07 18:28:05 +0900
categories:
    - Android
description: "At some point, we want to know whether the device is connected to network so that we can do some network process."
keywords: android, wifi, 3g, 4g
redirect_from: /p/20131207/
---

## Overview

1. [Check if WiFi or 3G/4G is Enabled (by User)]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#1)
   1. [WiFi]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#1-1)
   2. [3G/4G]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#1-2)
2. [Check if WiFi or 3G/4G is Connected]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#2)
   1. [WiFi]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#2-1)
   2. [3G/4G]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#2-2)
3. [Toggle WiFi or 3G/4G Programmatically]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#3)
   1. [WiFi]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#3-1)
   2. [3G/4G]({% post_url 2013-12-07-how-to-check-and-toggle-wifi-or-3g-4g-state-in-android %}#3-2)

At some point, we want to know whether the device is connected to network so that we can do some network processes. Also we want to know if _user_ make WiFi or 3G/4G disabled on purpose. Both things are able to know.

<!-- more -->

<h2 id="1">1. Check if WiFi or 3G/4G is Enabled (by User)</h2>

<h3 id="1-1">WiFi</h3>

`ACCESS_WIFI_STATE` permission must be added to `AndroidManifest.xml`.

``` xml
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

Checking code is simple. In activity, [WifiManager][] has a handy method.

[WifiManager]: http://developer.android.com/reference/android/net/wifi/WifiManager.html

``` java
WifiManager wifiManager = (WifiManager) getSystemService(WIFI_SERVICE);
boolean wifiEnabled = wifiManager.isWifiEnabled();
```

<h3 id="1-2">3G/4G</h3>

This is more complicated. As WiFi case, we have to add `ACCESS_NETWORK_STATE` permission.

``` xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

Then we get [NetworkInfo][] from [ConnectivityManager][].

[NetworkInfo]: http://developer.android.com/reference/android/net/NetworkInfo.html
[ConnectivityManager]: http://developer.android.com/reference/android/net/ConnectivityManager.html

``` java
ConnectivityManager connectivityManager =
    (ConnectivityManager) getSystemService(CONNECTIVITY_SERVICE);
NetworkInfo mobileInfo =
    connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_MOBILE);
```

See [getState()][] overview.

> Reports the current coarse-grained state of the network.

[getState()]: http://developer.android.com/reference/android/net/NetworkInfo.html#getState()

There are 6 types of [NetworkInfo.State][].

- `CONNECTED`
- `CONNECTING`
- `DISCONNECTED`
- `DISCONNECTING`
- `SUSPENDED`
- `UNKNOWN`

[NetworkInfo.State]: http://developer.android.com/reference/android/net/NetworkInfo.State.html

Also this is [getReason()][] overview.

> Report the reason an attempt to establish connectivity failed, if one is available.

[getReason()]: http://developer.android.com/reference/android/net/NetworkInfo.html#getReason()

We can realize that when `NetworkInfo.State` is `DISCONNECTED`, `getReason()` reports to us why mobile data is disconnected.

Also I tested several times with `getState()` and `getReason()`.

- Enable WiFi and 3G/4G

  When WiFi is connected, mobile data connection automatically closed.

  ``` java
  mobileInfo.getState()
  // => DISCONNECTED
  mobileInfo.getReason()
  // => "dataDisabled"
  ```

- Enable WiFi only

  ``` java
  mobileInfo.getState()
  // => DISCONNECTED
  mobileInfo.getReason()
  // => "specificDisabled"
  ```

- Enable 3G/4G only

  ``` java
  mobileInfo.getState()
  // => CONNECTED
  ```

- Disable both

  ``` java
  mobileInfo.getState()
  // => DISCONNECTED
  mobileInfo.getReason()
  // => "specificDisabled"
  ```

So the code would be like this.

``` java
String reason = mobileInfo.getReason();
boolean mobileDisabled = mobileInfo.getState() == NetworkInfo.State.DISCONNECTED
    && (reason == null || reason.equals("specificDisabled"));
```

<h2 id="2">2. Check if WiFi or 3G/4G is Connected</h2>

WiFi or 3G/4G may not be connected even if the user enables them. Checking connectivity is useful when we are going to do some network communication.

<h3 id="2-1">WiFi</h3>

``` java
NetworkInfo wifiInfo =
    connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_WIFI);
boolean wifiConnected = wifiInfo.getState() == NetworkInfo.State.CONNECTED;
```

<h3 id="2-2">3G/4G</h3>

``` java
NetworkInfo mobileInfo =
    connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_MOBILE);
boolean mobileConnected = mobileInfo.getState() == NetworkInfo.State.CONNECTED;
```

<h2 id="3">3. Toggle WiFi or 3G/4G Programmatically</h2>

<h3 id="3-1">WiFi</h3>

`CHANGE_WIFI_STATE` permission must be added to `AndroidManifest.xml`.

``` xml
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
```

Enabling or disabling WiFi is easy.

``` java
WifiManager wifiManager = (WifiManager) getSystemService(WIFI_SERVICE);
wifiManager.setWifiEnabled(isWifiEnabled);
```

<h3 id="3-2">3G/4G</h3>

There is an workaround with reflection on ["How can i turn off 3G/Data programmatically on Android?"][Stack Overflow].

[Stack Overflow]: http://stackoverflow.com/questions/12535101/how-can-i-turn-off-3g-data-programmatically-on-android#12535246

For Android 2.3 and above:

``` java
private void setMobileDataEnabled(Context context, boolean enabled) {
  final ConnectivityManager conman =
      (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
  try {
    final Class conmanClass = Class.forName(conman.getClass().getName());
    final Field iConnectivityManagerField = conmanClass.getDeclaredField("mService");
    iConnectivityManagerField.setAccessible(true);
    final Object iConnectivityManager = iConnectivityManagerField.get(conman);
    final Class iConnectivityManagerClass = Class.forName(
        iConnectivityManager.getClass().getName());
    final Method setMobileDataEnabledMethod = iConnectivityManagerClass
        .getDeclaredMethod("setMobileDataEnabled", Boolean.TYPE);
    setMobileDataEnabledMethod.setAccessible(true);

    setMobileDataEnabledMethod.invoke(iConnectivityManager, enabled);
  } catch (ClassNotFoundException e) {
    e.printStackTrace();
  } catch (InvocationTargetException e) {
    e.printStackTrace();
  } catch (NoSuchMethodException e) {
    e.printStackTrace();
  } catch (IllegalAccessException e) {
    e.printStackTrace();
  } catch (NoSuchFieldException e) {
    e.printStackTrace();
  }
}
```

It requires to `CHANGE_NETWORK_STATE` permission.

``` xml
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

In Activity:

``` java
setMobileDataEnabled(this, isMobileDataEnabled);
```

Codes for Android 2.2 and below are also in the same [answer][Stack Overflow], but it requires `MODIFY_PHONE_STATE` permission that can be used by [system applications only][].

[system applications only]: http://developer.android.com/reference/android/Manifest.permission.html#MODIFY_PHONE_STATE
