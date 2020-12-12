# DS-Lite-Debian
Dual Stack Lite (RFC 6333 + RFC 6334) implementation for Debian.

I was locking for a DS-Lite soliution for a router runnign with Debian. All consumer grade off the shelve routers (DS-Lite is not used in buisness grade environments and thus not supported by buisness grade hardware) I tried performed poorly on my FTTH connection. I had problems finding a sufficiently good soliution (OpenWRT supports it, but I do not have a fitting device) on the web, thus I decided to share the soliution which I figured out with you.

I want to make clear that I am not a specialist in the field. So please feel free to suggest improvements.

# Installation

## Prerequisites

It is asumed that you already have a network interface configured that connects to your ISP with with an IPv6 only connection. This interface should be configured to use dhcpcd as a client (the standard isc-dhcp-client might work as well, but I was not able get it to word with a ppp interface). The interface should already obtain a global IPv6 address, but no IPv4 address from your ISP.

To configure a ppp interface with ipv6 only add the following to your /etc/ppp/options file:
```
-ip
noip
noipx

+ipv6 ipv6cp-use-ipaddr
ipv6 ,
```

## Configure dhcpcd

In your /etc/dhcpcd.conf add the following:
```
allowinterfaces ppp0       # replace ppp0 with your interface
interface ppp0
    ipv6only               # dhcpv6 only
    ia_na                  # request a global ipv6 address
    ia_pd 1 eth0           # your LAN interfaces which you want to assign a prefix to
    define6 64 domain aftr # request the hostname of the AFTR element (this is mandatory accoriding to RFC 6333, see RFC 6334 for more info)
    require dhcp6_64
```

The last step is to place the 2 scripts dhcpcd.enter-hook and dhcpcd.exit-hook in your /etc/ directory.

## Testing

If everything is set up correctly you should see an additional interface called ppp0_dslite on your device. This interface is constructed and destructed when connecting or disconnecting to your ISP. If the interface is there, your should be able to send out and receive IPv4 packets on your device. It is worth mentioning that RFC 6333 specifies a Gateway-Based Architecture and a Host-Based Architecture. In my case I was not able to get the Host-Based Architecture to work (my ISP does not seem to support it). If you are not able to send out IPv4 datagrams, try to change the IPv4 address of your ppp0_dslite interface to any private IPv4 address (this is done in the dhcpcd.exit-hook script, keep the peer address as it is).
The RFC 6333 clearly states that your should NOT do a nat on your ppp0_dslite interface. The AFTR does the nat for you.

## Fragmentation and MTU

RFC 6333 says that the IPv4 packets should not be fragmented before entering the ipip6 tunnel. The fragmentation should only happen on the IPv6 packets. Tunneling the packets adds an additional IPv6 header to the packets and thus increases the packet size. The standard suggests that the ISP increases the MTU between you and the AFTR by at least 40 bytes to prevent fragmentation but that is dependent on your ISP and I can not give a general soliution here. Also not all network hardware supports a MTU bigger than 1500. Just make sure that your IPv4 packets are not fragmented by the tunnel. Ideally no fragmentation has to be done as this is bad for performance.
