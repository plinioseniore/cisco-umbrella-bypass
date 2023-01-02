# Cisco Umbrella

## Introduction

Cisco Umbrella is a cloud security product, the below is an analysis of how Cisco Umbrella works with the modules DNS and SWG (Secure Web Gateway).

> DISCLAIMER : This is a study of how Cisco Umbrella works and which are the possible bypass, there is no intention to promote the bypass of Cisco Umbrella. If your organization has placed Cisco Umbrella, is to protect your devices from cyberthreats.
> If you are using this information for any goal than is not a study or analysis, you are doing that at your own risk. 

In the verified configuration, Cisco Umbrella acts just after the IP stack of the device, apparently without changing the IP configuration of the device.

The DNS module intercept all DNS traffic and redirect the same to the Umbrella DNS for filtering and logging, this allows restriction of access to filtered website (like Social Media or Web Mail, based on configuration). In the ```ipconfig /all``` output the DNS configuration is not altered.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/ipconfigall.png)

The SWG module is a cloud proxy, HTTP and HTTPS traffic is sent to Umbrella proxy, where it could be inspected. The HTTPS traffic is inspected with a Man-in-the-Middle (MITM) approach, decrypted at proxy end and re-encrypted using a Cisco certificate installed on the device.

The below image shows the same website with the SWG module enabled and with a bypass active, in the first case, the site traffic is re-encrypted with a Cisco certificate. The browser will not alert the user as the certificate is installed and trusted in the device.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/SWG_SSL_Inspection_1.png)

Here the details of the certificates:

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/SWG_SSL_Inspection_2.png)

The decryption of the traffic, allows Cisco Umbrella to verify the content to identify potential issues, but at same time allow them to access all the traffic in plain, including sensitive information. In the below image, inspection has been trigged on an ecommerce website.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/SWG_SSL_Inspection_3.png)

Assuming that Cisco Umbrella cannot be disabled on the device, the following actions will bypass it:
- Rules in Firewall
- HTTP(S) Proxy in Android
- HTTP(S) Proxy in the router
- Virtual Machine
- Virtual Machine HTTP(S) Proxy
- Tunnel Traffic (VPN)

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/ciscoumbrellaservices.PNG)

### Rules in Firewall

Using a firewall to restrict access to Cisco Umbrella servers, will trigger the deactivation on the client after a timeout. The firewall rule can be deployed either on the Windows one or into a network device.

In the [Cisco Umbrella documentation](https://docs.umbrella.com/deployment-umbrella/docs/appx-a-status-and-functionality), is listed that if *There is at least one active network connection; however, the Umbrella roaming client can’t connect to 208.67.222.222 / 208.67.220.220 / 2620:119:53::53 / 2620:119:35::35 over port 53/UDP on any active connection. The user is not protected by Umbrella or reporting to Umbrella. The system's DNS settings are now back to their original settings (DHCP or Static).*

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/cisco_umbrella_status.PNG)

As result of a firewall rule that restrict the access to the above IPs, a yellow icon is shown on Cisco Umbrella client, according to the documentation this could be enough to have Cisco Umbrella DNS disabled.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/cisco_umbrella_dns_disabled.png)

In the detail pane the DNS protection is shown as disabled, the SWG will continue to run and the Umbrella filter will still be effective via the cloud proxy.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/DNS_1.png)

Including also the Cisco Umbrella proxy in the deny list of the firewall rule, will have this bypass working. The connection is established directly at the timeout, so a website that opens multiple TCP connections will require a longer than usual time to load, rather when a single TCP connection is enough, once the first timeout is gone, you will have the usual performances.

IP Addresses to block : (Proxy) 146.112.0.0/16, (DNS) 208.67.222.222, (DNS) 208.67.220.220 those addresses may vary for different regions (mostly for the proxy performances).

> The IP addresses may change in the future and make this ineffective, IP addresses are listed into the Cisco Umbrella documentation.

Based on the notes by [Andre Camillo](https://medium.com/swlh/a-study-on-how-cisco-umbrella-roaming-client-works-f3cd552c7112), there should be some redirect of the DNS traffic to the Cisco Umbrella services, rather in the implementation under test only TCP port 53 is bind to *dnscrypt-proxy.exe*.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/netstat.png)

Furthermore, there is no DNS traffic with 208.67.0.0/16

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/udp_probe.PNG)

and DNS traffic seems directly to the home router (even if some domains are not resolved via the local DNS)

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/dcs_requestes.PNG)

As all (or most) of the web traffic seems to go via the Cisco Umbrella Proxy, it could be that the implementation under test is not really using DNS and the filter is done at proxy level. The log of the DNS on the OpenWrt router shows DNS requests received, even if there is no proof that all the DNS traffic goes via local DNS and not via 208.67.0.0/16 over an encrypted communication.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/openwrt_dns_log.PNG)

> The proxy in Cisco Umbrella is defined *intelligent proxy* and from the documentation seems to not be supposed to proxy all the traffic. In this study, has been seen that all the traffic goes via the proxy, but not all traffic is inspected. So, some HTTPS website will be presented in the browser with their original certificate. 

### Use a local Proxy on Android

This approach leverage the use of local traffic to bypass Cisco Umbrella. The traffic to the local network is not redirected to the Cisco Umbrella Proxy, in the below image there is an HTTP request to the LAN and WAN interface of the router web console. 
The first request to the LAN address is resolved directly, rather the WAN one is redirected via Cisco Umbrella Proxy, so any traffic on the local network will not be inspected. 

So, having a proxy on a local interface is a bypass option. The local Proxy will resolve the DNS and handle all the traffic via the phone mobile connection.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/http_to_router_lan_wan_interface.png)

The IP addresses in the private range are not processed by Cisco Umbrella, even if the traffic is processed via the IP gateway, this makes sense as the Cisco Umbrella Proxy could not access the local resources for inspection. The actual implementation of Cisco Umbrella doesn't introduce any restriction on the traffic between local private addresses, so that any port is allowed. In the below image, two networks with private addresses can communicate on HTTPS via the gateway (the router).

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/local_address_multihop.png)

Having the Android device running an HTTP(S) Proxy app over the local network, allow the redirection of the traffic far from Umbrella. The application that need to bypass Umbrella should support the use of HTTP Proxy. Firefox and all major browser has this option included, VPN software like Softether also.

The Android phone shall be configured as USB Tethering interface, so that on your device a new local interface is available.

Generally Cisco Umbrella is deployed on the device via Cisco AnyConnect:

> If administrative rights are available, "Cisco AnyConnect" can be disabled from the local interface to the Android device and the *metrics* can be set to an high enough value (say 1000) to have only the local traffic (so the one to the local proxy) via the Android connection. Without, all traffic is via Android, even system updated or others that may consume high volume of data.
> ![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/tethering_interface_settings.PNG)

The use of the mobile network is not strictly required:

> If the Android device is connected to a WiFi network, it will work as long as the WiFi network has no Cisco Umbrella on it (or any other or restriction), so it will work on a home WiFi connection but likely no on a enterprise WiFi connection.

This, will work only if the traffic is redirected to the local interface where the proxy is listening, in the other cases it will not work.

> Cisco AnyConnect (that may include also Cisco Umbrella) when connected to a VPN may force all the traffic (even the one that could be resolved locally) via the VPN, that will make ineffective the local proxy bypass.

### Use a local Proxy on your home network

Cisco Umbrella usage is increasing while more people are working from home, so the same approach of using a proxy server running on Android can be rebuild using a proxy running on your OpenWrt router or any other local resource (a Raspberry or similar).

With an [OpenWrt](https://OpenWrt.org/) router, run [tinyproxy](http://tinyproxy.github.io/) and transfer the traffic to it instead of the Android device. Another alterative is NATting the traffic to 146.112.0.0/16 ports TCP 80, 443 to tinyproxy because as per below image the Cisco Umbrella Proxy is a standard HTTP Proxy (doesn't have any special syntax and doesn’t verify the connection authenticity).

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/cisco_umbrella_proxy_443.PNG)

> As side information, the traffic that is not HTTP/HTTPS may be allowed based on the configuration. In the above picture the QUIC protocol is not transferred to a proxy (even because is UDP and the Cisco Umbrella Proxy is an HTTP Proxy and not a SOCKS5 Proxy).

### NATting Traffic and perform an MITM

A *PREROUTING* rule in the iptables of your OpenWrt router, to transfer the TCP 80 and 443 to the router on the port where tinyproxy is listening (48241 in the example), will intercept all the traffic with destination Cisco Umbrella Proxy.

```
iptables -t nat -A PREROUTING -d 146.112.0.0/16 -p tcp  --dport 443 -j DNAT --to-destination 192.168.127.1:48241
iptables -t nat -A PREROUTING -d 146.112.0.0/16 -p tcp  --dport 80  -j DNAT --to-destination 192.168.127.1:48241
```

In the below image, even if the proxy IP address is in the subnet 146.112.0.0/16 the request is processed by tinyproxy. As result is not required to alter the browser (or any other application to be allowed) configuration to transfer the traffic to the local Proxy.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/cisco_umbrella_forwarding_tinyproxy.PNG)

> NATting the HTTP/HTTPS traffic and leaving access to the Cisco Umbrella DNS, means that URL are still resolved by it, this may imply a filtering and/or monitoring of which internet resources you are accessing (without an access to the dataflow).

When an application (like Firefox) is configured to use an HTTP Proxy, it will pass to the proxy the URL that is then resolved by the Proxy with it's own DNS.

### Use Virtual Machines

Virtual Machines doesn't use the TCP/IP stack of the host operating system if used in Bridged Mode, so the traffic doesn't trigger Cisco Umbrella. The easiest option is run a full system in the Virtual Machine, including the browser and the other applications you want to run without Cisco Umbrella proxy. This could have impact on the overall used resources.

Another option, is using the Virtual Machine for minimum network functionalities, in the below picture, OpenWrt is running via VirtualBox on the same host were Cisco Umbrella is installed.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/openwrt_vbox.PNG)

Running OpenWrt with two network adapters, one host-only adapter with a local address and the other bridged to the internet connected network, will allow to leverage the same proxy bypass used previously.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/vboxnetworkadapters.png)

The traffic could be redirected to the host-only interface *192.168.56.2* configuring the proxy in Firefox or in the other application in use, having OpenWrt running tinyproxy would emulate the proxy bypass seen above.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/openwrt_interfaces.PNG)

The NATting can also be applied with OpenWrt running in the virtual machine, but in that case, it shall be ensured that the traffic will not flow outside of the virtual machine. This can be achieved ensuring that the interface connected to internet has a working configuration only in the Virtual Machine and not in the host device Operative System.

### Tunnel Traffic (VPN)

The tunnelling of the traffic it doesn't bypass Cisco Umbrella by its own, because Umbrella will intercept the traffic directed to the virtual adapter of the VPN and apply the usual filtering. Using a tunnel, could help if on the local device cannot be deployed firewall rules, proxy and virtual machines as described above.

The bypass action can be deployed on a device available on the public internet and the VPN is used to redirect the traffic via it.

In order to estabilish the connection to the VPN Server, it should be leveraged one of those two options:
- Use TCP ports that are not proxied to Umbrella
- Use UDP for the connection, as Umbrella proxy only TCP traffic on ports 80 and 443.

The URL will still be processed by the DNS module of Umbrella, so the connection may have to point directly to the public IP address of the VPN Server. The test has been successful with Softether VPN and Wireguard VPN protocols, connecting the device under Umbrella with an own-hosted server.

> It cannot be excluded that Umbrella applies also filters at IP level for known addresses.

> The use of commercial VPNs servers will not be effective, because those cannot be configured to filter DNS and mimic the proxy. Furthermore, their IPs and URLs are likely in a deny list of Umbrella.

## Bypass and Cisco AnyConnect

Likely if you have Cisco Umbrella you also have Cisco AnyConnect as VPN, if so is also likely that the VPN client is configured to send all the traffic in the VPN tunnel.

In this case any local traffic is redirect in the tunnel and will never reach the proxy, so the bypass will no longer work. The VPN client achieve this forcing the IP routes on your computer.

There is a workaround, Umbrella is advertised as a protection mechanism and this overlap with one of the use of VPNs: tunnel traffic into a firewall and restrict traffic as protection. Umbrella already tunnel all HTTP(S) traffic to its proxy, so there is no longer a reason to have Umbrella traffic tunneled in a corporate VPN.
Looking at your routes while the VPN is active you may discover that Umbrella is allowed outside of the tunnel.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/routeprint_umbrella1.PNG)

If so, the workaround is quite straightforward, create a virtual interface and assign an IP address in the Umbrella range, you want need to really get in touch with those addresses but you can run your proxy on that interface.
AnyConnect will not force that traffic into the tunnel and you will still have a local resource accessible even while the VPN is active.

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/localinterface_umbrellaaddress.PNG)

As result, when the VPN client force the IP routes it will not touch the Umbrella IP addresses. 

![](https://raw.githubusercontent.com/plinioseniore/cisco-umbrella-bypass/pages/img/routeprint_umbrella2.PNG)

Attaching a VirtualBox VM on that virtual interface will complete the trick, in the above example the IPs 146.112.0.2 and 146.112.0.3 are available to run the local proxy.

> Umbrella doesn't tunnel HTTP traffic to itself into its HTTP proxy, is quite understandable why. As result, you cannot access local services running port 80 and 443 trough this approach.

## Bypass Analysis

Based on what seen above, the bypass options acts around the followings:
- Restricted access to the Umbrella IPs, so that Umbrella disable itself.
- Use local traffic to tunnel or proxy data, as the local traffic cannot be inspected by a cloud based device.
- Intercept traffic to Umbrella and redirect to a local DNS and Proxy server, as Umbrella doesn't verify the authenticity of the server it connects to.
- Tunnel traffic into UDP pipes or on alternate TCP ports, as only HTTP and HTTPS on standard ports seems redirected.

Those techniques can (or may need) to be combined together to be effective, as example, even tunnelling the traffic cannot be effective alone if the DNS is still resolved by Umbrella and the URL is filtered.

# Credits

This study is based on some experiments done by my friend [lalontra]() and me, with reference to the following web resources:

- [Cisco Umbrella - Appendix A – Status, States, and Functionality](https://docs.umbrella.com/deployment-umbrella/docs/appx-a-status-and-functionality)
- [A Study on How Cisco Umbrella Roaming Client Works](https://medium.com/swlh/a-study-on-how-cisco-umbrella-roaming-client-works-f3cd552c7112)
- [What is Cisco SWG](https://umbrella.cisco.com/blog/what-is-secure-web-gateway)
- [Use Umbrella with HTTP Proxy](https://support.umbrella.com/hc/en-us/articles/230563527-Using-Umbrella-with-an-HTTP-proxy)
- [Bypass Cisco Umbrella & OpenDNS website block](https://www.youtube.com/watch?v=SbN2Nzzy59Q)
- [OpenWrt on VirtuaBox](https://OpenWrt.org/docs/guide-user/virtualization/virtualbox-vm)
