# NEHotspotConfigurationManager

## 1. 必须添加配置  Hotspot Configuration Entitlement 


 
 A Boolean value indicating whether your app can use the hotspot manager to configure Wi-Fi networks.

Key: com.apple.developer.networking.HotspotConfiguration


This key indicates whether your app may use the NEHotspotConfigurationManager and NEHotspotConfiguration classes to configure Wi-Fi networks.

To add this entitlement to your app, enable the Hotspot Configuration capability in Xcode.


```
Signing & Capabilities > + Capability > Hotspot Configuration 

```

* [Code Signing Entitlements files
](https://developer.apple.com/library/archive/technotes/tn2415/_index.html#//apple_ref/doc/uid/DTS40016427-CH1-ENTITLEMENTSFILE) 

The following are relevant notes regarding Xcode's use of code signing entitlements files:

```
Certain capabilities require a code signing entitlements file. Xcode manages the creation of this file automatically when you adjust a target's capabilities.
When Xcode creates the entitlements for the app's signature, values from the code signing entitlements file are used to fill in any wildcard, asterisk portions of the entitlements that might exist in the code signing provisioning profile.
Aside from the entitlements that Xcode writes into code signing entitlements files for you, it’s possible to add entitlements into these files yourself. There used to be a need to do this but it is historic and more often the cause of entitlement problems nowadays.
``` 


## 2. 如何获取WiFi信息

Enable Access WiFi Information from capabilities


```
Signing & Capabilities > + Capability > Access WiFi Information  

```

通过 **CNCopyCurrentNetworkInfo**获取Wi-Fi信息。iOS 13后，获取WiFi信息有如下限制：

You can only access Wifi SSID in these three cases:

* If your app has permission to access the location.
* If your app has an enabled VPN profile.
* If your app is a networking app that uses NEHotspotConfiguration. You will need extra approval from Apple for this case.
