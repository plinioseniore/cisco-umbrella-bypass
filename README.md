# Cisco Umbrella

## Introduction

Cisco Umbrella is a DNS and Proxy cloud service used to secure and restrict traffic without a VPN, it intercept all DNS requests and will proxy all the traffic. Based on the configuration it could reject all traffic that is not HTTP/HTTPS.

> :warning: DISCLAIMER : Use the below at your own risk, if your company/school has set Cisco Umbrella is to protect the devices from cyber threats. A bypass of Cisco Umbrella is likely a breach in the local policy of your company/school, the information in this page shall be used for *educational purpose only* and are shared to provide a better understanding of Cisco Umbrella.

Assuming that you could not disable Cisco Umbrella, the following option can allow you to bypass it:

### Stop traffic to Cisco Umbrella Cloud DNS and Proxy

Creating a firewall rule to don't allow traffic to Cisco Umbrella DNS and Proxy, the Umbrella DNS will became inactive but the Proxy will still be there and will timeout. 

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/cisco_umbrella_dns_disabled.png)

The connection is estabilished directly at the timeout, so a website that opens multiple TCP connections will require a longer than usual time to load, rather when a single TCP connection is enough (like a VPN over HTTPS) once the first timeout is gone will have the usual performances.
Based on the rights you have on your PC you can have this rule in Windows Firewall or in your home Firewall (if it has configurable options like openWrt).

IP Addresses to block : 146.112.255.0 to 146.112.255.255, 208.67.222.222, 208.67.220.220

> The IP addresses may change and make this ineffective, to identify new IP addresses use the Cisco Umbrella documentation.

> A future update may stop all network connectivity when not able to connect to Cisco Umbrella cloud services, having this bypass no longer effective.

### Use a local Proxy on Android

The traffic to the local network is not redirected to the Cisco Umbrella Proxy, in the below image there is an HTTP request to the LAN and WAN interface of the router web console. The first request to the LAN address is resolved directly, rather the WAN one is redirected via Cisco Umbrella Proxy, so having a proxy on a local interface is a bypass option. The local Proxy will resolve the DNS and handle all the traffic via your mobile connection.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/http_to_router_lan_wan_interface.png)

The traffic that the IP stack could resolve without an IP gateway (the connections to IP addresses in the same IP subnet) are not processed via Cisco Umbrella.

Running the [Servers Ultimate](https://www.google.com/search?client=firefox-b-d&q=servers+ultimate) app, the Android device will act as Proxy, use the USB Tethering connection to create a local network interface. In Firefox and Softether (or any other application that support HTTP Proxy) configure the Proxy option to redirect the traffic through the local interface of the Android phone.

This bypass may not work if future updates of Cisco Umbrella will inspect local traffic for proxy connection.

> Cisco Anyconnect (that may include also Cisco Umbrella) when connected to a VPN will force all the traffic (even the one that could be resolved locally) via the VPN, this will make ineffective the local proxy bypass.

> Based on the administrative rights available, you could disable "Cisco AnyConnect" from the local interface to the Android device and set the *metrics* to an high enogh value (say 1000) to have only the local traffic (so the one to the local proxy) via your Android connection. Without all traffic is via Android, even system updated or others that may consume high volume of data.
> ![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/tethering_interface_settings.PNG)

> If the Android device is connected to a WiFi network it will work as long as the WiFi network has no Cisco Umbrella on it (or any other or restriction), so it will work on a home WiFi connection but likely no on a enterprice WiFi connection.

### Use a local Proxy on your home network

Cisco Umbrella usage is increasing while more people are working from home, so the same approach of using a proxy server running on Android can be rebuild using a proxy running on your openWrt router or any other local resource (a Raspberry or similar).

### Using scrcpy to remote into your Android

If your end goal is only to browse websites that are not allowed via Cisco Umbrella, another option is to use [scrcpy](https://github.com/Genymobile/scrcpy) that via USB will remote your Android device into a windows. Then your can browse with your mobile connection.
