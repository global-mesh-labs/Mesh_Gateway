**DIY Instructions for SMS Gateway**
Instructions created by Paul at [http://www.tourmalinewireless.com/](Tourmaline Wireless)

**Getting Started**

Download Raspberry Pi OS:[ https://www.raspberrypi.org/downloads/raspberry-pi-os/](https://www.raspberrypi.org/downloads/raspberry-pi-os/)

Select: Raspberry Pi OS (32-bit) Lite, Download ZIP file

 _(Current version as of this writing is August 2020, Release date: 2020-08-20)_

Download Balena Etcher:[ https://www.balena.io/etcher/](https://www.balena.io/etcher/)

Install and open Balena Etcher program

Insert SanDisk Ultra 32 GB microSD card into card slot on your PC

Select image: _2020-08-20-raspios-buster-armhf-lite.img_

Select target: _microSD card location_

Click_ _Flash! Button

After flashing is complete, remove card (eject if needed) and re-insert card into card slot on your PC

Add a blank text file with .ssh file extension anywhere in the boot directory

Eject card and insert into the microSD card slot on Raspberry Pi

Connect power and Ethernet cords to your Raspberry Pi and plug Ethernet cable into an open port on your router (or modem)

Find the IP address assigned to your Raspberry Pi 

Login to the Raspberry Pi:  ssh pi@xxx.xxx.xxx.xxx

User password: raspberry

**Configure Raspberry Pi**

sudo raspi-config

Select Option 8, Update (Update this tool to the latest version)

Select Option1, Change User Password (Change password for the ‘pi’ user)

	Update with your own secure password

Select Option 2, Network Settings (Configure network settings)

	Select N1 Hostname (Set the visible name for this Pi on a network)

		-> Provide a meaningful name for your gateway

Select Option 2, Network Settings (Configure network settings) again

	Select N2 Wireless LAN (Enter SSID and passphrase)

		-> Specify WiFi credentials here if you would like to administer your gateway via WiFi

Select Option 4, Localization Options (Set up language and regional settings to match your location)

	Select I2 Change Time Zone (Set up time zone to match your location)

Select Option 5, Interfacing Options (Configure connections to peripherals)

	Select P4 SPI (Enable/Disable automatic loading of SPI kernel module)

		-> Enable SPI 

Arrow down to Finish to exit & reboot

If you will be using WiFi, remove the Ethernet connection, identify the new IP address assigned by your router 

ssh pi@xxx.xxx.xxx.xxx

Use your new password to login

**Adding the Sixfab 3G/4G Base Hat and Quectel Module**

Read the getting started guide:[ https://docs.sixfab.com/docs/raspberry-pi-3g-4g-lte-base-hat-getting-started](https://docs.sixfab.com/docs/raspberry-pi-3g-4g-lte-base-hat-getting-started)

Use the QMI interface with Base Hat guide:[ https://docs.sixfab.com/docs/qmi-interface-with-base-hat](https://docs.sixfab.com/docs/qmi-interface-with-base-hat)

Below are the commands you will need to run, which are also detailed in the guide above. Only run these set of commands once:

sudo apt update && sudo apt upgrade -y

sudo apt-get install raspberrypi-kernel-headers

wget https://raw.githubusercontent.com/sixfab/Sixfab_RPi_3G-4G-LTE_Base_Shield/master/tutorials/QMI_tutorial/qmi_install.sh

sudo chmod +x qmi_install.sh

**STOP!!**

Make sure the short USB plug on the Sixfab Base Hat is left unplugged before running the installer

sudo ./qmi_install.sh

Press Enter key to reboot

**Before proceeding to the next step, make sure you have a Twilio SIM card that has been preconfigured on[ www.twilio.com](http://www.twilio.com) See steps below for this portion

—>Now insert configured SIM card into the Sixfab Hat and plug in the USB cable into the Raspberry PI on the following slot: (photo)

cd files/quectel-CM

sudo ./quectel-CM -s wireless.twilio.com

If you see the following line:  udhcpc: no lease, failing

Type Ctrl C

Then you need to add the following line:  denyinterfaces wwan0 to /etc/dhcpcd.conf

The easiest way to do this is use the nano file editor:  

nano /etc/dhcpcd.conf 

Add this line at the bottom of the file: denyinterfaces wwan0

then hit Ctrl X, Y (to save), Enter

Now reboot the Raspberry Pi using: sudo reboot 

Re-run the two lines above after logging in

Once the cellular modem is working properly, you should see the following line:

udhcpc: lease of 30.60.240.152 obtained, lease time 7200

You can open a new terminal tab, login, and try to run this command:

ping -I wwan0 -c 5 8.8.8.8

If successful, you will see a similar output:

**$** ping -I wwan0 -c 5 8.8.8.8

PING 8.8.8.8 (8.8.8.8) from 30.58.10.0 wwan0: 56(84) bytes of data.

64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=71.4 ms

64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=39.6 ms

64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=40.8 ms

64 bytes from 8.8.8.8: icmp_seq=4 ttl=117 time=47.3 ms

64 bytes from 8.8.8.8: icmp_seq=5 ttl=117 time=45.3 ms

--- 8.8.8.8 ping statistics ---

5 packets transmitted, 5 received, 0% packet loss, time 11ms

rtt min/avg/max/mdev = 39.638/48.902/71.382/11.589 ms

**Adding Python libraries and goTenna SDK**

_Recommend shutting down the Raspberry Pi, remove the USB-A plug and download the libraries over WiFi or Ethernet, not cellular modem.  Once complete you can return to plugging in the USB again._

sudo apt install python3-pip

sudo apt-get install git

sudo pip3 install cbor

sudo apt-get install minicom lsof

sudo apt-get install -y python3-spidev

mkdir goTenna

cd goTenna

git clone https://github.com/gotenna/PublicSDK.git

cd PublicSDK/python-public-sdk

pip3 install goTenna-0.12.5-py3-none-any.whl

Need to figure out if we clone using git of just pull these down with wget:

wget https://raw.githubusercontent.com/remyers/Signal-Android/v4.58.5e-mesh/gotenna-public-sdk/mesh_gateway.ini

wget https://raw.githubusercontent.com/remyers/Signal-Android/v4.58.5e-mesh/gotenna-public-sdk/mesh_gateway.py

Now we need to modify the mesh_gateway.ini file: nano mesh_gateway.ini

In particular, we need to edit the serial port portion, change from:

serial_port = /dev/cu.USB Application Port1433423

To:

serial_port = /dev/ttyUSB2

Also, we need to edit the imeshyou section credentials for our node, with both login username, password, and GPS coordinates

python3 mesh_gateway.py

**Finishing touches**

Add Sixfab auto-connect on reboot service to automatically start the cellular modem connection:

cd ~

wget https://raw.githubusercontent.com/sixfab/Sixfab_RPi_3G-4G-LTE_Base_Shield/master/tutorials/QMI_tutorial/install_auto_connect.sh

sudo chmod +x install_auto_connect.sh

sudo ./install_auto_connect.sh

It will then ask for APN. Type in your APN and then press ENTER

(use wireless.twilio.com if you using Twilio SIM)

To check if the service is active you can type:

sudo systemctl status qmi_reconnect.service

Add service to automatically start the sms gateway program:

Need to add command lines

**GPS preferred settings**: (Note: Sixfab Hat with Quectel module must be seated and plugged in to run these commands)

sudo minicom -b 115200 -D /dev/ttyUSB2

_Please note you will need to manually type these lines as a copy and paste will modify the characters and result in ERROR_

AT

AT+QGPSCFG=“autogps",1

AT+QGPSCFG="gnssconfig",3

AT+QGPSXTRA=0

To exit from minimum app: Ctrl A, z, x, Yes (Enter)

**Twilio SIM card setup:**

* Navigate to All Products & Services/ Super Network/ Internet of Things/ Programmable Wireless/ Manage SIMs/ SIMs

* Click the blue plus symbol (Add SIMs) / Register a SIM

* Enter registration code (found on the back of SIM card; no spaces)

* After registration is complete, choose a unique name for your device

* Create or choose a Rate Plan (we use the Enabled services: Data, Messaging; 1000 MBs/Month; $1 Quota; Roaming Disabled)

* Activate your SIM (this can take from 15 seconds to a few minutes once you click activate)

* Copy & Paste the SIM SID to a NotePad or TextEditor for later use

* Navigate to All Products & Services/ Super Network/ Phone Numbers/ Buy a Number

* Select your phone number and click purchase

* Navigate to All Products & Services/ Super Network/ Internet of Things/ Programmable Wireless/ Manage SIMs

* Select SIM for your new device

* Select Programmable Voice & SMS

* Navigate to Programmable SMS _Preview_ section at bottom of screen

* At SMS URL, select TwiML from the dropdown

* Click on the blue plus symbol to add TwiML Bin

* Replace the "YOUR_NUMBER_HERE" with your phone number in E164 format +1760xxxxxxx

* Replace the Friendly Name with something like "Delivery of outgoing messages: 760-xxx-xxxx”

* Click on Add TwilML Bin

* Click Save

* Navigate to All Products & Services/ Runtime/ TwiML Bins

* Click the red plus symbol 

* Add a friendly name like "Receive incoming messages to 760-xxx-xxxx”

*  Add the following code and insert your SIM underneath the following code line:

> &lt;?xml version="1.0" encoding="UTF-8"?>

> &lt;Response> &lt;Message to="sim:YOUR_SIM_SID_HERE" from="{{From}}"> {{Body}} &lt;/Message> &lt;/Response>

* Insert your SIM SID number that you saved earlier on a Text Editor, paste after “sim:

* Click Create

* Click Save

* Navigate to All Products & Services/ Super Network /Phone Numbers

* Find your phone number in the list and click

* Scroll down to Messaging/ A Message Comes In, change to TwiML Bin

* Change drop down selection to Receive incoming messages to 760-xxx-xxxx

* Click Save

* Done with adding SIM card and phone number in Twilio, now proceed to above step where you insert the SIM card into the Sixfab Hat
