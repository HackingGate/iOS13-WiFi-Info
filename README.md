# iOS13-WiFi-Info
Fix CNCopyCurrentNetworkInfo() does NOT work in iOS13 and later

## Get Wi-Fi SSID in iOS 12 and earlier

[stackoverflow](https://stackoverflow.com/a/37856496/4063462)

```swift
import Foundation
import SystemConfiguration.CaptiveNetwork

func getSSID() -> String? {
    var ssid: String?
    if let interfaces = CNCopySupportedInterfaces() as NSArray? {
        for interface in interfaces {
            if let interfaceInfo = CNCopyCurrentNetworkInfo(interface as! CFString) as NSDictionary? {
                ssid = interfaceInfo[kCNNetworkInfoKeySSID as String] as? String
                break
            }
        }
    }
    return ssid
}
```

>To use `CNCopyCurrentNetworkInfo()` in iOS 12 and later, enable the Access WiFi Information capability in Xcode. For more information, see [Access WiFi Information Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_networking_wifi-info).

## CNCopyCurrentNetworkInfo() returns nil in iOS 13 and later

Watch WWDC19 Session 713: [Advances in Networking, Part 2](https://developer.apple.com/videos/play/wwdc2019/713/).

>Now you all know the important privacy to Apple. And one of the things we realized. Is that... Accessing Wi-Fi information can be used to infer location.

>So starting now, to access that Wi-Fi information. You'll need the same kind of privileges that you'll need to get other location information.

![WWDC19-CNCopyCurrentNetworkInfo().png](/screenshots/WWDC19-CNCopyCurrentNetworkInfo().png)

Requires Capability: `Access Wi-Fi Information`

Must also meet at least one of criteria below

* Apps with permission to access location
* Currently enabled VPN app
* NEHotspotConfiguration (only Wi-Fi networks that the app configured)

Otherwise, returns `nil`

## Get Wi-Fi SSID in iOS 13 and later

Import [Core Location](https://developer.apple.com/documentation/corelocation) framework

```swift
import CoreLocation
```

Function to update UI

```swift
func updateWiFi() {
    print("SSID: \(currentNetworkInfos?.first?.ssid)")
    ssidLabel.text = getSSID()
}
```

Ask location permission

```swift
if #available(iOS 13.0, *) {
    let status = CLLocationManager.authorizationStatus()
    if status == .authorizedWhenInUse {
        updateWiFi()
    } else {
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
    }
} else {
    updateWiFi()
}
```

Implement [CLLocationManagerDelegate](https://developer.apple.com/documentation/corelocation/cllocationmanagerdelegate)

```swift
class ViewController: UIViewController, CLLocationManagerDelegate {
    ...
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        if status == .authorizedWhenInUse {
            updateWiFi()
        }
    }
    ...
}
```

## Update your app

If your app uses `CNCopyCurrentNetworkInfo()` and needs to solve the issue. Solve it now. There's no need to wait for Xcode 11 GM. The solution above is Xcode 10 compatible.
