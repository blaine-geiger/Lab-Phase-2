# Home Lab Phase 2 (buckle in, this phase is a long one)

## Installation of Home Lab Network Devices

Inspect input and output ports on all devices to plan for proper physical connections of network devices.
In this section we will visually inspect the physical ports and cabling for each device.

### Firewall (Netgate SG-1100 pfSense)
<p align="center">Inspection of firewall I/O ports
  <br/>
  <img src="https://imgur.com/r7VcRNf.png" height="80%" width="80%" alt=""/><br /><br />
</p>

<p align="center">Firewall I/O ports and VLAN tags
  <br/>
  <img src="https://imgur.com/gCf75F9.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

USB ports on the device can be used for a variety of purposes.
The primary use for the USB ports is to install or reinstall the operating system on the device. Beyond that, there are numerous USB devices which can expand the base functionality of the hardware, including some supported by add-on packages. For example, UPS/Battery Backups, Cellular modems, GPS units, and storage devices. Though the operating system also supports wired and wireless network devices, these are not ideal and should be avoided [Netgate.com]. 

<p align="center">Reverse view of firewall
  <br/>
  <img src="https://imgur.com/Ic7VQKJ.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

From left to right.
- **Power Connector**
  - 12VDC 2A Center Pin Positive
  - Power Consumption 3.48W (Idle)
- **Micro-USB Serial Console**
- **Recessed Reset Button**
  - Performs a hard reset, immediately turning the system off.


### Managed Switch (Netgear GS108Ev3)

<p align="center">I/O ports of Netgear Managed Switch
  <br/>
  <img src="https://imgur.com/BBQHv6t.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>



<p align="center">The Netgear GS108Ev3 has 8 Gigabit ports, with advanced supported features.
  <br/>
  <img src="https://imgur.com/QsKOt86.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

### Mini-PC (GMKtec M5 Pro)

<p align="center">Inspection of GMKtec PC  I/O ports
  <br/>
  <img src="https://imgur.com/EQEF0Yr.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

## Flow of ethernet cabling and associated ports on devices. 
Each separate cable section is named ETH1, ETH2...etc. as they connect to each device.
Main ethernet trunk:

- The main ethernet trunk ETH-1 exits the modem.
- ETH-1 enters the WAN port on the firewall.

**Home LAN:**
- ETH-2 exits the LAN port on the firewall.
- ETH-2 enters the WAN port on the existing ASUS router.
- ETH-3 exits the ASUS router to PC1 (existing desktop).
- ETH-4 exits the ASUS router to PC2 (existing desktop).
- ETH-5 exits the ASUS router to PC3 (existing laptop).

**Lab LAN:**
- ETH-6 exits OPT port on firewall (physical hardware segmentation).
- ETH-6 enters port-1 on the managed switch.
- ETH-7 exits port-8 on the switch.
- ETH-7 enters the ethernet port on the PC.
  
Network layout and flow of ethernet connections is shown in the image below.

<p align="center">
  <br/>
  <img src="https://imgur.com/ck9dgGX.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>



Before we start the next section and momentarily lose a reliable internet connection, we need to use the internet to download a few items we will need.
- Visit https://docs.netgate.com/manuals/pfsense/en/latest/sg-1100-security-gateway-manual.pdf and download the firewall manual.
- Visit https://www.downloads.netgear.com/files/GDC/GS108V3/GS108v3_IGprt_24sept2012.pdf  and download the managed switch manual.
- Visit https://www.netgear.com/support/product/netgear-discovery-tool/#download and download the Netgear Discovery Tool for the appropriate OS (Windows for our case)

## Install and Configure Firewall

***Note***: Some network devices may have the same default IP address. If we attempt to add a new device and its default IP matches a device that already exists on the network, an IP conflict will occur. If this is the case, communication between the devices cannot take place because they share the same IP address. Installation and configuration will not be possible.

First, we will change the IP address of the existing router. Its default IP is the same as the firewall we are trying to install. If we don’t change either the firewall IP or the router IP, we will have the problem from the note above. We will also disable the DHCP server on the router because the firewall will become the DHCP server (if there is more than one DHCP server, this will also cause conflicts). Finally, placing the router into AP mode will disable NAT on the router, because the firewall will also handle this task from now on (double NAT is possible, but redundant, unnecessary, and can decrease network performance).

Open a web browser while connected to the currently existing Wi-Fi signal:
1. Type `192.168.1.1` into the address bar and press Enter.
2. This takes you to the router’s configuration page.
3. Go to the router's Advanced Settings.
4. Select the LAN section from the left side column.
5. Select the LAN IP tab.
6. Change the IP address to `192.168.1.x`:
   - `x = 2 – 254` but should be low in range (2 – 10).
7. Click Apply to save the setting.
8. Go to LAN from the left side column.
9. Select the DHCP Server tab.
10. Under DHCP Server, set Status to Disabled.
11. Click Advanced Settings.
12. Click Administration.
13. Click Operation Mode.
14. Select Access Point (AP) mode.
15. Click Save.

> We have changed the IP of the router; its configuration page can now be accessed from the new address of `192.168.1.x`. We disabled DHCP services. We placed the router into AP mode.

1. Power down the existing router and modem.
2. Disconnect ethernet from the modem.
3. Do not connect the power source to the firewall at this time.
4. Place an ethernet cable connecting the modem to the WAN port of the firewall device (to be called ‘firewall’ from this point on).
5. Place a second ethernet cable from the LAN port of the firewall to the ethernet port of the laptop (laptop should be powered on).
6. Plug in the firewall and allow it to boot up fully (4-5 minutes).
7. One green light will be solid (power light).
8. One green light will flash as the firewall boots:
   - Once the flashing light stops and remains solid green, the firewall is fully booted.
9. On the laptop, open a web browser and enter `192.168.1.1` in the address bar:
   - This is the default IP of the firewall and will take us to the web GUI where we can configure the device.
10. Use the default credentials to enter GUI:
    - `admin : pfSense` (we will be asked to change these later).


- On the General Information page:
1. Hostname: change or leave default (pfsense).
2. Domain: change or leave default (home.arpa).
3. DNS Servers: we will use Quad9:
   - DNS Server 1: `9.9.9.9`.
   - DNS Server 2: `149.112.112.112`.
4. Disable DNS Server Override (avoids ISP overriding my choice of DNS server).
5. Disable DNS Resolver (avoids the firewall acting as the primary DNS resolver).
6. Enable DNS Forwarder (properly forwards to specific DNS server `9.9.9.9`).
7. Click Next.

- Timer Server Information page:
1. Time Server Hostname: default.
2. Timezone: New York (EST) or whatever is appropriate.
3. Click Next.

- WAN Interface page (public IP from ISP and modem):
1. Selected Type: DHCP.
2. Default for all other fields.
3. Click Next.

- LAN IP Address and Subnet Mask:
1. The firewall is the first device on the network:
    - Use a `192.168.1.x`:
      - `x = 1 – 254` but should be low in range (1 – 10) and not the same as the IP of the router we changed earlier.
2. Click Reload to save the configuration and wait for the completed message.

3. After the finish message loads:
4. Click Finish.

Next, we configure the OPT (optional) port of the firewall. This will serve as the lab segment of our network. Using this port provides physical hardware segmentation of our lab network. This is more secure than basic subnet addressing or even VLAN segmentation. However, we will also use subnet addressing. Use of the OPT port and subnet addressing provides both physical and logical segmentation of the lab network. This isolation is important to keep lab testing secure and separate from the main home LAN

- Navigate to **Interfaces**.
1. Click the **Assignments** tab.
2. Select the **OPT interface** tab.
3. Click **Add**.
4. Make sure the enable box is checked.
5. IP type: **Static**.
6. Set static IP `192.168.2.x` /24:
   - `x = 1 – 254` but should be low in range (1 – 10).
   - This sets our lab subnet of `2.x`.
     - This logically separates it from the home LAN subnet of `1.x`.
7. Upstream gateway: **None**.

- Click the **Services** tab:
1. Click the **OPT interface** tab.
2. Click the **DHCP server** tab:
    - Set pool for ~50 devices (example `192.168.2.50 – 192.168.2.100`).
    - Click **Save**.

- Click Services tab:
1. Click LAN interface tab.
2. Click DHCP server tab:
    - Set pool for ~50 devices (example `192.168.1.50 – 192.168.1.100`).
    - Click Save.

- Click Services tab:
1. Click NAT tab.
2. Click OPT interface tab.
3. Enable Outbound NAT for the new OPT interface (this is applied to the LAN interface by default, so we don’t need to activate NAT for the LAN interface).

## Firewall Configuration

Finally, we must configure manual firewall rules for the OPT interface we have enabled, or communication will not take place. Firewall rules are automatically applied to the LAN interface by default, so we only need to do this for the OPT interface.

1. Click the Firewall tab.
2. Click the Rules tab.
3. Click the OPT interface tab:
  - Click Add:
    - Action: Allow.
    - Protocol: TCP/UDP.
    - Source: `192.168.2.0 /24`.
    - Destination: Any.
    - Port: 53 (DNS).
    - Description: Allow DNS traffic.
    - Click Save.
    - Click Apply.

  ## Firewall Rules Configuration

- Click Add:
  - Action: Allow
  - Protocol: TCP
  - Source: `192.168.2.0 /24`
  - Destination: Any
  - Port: 80 (HTTP)
  - Description: Allow HTTP traffic
  - Click Save
  - Click Apply

- Click Add:
  - Action: Allow
  - Protocol: TCP
  - Source: `192.168.2.0 /24`
  - Destination: Any
  - Port: 443 (HTTPS)
  - Description: Allow HTTPS traffic
  - Click Save
  - Click Apply

- Click Add:
  - Action: Allow
  - Protocol: ICMP
  - Source: `192.168.2.0 /24`
  - Destination: Any
  - Description: Allow outbound ICMP traffic (outbound ping)
  - Click Save
  - Click Apply

- Click Add:
  - Action: Allow
  - Protocol: UDP
  - Source: `192.168.2.0 /24`
  - Destination: Any
  - Port: 67 (server) and 68 (client) (DHCP)
  - Description: Allow DHCP traffic
  - Click Save
  - Click Apply

- Click Add:
  - Action: Block
  - Protocol: Any
  - Source: `192.168.2.0 /24`
  - Destination: Any
  - Description: Block all other traffic (optional, but recommended)
  - Click Save
  - Click Apply

- Logout of the firewall GUI.
- Unplug power from the firewall device.

## Install and Configure Managed Switch:

1. Unzip the Discovery Tool (NSDT) we downloaded earlier.
2. Start the NSDT to install the program.
3. Connect an ethernet cable from your computer to port 1 on the switch.
4. Power on the switch (plug it in).
5. Open the NSDT program.
6. Click the ‘Choose a connection’ dropdown box.
7. Choose the available network.
8. Click **start searching**.
9. After it processes to 100%:
   - Click the **‘Admin Page’** button.
10. Enter the default login credentials that are printed on the label found on the bottom of the switch.
11. When prompted:
    - Change the admin name and password.
    - The switch’s GUI page will refresh.
    - Enter the new login credentials we just created.


1. Go to the **admin switch configuration** tab:
   - Disable the DHCP service (this allows us to set a static IP for the switch).
   - Set a static IP address on the `192.168.2.x /24` subnet:
     - Avoid using an IP address that is in the DHCP range we set earlier on the firewall (we used the example of `192.168.2.50 – 192.168.2.100`).
     - Also do not use the IP of the firewall’s OPT port we assigned earlier.
   - Set the gateway to the firewall’s OPT port IP address we assigned earlier.
2. Click **Save**.
3. Allow the configuration to be applied to the switch.
4. Once a finished message is received:
   - Unplug the ethernet cable from the computer and the switch.
5. Power off the switch (unplug).

## Installation of Personal Computer

1. This step can take place after all devices are connected and the PC is powered on for the first boot up.
2. Connect all devices according to the ethernet cabling descriptions provided in the earlier section:
   - All devices should be unplugged as the ethernet connections are made.
3. Use both the written and visual references as necessary.
4. After all devices are connected with ethernet cabling:
   - Power on the modem and wait for boot up and full/stable connection.
   - Power on the firewall and wait for full boot up.
   - Power on the main home LAN router/AP and wait for full boot up.
   - Power on the managed switch until port lights become stable and stop flashing.
   - Lastly, power on the PC with the appropriate peripheral devices connected (mouse, keyboard, monitor, etc.).
5. The PC boot could take a few minutes (approximately 5 minutes).
6. We should see Windows OS messages regarding installation and updates.
7. Follow the simple steps of Windows installation.
8. Now we see the Windows home screen and our process is complete.

## Diagram of subnets, domains, devices, IPs, gateways, subnet masks and DHCP pools. 
<p align="center">
  <br/>
  <img src="https://imgur.com/jYWfrDr.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
