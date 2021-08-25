# Cisco Umbrella

## Introduction

Cisco Umbrella is a DNS and Proxy cloud service used to secure and restrict traffic without a VPN, it intercept all DNS requests and will proxy all the traffic. Based on the configuration it could reject all traffic that is not HTTP/HTTPS.

> DISCLAIMER : Use the below at your own risk, if your company/school has set Cisco Umbrella is for a reason. These are provided only for information on how this software works and which are its weakness.

Assuming that you could not disable Cisco Umbrella, the following option can allow you to bypass it:

### Stop traffic to Cisco Umbrella Cloud DNS and Proxy

Creating a firewall rule to don't allow traffic to Cisco Umbrella DNS and Proxy, the Umbrella DNS will became inactive but the Proxy will still be there and will timeout. This works but is by overall slow, as you need to wait for the Umbrella timeout of each connection.

Based on the rights you have on your PC you can have this rule in Windows Firewall or your home Firewall (if it has configurable options like openWrt).

IP Addresses to block : 146.112.255.0 to 146.112.255.255, 208.67.222.222, 208.67.220.220

> The IP addresses may change and make this ineffective, to identify new IP addresses use the Cisco Umbrella documentation.

> A future update may stop all network connectivity when not able to connect to Cisco Umbrella cloud services, having this bypass no longer effective.

### Use a local Proxy on Android

The traffic to the local network could not be redirected to the Cisco Umbrella Proxy, so having a proxy on a local interface is a bypass option. The local Proxy will resolve the DNS and handle all the traffic via your mobile connection.

Running the [Servers Ultimate](https://www.google.com/search?client=firefox-b-d&q=servers+ultimate) app, the Android device will act as Proxy, use the USB Tethering connection to create a local network interface. In Firefox and Softether (or any other application that support HTTP Proxy) configure the Proxy option to redirect the traffic through the local interface of the Android phone.

As is now, this bypass works because the filtering is done on the cloud, but if a future update of Cisco Umbrella will inspect local traffic and stop proxy protocols this will no longer work.

> Cisco Anyconnect (that may include also Cisco Umbrella) when connected to a VPN will force all the traffic (even the one that could be resolved locally) via the VPN, this will make ineffective the local proxy bypass.

> Based on the administrative rights available, you could disable "Cisco AnyConnect" from the local interface to the Android device and set the **metrics** to an high enogh value (say 1000) to have only the local traffic (so the one to the local proxy) via your Android connection. Without all traffic is via Android, even system updated or others that may consume high volume of data.

> If the Android device is connected to a WiFi network it will work as long as the WiFi network has no Cisco Umbrella on it (or any other or restriction), so it will work on a home WiFi connection but likely no on a enterprice WiFi connection.

### Use a local Proxy on your home network

Cisco Umbrella usage is increasing while more people are working from home, so the same approach of using a proxy server running on Android can be rebuild using a proxy running on your openWrt router or any other local resource (a Raspberry or similar).

### Using scrcpy to remote into your Android

If your end goal is only to browse websites that are not allowed via Cisco Umbrella, another option is to use [scrcpy](https://github.com/Genymobile/scrcpy) that via USB will remote your Android device into a windows. Then your can browse with your mobile connection.
