#### Note: This repository is inherited from the [old Tuya Github repository](https://github.com/TuyaInc/tuyasmart_android_activator_sdk) that will be deprecated soon. Please use this repository for Tuya SDK development instead. For changing the existing remote repository URL, see this [tutorial](https://docs.github.com/en/free-pro-team@latest/github/using-git/changing-a-remotes-url).

# TuyaSmartActivator Android SDK

---

## Overview

TuyaSmartActivator android SDK supports three pairing modes, fast pairing (TLink. EZ mode), hotspot pairing (AP mode), and wired pairing through Zigbee gateway.

## Preparation

Add the following to the `build.gradle`:

```groovy
dependencies {
 implementation 'com.tuya.smart:deviceActivator:3.15.3'
 implementation 'com.alibaba:fastjson:1.1.67.android'
}
```

## Get authorization token

All activator APIs require incoming authorization tokens. 

First, you should request API `/v1.0/devices/token`, please refer to this document [Generate A Paring Token](https://docs.tuya.com/en/iot/open-api/api-list/api/paring-management#title-1-Generate%20a%20paring%20token)

And you will get the following response:

```json
{
  "secret":"reKE",
  "region":"AY",
  "token":"nqMwn1Nd"
}
```

authToken = region + token + secret , such as:

```java
String authToken = region + token + secret; // "AY" + "nqMwn1Nd" + "reKE" 
```

In this way, you get the authToken parameter that can be used in following methods.

## Pairing 

Tuya’s hardware modules support three pairing modes: fast pairing (TLink. EZ mode), hotspot pairing (AP mode), and wired pairing through Zigbee gateway. The EZ mode is  more direct. It is recommended to use the hotspot mode as an alternative only when the pairing fails in the EZ mode. 

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


### Zigbee gateway mode


```java
/**
* @param context  use the application's context
* @param authToken 
*/
void startConfig(Context context, String authToken);


/**
* stopConfig
*/
void stopConfig(Context context);
```

```java
TuyaConfig.getWiredConfigInstance().startConfig(context,"authToken");

TuyaConfig.getWiredConfigInstance().stopConfig(context);
```

### BLE + Wi-Fi dual mode device

BLE–Wi-Fi dual mode devices support both Bluetooth low energy (BLE) and Wi-Fi modules.

#### Permissions

To pair devices through BLE, you need to add the following permissions to your AndroidManifest.xml.

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

#### Get BLE + Wi-Fi methods

All BLE + Wi-Fi methods are in the `ItuyaBleWifiConfig`.

```java
ITuyaBleWifiConfig bleWifiInstance = TuyaConfig.getBleWifiInstance();
```

#### Scan BLE + Wi-Fi device

Only one device will be returned at a time and you need to save the scanned results yourself.

```java
/**
* Start to scan BLE device
* @param timeout scan timeout, the unit is ms
* @param scanCallback scan result callback
*/
public void startScan(int timeout, TuyaBleScanCallback scanCallback);
```

**Parameters**

`TuyaBleScanCallback` will call back `BLEScanBean` and you should focus on the following fields.

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

#### Stop scaning BLE + Wi-Fi device

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

#### Start configuring

```java
/**
 * start config 
 * @param configParams config parameters
 * @param listener config listener
 */
public void startConfig(ConfigParams configParams, TuyaBleConfigListener listener);
```

**Note:**

The `TuyaBleConfigListener` does not mean that the device is activated successfully. Instead, it only means that the SDK sent an activation command to the device successfully. After you receive the `onSuccess` callback, you need to continuously query the server for whether the device is activated successfully.

**Parameters**

`ConfigParams` can be created by `ConfigParamsBuilder`, the field descriptions are as follows.

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

#### Stop configuring

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

## Proguard-rules


```
#fastJson
-keep class com.alibaba.fastjson.**{*;}
-dontwarn com.alibaba.fastjson.**

-keep class com.tuya.**{*;}
-dontwarn com.tuya.**
```


## Changelog
### 3.15.3
* Fix the activator of Zigbee gateway memory leak.
### 3.15.1
* Fix the BLE + Wi-Fi device that can't be configured immediately after it is removed.
