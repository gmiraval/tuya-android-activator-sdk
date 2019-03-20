# TuyaSmartActivator Android SDK


---

## Features Overview

TuyaSmartActivator APP SDK supports three network configuration modes, quick connection mode (TLink, it is referred to as the EZ mode) and hotspot mode (AP mode),  wired network configuration of zigbee gateway.

## Rapid Integration



```groovy
dependencies {
 implementation 'com.tuya.smart:deviceActivator:3.0.0'
 implementation 'com.alibaba:fastjson:1.1.67.android'
}
```


## Network Configuration 

Tuya’s hardware module supports three modes of network configuration: fast connect mode (TLink, or EZ mode), and hotspot mode (AP mode), Wired network configuration of zigbee gateway. The EZ mode is relatively more straight- forward. It is recommended to use the hotspot mode as an alternative only when the network configuration fails with the EZ mode. 

##### EZ mode 


```java
/**
* @param ssid 
* @param password  
* @param token 
*/
void startConfig(String ssid, String password, String token);


/**
* stopConfig
*/
void stopConfig();

```

```java
TuyaConfig.getEZInstance().startConfig("xxxssid","xxxpwd","token");

TuyaConfig.getEZInstance().stopConfig();
```



##### AP mode


```java
/**
* @param router's ssid 
* @param router's password 
* @param token 
*/
void startConfig(String ssid, String password, String token);


/**
* stopConfig
*/
void stopConfig();
```

```java
TuyaConfig.getAPInstance().startConfig("xxxssid","xxxpwd","token");

TuyaConfig.getEZInstance().stopConfig();
```


##### Zigbee Gateway 


```java
/**
* @param token 
*/
void startConfig(Context context, String token);

}

/**
* stopConfig
*/
void stopConfig();
```

```java
TuyaConfig.getWiredConfigInstance().startConfig(context,"token");

TuyaConfig.getWiredConfigInstance().stopConfig();
```

