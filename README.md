![image](https://github.com/user-attachments/assets/d91edb33-105c-4787-a521-e403334216a0)


## Configuring OpenWrt to work with Japan Biglobe 10Giga (MAP-E) service


#### Issue
- I have switched from the Biglobe Hikari 1G package to the Biglobe 10Giga package, but the transition encountered difficulties because IPv4 is no longer supported. It only supports IPv6, and I had to find a way to use MAP-E to enable IPv4 functionality.
- Although I am eager to use other Router OS options like OPNsense or PFSense, they cannot obtain IPv6 from the ISP due to an incompatible prefix.
- OpenWrt is my last resort, but I will try again with OPNsense later.

#### Configuration
By default, upon booting after installation, three interfaces are preconfigured: lan, wan, and wan6.


1. System > Software: Install the required add-on package map for MAP-E/MAP-T support, you will need to reboot before you can use it.
![image](https://github.com/user-attachments/assets/d2aab642-8a88-417e-bbc2-3d229511f869)

2. Configure wan interface
```
config interface 'wan'
  option device 'eth7'
  option proto 'dhcp'
```


3. Configure wan6 interface

   Enable WAN6 with DHCPv6, firewall setting you probably need to add this to WAN Zone (same as IPv4 WAN) for protection.

   Under DHCP Server > IPv6 Setting, follow these settings:

    - Designated master ON
    - RA-Service: relay mode
    - DHCPv6-Service: relay mode
    - NDP-Proxy: relay mode
    - Learn routes: ON
  
```
config interface 'wan6'
	option device 'eth7'
	option proto 'dhcpv6'
	option reqaddress 'try'
	option reqprefix 'auto'
	option norelease '1'
```

![image](https://github.com/user-attachments/assets/f5af299c-145f-43ad-b2a9-878f6a208ec7)

4. Configure LAN interface

In the RA-Service, I use Server mode because I have already received an IPv6-PD from wan6. Therefore, I will assign it directly to the clients.

   DHCP Server > IPv6 settings, basically very similar to WAN6 but Designated master OFF
   
![image](https://github.com/user-attachments/assets/e8907795-a28f-479f-b3b9-be8630e20b02)

IPv6 RA Settings: Default 
![image](https://github.com/user-attachments/assets/202f95e2-98cb-4575-ae16-9b06d0c77a5f)


5. Before we create the MAP-E interface, the parameter calculation for MAP-E is needed, in reference section I have attached the IETF information but someone has created a page to calculate, you can copy the public IPv6 address from WAN6 interface and use this [online MAP-E rule calculator](http://ipv4.web.fc2.com/map-e.html):

* You can get `IPv6 プレフィックスかアドレスを入力` from wan6 interface

![image](https://github.com/user-attachments/assets/a1477f22-c498-4f25-b2a6-18c180ec6742)

*   Next will be setting up new MAP-E interface, create a new interface and name it (e.g. _WAN6MAPE_), and fill the parameters using above generated parameters:
    *   Protocol: MAP/LW4over6
    *   Type: MAP-E
    *   BR/DMR/AFTR: _\[peeraddr\]_
    *   IPv4 prefix: _\[ipaddr\]_
    *   IPv4 prefix length: _\[ip4prefixlen\]_
    *   IPv6 prefix: _\[ip6prefix\]_
    *   IPv6 prefix length: _\[ip6prefixlen\]_
    *   EA-bit length: _\[ealen\]_
    *   PSID-bits length: _\[psidlen\]_
    *   PSID offset: _\[offset\]_
    *   From advanced settings, make sure it has **WAN6 as Tunnel Link**, and check the box **Use legacy MAP**:
   ![image](https://github.com/user-attachments/assets/9877d47d-3f96-4eef-bb1e-a5ad1b281a53)

![image](https://github.com/user-attachments/assets/5d0bec45-9d6a-4112-aeab-45636f82376e)


Note: Don't forget to add this _WAN6MAPE_ interface to same firewall zone as WAN/WAN6 since this is also part of WAN.


6. All done. More infogmation https://github.com/fakemanhk/openwrt-jp-ipoe/blob/main/README.md
