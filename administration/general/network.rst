Network
#######

In this section you can set several system network related settings.

General
=======

Hostname and domain settings.

Interfaces
==========

The grid displays only network interfaces configured through the |webui|. It is normal to see this panel empty if your default interface was configured by DHCP or static IP address during installation. The installer does not setup the network using |omv|, it just configures :file:`/etc/network/interfaces`.

The dashboard contains a network widget that displays current status of interfaces.


Ethernet
^^^^^^^^

Just select DHCP or static. |omv| is a server so the recommended setting is to have static IP address, if you have a proper network infrastructure (separate router and switch ). In a reboot, if the router fails to boot you can still access the |webui| through the switch bridge. If the switch also fails you can use a direct Ethernet connection with the static IP address assigned to the server NIC.

When using static configuration, be aware that the configuration window does not expand completely. Scroll down for the IPv6 fields and DNS fields. 

Do not leave DNS setting empty; it is essential for fetching updates. A common value is to use the same IP address as the gateway. If unsure, just use google DNS ``8.8.8.8``.

Wake on LAN (WOL)
	This enables WOL in the kernel driver, make sure the NICs supports this and the feature is enabled in BIOS.

Wireless
^^^^^^^^

Support for wireless network was added in |omv| 3.0. The configuration window displays the same IP configuration fields as Ethernet, plus the relevant wireless values: SSID (the wireless network name) and the password.

Whenever possible,  use Ethernet for a NAS server. Wireless should not be used in a production server; this feature is intended for extreme cases only. 


VLAN
^^^^

If your network supports VLAN, just add the parent interface and the VLAN id.

Bond
^^^^

The configuration window provides all available `modes <https://www.kernel.org/doc/Documentation/networking/bonding.txt>`_ for the bond driver. To configure bonding, is necessary at least two physical network interfaces. The |webui| allows the selection of less than two. This is by design for configuration purposes. The workflow is as follow for dual NICs:

- If the primary NIC is already working either by the installer, configure it through the |webui| as static. If set as static using the same IP address given by DHCP, it should not be necessary to re-login to the |webui|.
- Click ``Network | Interfaces | Add | Bond``, select the second available NIC, select the bond mode, fill the IP field and subnet mask values, leave gateway and DNS empty. Save and hit apply. 
- Log out and access the |webui| using the new IP address assigned to the bond interface created.
- Now select the primary interface configured through |webui| in the first step, and delete it. Save and hit apply. 
- Select the newly created bond interface, click edit add now the physical nic that was deleted from the step before should be available to select. Save and hit apply.
- The dashboard should now report the bond interface information (including speed).


.. note::

	* 802.3ad LACP (Link Aggregation) mode only works if physical interfaces are connected to a managed switch that supports aggregation.
	* Is not possible to achieve 2GBit bandwidth (or more depending on the number of NICs) in a single client using LACP, even if the client also has a LACP-bonded NIC or 10Gbit card;  there is no multipath support in Samba or other |omv| services in the way  Windows Server has for file sharing using SMB.
	* Higher speeds using link aggregation are limited by disk speed. When serving simultaneous clients make sure the physical media is capable of reaching the speed of the bonded NIC (e.g. SSD or RAID array).


Advanced Interface Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Proxy
=====

This panel configures the server proxies using system wide environmental variables. All software that obeys Linux proxy environmental variables should be able to use the proxy. This is useful for example if there are many Debian servers in the network, when performing :command:`apt` operations, packages can be cached in the proxy if this configured appropiatly to reduce download bandwidth. 

The variables name are::

	http_proxy
	https_proxy
	ftp_proxy

This settings do not configure |omv| to act as a proxy server.


Service Discovery
=================

This panel configures avahi-daemon announce services. You can disable selectively by service and/or change the common name announce. Plugins can add their service here.
Avahi announces are recognized by Linux file browsers by default. Mac OSX only recognizes SMB and AFP protocol in their sidebar. 

Windows does not understand Avahi announces. Samba announces to windows client using the NetBIOS daemon (nmbd). If windows network section does not display the samba server, this setting does not change that behaviour.

Firewall
========

This is grid panel for adding iptables rules. This can be useful if you need to secure access in your local network. Currently it is only possible to add rules to the OUTPUT and INPUT chains in the filter table. The configuration to load the rules at boot or network restart is located in this file :file:`/etc/network/if-pre-up.d/openmediavault-iptables`. The mkconf |omv| script uses a run-parts folder :file:`/usr/share/mkconf/iptables.d` where is possbile to store custom scripts to add rules to the NAT and RAW table or the FORWARD chain.

.. tip::
	* To avoid locking yourself out while testing, create a cron command to run every five minutes that flushes the OUTPUT/INPUT chain.
	``*/5 * * * * root /sbin/iptables -F INPUT && /sbin/iptables -F OUTPUT``

	* Before adding the last rule to reject all, add a rule before the reject all, to LOG everything. This will help understand why some rules do not work. The log is saved in dmesg or syslog.

.. tip::
         When seeking support please avoid posting screenshots of the grid panel, this is useless because it does not give the full overview of your firewall ruleset. Instead use::

       $ iptables-save > /tmp/file.txt
 
       If you have no sensitive information in the ruleset then you can create a text link::

       $ iptables-save | curl -F 'sprunge=<-' http://sprunge.us
