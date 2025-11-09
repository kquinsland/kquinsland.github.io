# Using Qi charging to fix the biggest design flaw with the HidrateSpark Steel Pro bottle

{{< admonition info "Parts and Instructions" >}}
The majority of this post covers the "why" this mod came to be. If you're just looking for the mod, you can find
the 3d printable parts, BOM and instructions in [accompanying github repo](https://github.com/kquinsland/hidrate-spark-qi-charging).
{{< /admonition >}}

# What

[Hidrate Spark](https://hidratespark.com/) bottles are one of a small but growing number of 'smart' water bottles.
In this instance, 'smart' refers to some mechanism for reporting on and tracking the bottles content over time.

There are a few ways to measure liquid levels but the current generation of Hidrate Spark bottles uses a small sensor puck which contains a [load cell](https://en.wikipedia.org/wiki/Load_cell).
As the bottle is emptied or filled, the puck measures a difference in weight which can then be compared with the known weight of a full/empty bottle to determine exactly how full the bottle is.

There are a _ton_ of reasons why you might want to track your water intake but they all generally fall into one of a few broad categories:

- Dietary restrictions: Athletes and competitors of all sorts need to precisely track their diets and water intake is no exception.
- Medication side effects:  Any medication with 'diminishes thirst' as a side effect almost guarantees dehydration. A bottle that knows how long it's been since your last drink is a very effective way to mitigate these side effects.
- Personal curiosity / [quantified self](https://en.wikipedia.org/wiki/Quantified_self): If you're already tracking other aspects of your physical/mental health throughout the day, knowing when/how much water you had makes for some more interesting graphs.
  - In the same vein, your levels of hydration can effect: cognition, blood pressure, mood, blood glucose levels, energy, complexion and more.

Now as for why you'd need to bolt a kitchen scale and a BTLE radio onto the bottom of a water bottle in order to record this, there's really only one reason: Automation.

{{< admonition warning >}}
I'm not going to entertain the "now you're just being silly; there's no need for a water bottle to have a radio or battery in it" crowd. If you don't value any of the reasons outlined above then we agree... this is a silly water bottle and **you** don't have a good reason to own one.
{{< /admonition >}}

## Design evaluation

Overall, the Hidrate Spark bottle works. It is not without some drawbacks, though.
The primary and obvious downside is charging the battery 🪫.

Depending on how aggressive you have the LEDs configured, the battery will need charging somewhere between 1-5 times a week.
The app will inform you when the battery is low, but sometimes the notifications come far to late to be useful. I have lost data because the warning came less than an hour before the battery was dead and about 9 hours before I'd be able to charge it again.

To charge the battery, you need to remove the sensor puck from the bottle and then attach a fiddly and proprietary USB connector. Worse, this connector uses prone-to-failure [Pogo pins](https://en.wikipedia.org/wiki/Pogo_pin).

Ignoring the shameful and unnecessary use of a non-standard USB micro/C port for charing, you now have to wait a few hours for the battery to charge; during this time, the puck can't be inserted into the bottle so it can't record data while it's being charged.
Remind you of anything?

{{<figure name="mighty_mouse">}}

So to re-cap:

- Anywhere from 1-5 times a week you'll need to remove the sensor from the bottle. During this time you'll have to manually track any consumed water.... defeating the entire point of the bottle!
- If you don't charge the puck in time, you'll also loose automatic tracking of consumption... with little or no notice from either the puck or the app. The puck does not store measurements in flash so any data recorded before the battery died but not yet synced to the phone is _gone_.
- You will need to pack a dedicated cable for charging the bottle if you travel. Don't loose or damage it because you can't go to the closest electronics shop to get another compatible cable!

When I did my initial tear down, I didn't spot any electrical reason why the bottle could not charge and measure at the same time.
After some additional testing, I was able to confirm that the firmware also does not object.

At the **_very least_** the Hidrate team could have put the pogo pins on the bottom and shipped the bottle with a charging dock/base similar to how Ember does their mugs.

{{<figure name="ember_mug">}}

The proprietary nature of the charge connection is still not ideal but at least the bottle could be "used" while it was charging.
If the charging dock was centrally or conveniently located, the sensor puck could be consistently 'topped off'; trickle charing while idle in the same way wireless charging of phots works; take a sip and put the bottle back down to charge and measure the consumption.

Alternatively, the Hidrate designers could have at least used a standard USBC or micro USB jack in the same location as the current charging pogo pins.
Since the charge location is at the 'top' of the sensor puck which is deeply embedded in the base of the water bottle, they might not have even needed to bother with a 'fancy' waterproof USB port and the additional electronics necessary to detect water/debris and cut off charging.

Going with a standard USB C port in place of pogo pins would have the "can't use it and charge it at the same time" draw back but it would have at least eliminated a proprietary cable!

## The other draw backs

While the biggest design flaw is related to battery charging, the Hidrate spark does have a few other noteworthy issues.

### Recalibration

The sensor puck measures changes in weight.
To know how full the bottle is, the sensor puck must also be aware of the weight of a full and empty bottle.
This weight range will change based on the density of the liquid inside.
A bottle that is full of water will have a different weight compared to a bottle that is full of orange juice, for example.
Any time the liquid in the bottle changes, the sensor puck will need to be recalibrated.

To measure changes in weight/weight, the sensor puck relies on a load cell which (slightly) deforms as the load on it changes.
All mechanical parts will wear with time so the bottle will periodically need to be recalibrated from time to time just to stay accurate.
Likewise, if the bottle is dropped or otherwise experiences a sharp impact, the sensor will most certainly need to be recalibrated.

### Misc

- No temperature. The sensor is completely independent from the primary bottle so there is no easy way to record the temperature of the liquid. It would be nice to know how cold the liquid inside is from time to time. The hidrate team could have engineered a temperature sensor into the bottle body that would come into contact with the sensor puck when inserted but this almost certainly would have increased manufacturing costs for a relatively small benefit.
- Imprecise and limited user feedback. The only way the sensor puck can communicate with the user is through the ring of addressable LEDs around the permitter and through BTLE; the latter of which is unreliable as the user might not have their phone nearby or the app might be closed. There is no way for the lights to communicate _exactly_ how long it's been since the last drink or how much battery life is left. There are 10 LEDs and they can each be individually controlled so this drawback probably can be addressed through software; let the user pick when/how to display battery life remaining or to design a sequence of lights to indicate when the last drink took place.
- Most of the mass needs to be suspended above and not directly connected to the sensor puck. This means that the bottle has a high center of gravity which - when coupled with the small base - makes for a slightly unstable bottle. There isn't any real way to avoid this though; if you're going to use a load cell, the liquid must ~~necessarily~~ reasonably be above the load cell.

## A quick look at the TuYa bottle

If you search the usual eastern import sites, you'll see that the _massive_ TuYa ecosystem has it's own smart water bottle that can keep track of liquid intake.
As best as I can tell, this is branded for TuYa but the OEM is [`Maxevis Smart Bottle(BLE)`](https://expo.tuya.com/product/839372) but the best FCC ID listing that I could find has [`2ALQ5-M1`](https://fccid.io/2ALQ5-M1/Internal-Photos/Internal-photos-5000615) listed.

{{<figure name="tuya_bottle">}}

Rather than use a load cell to measure the weight of the bottle and then calculate the liquid volume, the TuYa bottle uses an AMS IR range detector (probably `TMF8805`) to measure the distance to the top of the liquid.

This approach is superior in virtually every way:

- All the electronics are in the lid: The entire top of the lid 'free' to communicate information to the user via a numeric screen. As a bonus, the TuYa lid supports charging while measuring the liquid level!
- Cheaper to manufacture: There is no complicated silicon overmolding as there is nothing in the lid that needs to fled or bend or otherwise deform.
- Sensor measures distance to the top of the liquid. Assuming a fixed volume, the density of the liquid does not matter; as long as the surface of the liquid is level and reflects IR, the sensor can tell you how full the bottle is.

Downsides:

- Similar recalibration will be required if changing the bottle size... but at least no re-calibration needed when changing the liquid! As far as I can tell, there is only one size of bottle so this is a bit of a moot point.
- The data is not easily accessible. You can only get the data using their proprietary app and the TuYa app does not seem to have any easy way to exfiltrate this data to the sources where I'd prefer to record and manage it: FitBit.

### Brief bit of TuYa / Maxevis / Sguai M1 BTLE logs

Below is a (lightly) obfuscated log file from the _amazing_ [nRF connect app](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp&hl=en_US&gl=US).
I am dumping the BTLE characteristics and data here for SEO purposes. If you do find this post and these logs and _do_ manage to reverse engineer the packet format... [do let me know!]({{< relref "/contact" >}}).

<!-- markdownlint-disable-file MD010 -->
```text
nRF Connect, 2022-06-12
M1 (DC:23:4D:DE:AD:BE)
D	11:35:37.604	[Broadcast] Action received: android.bluetooth.device.action.ACL_CONNECTED
V	11:35:37.796	Connecting to DC:23:4D:DE:AD:BE...
D	11:35:37.796	gatt = device.connectGatt(autoConnect = false, TRANSPORT_LE, preferred PHY = LE 1M)
D	11:35:37.799	[Callback] Connection state changed with status: 0 and new state: CONNECTED (2)
I	11:35:37.799	Connected to DC:23:4D:DE:AD:BE
V	11:35:37.809	Discovering services...
D	11:35:37.809	gatt.discoverServices()
D	11:35:37.811	[Callback] Services discovered with status: 0
I	11:35:37.811	Services discovered
V	11:35:37.816	Generic Access (0x1800)
- Device Name [R] (0x2A00)
- Appearance [R] (0x2A01)
Generic Attribute (0x1801)
Unknown Service (00001910-0000-1000-8000-00805f9b34fb)
- Unknown Characteristic [W WNR] (00002b11-0000-1000-8000-00805f9b34fb)
- Unknown Characteristic [N] (00002b10-0000-1000-8000-00805f9b34fb)
   Client Characteristic Configuration (0x2902)
D	11:35:37.816	gatt.setCharacteristicNotification(00002b10-0000-1000-8000-00805f9b34fb, true)
V	11:36:09.013	Refreshing device cache...
D	11:36:09.013	gatt.refresh() (hidden)
I	11:36:09.015	Cache refreshed
V	11:36:09.019	Discovering services...
D	11:36:09.019	gatt.discoverServices()
D	11:36:09.928	[Callback] Services discovered with status: 0
I	11:36:09.928	Services discovered
V	11:36:09.931	Generic Access (0x1800)
- Device Name [R] (0x2A00)
- Appearance [R] (0x2A01)
Generic Attribute (0x1801)
Unknown Service (00001910-0000-1000-8000-00805f9b34fb)
- Unknown Characteristic [W WNR] (00002b11-0000-1000-8000-00805f9b34fb)
- Unknown Characteristic [N] (00002b10-0000-1000-8000-00805f9b34fb)
   Client Characteristic Configuration (0x2902)
D	11:36:09.931	gatt.setCharacteristicNotification(00002b10-0000-1000-8000-00805f9b34fb, true)
V	11:36:12.944	Reading all characteristics...
V	11:36:12.944	Reading characteristic 00002a00-0000-1000-8000-00805f9b34fb
D	11:36:12.944	gatt.readCharacteristic(00002a00-0000-1000-8000-00805f9b34fb)
I	11:36:13.027	Read Response received from 00002a00-0000-1000-8000-00805f9b34fb, value: 0 bytes
V	11:36:13.030	Reading characteristic 00002a01-0000-1000-8000-00805f9b34fb
D	11:36:13.030	gatt.readCharacteristic(00002a01-0000-1000-8000-00805f9b34fb)
I	11:36:13.126	Read Response received from 00002a01-0000-1000-8000-00805f9b34fb, value: (0x) 00-00
A	11:36:13.126	"[0] Unknown" received
V	11:36:13.128	2 characteristics read
I	11:36:14.079	Connection parameters updated (interval: 7.5ms, latency: 0, timeout: 5000ms)
I	11:36:14.176	Connection parameters updated (interval: 50.0ms, latency: 0, timeout: 5000ms)
V	11:36:22.198	Reading characteristic 00002a01-0000-1000-8000-00805f9b34fb
D	11:36:22.198	gatt.readCharacteristic(00002a01-0000-1000-8000-00805f9b34fb)
I	11:36:22.276	Read Response received from 00002a01-0000-1000-8000-00805f9b34fb, value: (0x) 00-00
A	11:36:22.276	"[0] Unknown" received
V	11:36:22.734	Reading characteristic 00002a01-0000-1000-8000-00805f9b34fb
D	11:36:22.734	gatt.readCharacteristic(00002a01-0000-1000-8000-00805f9b34fb)
I	11:36:22.826	Read Response received from 00002a01-0000-1000-8000-00805f9b34fb, value: (0x) 00-00
A	11:36:22.826	"[0] Unknown" received
V	11:36:27.086	Reading all characteristics...
V	11:36:27.086	Reading characteristic 00002a00-0000-1000-8000-00805f9b34fb
D	11:36:27.086	gatt.readCharacteristic(00002a00-0000-1000-8000-00805f9b34fb)
I	11:36:27.176	Read Response received from 00002a00-0000-1000-8000-00805f9b34fb, value: 0 bytes
V	11:36:27.179	Reading characteristic 00002a01-0000-1000-8000-00805f9b34fb
D	11:36:27.179	gatt.readCharacteristic(00002a01-0000-1000-8000-00805f9b34fb)
I	11:36:27.277	Read Response received from 00002a01-0000-1000-8000-00805f9b34fb, value: (0x) 00-00
A	11:36:27.277	"[0] Unknown" received
V	11:36:27.281	2 characteristics read
V	11:36:41.136	Reading PHY...
D	11:36:41.136	gatt.readPhy()
I	11:36:41.160	PHY read (TX: LE 1M, RX: LE 1M)
V	11:36:45.223	Reading all characteristics...
V	11:36:45.223	Reading characteristic 00002a00-0000-1000-8000-00805f9b34fb
D	11:36:45.223	gatt.readCharacteristic(00002a00-0000-1000-8000-00805f9b34fb)
I	11:36:45.326	Read Response received from 00002a00-0000-1000-8000-00805f9b34fb, value: 0 bytes
V	11:36:45.329	Reading characteristic 00002a01-0000-1000-8000-00805f9b34fb
D	11:36:45.329	gatt.readCharacteristic(00002a01-0000-1000-8000-00805f9b34fb)
I	11:36:45.426	Read Response received from 00002a01-0000-1000-8000-00805f9b34fb, value: (0x) 00-00
A	11:36:45.426	"[0] Unknown" received
V	11:36:45.429	2 characteristics read
V	11:36:48.225	Enabling services...
V	11:36:48.225	Enabling notifications for 00002b10-0000-1000-8000-00805f9b34fb
D	11:36:48.225	gatt.setCharacteristicNotification(00002b10-0000-1000-8000-00805f9b34fb, true)
D	11:36:48.226	gatt.writeDescriptor(00002902-0000-1000-8000-00805f9b34fb, value=0x0100)
I	11:36:48.327	Data written to descr. 00002902-0000-1000-8000-00805f9b34fb, value: (0x) 01-00
A	11:36:48.327	"Notifications enabled" sent
V	11:36:48.331	Notifications enabled for 00002b10-0000-1000-8000-00805f9b34fb
V	11:36:48.335	All services enabled
I	11:36:48.427	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-3A-05-27-3C-3B-DF-88-0F-BF-B6-2C-9A-28-E5-47-9D-93-2C
A	11:36:48.427	"(0x) 00-31-3A-05-27-3C-3B-DF-88-0F-BF-B6-2C-9A-28-E5-47-9D-93-2C" received
I	11:36:48.427	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-C3-81-F8-D3-88-1F-C5-F0-68-9E-3E-21-71-97-E0-CF-69-B8-D2
A	11:36:48.427	"(0x) 01-C3-81-F8-D3-88-1F-C5-F0-68-9E-3E-21-71-97-E0-CF-69-B8-D2" received
I	11:36:48.427	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-76-24-15-EA-88-4C-E9-F6-2D-98-D3-F5-30
A	11:36:48.427	"(0x) 02-76-24-15-EA-88-4C-E9-F6-2D-98-D3-F5-30" received
V	11:36:51.059	Reading all characteristics...
V	11:36:51.059	Reading characteristic 00002a00-0000-1000-8000-00805f9b34fb
D	11:36:51.059	gatt.readCharacteristic(00002a00-0000-1000-8000-00805f9b34fb)
I	11:36:51.126	Read Response received from 00002a00-0000-1000-8000-00805f9b34fb, value: 0 bytes
V	11:36:51.128	Reading characteristic 00002a01-0000-1000-8000-00805f9b34fb
D	11:36:51.128	gatt.readCharacteristic(00002a01-0000-1000-8000-00805f9b34fb)
I	11:36:51.226	Read Response received from 00002a01-0000-1000-8000-00805f9b34fb, value: (0x) 00-00
A	11:36:51.226	"[0] Unknown" received
V	11:36:51.227	2 characteristics read
I	11:37:29.428	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-3B-05-29-D8-E0-4B-72-0B-28-02-BC-F6-A8-11-D5-59-34-38
A	11:37:29.428	"(0x) 00-31-3B-05-29-D8-E0-4B-72-0B-28-02-BC-F6-A8-11-D5-59-34-38" received
I	11:37:29.478	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-77-3B-AE-22-0C-A7-5A-F8-31-41-0C-FF-53-B7-97-D0-17-BE-43
A	11:37:29.478	"(0x) 01-77-3B-AE-22-0C-A7-5A-F8-31-41-0C-FF-53-B7-97-D0-17-BE-43" received
I	11:37:29.478	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-8F-38-BE-8E-5C-9F-1F-40-FC-15-66-1F-01
A	11:37:29.478	"(0x) 02-8F-38-BE-8E-5C-9F-1F-40-FC-15-66-1F-01" received
I	11:37:29.478	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-3C-05-63-F4-26-37-1F-87-CD-CE-7C-D2-60-BD-DE-95-6A-C4
A	11:37:29.478	"(0x) 00-31-3C-05-63-F4-26-37-1F-87-CD-CE-7C-D2-60-BD-DE-95-6A-C4" received
I	11:37:29.479	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-75-AA-47-4F-6F-4D-5C-3F-8E-DC-32-3F-A3-F5-36-51-8C-8F-AA
A	11:37:29.479	"(0x) 01-75-AA-47-4F-6F-4D-5C-3F-8E-DC-32-3F-A3-F5-36-51-8C-8F-AA" received
I	11:37:29.479	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-4C-6F-7E-67-92-77-D4-E7-E4-18-2C-59-E5
A	11:37:29.479	"(0x) 02-4C-6F-7E-67-92-77-D4-E7-E4-18-2C-59-E5" received
I	11:37:38.427	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-3D-05-45-90-BD-A3-80-83-E1-1A-DC-2E-02-E9-53-51-66-D0
A	11:37:38.427	"(0x) 00-31-3D-05-45-90-BD-A3-80-83-E1-1A-DC-2E-02-E9-53-51-66-D0" received
I	11:37:38.428	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-6D-C3-48-81-2B-E4-D1-CD-BB-55-20-54-31-97-4F-22-FB-A7-C7
A	11:37:38.428	"(0x) 01-6D-C3-48-81-2B-E4-D1-CD-BB-55-20-54-31-97-4F-22-FB-A7-C7" received
I	11:37:38.428	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-E1-EF-8D-FC-A7-AC-17-C7-10-80-EC-09-2E
A	11:37:38.428	"(0x) 02-E1-EF-8D-FC-A7-AC-17-C7-10-80-EC-09-2E" received
I	11:37:41.428	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-3E-05-3E-AC-56-8F-84-FF-91-E6-4B-0A-3E-95-23-8D-58-5C
A	11:37:41.428	"(0x) 00-31-3E-05-3E-AC-56-8F-84-FF-91-E6-4B-0A-3E-95-23-8D-58-5C" received
I	11:37:41.428	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-DA-FC-E3-F6-BC-AA-A0-C8-15-84-7B-80-AA-29-C4-61-7A-91-DA
A	11:37:41.428	"(0x) 01-DA-FC-E3-F6-BC-AA-A0-C8-15-84-7B-80-AA-29-C4-61-7A-91-DA" received
I	11:37:41.428	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-E9-78-C0-8A-F4-89-EC-8B-9E-B3-99-74-DC
A	11:37:41.428	"(0x) 02-E9-78-C0-8A-F4-89-EC-8B-9E-B3-99-74-DC" received
I	11:38:32.829	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-3F-05-BF-48-A0-FB-1C-FB-10-32-39-66-C3-C1-3F-49-6F-68
A	11:38:32.829	"(0x) 00-31-3F-05-BF-48-A0-FB-1C-FB-10-32-39-66-C3-C1-3F-49-6F-68" received
I	11:38:32.829	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-FF-00-EB-5E-50-22-71-6C-5F-C1-82-8D-95-13-C9-F4-5F-9F-E0
A	11:38:32.829	"(0x) 01-FF-00-EB-5E-50-22-71-6C-5F-C1-82-8D-95-13-C9-F4-5F-9F-E0" received
I	11:38:32.830	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-C1-47-3D-34-8A-5D-85-69-C8-7F-21-DB-4E
A	11:38:32.830	"(0x) 02-C1-47-3D-34-8A-5D-85-69-C8-7F-21-DB-4E" received
I	11:38:33.430	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-30-05-37-64-4C-E7-38-77-8B-FE-17-42-42-6D-96-85-DB-F4
A	11:38:33.430	"(0x) 00-31-30-05-37-64-4C-E7-38-77-8B-FE-17-42-42-6D-96-85-DB-F4" received
I	11:38:33.430	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-7D-C6-26-19-9C-C4-64-C3-CE-9B-BD-1F-8F-41-30-4E-D5-81-31
A	11:38:33.430	"(0x) 01-7D-C6-26-19-9C-C4-64-C3-CE-9B-BD-1F-8F-41-30-4E-D5-81-31" received
I	11:38:33.430	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-FE-01-83-24-0F-A7-4C-2A-07-0E-00-22-F2
A	11:38:33.430	"(0x) 02-FE-01-83-24-0F-A7-4C-2A-07-0E-00-22-F2" received
I	11:38:35.830	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-30-05-17-00-0A-53-C7-73-35-4A-54-9E-6A-99-19-41-CE-00
A	11:38:35.830	"(0x) 00-31-30-05-17-00-0A-53-C7-73-35-4A-54-9E-6A-99-19-41-CE-00" received
I	11:38:35.830	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-DB-8B-0F-A0-8E-61-43-4F-EC-F2-70-91-17-CE-A7-5F-2D-CA-64
A	11:38:35.830	"(0x) 01-DB-8B-0F-A0-8E-61-43-4F-EC-F2-70-91-17-CE-A7-5F-2D-CA-64" received
I	11:38:35.830	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-00-0E-6B-D8-FF-9E-47-CA-AB-45-1F-A3-EB
A	11:38:35.830	"(0x) 02-00-0E-6B-D8-FF-9E-47-CA-AB-45-1F-A3-EB" received
I	11:38:41.880	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-30-05-CE-1C-89-3F-B9-EF-3C-16-61-7A-EC-45-B7-7D-75-8C
A	11:38:41.880	"(0x) 00-31-30-05-CE-1C-89-3F-B9-EF-3C-16-61-7A-EC-45-B7-7D-75-8C" received
I	11:38:41.880	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-03-CF-67-7F-14-6C-54-5A-B7-DB-4D-D7-6E-E9-D7-28-17-3D-64
A	11:38:41.880	"(0x) 01-03-CF-67-7F-14-6C-54-5A-B7-DB-4D-D7-6E-E9-D7-28-17-3D-64" received
I	11:38:41.880	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-D0-68-E2-25-13-D9-18-A3-1E-A9-39-C5-5C
A	11:38:41.880	"(0x) 02-D0-68-E2-25-13-D9-18-A3-1E-A9-39-C5-5C" received
I	11:38:46.881	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-30-05-CC-B8-79-AB-FF-EB-D0-62-AD-D6-77-71-61-39-02-98
A	11:38:46.881	"(0x) 00-31-30-05-CC-B8-79-AB-FF-EB-D0-62-AD-D6-77-71-61-39-02-98" received
I	11:38:46.881	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-4C-68-73-AB-5E-37-67-25-64-7F-EF-7D-2C-EA-CF-8D-4E-CB-90
A	11:38:46.881	"(0x) 01-4C-68-73-AB-5E-37-67-25-64-7F-EF-7D-2C-EA-CF-8D-4E-CB-90" received
I	11:38:46.881	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-A2-01-AF-59-35-60-7F-4F-29-AC-7A-0A-0A
A	11:38:46.881	"(0x) 02-A2-01-AF-59-35-60-7F-4F-29-AC-7A-0A-0A" received
I	11:38:52.881	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 00-31-30-05-82-D4-8B-97-89-67-22-2E-A9-B2-BC-1D-06-75-A5-24
A	11:38:52.881	"(0x) 00-31-30-05-82-D4-8B-97-89-67-22-2E-A9-B2-BC-1D-06-75-A5-24" received
I	11:38:52.881	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 01-93-D9-CB-C6-6B-37-89-31-5E-A3-9F-8A-ED-B4-D1-13-7D-0A-FD
A	11:38:52.881	"(0x) 01-93-D9-CB-C6-6B-37-89-31-5E-A3-9F-8A-ED-B4-D1-13-7D-0A-FD" received
I	11:38:52.881	Notification received from 00002b10-0000-1000-8000-00805f9b34fb, value: (0x) 02-6E-4D-34-CD-EF-B0-C5-81-F1-E8-7A-1E-E3
A	11:38:52.881	"(0x) 02-6E-4D-34-CD-EF-B0-C5-81-F1-E8-7A-1E-E3" received
```

Perhaps `00002b10-0000-1000-8000-00805f9b34fb` is [temperature](https://developer.tizen.org/community/code-snippet/embed/20776) or [`tempRange`](https://pub.dev/documentation/flutter_web_bluetooth/latest/flutter_web_bluetooth/BluetoothDefaultCharacteristicUUIDS/temperatureRange-constant.html)?

And `00002b11-0000-1000-8000-00805f9b34fb` is [`temperatureStatistics`](https://pub.dev/documentation/flutter_web_bluetooth/latest/flutter_web_bluetooth/BluetoothDefaultCharacteristicUUIDS/temperatureStatistics-constant.html)?

Not sure about packets for volume or battery level (if they're even in the above log dump).

# The mod

I still don't know why the Hidrate team didn't make the puck charge wirelessly to begin with but that's the simplest modification to address most of the biggest shortcomings:

- Qi is a standard. If the charging base is lost/damaged, replacements are not difficult to get and any wireless charging that a user might travel with... will just work with the bottle.
- Wireless charging means no liquid to accidentally get in a USB port or onto pogo pins / slip rings. Electric current through a liquid results in corrosion... which is how my original cable got damaged!
- The bottle can be "in use" while it's charging. Take a sip, place bottle back on charger!

Here is a quick "proof of concept" video taken just before final testing and assembly.

{{< video src="poc_video" controls="yes" >}}

The version shown above is "mark 2". It is simpler to assemble, uses less material and is about 30% thinner relative to the "mark 1" designs.
I put a _ton_ of effort into making the design as approachable and affordable as possible. The idea was to make it easy for average people to make their bottle _better the stock_ in just a few hours.

- Parts can be printed easily and quickly on all but the sloppiest printers.
- Basic hand / power tools are all that are required. At no point do you need an extremely expensive and precise machine to perform some critical step.
- Cheap! Requires a few $ in easy to source parts and pennies in plastic and electricity.

My hope is that the engineers at Hidrate will see that it's not _**that difficult**_ to design an objectively better experience into their products, that it's an anti-consumer dark pattern to needlessly require a proprietary charging cable.

Fingers crossed that their next generation of hardware won't look like this 🤞:

{{<figure name="hidrate_spark_asshole_design">}}

You can find a through set of instructions, printable part files and BOM at the [`kquinsland/hidrate-spark-qi-charging`](https://github.com/kquinsland/hidrate-spark-qi-charging) repo.
If you do follow the instructions and manage to make your bottle charge wirelessly, let me know!

