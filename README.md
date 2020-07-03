# TuyaSmartActivator Android SDK

---

## Features Overview

TuyaSmartActivator APP SDK supports three network configuration modes, quick connection mode (TLink, it is referred to as the EZ mode) and hotspot mode (AP mode),  wired network configuration of zigbee gateway.

## Preparation

Put the following configuration in your build.gradle:

```groovy
dependencies {
 implementation 'com.tuya.smart:deviceActivator:3.15.1'
 implementation 'com.alibaba:fastjson:1.1.67.android'
}
```

## Get Authorization Token

All activator interfaces require incoming authToken. 

First, you shoud request api `/v1.0/devices/token`, please refer to this document [Generate A Paring Token](https://docs.tuya.com/en/iot/open-api/api-list/api/paring-management#title-1-Generate%20a%20paring%20token)

And you will get such response:

```json
{
  "secret":"reKE",
  "region":"AY",
  "token":"nqMwn1Nd"
}
```

authToken = region+token+secret , such as:

```java
String authToken = region + token + secret; // "AY" + "nqMwn1Nd" + "reKE" 
```

In this way, you get the authToken parameter, which can be used in following methods.

## Network Configuration 

Tuya’s hardware module supports three modes of network configuration: fast connect mode (TLink, or EZ mode), and hotspot mode (AP mode), Wired network configuration of zigbee gateway. The EZ mode is relatively more straight- forward. It is recommended to use the hotspot mode as an alternative only when the network configuration fails with the EZ mode. 

### EZ mode 


```java
/**
* @param ssid 
* @param password  
* @param authToken 
*/
void startConfig(String ssid, String password, String authToken);


/**
* stopConfig
*/
void stopConfig();

```

```java
TuyaConfig.getEZInstance().startConfig("xxxssid","xxxpwd","authToken");

TuyaConfig.getEZInstance().stopConfig();
```

### AP mode


```java
/**
* @param router's ssid 
* @param router's password 
* @param authToken 
*/
void startConfig(Context context,String ssid, String password, String authToken);


/**
* stopConfig
*/
void stopConfig();
```

```java
TuyaConfig.getAPInstance().startConfig(context,"xxxssid","xxxpwd","authToken");

TuyaConfig.getEZInstance().stopConfig();
```


### Zigbee Gateway 


```java
/**
* @param context  use the application's context
* @param authToken 
*/
void startConfig(Context context, String authToken);

}

/**
* stopConfig
*/
void stopConfig();
```

```java
TuyaConfig.getWiredConfigInstance().startConfig(context,"authToken");

TuyaConfig.getWiredConfigInstance().stopConfig();
```

### BLE-WiFi Dual Mode Device

BLE-WiFi dual mode device has both Bluetooth Low Energy(BLE) and WiFi modules.

#### Permissions

To use Bluetooth to scan and pair devices, you need to add the following permissions to your AndroidManifest.xml.

```xml
<!-- Required. Allows applications to connect to paired bluetooth devices.  -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<!-- Required. Allows applications to discover and pair bluetooth devices.  -->
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<!-- Required.  Allows an app to scan bluetooth device.  -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<!-- Required.  Allows an app to scan bluetooth device.  -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<!--  Allows an app to use bluetooth low energy feature  -->
<uses-feature
	android:name="android.hardware.bluetooth_le"
	android:required="false" />
```

#### getBleWifiInstance

All  Ble-WiFi methods are in ItuyaBleWifiConfig interface.

```java
ITuyaBleWifiConfig bleWifiInstance = TuyaConfig.getBleWifiInstance();
```

#### Scan BLE-WiFi Device

Only one device will be returned at a time, you need to save the scanned results yourself.

```java
/**
* Start to scan BLE device
* @param timeout scan timeout, the unit is ms
* @param scanCallback scan result callback
*/
public void startScan(int timeout, TuyaBleScanCallback scanCallback);
```

**Parameters**

`TuyaBleScanCallback` will callback `BLEScanBean`,  you should focus on the following fields.

| Field     | Type    | Description                         |
| --------- | ------- | ----------------------------------- |
| devUuId   | String  | device uuid                         |
| productId | String  | device product id                   |
| isBind    | boolean | Whether the device has been bound   |
| support5G | boolean | Whether the device supports 5g WIFI |

**Example**

```java
HashMap<String, BLEScanBean> map = new HashMap<String, BLEScanBean>();
bleWifiInstance.startScan(20 * 1000, new TuyaBleScanCallback() {
    @Override
    public void onResult(BLEScanBean bleScanBean) {
        Log.i(TAG, "startScan onResult :" + bleScanBean);
        map.put(bleScanBean.devUuId, bleScanBean);
    }
});
```

#### Stop Scan BLE-WiFi Device

```java
/**
 *  Stop to scan BLE device
 */
public void stopScan();
```

**Example**

```java
bleWifiInstance.stopScan();
```

#### Start Config

```java
/**
 * start config 
 * @param configParams config parameters
 * @param listener config listener
 */
public void startConfig(ConfigParams configParams, TuyaBleConfigListener listener);
```

**NOTE:**

The `TuyaBleConfigListener` here does not mean that the device is actually activated successfully, only that the SDK successfully sent an activation command to the device. After you receive the `onSuccess` callback, you need to continuously request server for whether the device is activated successfully.

**Parameters**

`ConfigParams` can create by `ConfigParamsBuilder`, the field descriptions are as follows.

| Field     | Type        | Description                                                  |
| --------- | ----------- | ------------------------------------------------------------ |
| scanBean  | BLEScanBean | Scanning result bean                                         |
| ssid      | String      | Your wifi name                                               |
| password  | String      | Your wifi password                                           |
| authToken | String      | authToken, see [Get Authorization Token](#get-authorization-token) |
| random    | String      | random requested from the server                             |
| authKey   | String      | authKey requested from the server                            |

**Example**

```java
ConfigParams configParams = new ConfigParamsBuilder()
        .scanBean(bleScanBean)
        .authKey(encryptedAuthKey)
        .random(random)
        .authToken(authToken)
        .ssid("your_wifi_name")
        .password("your_wifi_passowrd")
        .build();
bleWifiInstance.startConfig(configParams, new TuyaBleConfigListener() {
    @Override
    public void onSuccess(String uuid) {
        Log.i(TAG, "startWifiConfig onSuccess : " + uuid);
    }

    @Override
    public void onFail(String uuid, String code, String msg) {
        Log.i(TAG, "startWifiConfig onFail : uuid = "+uuid+" code:" + code + "  message=" + msg);
    }
});
```

#### Stop Config

```java
/**
 * stop config
 * @param devUuid device uuid
 */
public void stopConfig(String devUuid);
```

**Example**

```java
bleWifiInstance.stopConfig("your_device_uuid");
```

## proguard-rules


```
#fastJson
-keep class com.alibaba.fastjson.**{*;}
-dontwarn com.alibaba.fastjson.**

-keep class com.tuya.**{*;}
-dontwarn com.tuya.**
```


## Changelog

### 3.15.1
* Fix Ble-WiFi Device can't config immediately after remove the device.