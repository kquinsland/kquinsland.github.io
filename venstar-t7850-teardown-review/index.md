# Quick look inside Venstar T7850 - One of the only 'no-cloud' WiFi Thermostats that plays nice with Home Assistant



{{< admonition info >}}
As of 2022-05-11, there is an update on my experience with the thermostat [below](#an-update-on-wifi-connectivity)!
{{< /admonition >}}

This is another one of those posts from my never ending quest to integrate Home Assistant with All The Things!

The thermostat that was installed when I moved in was an early Nest thermostat. These thermostats are - for the most part - well reviewed and liked. I had no complaints... except one.

Google only permits programmatic interaction with Nest devices through their [Smart Device Management API](https://developers.google.com/nest/device-access/api).
This API is essentially cloud hosted MQTT and requires a small fee to access.
There is no way to control a supported Nest device without an Internet connection... even if the Home Assistant server and the Nest device are on the same subnet! 👎

This policy makes it needlessly complicated to integrate the thermostat with Home Assistant and that cripples my climate control automations beyond a tolerable level.

With summer approaching, I started looking for a thermostat that would play nice with Home Assistant.
My criteria:

- Work with existing HVAC system and wiring.
- Play nice with Home Assistant, preferably with a _**local**_ API.
- Cost no more than the Nest thermostat.
- Use TCP/IP over WiFi instead of Zigbee or Zwave.


As it turns out, I am not the only one on a similar hunt:

- [Which of these thermostats have the best experience with HA?](https://community.home-assistant.io/t/which-of-these-thermostats-have-the-best-experience-with-ha/207520)

- [Smart Thermostat Recommendations for 2021](https://community.home-assistant.io/t/smart-thermostat-recommendations-for-2021/359434)

- [Best, easiest and of course lowest cost smart thermostat for Home Assistant](https://community.home-assistant.io/t/best-easiest-and-of-course-lowest-cost-smart-thermostat-for-home-assistant/384927)

- [What is the best thermostats to use with Home Automation](https://www.reddit.com/r/homeassistant/comments/kzavi9/what_is_the_best_thermostats_to_use_with_home/)


The "TL;DR:" of most of those threads is: _"Any thermostat with zwave of zigbee will work"_.

As most people don't particularly care about the network protocol(s) used to link Home Assistant to the thermostat, that's fine advice.

For whatever reason, you have to dig much deeper to find people discussing **WiFi** based thermostats that play nice with Home Assistant w/o an internet connection but there are a few out there. I settled on the Venstar T7850.

{{<figure name="oem_render">}}

# Initial Impressions:

Below is a super concise review that is based mostly on my initial impressions / install experience.
I have only recently acquired and installed the T7850 so I can't comment on any of the finer points or drawbacks that could only be known after several months experience with the thermostat.

TL;DR: I wish I had more insight into Google's disappointing decision to not implement a local API for the Nest; I'll uninstall the Venstar and put it up on eBay the nanosecond the Nest gets a local API!


## An Update on WiFi connectivity

Less than 48 hours after installing the thermostat, I noticed that Home Assistant was no longer able to control it.
Every entity on the device showed `Unavailable`. Apparently, the thermostat had fallen off the network and was struggling to get back on.

I went through all the usual troubleshooting steps and was able to confirm a few interesting things:

- Changing the network name and password didn't help.
- I disabled the SSID on my access point and created an identical SSID/network key on an old raspberry pi. The thermostat was _usually_ able to connect to the AP. If I powered down the raspberry pi AP and re-enabled the same network name/key on my AP, the thermostat would not connect. If left on the raspberry pi AP, the thermostat would eventually eventually "fall off".

I initially suspected that the initial firmware update I didn't consent to may have broken something and started to look for a way to downgrade the firmware to the previous version that _did_ manage to connect to my AP quickly.

No such luck.

While the thermostat does have an A/B partition scheme, I couldn't find any way to force a downgrade or even get older firmware files from the manufacturer.

After a few factory resets and additional troubleshooting, I started to look around... and apparently this is not a new problem.

Here are two 'relatively new' reddit threads that mention similar connection issues with similar models in the same product family:


{{<figure name="wifi_issue_01">}}

{{<figure name="wifi_issue_02">}}


And an Amazon question from years ago indicating that this might not be a 'new' issue:

{{<figure name="wifi_issue_03">}}

A Home Assistant community/support form poster seems to have a similar issue and even offers a fix... which did not work.

{{<figure name="wifi_issue_04">}}

My access point is also a Unifi AP so of course there are a few threads on the unifi support forums:

{{<figure name="wifi_issue_05">}}

{{< admonition note >}}
Note: [The FCC documents indicate](https://fccid.io/MUH-SKYPORT2/Attestation-Statements/Models-Covered-2531030) that Venstar did re-brand the thermostat for `First Alert` and `Bionaire`. It is likely that they also did this for `Carrier` as well judging by [pictures](https://images.google.com/search?q=carrier+infinity+touch+control&tbm=isch) of a `Carrier Infinity Touch Control` thermostat.

{{< /admonition >}}

{{<figure name="wifi_issue_06">}}

The user name seems familiar; probably the same user from one of the above reddit threads:
{{<figure name="wifi_issue_07">}}

{{<figure name="wifi_issue_08">}}

{{< admonition warning "TL;DR:" >}}

Ubiquity's Unifi line of wireless access points has a pretty solid reputation for a reason; by and large they just work. I have installed dozens of them over the years and have supported installs with _thousands_ of them and had far fewer issues with those customers/sites than with your typical 'soho' routers and other consumer-grade access points.

As of **right now**, I have a little more than a hundred wireless devices hanging off of the same AP that the thermostat failed to reliably connect to. Over the past several years, I have acquired many more devices and - with the exception of **one other device** - I have never had issues connecting anything to my wireless network.

Even if only some users experience issues with Venstar thermostats connecting to WiFi, I have to wonder why the issue exists at all. It's 2022 and there's no excuse for basic WiFi network compatibility issues like the kind that were more common in the bad old days of early WiFi circa _2005_. Somehow, the hundred+ other devices got their WiFi right... what is Venstar missing?

While I didn't want to introduce _yet another_ wireless's networking standard to my home, I am _considerably_ happier with the Honeywell TH6320ZW2003 that has replaced the T7850.

{{< /admonition >}}

## The Good

- It has _optional_ cloud connectivity! It's trivial to turn this off and - as far as I can tell - it is **mostly** disabled. More on that [below](#network).
- API is [documented](https://developer.venstar.com/documentation/)!
- API accessible over HTTP or HTTPS and can be put behind [BasicAuth](https://en.wikipedia.org/wiki/Basic_access_authentication)! 👏
- Thermostat has _lots_ of tweaks and options meant for 'power users'. These settings ship with reasonably sane default values. I suspect that most of these options are included in the "residential" devices only because they're already baked into the firmware for "commercial" customers that legitimately do need lots of knobs to adjust.
- It's a reasonably good looking LCD screen; viewing angles are decent and the screen is plenty readable in direct sunlight when the brightness is turned all the way up.
- The piezoelectric beeper can be disabled in software! 🔇

## The Bad

- The API is [pretty limited](#api-limitations).
- The screen has an integrated _resistive_ touch panel. This was a poor choice for for a product design in 2014/2015 and is inexcusable for in 2022! 👎
- There is no way to use your own TLS certificate. You have to use the certificate generated by the device. 🙁
- There is no way to disable automatic firmware updates or view release notes on device. If there's an update found, the thermostat will download it and restart... even if the user is _actively using the device_. 😡 Even [windows (finally) let you push the update to _after_ you're done using the device](https://support.microsoft.com/en-us/windows/defer-feature-updates-in-windows-10-770c0ea8-eee5-ae25-f695-8e33f541e04d)!
- There is no ambient brightness sensor or presence detection sensor so the screen is always on at one brightness level.
- The device has WiFI and communicates with the mothership using TLS1.2... but somehow it does not have [NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol) support! Yep! This is literally the **only** internet enabled _thing_ that I own that **still requires me to manually set the date/time** 😡. What a glaring oversight!

### API limitations

{{< admonition warning >}}
The only _documented_ portions of the API [cover interrogating the device for state about sensors and mode](https://developer.venstar.com/documentation/#query).
Keep these limitations in mind when considering exactly what you want to be able to control via the API.

E.G.: It was _very_ disappointing to find out that I can't adjust the screen on/off/brightness state via the API! So much for automatically turning the screen at night and waking it only when nearby motion was detected!
{{< /admonition >}}

There is no _documented_ way to control most of the other device settings that can be manipulated through the screen.
Things you can do via the screen but not via the (documented) API:

- Adjust the date / time
- Adjust the screen brightness
- Adjust any program/schedule settings or adjust vacation mode settings
- Adjust setpoint limits (e.g.: don't let the user try to cool below 25°C or heat above 35°C)
- Set/Disable Screen lock (requires a PIN code to access controls)
- Set 'screensaver' functionality. You can't toggle this via the API nor can you set the "idle timeout" setting or upload custom photos. You have to resort to a desktop / electron app to do so!


I have not bothered with the "skyport" remote control functionality that Venstar offers but their marketing literature implies that some of the above points _can_ be manipulated via their remote service. This leads me to believe that there is more to the API that what is documented publicly.

Given that the thermostat seems to have it's own CA created by Venstar engineering, I would bet that simple TLS proxy will not be enough to discover any undocumented API endpoints in the unit that could allow for more control.

If anybody does know of a local API endpoint that allows for controlling the screen brightness...[please do get in touch]({{<ref contact>}})!

# Teardown

It's not a particularly complicated device: everything is on one side of a multi-layer PCB.
A ton of electronics for directly interfacing with thermostat wires and a few ICs for all the fun stuff!

The photos are mine, taken right before installing the unit.
There are more / similar photos at the [FCC filing](https://fccid.io/MUH-SKYPORT2/Internal-Photos/Internal-Photos-Revised-2531033) if you're curious.

I am including mine for some additional detail and as a 'mirror' of the FCC photos. Shoutout to the _amazing_ `fccid.io` ❤️!

The main PCB is readily accessed - just remove the rear panel / wall mount bracket.

{{<figure name="main_pcb" >}}

Lots of passives and misc ICs and a few unpopulated footprints. This is consistent with [some of the documentation on file with the FCC](https://fccid.io/MUH-SKYPORT2/Attestation-Statements/Models-Covered-2531030): there is 1 PCB for all 6 products in the family and the only significant difference is the software and the humidity sensor.
If i had to guess, I'd be that `U13` is the humidity sensor.

Only one obvious pin header but the silkscreen `JP2` implies that there's another one somewhere (`JP1`).
The pins are suspiciously close to the full sized SD card and don't match the pinout for an obvious UART but 6 pins could be JTAG.

{{<figure name="main_pcb_rear" >}}

There's nothing remarkable on the other side; just a connection for the LCD and a beeper. At least the beeper module is trivial to "factory delete" if the software option to disable it ever stops working!

The full sized SD card is for users to store photos and firmware upgrades.
Photo parsing is notoriously tricky so there's a decent chance that a malicious photo could be crafted to attack the thermostat. 🤔

All the business logic lives under the big metal shield. I didn't probe the two test points but they certainly look like they could be UART or similar interface to the app processor.

{{<figure name="feat_main_app_proc" >}}

Here's a partial shot of the LCD panel barcode.
{{<figure name="lcd_beeper_closeup" >}}

And a quick bonus picture! I took this just a few moments after the thermostat had been installed and powered up.
I had connected it to the WiFi network just 45 seconds before and was exploring the system settings when the unit locked up for a second and then kicked me to this screen:

{{<figure name="surprise_upgrade" >}}

There was no explanation for the abrupt reboot but after the device came back, the system settings indicated the device was running the latest firmware.

**Note:** Look at the date/time! The device _obviously_ has been able to phone home and download the latest firmware file... but does nothing with the NTP server it asked for (and received!)

# Other technical details

## Firmware

The firmware updates can be obtained through the desktop app. They are written out to a SD card with this directory structure:

```shell
❯ tree
.
├── FOUND.000
│   └── FILE0000.CHK
├── System Volume Information
│   ├── IndexerVolumeGuid
│   └── WPSettings.dat
├── VC
│   └── firmware
│       └── Update.bin
├── VH
│   ├── dealer.bin
│   ├── dealerlogo.bin
│   ├── dealerlogo - Copy.jpg
│   ├── firmware1a
│   │   └── update.bin
│   ├── gallery
│   │   ├── 0.bin
│   │   ├── gallery.bin
│   │   └── thumbs
│   │       └── 0.bin
│   ├── name.bin
│   ├── schedule.bin
│   └── settings.bin
├── VR
│   └── firmware
│       └── Update.bin
└── VW
    └── firmware1a
        └── update.bin

12 directories, 16 files
```

The `dealerlogo - Copy.jpg` was me testing something related to the "user photos" function of the desktop app. The `gallery/*.bin` files are just re-sized jpeg files.
There is a mechanism for backing up / importing settings using `bin` files which are just `json` files store in the root directory for the device model; `VH` in this case. Those files are not pictured above as I discovered the export function after I drafted this section of the post.

I could do a whole _series_ of posts on reverse engineering the firmware but that's going to have to wait for another day / more time. In the interim, here's some of my findings:

- The user interface is javascript. Yep! The entire interface is a single page web app that seems to be hosted by a binary called `maestero2`. Not much comes up on google, but [this repo](https://github.com/grassroot72/Maestro2) does seem like it could be related.
  - I can _see_ in the source code where the javascript controls the LCD backlight... so there is **absolutely no good reason** why I can't do the same via the "local API".
- The device appears to use mutual TLS when talking to the remote endpoints. Try to make a request to `https://ctupdate.skyport.io/feed` and you'll see the server ask you for a certificate :D.
  - I don't know if the certificates are _per device_ or not. The certificates are stored in a separate `jffs2` partition which is _not_ distributed in the firmware updates (as best I can tell).
  - There are a few strings that mention code signing certificates but I have not probed the firmware update routines in depth to know how it all works.
- It appears that the TTY is disabled and there are no telnet/ssh services started on boot so it's unclear how the `root` user can be used remotely... but I did find this in `/etc/shadow`: `root:$1$JEstzl9y$Ed7nAJIsY/0irewnqZoqn1:10933:0:99999:7:::`.

## network

{{< admonition warning >}}
Even though the `skyport` functionality has been switched off, [I still see the thermostat phoning home to `ctupdate.skyport.io` on startup](#dns). In addition to the usual "write firewall rules preventing WAN access", I would suggest sinkholing the `skyport.io` domain.
{{< /admonition >}}

Like with all new devices, I ran `tcpdump` while setting it up. Almost all of the traffic was TLS1.2 protected but I did notice a few interesting things from the packet capture.

Immediately after booting / joining the network, the device sends out a DHCP request:

```text
Dynamic Host Configuration Protocol (Discover)
    Message type: Boot Request (1)
    Hardware type: Ethernet (0x01)
    Hardware address length: 6
    Hops: 0
    Transaction ID: 0x1308ee40
    Seconds elapsed: 0
    Bootp flags: 0x0000 (Unicast)
    Client IP address: 0.0.0.0
    Your (client) IP address: 0.0.0.0
    Next server IP address: 0.0.0.0
    Relay agent IP address: 0.0.0.0
    Client MAC address: MurataMa_de:ad:bf (10:98:c3:de:ad:bf)
    Client hardware address padding: 00000000000000000000
    Server host name not given
    Boot file name not given
    Magic cookie: DHCP
    Option: (53) DHCP Message Type (Discover)
        Length: 1
        DHCP: Discover (1)
    Option: (61) Client identifier
        Length: 7
        Hardware type: Ethernet (0x01)
        Client MAC address: MurataMa_de:ad:bf (10:98:c3:de:ad:bf)
    Option: (57) Maximum DHCP Message Size
        Length: 2
        Maximum DHCP Message Size: 576
    Option: (55) Parameter Request List
        Length: 7
        Parameter Request List Item: (1) Subnet Mask
        Parameter Request List Item: (3) Router
        Parameter Request List Item: (6) Domain Name Server
        Parameter Request List Item: (12) Host Name
        Parameter Request List Item: (15) Domain Name
        Parameter Request List Item: (28) Broadcast Address
        Parameter Request List Item: (42) Network Time Protocol Servers
    Option: (60) Vendor class identifier
        Length: 12
        Vendor class identifier: udhcp 1.29.2
    Option: (255) End
        Option End: 255
    Padding: 0000000000000000000000000000000000000000
```

**Note:** `de:ad:bf ` is replacing the _actual_ octets of my thermostat MAC :).

Interesting! The DHCP server _explicitly asks for a NTP server_ and then the thermostat ... does not use it!

The `udhcp 1.29.2` string implies a relatively recent build of - possibly - busybox running the show...

The next packets after that are basic SSDP and IGMP:

```text
NOTIFY ALIVE SDDP/1.0
From: "123.456.789.012:1902"
Host: "10:98:c3:de:ad:bf"
Max-Age: 300
Type: "venstar:control4_thermostat_proxy:colortouch"
Primary-Proxy: "thermostatV2"
Proxies: "thermostatV2"
Manufacturer: "Venstar"
Model: "ColorTouch"
Driver: "venstar_ip_colortouch_hvac.c4z"
```

```text
Internet Group Management Protocol
    [IGMP Version: 3]
    Type: Membership Report (0x22)
    Reserved: 00
    Checksum: 0xea03 [correct]
    [Checksum Status: Good]
    Reserved: 0000
    Num Group Records: 1
    Group Record : 239.255.255.250  Change To Exclude Mode
        Record Type: Change To Exclude Mode (4)
        Aux Data Len: 0
        Num Src: 0
        Multicast Address: 239.255.255.250
```

### DNS

And _just before_ the TLS traffic, there is a single DNS query:

```text
Queries
    ctupdate.skyport.io: type A, class IN
        Name: ctupdate.skyport.io
        [Name Length: 19]
        [Label Count: 3]
        Type: A (Host Address) (1)
        Class: IN (0x0001)
```

### Web server

After setting a user/password and selecting the `https` option for the local API, here's what I see:

```shell
❯ openssl s_client -connect 123.456.789.012:443
CONNECTED(00000003)
depth=0 C = US, ST = California, L = Chatsworth, O = Venstar Inc., OU = Engineering, CN = CT1A_100163664
verify error:num=20:unable to get local issuer certificate
verify return:1
depth=0 C = US, ST = California, L = Chatsworth, O = Venstar Inc., OU = Engineering, CN = CT1A_100163664
verify error:num=21:unable to verify the first certificate
verify return:1
depth=0 C = US, ST = California, L = Chatsworth, O = Venstar Inc., OU = Engineering, CN = CT1A_100163664
verify return:1
---
Certificate chain
 0 s:C = US, ST = California, L = Chatsworth, O = Venstar Inc., OU = Engineering, CN = CT1A_100163664
   i:C = US, ST = California, O = Venstar Inc., OU = Engineering, CN = Skyport Root CA
---
Server certificate
-----BEGIN CERTIFICATE-----
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
kVBzZrkbvA==
-----END CERTIFICATE-----
subject=C = US, ST = California, L = Chatsworth, O = Venstar Inc., OU = Engineering, CN = CT1A_100163664

issuer=C = US, ST = California, O = Venstar Inc., OU = Engineering, CN = Skyport Root CA

---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 2277 bytes and written 430 bytes
Verification error: unable to verify the first certificate
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 4096 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 1391DF607DB2888CA81220E321CB06CA6B5CBDE2031483A3E4AC075AD95A6A5D
    Session-ID-ctx:
    Master-Key: 7A37EEBC584574EBA6B114E85CCD685B30F640D17D38A9B2E90AC8739BC76EB4A15E38A8F3B74F3D428328F874C9807C
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1650744968
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)
    Extended master secret: yes
```

The thermostat was pretty well closed off. `nmap` came back with port 53 and 443 open. Taking a closer lok at 443:

```shell
PORT    STATE SERVICE  VERSION
443/tcp open  ssl/http Neato Botvac Connected
| ssl-cert: Subject: commonName=CT1A_100163664/organizationName=Venstar Inc./stateOrProvinceName=California/countryName=US
| Issuer: commonName=Skyport Root CA/organizationName=Venstar Inc./stateOrProvinceName=California/countryName=US
| Public Key type: rsa
| Public Key bits: 4096
| Signature Algorithm: sha512WithRSAEncryption
| Not valid before: 2019-11-08T05:19:36
| Not valid after:  2037-05-17T23:59:59
| MD5:   3efc 609c bbf3 cfb6 ddfb 4cbe bb54 1c63
|_SHA-1: 1e65 43f9 8404 9bf6 acf4 0be8 5116 78ff bc02 f024
Service Info: Device: specialized
```

And taking a closer look @ the web server:

```text
❯ curl -vvv --insecure -L 'https://123.456.789.012/' -H 'Authorization: Digest username="SomeUserNameHere", realm="thermostat", nonce="1234567890", uri="/", response="some_64_char_string_here", qop=auth, nc=00000003, cnonce="some16CharStrHere"'
*   Trying 123.456.789.012:443...
* Connected to 123.456.789.012 (123.456.789.012) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: C=US; ST=California; L=Chatsworth; O=Venstar Inc.; OU=Engineering; CN=CT1A_100163664
*  start date: Nov  8 05:19:36 2019 GMT
*  expire date: May 17 23:59:59 2037 GMT
*  issuer: C=US; ST=California; O=Venstar Inc.; OU=Engineering; CN=Skyport Root CA
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
> GET / HTTP/1.1
> Host: 123.456.789.012
> User-Agent: curl/7.82.0
> Accept: */*
> Authorization: Digest username="SomeUserNameHere", realm="thermostat", nonce="1234567890", uri="/", response="some_64_char_string_here", qop=auth, nc=00000003, cnonce="some16CharStrHere"
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Transfer-Encoding: chunked
<
* Connection #0 to host 123.456.789.012 left intact
{"api_ver":9,"type":"residential","model":"COLORTOUCH","firmware":"6.91"}%
```

Unfortunately no headers that leak details about the web server implementation. 😒

## ICs and distinguishing markings

As noted above, the WiFi module was covered with a soldered on shield. Fortunately, the FCC filings give a good look!
The chip is a `muRata Type ZX WiFi Module` which is the [_same_ WiFi radio thats inside the Nest Protect](https://learn.sparkfun.com/tutorials/nest-protect-teardown/sensor-scavenger-hunt)!

The larger of the two metal shields on the PCB can be removed giving a look at the main application processor:

- `W971GG` (probably [`W971GG6JB`](https://www.alldatasheet.com/datasheet-pdf/pdf/555599/WINBOND/W971GG6JB-25.html)) - DRAM.

- [`W29N01HVBINA`](https://www.digikey.com/en/products/detail/winbond-electronics/W29N01HVBINA/5804021): 1gigabit flash.

- [`AT91SAM9G15`](https://www.microchip.com/en-us/product/AT91SAM9G15) - ARM926 with the peripherals needed for driving a LCD and reading from the resistive (🤮) touchscreen

The LCD is marked with (partially visible):

    AT043HS40D07R2
    30671T051KD
    190805527 (0000000) L101661

