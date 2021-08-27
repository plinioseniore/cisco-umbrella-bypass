# Cisco Umbrella

## Introduction

Cisco Umbrella is a DNS and Proxy cloud service used to secure and restrict traffic without a VPN, it intercept all DNS requests and will proxy all the traffic. Based on the configuration it could reject all traffic that is not HTTP/HTTPS.

> DISCLAIMER : Use the below at your own risk, if your company/school has set Cisco Umbrella is to protect the devices from cyber threats. A bypass of Cisco Umbrella is likely a breach in the local policy of your company/school, the information in this page shall be used for *educational purpose only* and are shared to provide a better understanding of Cisco Umbrella.

Assuming that you could not disable Cisco Umbrella, the following option can allow you to bypass it:

### Stop traffic to Cisco Umbrella Cloud DNS and Proxy

Creating a firewall rule to don't allow traffic to Cisco Umbrella DNS and Proxy, the Umbrella DNS will became inactive but the Proxy will still be there and will timeout. 

In the ![Cisco Umbrella documentation](https://docs.umbrella.com/deployment-umbrella/docs/appx-a-status-and-functionality) is listed that if *There is at least one active network connection; however, the Umbrella roaming client can’t connect to 208.67.222.222 / 208.67.220.220 / 2620:119:53::53 / 2620:119:35::35 over port 53/UDP on any active connection. The user is not protected by Umbrella or reporting to Umbrella. The system's DNS settings are now back to their original settings (DHCP or Static).*

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/cisco_umbrella_status.PNG)

As resut a yellow icon is shown on Cisco Umbrella client, according to the documentaiton this could be enough to have Cisco Umbrella disabled, but in the implementation I've tested, even with the Cisco Umbrella DNS is unreachable the proxy features will still run.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/cisco_umbrella_dns_disabled.png)

Including also the Cisco Umbrella proxy in the deny list of the firewall rue will have this bypass working. The connection is estabilished directly at the timeout, so a website that opens multiple TCP connections will require a longer than usual time to load, rather when a single TCP connection is enough (like a VPN over HTTPS) once the first timeout is gone will have the usual performances.
Based on the rights you have on your PC you can have this rule in Windows Firewall or in your home Firewall (if it has configurable options like openWrt).

IP Addresses to block : (Proxy) 146.112.0.0/16, (DNS) 208.67.222.222, (DNS) 208.67.220.220 those ![addresses](https://support.umbrella.com/hc/en-us/articles/230563527-Using-Umbrella-with-an-HTTP-proxy) may be based on your region (mostly for the proxy performances).

> The IP addresses may change in the future and make this ineffective, to identify new IP addresses use the Cisco Umbrella documentation.

> The proxy in Cisco Umbrella is defined *intelligent proxy* and is not supposed to proxy all your web traffic (even if in my test all traffic were via Cisco Umbrella proxy) so you may have some cache in the Cisco Umbrella client that could stop your traffic.

> A future update may stop all network connectivity when not able to connect to Cisco Umbrella cloud services, having this bypass no longer effective.

### Use a local Proxy on Android

The traffic to the local network is not redirected to the Cisco Umbrella Proxy, in the below image there is an HTTP request to the LAN and WAN interface of the router web console. The first request to the LAN address is resolved directly, rather the WAN one is redirected via Cisco Umbrella Proxy, so having a proxy on a local interface is a bypass option. The local Proxy will resolve the DNS and handle all the traffic via your mobile connection.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/http_to_router_lan_wan_interface.png)

The IP addresses in the private range are not processed by Cisco Umbrella even if the traffic is processed via the IP gateway, this make sense as the Cisco Umbrella Proxy could not access the local resources for inspection. The actual implementation of Cisco Umbrella doesn't introduce any restriction on the traffic between local private addresses, so that any port port is allowed. In the below image, two networks with private addresses can communicate on HTTPS via the gateway (the router).

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/local_address_multihop.png)

Running the [Servers Ultimate](https://www.google.com/search?client=firefox-b-d&q=servers+ultimate) app, the Android device will act as Proxy, use the USB Tethering connection to create a local network interface. In Firefox and Softether (or any other application that support HTTP Proxy) configure the Proxy option to redirect the traffic through the local interface of the Android phone.

This bypass may not work if future updates of Cisco Umbrella will inspect local traffic for proxy connection.

> Cisco Anyconnect (that may include also Cisco Umbrella) when connected to a VPN will force all the traffic (even the one that could be resolved locally) via the VPN, this will make ineffective the local proxy bypass.

> Based on the administrative rights available, you could disable "Cisco AnyConnect" from the local interface to the Android device and set the *metrics* to an high enogh value (say 1000) to have only the local traffic (so the one to the local proxy) via your Android connection. Without all traffic is via Android, even system updated or others that may consume high volume of data.
> ![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/tethering_interface_settings.PNG)

> If the Android device is connected to a WiFi network it will work as long as the WiFi network has no Cisco Umbrella on it (or any other or restriction), so it will work on a home WiFi connection but likely no on a enterprice WiFi connection.

### Use a local Proxy on your home network

Cisco Umbrella usage is increasing while more people are working from home, so the same approach of using a proxy server running on Android can be rebuild using a proxy running on your openWrt router or any other local resource (a Raspberry or similar).

If you have an ![openWrt](https://openwrt.org/) router, you can run ![tinyproxy](http://tinyproxy.github.io/) and trasfer your traffic to it instead to the Android device or NATting the traffic to 146.112.0.0/16 ports TCP 80, 443 to your tinyproxy because as per below image the Cisco Umbrella Proxy is a standard HTTP Proxy (doesn't have any special sintax).

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/cisco_umbrella_proxy_443.PNG)

> As side information, the traffic that is not HTTP/HTTPS may be allowed based on the configuration. In the above picture the QUIC protocol is not transferred to a proxy (even because is UDP and the Cisco Umbrella Proxy seems an HTTP Proxy and not a SOCKS5 Proxy) but other protocols may be stopped.

A *PREROUTING* rule in the iptables of your openWrt router to trasfer the TCP 80 and 443 to your router on the port were tinyproxy is listing (48241 in the example) will intercept all the traffic with destination Cisco Umbrella Proxy.

```
{
iptables -t nat -A PREROUTING -d 146.112.0.0/16 -p tcp  --dport 443 -j DNAT --to-destination 192.168.127.1:48241
iptables -t nat -A PREROUTING -d 146.112.0.0/16 -p tcp  --dport 80  -j DNAT --to-destination 192.168.127.1:48241
}
```

In the below image, even if the proxy IP address is in the subnet 146.112.0.0/16 the request is processed by tinyproxy. As result is not required to alter the browser (or any other application to be allowed) configuration as in case of an Android Proxy.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/main/img/cisco_umbrella_forwarding_tinyproxy.PNG)

### Using scrcpy to remote into your Android

If your end goal is only to browse websites that are not allowed via Cisco Umbrella, another option is to use [scrcpy](https://github.com/Genymobile/scrcpy) that via USB will remote your Android device into a window. Then your can browse with your mobile connection.
