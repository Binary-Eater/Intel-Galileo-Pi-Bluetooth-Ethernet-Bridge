# BlueZ Ethernet Bridging With DHCP Server Binded To Ethernet Device

Keep in mind to use this you will have to compromise sing the ethernet port for network access while running this if you choose to run the DHCP server on Pi instead of a laptop.

  - Prerequisites: An ethernet controller and bluetooth module (The Pi 3 has both built-in)

## Needed Items To Download Before Proceeding

  - On Any Debian Based System To Run The Bluetooth-Etheret Bridging (aka a Pi)
 - ```sudo apt-get install bluetooth dbus bluez bluez-utils bridge-utils python-bluez python-dbus
    ```
  - For whatver device you want running the DHCP server (could be the Pi or a computer)
  - ```sudo apt-get install isc-dhcp-server bind9
    ```
  - Now, you will need Webmin on whatever device is running the DHCP server, so that it may take care of ip-table rules for you.
    - Be sure to download the Debian package 
    - http://www.webmin.com/
    - Once downloaded run:
    - ```
sudo dpkg -i <whatever the name of the deb file downloaded was + its extension>

        #in the directory that the deb file was downloaded in

        #for instance, cd ~/Downloads if your Downloads folder is Downloads before running this command
      ```
    
## Installation
#### DHCP Server Device

Run this command
```sh
ifconfig
```
If you see something other than eth0 in the list like enp something, do the steps in the box below and afterwards the steps below the box.
Otherwise, just do the steps below the box.

>Note down the Hardware address for the enp something device. It should be the string with colons to the right of Hwaddr

>Now run
>```sh
>sudo nano /etc/udev/rules.d/70-persistent-net.rules
>```
>```sh
>SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="<the hardware address you noted down from ifconfig>", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
>```
>Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.
>```sh
>sudo reboot
>```

Type this command in Terminal
```sh
sudo nano /etc/network/interfaces
```
Copy this into the file and delete everything else.
```sh
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 10.10.10.1
        netmask 255.255.255.0
```
Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.

Type this command in Terminal
```sh
sudo service networking restart
```

Type in the terminal
```sh
sudo nano /etc/dhcp/dhcpd.conf
```
Now delete everything in the file and insert this
```sh
ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;
authoritative;
log-facility local7;
option subnet-mask 255.255.255.0;
option broadcast-address 10.10.10.255;
option routers 10.10.10.1;
option domain-name-servers 10.10.10.1;
option domain-name "ubuntu.firewall";
subnet 10.10.10.0 netmask 255.255.255.0 {
range 10.10.10.2 10.10.10.200;
}
```
Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.

Now run in the Terminal
```sh
sudo service isc-dhcp-server restart 
```
```sh
sudo nano /etc/sysctl.conf
```
Uncomment the line (delete the #) for net.ipv4.ip_forward=1

Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.

Now login to webmin. Either use your web browser to go to localhost:10000 if you are on a laptop and go to "whatever hostname".local:10000 if you are accessing a device remotely on a local network.

Use your computer username and password to login

Now follow these steps to enable NAT as the picture illustrates

![Enable NAT Image](https://rbgeek.files.wordpress.com/2012/05/129.jpg?w=819&h=403)

To save iptable settings, follow this image

![Save Firewall Settings Image](https://rbgeek.files.wordpress.com/2012/05/134.jpg?w=819&h=403)

Now run in Terminal
```sh
sudo service networking restart
```

#### Bluetooth Bridging Device

On the bluetooth bridging device (probably a Pi), run this command
```sh
ifconfig
```
If you see something other than eth0 in the list like enp something, do the steps in the box below and afterwards the steps below the box.
Otherwise, just do the steps below the box.

>Note down the Hardware address for the enp something device. It should be the string with colons to the right of Hwaddr

>Now run
>```sh
>sudo nano /etc/udev/rules.d/70-persistent-net.rules
>```
>```sh
>SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="<the hardware address you noted down from ifconfig>", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
>```
>Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.
>```sh
>sudo reboot
>```

Now run
```sh
sudo nano /etc/network/interfaces
```
Add the following to the end of the file
```sh
auto br0
iface br0 inet dhcp
        bridge_ports    eth0
        bridge_fd       5
        bridge_stp      yes
```
Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.

Type this command in Terminal
```sh
sudo service networking restart
```

Now run
```sh
ifconfig
```
And make sure you see a device named br0 among the things printed

Now run,
```sh
brctl show
```
And make sure br0 is listed there

Now type the following command and follow the instructions to enable ip_forwarding
```sh
sudo nano /etc/sysctl.conf
```
Uncomment the line (delete the #) for net.ipv4.ip_forward=1

Hit Control-X and then save the file by following nano's instructions. Probably just have to hit enter.

Now run the bluetooth client on the bridging device.
```sh
bluetoothctl
```
Now, run
```sh
scan on
```
For each device seen, copy the hardware address that pops up associated to the device you want to pair
```sh
pair <hardware address copied>
```
Now after it has said paired successful
```sh
trust <hardware address copied>
```

Once done pairing devices, type
```sh
exit
```

Now cd into the directory that has the test-nap script that came with this repo.

Now run if using the script for the first time
```sh
sudo chmod 755 test-nap
```
An now you can run
```sh
./test-nap br0
```

### Version
0.0.1
Copyright Babu

### Tech

Uses Bluez, DBus, Webmin, and DHCP Server
For extra help, use the link below or contact Babu at (408)833-5642 or rahul_ram@me.com

For Bluetooth Ethernet Bridge
http://www.hkepc.com/forum/viewthread.php?tid=1710030

For DHCP Server
https://rbgeek.wordpress.com/2012/05/14/ubuntu-as-a-firewallgateway-router/

