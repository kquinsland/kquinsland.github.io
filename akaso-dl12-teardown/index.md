# AKASO DL12 Dashcam Teardown

<!-- markdownlint-disable-file MD002 -->
# AKASO DL12 Dashcam Teardown

{{<figure name="feat-product01">}}

This was my dashcam for a while until the front camera stopped working.
It was a good dashcam while it lasted but there are better options now... so let's take it apart!

Ultimately, it's a highly integrated consumer-grade bit of electronics.
Everything's built down to a price and anything that didn't need to be done ... wasn't.
There's not much to see and not much to say so this is a ["two minute teardown" ]({{<relref "tags/two-minute-teardown">}}).

## Teardown

There are 4 screws holding the case together. They are hidden under the rubber feet.

{{<figure name="td02">}}

Once the screws are removed, the case comes apart easily to reveal a mostly empty shell.
The LiPo battery is an interesting choice given the heat that can build up in a car.
As far as I know, this is only needed for the few seconds after power loss so the camera can save the current recording and shut down cleanly.

{{<figure name="td05">}}

Close up of the rear of the main PCB shows a few test points and populated but unused connectors.

I love the `TV_IN` silkscreen for the rear camera connector. Almost makes me wonder if this was a re-purposed schematic from a different product.

Why there's a dedicated `RTC_BAT` connector and not sipper power from the main battery is a mystery to me.
The unit does have a GPS module so there's really no need for an RTC battery... just use the GPS time!

{{<figure name="td03">}}

The 'forward facing' side of the PCB has a few more populated/un-used connectors and a heat-spreader over the main CPU.
The two push buttons in the top left are not accessible from the outside of the case so there's probably something interesting that happens when they're pressed.

{{<figure name="td01">}}

Removing it reveals a HiSilicon Hi3556 SoC, EEPROM and what is probably the image processor for the rear camera.
The large unpopulated footprint in the top right looks like it's meant for a radio module.
Not sure if it's for a wireless link to the rear camera or for something like a [TPMS sensor](https://en.wikipedia.org/wiki/Tire-pressure_monitoring_system) or possibly WiFi for a phone app.

{{<figure name="td06">}}

{{<figure name="td04">}}

## PCB Markings

AKA SEO optimization 😉

Main PCB is marked:

  > 77-56V200(20) V3.411
  >
  > 6105
  >
  > V200-AMWG-V3.411
  > 2019.12.03

And on the rear:

  > F02Z1B
  > F15Z1B

The main CPU is [meant for consumer cameras](https://silicondevice.com/file.upload/images/Gid1551Pdf_Hi3556%20V100%20HD%20Mobile%20Camera%20SoC%20Brief%20Data%20Sheet.pdf) and is marked:

  > Hi 3556
  >
  > RBCV200
  >
  > CP9881938
  >
  > 1939-CN

The LCD is marked:

  > XTY TB118-I4018E30A-01
  >
  > T B118-45a164200402-J

I can't find a datasheet for what I suspect is the rear-camera processor but it's marked:

  > pixelplus
  >
  > PR2020
  >
  > S3CWN
  >
  > KAG2006

## Firmware

Just for giggles, I dumped the firmware from the SPI flash to get an idea of what software powers this device.

```shell
❯ flashrom --force -c "GD25B128B/GD25Q128B" --verbose --programmer ch341a_spi --read oem.bin
<...>
Reading flash... 
<...>
```

I did run `strings` on it and there's some mildly interesting stuff in there but nothing interesting enough to warrant a deep dive.

`binwalk` didn't have any trouble extracting things but I'm not going to spend any time reversing this firmware.

It looked like the usual "bootloader and two identical copies of the application firmware" setup.
Taking a look at what I think is the main application:

```shell
Scan Time:     2024-08-02 16:52:43
Target File:   /home/karl/scratch/dash-fw/_oem.bin.extracted/media_app.bin
MD5 Checksum:  557e5a7578dfb20ed8328888479bc973
Signatures:    436

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
510070        0x7C876         JBOOT STAG header, image id: 10, timestamp 0x3AF8E51B, image size: 454878491 bytes, image JBOOT checksum: 0xE51B, header JBOOT checksum: 0x3233
2689033       0x290809        Certificate in DER format (x509 v3), header length: 4, sequence length: 5380
3038273       0x2E5C41        Certificate in DER format (x509 v3), header length: 4, sequence length: 5380
3828721       0x3A6BF1        Certificate in DER format (x509 v3), header length: 4, sequence length: 13692
4056525       0x3DE5CD        ESP Image (ESP32): segment count: 15, flash mode: QUIO, flash size: 2MB, entry address: 0x40006a7
4732974       0x48382E        eCos RTOS string reference: "eCostInSystem"
4732991       0x48383F        eCos RTOS string reference: "eCostInSystem"
4771774       0x48CFBE        TIFF image data, big-endian, offset of first image directory: 8
4773180       0x48D53C        TIFF image data, big-endian, offset of first image directory: 8
4980260       0x4BFE24        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/components/ipcm/ipcm/message/ipcm.c
4982320       0x4C0630        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/components/ipcm/ipcm/message/ipcm_data.c
4983576       0x4C0B18        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/kernel/base/ipc/los_mux.c
4983960       0x4C0C98        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/kernel/base/core/los_task.c
4986708       0x4C1754        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/kernel/base/mem/mem_bestfit/los_memory.c
4998096       0x4C43D0        Unix path: /usr/local/etc/zoneinfo
4999016       0x4C4768        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/lib/libc/src/time/time64.c
5000336       0x4C4C90        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/compat/posix/src/pthread_mutex.c
5001228       0x4C500C        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/fs/vfs/inode/fs_inode.c
5006876       0x4C661C        Unix path: /home/user04/work/kashite/3556-11.8-v1013-AKASO/osdrv/platform/liteos/platform/bsp/board/hi3556v200/include/hisoc/spi.h
5194272       0x4F4220        ESP Image (ESP32): segment count: 11, flash mode: QUIO, flash speed: 40MHz, flash size: 1MB, entry address: 0x4003d1
5363994       0x51D91A        ESP Image (ESP32): segment count: 1, flash mode: QUIO, flash speed: 40MHz, flash size: 1MB, entry address: 0x0
```

