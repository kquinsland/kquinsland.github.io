# Unbrick a rievtech PLC after failed firmware upgrade


This is "reference" post for anybody else that happens to have this same very specific problem. 

I was looking for a way to incorporate some of the _many_ cheap / industrial grade sensors from AliExpress with Home Assistant.
Long story short: almost everything electronic in the industrial space uses [Modbus](https://modbus.org/) to communicate, typically with a [PLC](https://en.wikipedia.org/wiki/Programmable_logic_controller). While Home Assistant [does have support](https://www.home-assistant.io/integrations/modbus/) for the Modbus protocol, but wanted to use a PLC that could manage the sensors directly and expose the values over the network in a more standard format; MQTT.

Turns out, quite a few PLCs come with network interfaces and can speak MQTT now!

So I picked up a [`PR-18DC-DAI-R-N`](https://www.rievtech.com/PR-18DC-DAI-R-N-pd72286477.html) PLC from the 'budget friendly' supplier rievtech. The PLC was put into service in 2018 and never updated; the firmware version it was running was almost certainly below `150` but I didn't record the specific version.

{{< figure name="plc_stock" >}}

Hoping to squash a small bug, I chose to upgrade the firmware to version `152` which was released at the end of 2020 and somehow managed to brick the device 🤦.

I am _speculating_ but it looks like something significant changed around firmware version `150`. If you try to update a device from a version prior to `150` - like I did - something in memory is not properly migrated to the format required by versions after `150` and this causes the update process to fail before completion:


{{< figure name="update_fail-feature" >}}

That failure message came from the [Update_Net_V152_20201205.zip](https://www.rievtech.com/phoenix/admin/download?fileId=jGUKpAvYWcQE&dp=GvUApKfKKUAU) file which is meant for the ethernet equipped PLCs in the `PR` line. The failure apparently soft bricks the PLC.

{{< figure name="plc_brick" >}}


The PLC seemed 'alive' as I could still see `arp` packets coming from it's ethernet port on boot and the web server seemed to accept my connection but never return any data. Even after restarting the PLC, the firmware updater was able to open a new connection and flash the firmware... always failing at the same spot: 99.17%.

Clearly _something_ is alive and well inside the PLC... If I can connect to whatever *is* running and convince that process to update the flash then I have a good shot to do 'solderless' recovery. Absolute worst case, I buy another one and clone the flash memory from the working one onto the bricked one.



## Recovery process

Turns out, their support team has a file ready to go for this exact problem! I guess I'm not the first person. Really wish they'd publish this file and/or procedure on their website. I could have recovered from my failure in minutes with a quick google search and saved myself a few hours of trouble shooting before drafting a support ticket... not to mention the days of waiting for a reply. Oh well. That's why _I'm_ documenting it.

I was given a 2.6MB file named `UpdateFail_PR-18DC-DAI-R-N_V150.zip`. After extracting, it looks a lot like their _regular_ firmware update archives, just much smaller. I suspect that the archive is much smaller because it's only got flash images for a few affected models/devices. In any event, the archive comes with a `doc` file that I couldn't open and was able to eventually convert it to a `pdf` which is also attached to this post.

In short:

- Connect the PLC using their [programming cable](https://www.rievtech.com/USB-Cable-pd67130.html)
- Open the COM port and 'prepare' the device. This takes only a few seconds
- Click the update button. In my case, I got a failure telling me to `Please power off the PLC first, and then power on again!"
- Power cycle the PLC, keeping the update program and COM port open
- Click `Start`. The process of flashing version `150` started and took several minutes to finish


This left me with a *completely working* PLC 🥳! I was able to access the web interface and use the LCD screen. I was _then_ able to use the network update tool to the firmware version released recently; version `152`.


So there you go. If a failed network update leaves your `PR` series PLC in a mostly-not-working state, there is hope!

{{< figure name="recovery_ok" >}}


## Files

{{< bundle-files >}}

