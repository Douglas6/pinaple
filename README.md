pinaple
=======

Raspberry Pi scripts for Bluetooth NAP service

Installation
------------

Install the needed dependencies:

    sudo apt-get install bluez bluez-utils bridge-utils

Verify with hciconfig:

    hciconfig
    hci0:   Type: BR/EDR  Bus: USB
            BD Address: 00:15:83:0C:BF:EB  ACL MTU: 339:8  SCO MTU: 128:2
            UP RUNNING PSCAN
            RX bytes:1346 acl:0 sco:0 events:44 errors:0
            TX bytes:422 acl:0 sco:0 commands:37 errors:0

Install the Pinaple scripts and make executable:

    sudo mv pinaple-agent /usr/local/bin
    sudo chmod 755 /usr/local/bin/pinaple-agent

    sudo mv pinapled /usr/local/bin
    sudo chmod 755 /usr/local/bin/pinapled

    sudo mv pinaple /etc/init.d/
    sudo chmod 755 /etc/init.d/pinaple
    sudo update-rc.d pinaple defaults

Setup the Network
-----------------

Set up network bridging; edit /etc/network/interfaces:

    sudo nano /etc/network/interfaces

Modify to suit your network. A minimal configuration is given here. Do NOT assign
an IP address to your eth0 device; only the bridge (br0) shaould have an address.

    auto lo
    iface lo inet loopback

    auto br0
    iface br0 inet dhcp
    bridge-ports eth0
    bridge_fd 5

Allow IP packet forwarding:

    sudo nano /etc/sysctl.conf

And un-comment (remove the leading '#') the following line:

    net.ipv4.ip_forward = 1

Connect with your Phone
-----------------------

AFter rebooting to make sure all changes have taken effect, you should be able to 
connect your phone to the Raspberry Pi via bluetooth.
Note, the following instructions are based on a Nexus 4 running Android 4.4.2.

1. Turn off the phone's Wifi. The phone will not connect to a bluetooth NAP
network if wireless is available.

2. Start the pinaple agent on the Raspberry Pi. You may specify a pairing PIN
code, or use the default of "0000".

    sudo pinaple-agent --pin 1234

This will make the Raspberry Pi discoverable by your phone, and will accept
the PIN code and trust the device. After pairing, press ctrl-C to end the
agent and make the Raspberry Pi undiscoverable again. You will only need
to perform this step once for each device, or until you un-pair the phone.

3. Turn Bluetooth on on the phone, and search for devices. Select the 
raspberrypi-0 and pair to it, with the pin code you specified when starting
pinaple-agent.

4. Click the 'settings' icon next to the raspberrypi-0, then click to check
the 'Internet access' check box. If all goes well, after several seconds you 
should see a small '3G' appear above the 'bars' icon of the phone's status bar.

You are now connected to the Raspberry Pi's network via Bluetooth.
