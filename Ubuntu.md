

# How to create an AllStar to DMR Bridge (concise version)

We are going to take you through installing one of the most requested DVSwitch bridge types, an Analog to DMR bridge. Throughout this document, we will use real node numbers, real DMR IDs and real TG numbers. Where possible those values will be green. Please substitute your numbers. We will start with a already running ASL node and add the required parts to bridge it to BrandMeister DMR. Throughout this document we will be testing along the way. If you do not get the expected results as shown, stop, check your work and if it all looks right, ask for help. It is a LOT easier to troubleshoot at the points along the way rather than at the end. 

## Prep work
Execute the following commands:

    sudo -s  
    apt-get update
    cd /etc/asterisk 
    systemctl stop asterisk  nano
    apt-get install dvswitch -y

## Asterisk

### Edit rpt.conf
Open the rpt.conf file by executing:
`nano /etc/asterisk/rpt.conf`

Copy the [1999] stanza below and paste it at the top of rpt.conf file. 

*Note: All node numbers below 2000 are reserved for private use. So assuming you don't already have a private node, you don't need to edit this - just copy and paste it as is to the top of the file.*

    [1999]
    rxchannel = USRP/127.0.0.1:34001:32001 	; Use the USRP channel driver. Must be enabled in modules.conf
											; 127.0.0.1 = IP of the target application
											; 34001 = UDP port the target application is listening on
											; 32001 = UDP port ASL is listening on
    duplex = 0 								; 0 = Half duplex with no telemetry tones or hang time.
    hangtime = 0 							; squelch tail hang time 0
    althangtime = 0 						; longer squelch tail hang time 0
    holdofftelem = 1 						; Hold off all telemetry when signal is present on receiver or from connected nodes
										    ; except when an ID needs to be done and there is a signal coming from a connected node.
    telemdefault = 0 						; 0 = telemetry output off.
    telemdynamic = 0 						; 0 = disallow users to change the local telemetry setting with a COP command,
    linktolink = no 						; disables forcing physical half-duplex operation of main repeater while
										    ; still keeping half-duplex semantics (optional)
    nounkeyct = 1 							; Set to a 1 to eliminate courtesy tones and associated delays.
    totime = 180000 						; transmit time-out time (in ms) (optional, default 3 minutes 180000 ms)
    idrecording = |ie 						; id recording or morse string see http://ohnosec.org/drupal/node/87
    idtalkover = |ie 						; Talkover ID (optional) default is none see http://ohnosec.org/drupal/node/129

In the [nodes] stanza, add: 

    1999 = radio@127.0.0.1:4569/1999,NONE 

### Edit modules.conf
Open the rpt.conf file by executing:

    nano /etc/asterisk/modules.conf 

In the [modules] stanza change this line: 

    noload => chan_usrp.so 

to this: 

    load => chan_usrp.so 

### Edit extensions.conf
Open this file by executing:

    nano /etc/asterisk/extensions.conf 

In the [globals] stanza add: 

    NODE1 = 1999 

and in the [radio-secure] stanza add: 

    exten => ${NODE1},1,rpt,${NODE1}

## DVSwitch

    nano /opt/MMDVM_Bridge/MMDVM_Bridge.ini 

In the [General] stanza, change the following line to your callsign: 

    Callsign=W1AW 

In the [General] stanza, change the following line to your DMR ID + any two digits

    Id=1234567

In the [DMR] stanza change `Enable=0` to `Enable=1`

In the [DMR_Network] stanza Change: `Address=dvswitch.org` to your regional DMR master.  In Arkansas we use 3103.repeater.net.  You may need to research to find yours.

Login to your selfcare page at [https://brandmeister.network/?page=selfcare](https://brandmeister.network/?page=selfcare). On the left hand side you should see “My hotspots”.

Notice that the [DMR ID + any two digits] you entered earlier is showing the green “plug” icon. That means it is connected to BrandMeister. Click on that hotspot number. 

In the center of the page you should see Sysops (system operators). You should be the only sysop for now, but you could add others if you want.  You can also setup a static talk group to pass through the bridge. To do so, add that talkgroup you want to always be connected in the "Static Talkgroups" section.

## Analog Bridge
Edit the analog bridge configuration by executing: 

    nano /opt/Analog_Bridge/Analog_Bridge.ini

In the [GENERAL] stanza, change `decoderFallBack` to `true`.

In the [AMBE_AUDIO] stanza make the following changes: 

gatewayDmrId = *your DMR ID number* 
repeaterID = *the [DMR ID + any two digits] you selected earlier*
txTg = *the talkgroup to which you want to send traffic * 
txPort = 31103 
rxPort = 31100

## Wrap it up
Reboot by executing

    reboot

When it comes back up just link your public and private nodes together and you should be bridged from AllStar <> analog <> DMR.
