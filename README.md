# 5G Linksys EA8300 PoE Modem Build
Repository of all information related to my Linksys EA8300 5G PoE modem build.

If this project benefitted you in some way please consider supporting my efforts with a small donation below:

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/Donate_QR_Code.png" />

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=AB6H6ER4RWT74)

- [Table of Contents](#5g-linksys-ea8300-poe-modem-build)
  * [Design Philosophy & Guiding Principles](#design-philosophy--guiding-principles)
  * [Parts List](#parts-list)
  * [Important Component Selection Information](#important-component-selection-information)
    + [Quectel RM502Q-AE](#quectel-rm502q-ae)
    + [NGFF to USB 3.0 Adapter](#ngff-to-usb-30-adapter)
    + [Linksys EA8300](#linksys-ea8300)
    + [Amazon Basics HU3641V1](#amazon-basics-hu3641v1)
    + [80mm USB PC Fans (2x)](#80mm-usb-pc-fans-2x)
    + [Gigabit PoE Splitter](#gigabit-poe-splitter)
    + [Buck Converter (DC voltage step-down regulator)](#buck-converter-dc-voltage-step-down-regulator)
    + [FPC Antennas](#fpc-antennas)
  * [Hardware Build](#hardware-build)
    + [Vent, Fan, and Wire Gland Install](#vent-fan-and-wire-gland-install)
    + [Mounting, Connections, and Cabling](#mounting-connections-and-cabling)
      - [Buck Converter and PoE Splitter](#buck-converter-and-poe-splitter)
      - [Layout and Cable Routing](#layout-and-cable-routing)
      - [EA8300 PCB](#EA8300-pcb)
      - [NGFF USB Adapter](#ngff-usb-adapter)
      - [Power Cables](#power-cables)
      - [Voltage Validation](#voltage-validation)
      - [USB and Ethernet Connections](#usb-and-ethernet-connections)
      - [FPC Antennas](#fpc-antennas)
  * [Software Build](#software-build)
    + [Operating System Selection](#operating-system-selection)
    + [OpenWRT Install and Initial Configuration](#openwrt-install-and-initial-configuration)
    + [Install All Required Packages](#install-all-required-packages)
    + [Modem Firmware Update and Data Interface Configuration](#modem-firmware-update-and-data-interface-configuration)
    + [Configure Modem Interface and Remove Ethernet WAN](#configure-modem-interface-and-remove-ethernet-wan)
    + [Add Custom Firewall Rules](#add-custom-firewall-rules)
    + [Configure DMZ to Main Router](#configure-dmz-to-main-router)
    + [Helper Scripts](#helper-scripts)
      - [fancontrol.sh](#fancontrolsh)
      - [modemwatcher.sh](#modemwatchersh)
      - [quickycom.sh](#quickycomsh)
      - [nsacheck.sh](#nsachecksh)
      - [failsafe.sh](#failsafesh)
    + [Switch Modem to Generic Image](#switch-modem-to-generic-image)
    + [Disable Modem NR SA](#disable-modem-nr-sa)
    + [Build and Configure SMS Tool](#build-and-configure-sms-tool)
  * [Results](#results)  
  * [ToDo List](#todo-list)
  * [Historical Background](#historical-background)

## Design Philosophy & Guiding Principles
This project strives to align with the same overall goals and principles originally laid out for my [5G RPi PoE Modem Build](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#design-philosophy--guiding-principles). Due to supply chain issues the RPi 4 have become prohibitively expensive for such a simple thing as a modem host and, in general, simply unobtainable for nearly any hobbyist project as of late (blame the early Helium crypto miners and COVID, I guess). This project was birthed as I began the search for an SBC or equivalent hardware platform which folks could actually get their hands on without paying a king's ransom or waiting months to receive. The basic qualifiers for the new were host board were:

* Cost under $50
* Immediate domestic availability
* At least one USB 3.x port
* Gigabit Ethernet
* Mainline OpenWRT support on the latest stable releases (so we can use ModemManager etc. easily)
* Healthy amount of NAND flash (128MB+) for initial OpenWRT image flashing, package installations, future upgrades, etc.
* Still compact in size (when compared to RPi 4 or similar SBC as a reference)

Fortunately the [OpenWRT "Table of Hardware" (Full Details view)](https://openwrt.org/toh/views/toh_extended_all) made this search relatively easy. Filtering the table down by my basic hardware requirements and then searching for device supply pricing/availability from there quickly landed me on the Linksys EA8300/MR8300 units which have been out since ~2017 and are available in a healthy number of places (Amazon, eBay, Facebook Marketplace, Mercari, OfferUp, etc.). The EA8300 is the more ideal candidate with 256MB NAND and nearly 200 unique item listings on eBay as of this writing. The MR8300 is sufficient at 128MB NAND with listings less than $50 on Amazon as well. Yes, most of the units being sold at the > $50 price point are used or refurbished condition but, with only one or two small capacitors used in their design, most of these units will still have plenty of life left in them beyond their current ~5 year existing age. Besides all this, being able to leverage "up-cylcing" as the base of our project earns us a few environmentally conscious points which is always a nice bonus.

## Parts List
I've included affiliate links to items which are sold by The Wireless Haven as I prefer to support local merchants were possible. The rest of the items can be sourced from the usual suspects via the non-affiliate links provided.

* **Quectel RM502Q-AE**
  * Supports all current LTE and NR low and mid-spectrum bands by US carriers (no mmWave)
  * SDX55 chipset supporting NR SA, NR CA (FDD+FDD, TDD+TDD), NSA (LTE+NR), and VoLTE
  * M.2 Key B connection supporting USB 3.x and PCIe w/ IPEX4/MHF4 antenna connectors
  * https://thewirelesshaven.com/shop/modems-hotspots/5g-modems/quectel-rm502q-ae-5g-cellular-modem?aflt=3345
* **Copper Modem Heatsink**
  * Phyiscal dimensions: 40x26x4mm
  * https://thewirelesshaven.com/shop/accessories/cooling/copper-heatsink-for-5g-modems-4mm-low-clearance?aflt=3345
* **NGFF(M.2) to USB 3.0 Adapter**
  * Compact size
  * Provides supplemental power to modem when used with USB Y-Cable
  * M.2 Key B connector for modem
  * https://a.co/d/bR93nxd
  * Alternative single SIM slot version with supplemental DC input: https://thewirelesshaven.com/shop/mini-pcie-m2-adapters/modem-enclosure/usb3-0-to-ngff-m-2-key-b-4g-5g-modem-adapter-enclosure-with-sim-card-slot-new-style?aflt=3345
* **Quectel 5G FPC Omnidirectional Antennas (x4)**
  * 20cm cable
  * IPEX4/MHF4 connectors
  * Frequency range: 600-6000Mhz
  * https://www.mouser.com/ProductDetail/Quectel/YF0020AA?qs=QNEnbhJQKva2Xjy9%252B1VCOw%3D%3D&countrycode=US&currencycode=USD
* **Linksys EA8300**
  * 256MB NAND Flash Storage
  * https://www.ebay.com/sch/i.html?_from=R40&_trksid=p2334524.m570.l1313&_nkw=ea8300&_sacat=0&LH_TitleDesc=0&LH_BIN=1&_sop=15&_oac=1
* **IP67 Outdoor Project Box**
  * Plastic, pre-drilled mounting plate and included hardware
  * Stainless steel latches
  * Physical external dimensions 350x250x150mm
  * https://a.co/d/1PUAqCX
* **Large Air & Moisture Vent (x2)**
  * Bud Industries IPV-1116
  * Physical dimensions 100x96x65.5mm
  * Hole dimension diameter: 88mm
  * https://www.solutionsdirectonline.com/bud-industries-large-vent-kit-for-enclosures-ipv-1116
* **88mm Hole Saw**
  * Steel alloy
  * https://www.aliexpress.com/item/2251832539521487.html
  * Domestically sourced version with diamond blade instead of sawtooth: https://a.co/d/jfRxoul
* **80mm PC Fan (x2)**
  * USB powered (5v)
  * https://a.co/d/2aagEVy
* **80mm PC Fan Filter Grills (x2)**
  * Aluminum frame
  * Fine Stainless Steel mesh
  * https://a.co/d/8LybAtF
* **Apricorn USB 3.0 USB Y-cable**
  * 1x female USB-A Data+Power
  * 2x Male USB-A (1x Power only, 1x Data+Power)
  * Better quality than other competing cables I tested previously
  * https://a.co/d/c9gIp8W
* **4-Port USB 3.0 Hub**
  * Amazon Basics HU3641V1 (aka GWC HU3640)
  * Discontinued but still available used
  * Provides PPPS (Per Port Power Switching)
  * Discontinued but still available under $25 via used channels: https://www.ebay.com/sch/i.html?_from=R40&_trksid=p2553889.m570.l1313&_nkw=HU3641V1&_sacat=0
  * Alternative current model which has one switched port is the Amazon Basics U3-4HUB which can be used with the linked fans' included USB splitter to switch power on both fans leaving the remaining 3 ports unswitched for modem connection etc.: https://a.co/d/0gvHBJy
* **1-1/8" Corner Guard**
  * Comes in 4' but can be cut down to use as 90 degree mount bracket for the USB hub
  * Available at most hardware stores. Because of the odd shape the price on Amazon etc. is ridiculous due to shipping considerations.
  * https://www.menards.com/main/mouldings/panel-mouldings/trimaco-1-1-8-x-4-corner-guard-with-nails/01184g/p-1549351771019-c-8164.htm?tid=-8168026590023558773&ipos=3
* **18AWG DC Power Cables**
  * 5.5x2.1mm 
  * Male+Female pairs (6x)
  * https://a.co/d/5KNvNhA
* **DC Step down voltage regulator (i.e. buck converter)**
  * 9v-36v input / 5v 6a output
  * DC barrel connector input/output
  * Additional USB-A connector output
  * https://a.co/d/bhq6Paz
* **Double Sided Mounting Tape**
  * 1" wide
  * https://www.amazon.com/dp/B01AOFK85C
* **802.3AT PoE Gigabit Splitter**
  * DC 48v input
  * DC 12v output (switchable 5-18v output but 12v selected for this project)
  * https://a.co/d/iYKScf1
  * A better version that takes up less space and does not have a voltage switch: https://thewirelesshaven.com/shop/poe/poe-splitters/48v-to-12v-3a-poe-gigabit-splitter-5-5mm-x-2-1mm-power-plug-tip-2?aflt=3345
* **802.3AT PoE Gigabit Injector**
  * AC 120v input
  * DC 48v output
  * https://thewirelesshaven.com/shop/power-supplies-adapters/poe-power-supplies/poe-desktop-adapter-48v-1-5a-72w-gigabit-2?aflt=3345
* **320pc Nylon Standoff/Screw/Washer Kit**
  * Assorted lengths
  * M3 size
  * https://a.co/d/c1gfxq5
* **Outdoor-grade Zip Cable Ties (2 sizes)**
  * 10cm length, 18LB loop tensile strength (1000ct): https://a.co/d/1FN3j6I
  * 20cm length, 50LB loop tensile strength (100ct): https://a.co/d/1di86hL
* **Wago LEVER-NUTS**
  * 3-Conductor
  * 24-12 AWG
  * https://a.co/d/6RDEEAa
* **USB-A to 5.5x2.1mm DC Power**
  * Connects modem USB Y-cable to supplemental power
  * https://a.co/d/70YoDLQ
* **1.64' DC Male to DC Male**
  * Connects buck converter to Amazon USB Hub supplemental power
  * https://a.co/d/1CFrU4q
* **1' Cat6 Ethernet Cable**
  * Connects PoE Splitter to EA8300 LAN
  * https://a.co/d/4bQSsnU
* **Terminated Ethernet Wire Gland - Gray Nylon**
  * Accepts pre-terminated Ethernet (RJ45)
  * https://thewirelesshaven.com/shop/enclosures/wifix-outdoor-enclosure-cable-gland-heavy-duty?aflt=3345

## Important Component Selection Information
### Quectel RM502Q-AE
Reasons for selection covered [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#quectel-rm502q-ae). 

### NGFF to USB 3.0 Adapter
Instead of a 5G modem evaluation board (EVB) this build will use a simpler adapter board. The disadvantage here is no direct DC power supplementation unless you go with the Wireless Haven single SIM version linked in the parts list. On the flipside, this item can be sourced domestically for less cost than a specialized EVB and we can supplement power input with a USB Y-cable to cover the power demands of the modem. NOTE: In later testing I found that neither this adapter nor the EVB used in the RPi 4 build had the proper SIM-presence pin wired for the RM502Q-AE to recognize the second SIM slot so unfortunately only the primary SIM slot is useable.

### Linksys EA8300
Already covered selection [reasons for this component](https://github.com/hazarjast/5g_linksys_build/blob/main/README.md#design-philosophy--guiding-principles).

### Amazon Basics HU3641V1
In the RPi build I found the D-Link hub had a nasty habit of sometimes disconnecting completely from the system when power cycled. Since that hub was only used for controlling the fans and not connecting the modem I was able script around the issue by adding a HotPlug rule which reset the fan status when the hub disconnected/reconnected. However, this behavior isn't really acceptable when we only have one USB port to work with on the Linksys and rely on the hub to always keep the modem connected while *only* cycling power to the fans. So, I tested the Amazon Basics HU3641V1 this time which is equally supported by usbhubctl but without the disconnect behavior of the D-Link.

### 80mm USB PC Fans (2x)
Already covered [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#80mm-usb-pc-fans-2x).

### Gigabit PoE Splitter
Already covered [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#gigabit-poe-splitter).

### Buck Converter (DC voltage step-down regulator)
Already covered [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#buck-converter-dc-voltage-step-down-regulator).

### FPC Antennas
Switched to FPC antennas this time as they are first-party manufacturered by Quectel for a perfect pairing with the modem and are a little smaller and easier to use since they are flexible vs the rigid PCB antennas.

## Hardware Build
### Vent, Fan, and Wire Gland Install
Description of vent and fan install already covered in detail [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#vent-fan-and-wire-gland-install). This time around I have added pictures of the Ethernet cable gland installed as well (drilled 1/2" hole and widened further with side of drill bit to taper the opening for a nice, tight install).

<table >
	<tbody>
		<tr>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/IMG_5377.jpg" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h45_46.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h46_32.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h44_33.png" width="200" height="200" /></td>
		</tr>
		<tr>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h47_55.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h50_13.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h48_29.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_08h48_59.png" width="200" height="200" /></td></td>
		</tr>
	</tbody>
</table>

### Mounting, Connections, and Cabling
#### Buck Converter and PoE Splitter
The buck converter was secured to the top of the PoE splitter with layers of mounting tape to bridge the gap created by the legs and solder beads on the underside of its PCB. As insurance against any future adhesive failure, a zip-tie was used to secure it in the middle as well. This will not create a heat issue since it is right in front of the cold air intake fan and the PoE splitter heat dissipation vents are on its sides. Mounting it on top of the PoE splitter created more space to access the Ethernet ports on the router PCB and allowed me to rely less on zip-ties this time since, on the RPi build, I found it was easy to overtighten them and shear off capacitors from the buck converter which is a show-stopper. The PoE splitter already had nice mounting holes and does not generate much heat so I left its cover on and used the shortest height standoffs to mount it. This still left some nice space underneath it to route power cabling. 

#### Layout and Cable Routing
Comoponents were mounted to the plastic mounting plate of the enclosure using a combination of nylon standoffs/screws/nuts and smaller zip-ties. I made greater use of the nylon standoffs in this build to better secure each component leaving the zip-ties for things like the Lever-Nuts which have no mounting holes themselves. Components were oriented on the mounting plate from coldest to hottest running for most efficient heat dissipation: PoE splitter and buck converter on the bottom, router PCB in the middle, and modem at the top. One set of horizontally oriented Lever-Nuts was used for power distribution out from the PoE splitter to the DC barrell connector inputs on the router and buck converter. A second set of vertically oriented Lever-Nuts was used to distribute power out from the bare wire crimp connector on the buck converter to the USB hub and the power-only end of the USB Y-cable (via USB-A to DC adapter) to supplement the modem.

Since the package of DC Male and Female connectors I bought had excess wire, I was able to clip them shorter and use the additional lengths of wire to distribute the power from the buck converter to the vertically oriented Lever-Nuts. Wires and connectors were routed into natural channels between components and secured with zip-ties to reduce strain, eliminate pinch points, and leave sufficient access for fingers to access connectors, SIMs, etc. without snagging anything. After all bare wire ends were terminated in the Lever-Nuts, the Lever-Nuts themselves were secured to the mounting plate with some of the smaller zip-ties to ensure they were not snagged and unlocked inadvertently.

#### EA8300 PCB
This was the largest, widest component so it had to go as far up as it could in the enclosure while leaving space for the modem to mount just above it as well. The Ethernet ports were oriented downward for easier routing of power to the DC connector and access to plug in Ethernet cables. Two out of the four factory screw holes in the PCB were inside the edges so nylon standoffs were used to secure it easily. The top two holes were not full holes since they were on the edge of the PCB and needed washers added to fully secure them.

#### NGFF USB Adapter
The USB adapter used for the modem in this build already had mounting holes so using the tall nylon standoffs to mount it was easy and raised it above the router PCB far enough to allow for easy SIM insertion/ejection. The adapter and modem were centered over the router PCB to allow room for fingers in the channel between the two router heatsinks.

#### Voltage Validation
Because we are working with different voltages and have invested a good chunk of change into the components we are connecting, it is always wise to check all voltage outputs before making the individual component power connections and plugging the PoE injector into mains. First up, I plugged the PoE injector into the wall outlet and connected it via Cat6 cable to the PoE splitter, ensuring the PoE splitter was set to 12v. Nothing was connected to the buck converter at this point. Once I verified the splitter was ouputting 12v for the router and buck converter I connected it to the buck converter and verified the buck converter was providing 5v output. The multimeter reading of a steady 5.20v confirmed we were well within USB spec (official range is 5.25 down to 4.45 for most USB devices powered from a 5v power supply rail).

I then tested the male DC connectors attached to the vertically oriented Lever-Nuts (connected to buck converter output) with the multimeter and verified they both read 5.20v as well. From there I connected these same male DC connectors to the USB hub and USB Y-cable to test the voltage of each USB connection with the USB power meter. Both tested in range at 5.15v which is within margin of error compared to the multimeter readings. In the photo matrix under this section I have included a rough electrical wiring diagram showing the voltages and how each component is connected to its power source. 

#### USB and Ethernet Connections
Once power connections were in place I removed power from the injector to run data cabling. First I connected the NGFF to USB modem adapter to the female end of the USB Y-cable and the data-only end of the same to the USB hub. The fans were then connected to other ports on the USB hub. The hub was then connected to the USB 3.0 port of the router PCB. The USB hub was positioned to the left of the router PCB. The hub was attached to a 4" cut section of the clear, 90 degree, plastic "corner protector" with mounting tape and mounted to the mounting plate of the enclosure with medium height nylon standoffs and nuts (the medium height was used to create a channel for the USB Y-cable power cable connection). A small foam-tape adhered zip-tie tie-down was used to secure the fan cables to the inside of the lid to eliminate pinching of their USB power cables. Finally a 1' Cat6 cable was connected from the RJ45 Data port of the PoE splitter to a LAN port on the router. 

#### FPC Antennas
The 4x FPC antennas were mounted equal distances apart in the top of the enclosure body just above the modem. The 20cm cables length necessitated this positioning but you could mount them in the sides of the body as well if you purchased some MHF4 extension cables. The back of the antennas features a 3M brown label self-adhesive which should provide sufficient adhesion in both hot and cold environments. The MHF4 connectors of the antennas were connected to the modem *very* gingerly using the end of a plastic spudger. The trick is to line them up exactly over the modem connector and apply centered, even pressue until they snap on. Do *NOT* use a metal tool (such as a screwdriver) for this. I have seen way too many people slice through cables and/or shear connectors off the modem when the two metal surfaces invariabley slip away from each other under pressure.

<table >
	<tbody>
		<tr>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h50_41.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h26_25.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h42_16.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h34_42.png" width="200" height="200" /></td>
		</tr>
		<tr>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h33_36.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h31_07.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h31_51.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h34_09.png" width="200" height="200" /></td>
		</tr>
		<tr>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_11h12_59.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_11h12_36.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h37_09.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h37_36.png" width="200" height="200" /></td>
		</tr>
		<tr>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h36_29.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h36_02.png" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/5g_linksys_wiring.jpg" width="200" height="200" /></td>
			<td><img src="https://github.com/hazarjast/5g_linksys_build/blob/main/assets/2022-06-28_09h40_25.png" width="200" height="200" /></td>
		</tr>
	</tbody>
</table>

## Software Build
### Operating System Selection
I decided to go with the latest stable OpenWRT (21.02.3) and ModemManager (1.16.6-1) for this build. All other rationale for the decision to go with OpenWRT and ModemManager as the platform combo is covered in my earlier RPi build info found [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#operating-system-selection).

### OpenWRT Install and Initial Configuration
The starting point for this build was of course RTFM (reading the 'fine' manual). In this case OpenWRT already has a nice wiki page for the EA8300 router: https://openwrt.org/toh/linksys/ea8300 . I connected the Ethernet data connection from the injector to my workbench PC's NIC so that I could follow the flashing instructions provided. Once booted up, I logged in to the web interface and set the root password to proceed from there. The following screenshots are from my RPi build and thus show the 21.02.1 UI. If I get some time I'll update them showing the newer 21.02.3 UI but essentially they are exactly the same outside of some minor visual design differences.

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h39_15.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h39_30.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h39_50.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h40_00.png" />

Given the default IP will conflict with many existing routers running on 192.168.1.1 it will be best to change this as well:
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h40_58.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h41_12.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h41_32.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-14_08h25_31.png" />

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h45_07.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_10h45_24.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-08_11h21_12.png" />

### Install All Required Packages
Once the EA8300 LAN subnet has been changed, I then connected the WAN port to my existing home LAN to provide Internet connectivity. Once Internet connectivity was established, I used Putty to connect to the EA8300 over SSH and issued the following commands to update the software package lists and install the packatges required for this build (there are actually more packages which will be installed but they will be installed automatically as dependencies for the packages listed below):

```bash
opkg update
opkg install usbutils kmod-usb-net-qmi-wwan kmod-usb-net-cdc-mbim kmod-usb-serial-option luci-proto-modemmanager uhubctl socat coreutils-timeout iptables-mod-ipopt pservice
reboot
```

### Modem Firmware Update and Data Interface Configuration
Flashing of Quectel firmware updates is covered in the original RPi build [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#flash-modem-firmware-update). Additionally, in this EA8300 build the RM502Q-AE I sourced was pulled from another existing build and was configured with a USB composition type of QMI along with passing data over PCIe (aka MHI or 'Modem Host Interface'). Because the USB controller interface on the EA8300 is different/older than the one in our RPi build it did not handle the many simultaneous connection streams utilized by the QMI interface driver so it was necessary to switch the modem to use MBIM for this build. Additionally, since we will be passing data over the USB interface we will need to set the modem to do this since it was using the PCIe interface to do this in the router it was removed from. This was accomplished by issuing the following AT commands:

NOTE: This is not necessary if your modem is already in MBIM composition and passing data over USB.
```bash
echo -e AT+QCFG=\"data_interface\",0,0 | socat - /dev/ttyUSB2,crnl
echo -e AT+QCFG=\"usbnet\",2 | socat - /dev/ttyUSB2,crnl
echo -e AT+CFUN=1,1 | socat - /dev/ttyUSB2,crnl
```

### Configure Modem Interface & Remove Ethernet WAN
Once packages are installed and OpenWRT has been rebooted, log back into the web interface to configure the modem interface (I have called mine 'WWAN'):
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-10_17h39_30.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-10_17h40_28.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-10_17h41_10.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-10_17h41_47.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-10_17h42_12.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-11_16h37_11.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-11_16h42_03.png" />

### Add Custom Firewall Rules
It will be necessary to add custom firewall rules ('Network > Firewall > Custom Rules') if you are using a SIM provisioned to a plan that differntiates on-device data from hotspot data, else you will exhaust the hotspot bucket and be left with greatly throttled speeds in some cases (Caveat Emptor: this is a ToS violation on some cellular plans so if you get your line/plan canceled by your carrier it's not my fault; you've been warned):

```bash
#IPv4 TTL mod
iptables -w -t mangle -C POSTROUTING -o wwan0 -j TTL --ttl-set 65 > /dev/null 2>&1 || \
iptables -w -t mangle -I POSTROUTING 1 -o wwan0 -j TTL --ttl-set 65

#IPv6 TTL mod (prevents leaks not covered by IPv4 rules)
ip6tables -w -t mangle -C POSTROUTING -o wwan0 -j HL --hl-set 65 > /dev/null 2>&1 || \
ip6tables -w -t mangle -I POSTROUTING 1 -o wwan0 -j HL --hl-set 65`
```

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-01-15_12h16_28.png" />

If you modem device is not 'wwan0' updated it accordingly in the rules above. The TTL value of 65 is used here because TTL decrements and you need it to hit the cellular network with a TTL of 64 (65-1=64). This value can vary with some LAN configurations and cellular carriers so you may have to use something like 64 or 66 (some claim simply setting it to 64 is carrier agnostic but this has not been my experience, YMMV).

### Configure DMZ to Main Router
Since I am using OPNSense as my main home router and will be using the RPi OpenWRT install as a modem host only, I will be statically assigning the IP address of my OPNSense WAN interface in the same IPv4 subnet as the EA8300 LAN and setting the EA8300 as the gateway for Internet traffic. My full rationale for this DMZ configuration is explained in the original RPi build [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#configure-dmz-to-main-router). OpenWRT doesn't have a specific "DMZ" feature but under "Port Forwards" we have the ability to accomplish the same thing:
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-02-10_12h09_52.png" />

In creating our rule, we will allow any protocol with a source zone of 'WWAN' (or whatever you named your modem interface), destination zone of 'lan', and an internal ip address of '192.168.21.2' (the IP address we will have statically assigned to the WAN of our main router). Be sure to 'Save & Apply' the changes:
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-02-10_12h11_37.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-02-10_12h11_58.png" />

Once that is done we will disable all DHCP server features on the LAN interface (beware that once you do this, if you connect the RPi to a PC to debug it later on, you will need to statically set the IP address for the NIC at OS level to something in the same subnet ex. 192.168.21.5 in order to access the RPi since it won't be automatically assigned an address anymore on connection):
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-02-10_12h06_54.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-02-10_12h07_19.png" />

At this point OpenWRT can be restarted and connect to the WAN of your main router which has been configured with the static IP of 192.168.21.2.

### Helper Scripts
For this project we will need some scripts to achieve the goals we made at the outset I cover each in detail below. Such scripts may be added/removed/consolidated at a later date depending on subsequent testing of the overall build and its performance. All such scripts are available under 'scripts' folder of this repository and live under '/scripts' on the OpenWRT installation. Some scripts are written with the intenion they should be running at all times in 'daemon' mode (aka 'service' if you come from Windows background), others are created to be run only interactively at the command line.

For daemonized scripts, we control them using the 'pservice' package. This is a very simple OpenWRT package which is a wrapper for shell scripts which should run as daemons. The reason to use this package to manage such scripts is that it saves us from having to create and maintain individual 'init.d' service defintions for each script. One downside is that 'pservice' does not handle child processes (descendants) which are launched from inside our script functions. Thus, our scripts must track any subshell processes created so that we can intercept signals on termination by 'pservice' (mostly 'SIGTERM') and end them prior to the main script ending so as not to leave orphan processes when stopping/starting/restarting 'pservice'. Final point to note is that 'pservice' start/stop controls all scripts together and not individually. If one wishes to control each script daemon individually then one would be encouraged to write proper service files for each one to be called by procd directly on boot.

#### fancontrol.sh
This script controls our case fans. On first run this script will add itself to 'pservice' config if not present already. Also, it checks that the selected modem AT interface is unbound from ModemManager so we can use it. If this isn't the case, it creates the necessary 'udev' rule to unbind the interface and prompts the user to reboot for the change to take effect. The script runs as a daemon under 'pservice' and checks the modem SoC temperature once per minute. If the temperature is over the defined limit threshold (55c by default), it will power on the fans. Once the modem has cooled below the limit, the fans are deactivated. Fan activation/deactivation by this script is logged to the system log; the history can be viewed with 'logread -e FAN_CONTROL'. Before running this script the following variables should be entered appropriately:

**$HUB** - Obtain w/ 'lsusb' ('idVendor:idProduct').

**$PORTS** - Populate with hub port numbers of connected fans using appropriate uhubctl syntax. Ex. '2-3' (ports two through three), '1,4 (ports one and four), etc.

**$ATDEVICE, $MMVID, $MMPID, $MMUBIND** - Found in '/lib/udev/rules.d/77-mm-[vendor]-port-types.rules'. '...ttyUSB2...AT primary port...ATTRS{idVendor}=="2c7c", ATTRS{idProduct}=="0800", ENV{.MM_USBIFNUM}=="02"...'. Ex. ATDEVICE="/dev/ttyUSB2", MMVID="2c7c", MMPID="0800", MMUBIND="02".

**$LIMIT, $INTERVAL** - Temperature threshold in degrees celsius when fans should be activated and time in seconds between polling modem temperature.

#### modemwatcher.sh
This script is necessary because ModemManager does not automatically cycle the modem interface on state changes that can occur during normal 'modem <-> tower' communications. Other ModemManager OS platforms (especially Debian based) use NetworkManager to handle this scenario but OpenWRT has no direct feature parity here. This is a known 'issue' when using ModemManager on OpenWRT and not a defect of ModemManager itself (see: https://lists.freedesktop.org/archives/modemmanager-devel/2021-July/008739.html). Other solutions I found relied on 'watchdog' scripts scheduled at a regular interval under cron to wait and re-query the modem before taking action which resulted in some seconds or minutes of actual Internet connectivity to downstream clients which is really undesirable. Thus, this script was created to watch the modem in realtime and take immediate action as required.

Functionality-wise, this script watches the modem to see if it leaves the 'connected' state which would result in loss of Internet connectivity. On first run, the script will add itself to 'pservice' config if not present already. From then on it runs as a daemon under 'pservice' and watches the system log in real time to check for state changes. If the modem does leave the 'connected' state, it checks internet connectivity by pinging google.com and cloudflare.com (two sites with incredibly reliable uptime). If Internet connectivity is not found, the script restarts the modem using mmcli (ModemManager command line interface) and re-checks connectivity. The following inputs should be entered appropriately prior to first run:

**$PINGDST, $LIFACE** - Domains to ping, logical (uci) name of the modem interface.

#### quickycom.sh
This is an interactive wrapper for the 'socat' utility which allows us to communicate easily with the modem's AT interface for sending commands and receiving return output (for scripts which interface with the AT port, we use 'socat' directly). On first run this script checks that the selected modem AT interface is unbound from ModemManager so we can use it. If this isn't the case, it creates the necessary 'udev' rule to unbind the interface and prompts the user to reboot for the change to take effect. Also, the script aliases itself as 'qcom' by creating a symlink in bin $PATH ('/usr/sbin/qcom') so that one can simply call it as 'qcom' from under any directory going forward. The following inputs should be entered appropriately prior to first run:

**$CMD, $TIMEOUT** - AT command, timeout period before termindation (in seconds)

**$ATDEVICE, $MMVID, $MMPID, $MMUBIND** - Found in '/lib/udev/rules.d/77-mm-[vendor]-port-types.rules'. '...ttyUSB2...AT primary port...ATTRS{idVendor}=="2c7c", ATTRS{idProduct}=="0800", ENV{.MM_USBIFNUM}=="02"...' Ex. $ATDEVICE="/dev/ttyUSB2", MMVID="2c7c", MMPID="0800", MMUBIND="02".

#### nsacheck.sh
I noticed that sometimes after being connected to the cell for a long period of time (1 week+) the NSA aggregation abilities between the LTE control channel and NR channels seem to stop functioning. This was easily shown by a speedtest which failed to break the 100Mbps barrier when normally the properly functioning NSA connection would exceed this easily (200Mbps+ average). This script is called by cron once daily during off hours (5AM my local time). It calls the Ookla speedtest CLI app for armhf (you must download this separately and place under '/usr/bin/speedtest'), then disables the modem via mmcli if the download result is not greater than *$THRESHOLD* (set to 100Mbps by default). The daemonized modemwatcher.sh script then restarts the modem when finding it disabled. If the speedtest download result is not greater than *$THRESHOLD* then no action is taken. The following inputs should be entered appropriately prior to first run:

**$THRESHOLD** - Threshold in Mbps; a result less than or equal to this will trigger disabling modem.

#### failsafe.sh
I noticed one edge case of connectivity failure that modemwatcher.sh was not catching. In the very rare case that the modem attaches to a cell and receives an IP but the backend carrier routing is broken the modem showed it was in 'connected' state in ModemManager but could not pass any traffic. Again, this is quite rare and a restart of the modem connected it to another cell while the other one was in this error state so apparently the carrier does steer new connections onto a cell that does have proper routing out to the internet. When I observed this issue I am thinking I may have just caught it when the backend connection to the problem cell was undergoing maintenance. In order to prevent manual intervention in these cases, I created this one additional script running as a service (under pservice) which performs a ping test once every five minutes and then disables the modem to be reset by modemwatcher.sh if it cannot reach google or cloudflare. The following inputs should be entered appropriately prior to first run:

**$PINGDST, $LIFACE, $INTERVAL** - Domains to ping, logical (uci) name of the modem interface, and interval.

### Switch Modem to Generic Image
In initial testing I found that the RM502Q-AE had Quectel's auto-image-switching feature activated by default. This 'feature' switches its firmware image (MBN) based on the carrier SIM which is inserted. Thus, when I inserted my carrier SIM it promptly switched to using the commercial image for my carrier. While this first party image allowed me to obtain IP assignment which was very geo-local (lowest latency), I noticed a significant loss of ICMP and UDP packets. Thus, ping and connectivity to UDP (such as external DNS, WireGuard, etc.) was completely broken at worst or unreliable at best.

I found that if I disabled the auto-image-switching and selected the 'Generic' 3GPP image from Quectel instead of my carier image, the ICMP/UDP packet loss issues disappeared. The only downside is that the IP that the carrier then routed me out of on the generic firmware was less geo-local (higher latency). I have opened a support thread with Quectel on this issue but have not received any resolution at this time so in the interim I am staying on the generic image. AT commands for disabling auto-image-switching and switching manually to the generic image are below (utilizing our qcom/quickycom.sh wrapper script):

```bash
qcom AT+CFUN=0
qcom AT+QMBNCFG=\"AutoSel\",0
qcom AT+QMBNCFG=\"Deactivate\"
qcom AT+QMBNCFG=\"select\",\"ROW_Generic_3GPP_PTCRB_GCF\"
qcom AT+CFUN=1,1
```

### Disable Modem NR SA
The only NR SA support in my area is N71 which does not have a lot of bandwidth allocated so for now I have disabled NR SA mode to leverage the significat throughput gains offered by NSA. The command to disable NR SA is below (leveraging our qcom wrapper):

```bash
qcom AT+CFUN=0
qcom AT+QNWPREFCFG=\"nr5g_disable_mode\",1
qcom AT+CFUN=1,1
```

### Build and Configure SMS Tool
Given many folks who setup service on a cellular line may need to periodically receive OTP or other SMS notifications for their account it will be helpful to have a way to send/receve text messages from the web interface of OpenWRT. Luckily such a tool exists for us: https://github.com/4IceG/luci-app-sms-tool . Only problem is that it is not part of the standard OpenWRT repositories for our release so we must cross-compile it ourselves. While GCC libraries and 'make' are availabe as OpenWRT packages, they don't include the rest of the toolchain which we need to succesfully compile the .ipk packages from the GitHub source code. The easiest way to cross-compile is to use another Linux machine to download the appropriate SDK for our architecture and release and then compile it there. I used by Ubuntu VPS which made quick work of it. Mostly the instructions and prerequisites are covered in high level here:
https://openwrt.org/docs/guide-developer/toolchain/using_the_sdk
https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem

The exact steps on my Ubuntu VPS are below for reference:

```bash
cd ~

apt update

apt install build-essential ccache ecj fastjar file g++ gawk gettext git java-propose-classpath libelf-dev libncurses5-dev libncursesw5-dev libssl-dev python python2.7-dev python3 unzip wget python3-distutils python3-setuptools python3-dev rsync subversion swig time xsltproc zlib1g-dev apt-transport-https ca-certificates -y

update-ca-certificates

curl https://archive.openwrt.org/releases/21.02.3/targets/ipq40xx/generic/openwrt-sdk-21.02.3-ipq40xx-generic_gcc-8.4.0_musl_eabi.Linux-x86_64.tar.xz -o sdk.tar.xz

mkdir sdk

tar -xf sdk.tar.xz -C ./sdk/

cd sdk/openwrt-sdk-21.02.3-ipq40xx-generic_gcc-8.4.0_musl_eabi.Linux-x86_64/package

git clone https://github.com/4IceG/luci-app-sms-tool

cd ..

./scripts/feeds update -a
(may run for a few minutes as it downloads ~2GB)

./scripts/feeds install -a

make menuconfig
```
At this point the menuconfig will open. Enter Global Build Settings and in the submenu, deselect/exclude the following options:
- Select all target specific packages by default
- Select all kernel module packages by default
- Select all userspace packages by default
(This cuts down on compile time for these we don't need which are selected by default)

Go to LuCI > Applications and select 'luci-app-sms-tool' by pressing 'm'. Exit and choose 'Yes' to save changes to '.config'. Then we can compile:
```bash
make package/luci-app-sms-tool/compile
```
It will take some minutes to complete depending on CPU as it has to compile all dependencies along with the app source code. You can increase speed by adding the '-j[x]' switch where '[x]' is the number of CPUs the compile should be threaded across. With the default single CPU selection, it took 10-15 minutes to complete.The .ipk files for the resulting two apps, sms-tool and luci-app-sms-tool, will be located under './bin/packages/arm_cortex-a7_neon-vfpv4/base'. I used WinSCP to download them from my VPS and then upload them to the OpenWRT install. I've also placed a copy under https://github.com/hazarjast/5g_linksys_build/tree/main/images for easy download if you would like to use them. Once copied over to OpenWRT they can be instaled with opkg as normal:
```bash
root@OpenWrt:~# opkg install sms-tool_2022-03-21-f07699ab-1_arm_cortex-a7_neon-vfpv4.ipk
Installing sms-tool (2022-03-21-f07699ab-1) to root...
Configuring sms-tool.
root@OpenWrt:~# opkg install luci-app-sms-tool_1.9.4-20220325_all.ipk
Installing luci-app-sms-tool (1.9.4-20220325) to root...
Installing luci-compat (git-22.046.85744-f08a0f6) to root...
Downloading https://downloads.openwrt.org/releases/21.02.3/packages/arm_cortex-a7_neon-vfpv4/luci/luci-compat_git-22.046.85744-f08a0f6_all.ipk
Configuring luci-compat.
Configuring luci-app-sms-tool.
//usr/lib/opkg/info/luci-app-sms-tool.postinst: /etc/uci-defaults/start-smsled: line 5: /etc/init.d/smsled: Permission denied
uci: Entry not found
uci: Entry not found

uci: Entry not found


//usr/lib/opkg/info/luci-app-sms-tool.postinst: line 284: /etc/init.d/smsled: Permission denied
//usr/lib/opkg/info/luci-app-sms-tool.postinst: line 286: /etc/init.d/smsled: Permission denied
root@OpenWrt:~#
```
As you can see there were some errors on install but they do not seem to have affected the functionality of the packages in any way from my testing so far. Now we can login to the web interface of OpenWRT and we should see a new menu option called 'Modem':

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-17_10h12_13.png" />

The first time we click on it it may hang for a minute or two because by default it is trying to access '/dev/ttyUSB0' which our AT port is not located at. Once it loads we need to update the Configuration so that it works properly with our modem:

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-17_10h26_38.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-17_10h27_16.png" />
<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-17_10h27_25.png" />

Be sure to click 'Save & Apply' when done.

***NOTE***
There is also a handy tab which allows us to send AT comands from the gui. Be aware that some of the AT commands included by default can break your configuration (ex. AT+QCFG="usbnet" etc.) so I would not recomend executing any of them unless you know what they do :)

# Results
My local tower offers only n71 SA which is not allocated much bandwidth at present so I am operating in NSA mode with a PCC of B2 or B4/B66 aggregated with n41. The initial results are a solid improvement over my previous average speeds on LTE only and ping is much improved. The device has so far only been tested indoors so I am excited to get it outside and up high to see what additional speed improvements I may achieve under those conditions.

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-09_17h31_49.png" />

<img src="https://github.com/hazarjast/5g_rpi4_build/blob/main/assets/2022-03-09_17h29_09.png" />

# ToDo List
* Cover band/cell locking; maybe add helper/watcher scripts for this
* Extroot for addition of USB storage


## Historical Background
This project is a spinoff of my original [RPi 4 based build](https://github.com/hazarjast/5g_rpi4_build). Full historical context on what motivated that project, and this one, can be found [here](https://github.com/hazarjast/5g_rpi4_build/blob/main/README.md#historical-background).
