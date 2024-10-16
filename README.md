# Lab Phase 2: Installation and Configuration of Devices

### Private IP Address Ranges
Private IP addresses are not routable on the public internet and can be used within private networks. The standard ranges for private IP addresses are:
- 10.0.0.0 to 10.255.255.255
- 172.16.0.0 to 172.31.255.255
- 192.168.0.0 to 192.168.255.255
  
#### For the sake of clarity we will use the following examples:
  - 10.1.1.0 /24 for subnet 1
  - 10.1.2.0 /24 for subnet 2

> **Note:** All device IP addresses are examples and do not reflect the actual addressing scheme in my network setup. They are given as placeholder IP addresses for
> clarity to the reader.

## Summary of Sections
### Installation of Home Lab Network Devices:
- Inspect the input/output ports on the firewall, managed switch, and mini-PC to ensure proper cabling and connections.
### Firewall Configuration:
- Change the IP address of the existing router to avoid conflicts with the firewall. Disable its DHCP service and switch it to Access Point mode.
Connect the modem to the WAN port of the firewall and the LAN port to a laptop for initial configuration.
Set up DNS servers, time settings, and configure the LAN and OPT interfaces for network segmentation.
### Managed Switch Configuration:
- Disable DHCP on the switch and set a static IP within the lab’s subnet.
### Personal Computer Installation:
- Connect all devices according to the planned ethernet cabling. Power on the modem, firewall, and switch before booting the PC to complete the installation.
### Diagram of example Subnets, Domains, Devices, IPs, Gateways, Subnet Masks, and DHCP Pools
- Create detailed diagram for easy reference 
## Installation of Home Lab Network Devices
Inspect input and output ports on all devices to plan for proper physical connections of network devices.
- Firewall (Generic Security Gateway)
- Managed Switch (Generic Managed Switch)
- Mini-PC (Generic Mini-PC)

Before we start the next section and momentarily lose a reliable internet connection, we need to use the internet to download a few items we will need.
- Visit the manufacturer's documentation site to download the firewall manual.
- Visit the manufacturer's documentation site to download the managed switch manual.
- Visit the manufacturer's support site to download any necessary tools.

Network layout and flow of ethernet connections is shown in the image below.

<p align="center">
  <br/>
  <img src="https://imgur.com/H4sgfGe.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

## Firewall Configuration

> **Note**: Some network devices may have the same default IP address. If we attempt to add a new device and its default IP matches a device that already exists on the network, an IP conflict will occur. If this is the case, communication between the devices cannot take place because they share the same IP address. Installation and configuration will not be possible.

First, we will change the IP address of the existing router. Its default IP is the same as the firewall we are trying to install. If we don’t change either the firewall IP or the router IP, we will have the problem from the note above. We will also disable the DHCP server on the router because the firewall will become the DHCP server (if there is more than one DHCP server, this will also cause conflicts). Finally, placing the router into AP mode will disable NAT on the router, because the firewall will also handle this task from now on (double NAT is possible, but redundant, unnecessary, and can decrease network performance).

### Change Router IP
1. Open a web browser while connected to the currently existing Wi-Fi signal:
2. Type `10.1.1.1` into the address bar and press Enter.
3. This takes you to the router’s configuration page.
4. Locate the LAN settings.
5. Select the LAN IP.
6. Change the IP address to `10.1.1.2`:

### Disable Router's DHCP Server
1. Locate LAN settings
2. Select the DHCP Server
3. Under DHCP Server, set Status to Disabled.

### Change Router to AP mode
5. Locate router operation modes
6. Select Access Point (AP) mode.

> We have changed the IP of the router; its configuration page can now be accessed from the new example address of `10.1.1.2`. We disabled DHCP services. We placed the router into AP mode.

### Access Firewall Configuration
1. Power down the existing router and modem.
2. Disconnect ethernet from the modem.
3. Do not connect the power source to the firewall at this time.
4. Place an ethernet cable connecting the modem to the WAN port of the firewall device (to be called ‘firewall’ from this point on).
5. Place a second ethernet cable from the LAN port of the firewall to the ethernet port of the laptop (laptop should be powered on).
6. Plug in the firewall and allow it to boot up fully (4-5 minutes).
7. On the laptop, open a web browser and enter the default IP of the firewall and will take us to the web GUI where we can configure the device.
8. Use the default credentials to enter GUI:

### Set DNS Server
1. Locate DNS server:
2. Hostname: change or leave default
3. Domain: change or leave default
4. DNS Servers: There are many public DNS servers to choose from, some are more popular than others.
  - We can use Google for our example. 
   - DNS Server 1: `8.8.8.8`.
   - DNS Server 2: `8.8.4.4`.

### Configure WAN
- WAN Interface page (public IP from ISP and modem):
  - Select Type: DHCP.

### Assign and Configure LAN
- LAN IP Address and Subnet Mask:
  - The firewall is the first device on the network:
    - Example `10.1.1.1`:
      - cannot be the same as the IP of the router we changed earlier.

Next, we configure the OPT (optional) port of the firewall. This will serve as the lab segment of our network. Using this port provides physical hardware segmentation of our lab network. This is more secure than basic subnet addressing or even VLAN segmentation. However, we will also use subnet addressing. Use of the OPT port and subnet addressing provides both physical and logical segmentation of the lab network. This isolation is important to keep lab testing secure and separate from the main home LAN

### Assign OPT Interface
- Navigate to **Interfaces**.
1. Click the **Assignments** tab.
2. Select the **OPT interface** tab.
3. Click **Add**.
4. Make sure the enable box is checked.
5. IP type: **Static**.
6. Set static IP `10.1.2.1` /24:
   - This sets our lab subnet
     - This logically separates it from the home LAN subnet

### OPT DHCP setup using examples
- Click the **Services** tab:
1. Click the **OPT interface** tab.
2. Click the **DHCP server** tab:
    - Set pool for ~50 devices (example: `10.1.2.50 – 10.1.2.100`).
    - Cannot overlap with other static IPs

### LAN DHCP setup using examples
- Click Services tab:
1. Click LAN interface tab.
2. Click DHCP server tab:
    - Set pool for ~50 devices (example: `10.1.1.50 – 10.1.1.100`).
    - Cannot overlap with other static IPs

### Enable NAT for OPT (Lab) Interface
- Locate Services:
1. Click NAT.
2. Click OPT interface tab.
3. Enable Outbound NAT for the new OPT interface (this is applied to the LAN interface by default, so we don’t need to activate NAT for the LAN interface).

## Firewall Rules
Finally, we must configure manual firewall rules for the OPT interface we have enabled, or communication will not take place. Firewall rules are automatically applied to the LAN interface by default, so we only need to do this for the OPT interface.

### Generic Firewall Rules for Interfaces:

- WAN Interface: "Configured to allow established connections while blocking unsolicited inbound traffic, ensuring a secure boundary for external interactions."

- LAN Interface: "Configured to allow all traffic within the local network to facilitate communication among devices, with optional blocks for specific services as needed."

- OPT Interface: "Setup allows necessary services for the lab environment, including DNS, HTTP, and DHCP traffic, while blocking any unsolicited inbound requests."

- Example Rule for OPT Interface: "For DNS resolution within the lab network, an allow rule is configured as follows:
Action: Allow
Protocol: TCP/UDP
Source: 10.1.2.0/24 (lab subnet)
Destination: Any
Port: 53 (DNS)
Description: Allow DNS traffic from the lab subnet to ensure devices can resolve domain names effectively.

## Install and Configure Managed Switch:

### Access GUI and change password
1. Locate web GUI for managed switch at default address
2. Enter the default login credentials that are printed on the label found on the bottom of the switch.
   - Locate **‘Admin Settings’** or similar  
3. When prompted:
    - Change the admin name and password.
    - The switch’s GUI page will refresh.
    - Enter the new login credentials we just created.

### Disable DHCP on managed switch
4. Disable the DHCP service (this allows us to set a static IP for the switch).
5. Set a static IP address on the `10.1.2.0 /24` subnet:
     - Avoid using an IP address that is in the DHCP range we set earlier on the firewall (we used the example of `10.1.2.50 – 10.1.2.100`).
     - Also do not use the IP of the firewall’s example OPT port we assigned earlier which was `10.1.2.1`
   - Set the gateway to the firewall’s OPT port IP address we assigned earlier `10.1.2.1`.

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
6. We should see OS messages regarding installation and updates.
7. Follow the simple steps of OS installation.
8. Now we see the OS home screen and our process is complete.

## Diagram of example subnets, domains, devices, IPs, gateways, subnet masks and DHCP pools. 
<p align="center">
  <br/>
  <img src="https://imgur.com/ZRBmXI0.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
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
