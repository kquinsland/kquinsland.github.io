# Amcrest IP Cameras: security isn't a feature, it's a punchline


This is part rant, part "reference" for anybody else that's struggling to get their Amcrest IP Camera to work with [Home Assistant](https://www.home-assistant.io/integrations/onvif/) via ONVIF. Skip to [TL;DR]({{< relref "#tldr">}}) for a working Home Assistant config.


Briefly, [ONVIF](https://www.onvif.org/profiles/) is an industry group that maintains a set of standards to allow for interoperability between IP Cameras and related devices from multiple vendors. One set of protocols so your cameras from `$vendorA` will work with with the recording/analytics software from `$vendorB` which can then pipe events into software from `$vendorC`.
How ONVIF works and how it's implemented are beyond the scope of this rant, but, like most standards that haven't aged well, [SOAP](https://en.wikipedia.org/wiki/SOAP) is involved. 🤮.


The IP Camera in question is the `Amcrest IP4M-1051`, though it does appear that other devices from Amcrest are effected by some of the same issues I'll detail below. 
Additionally, Amcrest appears to be a budget brand offering cheaper versions of [DaHua](https://en.wikipedia.org/wiki/Dahua_Technology) hardware, so the firmware Amcrest uses likely originates from DaHua.

The hardware is pretty good for the price. About $80 and you get a camera with a pretty wide field of view, 4K resolution, automatic IR leds/filter, PTZ servos, 10/100 Ethernet *and* 2.4/5ghz WiFi. The software, however is scarry 😱.


I've only spent a few hours with the camera so far, but here's an incomplete list of the security-related issues i've noticed in that time:

- There's no way to require the web interface be served over HTTPS. You can turn on HTTPS support, but you can't turn of HTTP support!
- ONVIF runs over HTTP ... on port 80!
- Only the ADMIN user is supported for ONVIF
- Passwords are limited to 32 characters in length... but the web UI won't inform you about this.


### Firmware

Before anybody asks, yes, I did update the firmware to the latest version immediately after unboxing.
As of writing, the latest firmware available is version `V2.620.00AC000.3.R.20191218` which can be downloaded [here](https://sup-files.s3.us-east-2.amazonaws.com/Firmware/IP4M-1051/Amcrest_IPC-AWXX-V2-Rhea_Eng_NP_AMCREST_V2.620.00AC000.3.R.20191218.bin).


{{< figure name="latest_firmware" >}}



In the unlikely event that anybody sees this post and somehow gets Amcrest to correct their issues, the versions of software on the affected cameras are:

```txt
Software Version:       V2.620.00AC000.3.R, Build Date: 2019-12-18
WEB Version:            3.2.1.619604
ONVIF Version:          16.12(V2.4.1.513183)
```


## Web UI

Nothing shocking about a networking equipped, consumer-grade bit of electronics with a web based management interface. The only reason I'm noting it here is because I can't find a way to
force the web UI to be served over HTTPS 😢.

{{< figure name="port_conflict" >}}

And the `HTTPS` tab has no way to disable the web server on port 80 or at least do a simple redirect to the HTTPS URL 😞.

{{< figure name="https_config" >}}

I suspect that the next issue might have something to do with the 'always on' HTTP server... 🤔


## ONVIF over port 80


The inciting 'incident' for this post was a 'generic' error from Home Assistant while trying to connect the camera to HA.
While going through the config flow, I'd get a super useful `an unknown error occurred` message 🙃.

After a bit of digging through verbose HA logs, I started to suspect that the default port of `5000` was not the correct port despite the port being 'open' on the camera:

```bash
$ nmap -p1-6553 cameraIPv4Here
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-27 16:14 PST
Nmap scan report for cameraIPv4Here
Host is up (0.010s latency).
Not shown: 6547 closed ports
PORT     STATE    SERVICE
53/tcp   open     domain
80/tcp   open     http
123/tcp  filtered ntp
443/tcp  open     https
554/tcp  open     rtsp
5000/tcp open     upnp

Nmap done: 1 IP address (1 host up) scanned in 2.21 seconds
```

Sure enough, Home Assistant found a responsive ONVIF endpoint on port 80. Repeated testing showed that port 80 is the ONLY port with a responsive ONVIF endpoint 🤦.


So yeah. ONVIF traffic, just like the web UI traffic, can be sniffed by anybody between the client and the camera. That's not OK for a device that's intended to be used for security.

So this little setting is basically useless:

{{< figure name="onvif_auth" >}}

That screenshot is from the PDF file [here](https://drive.google.com/file/d/1X-f5xH4aSjhd4vXpIT9yPN_y9-LSgVWN/view) which has this revision info:

```
Amcrest IP4M-1051B / IP4M-1051W
4MP ProHD Indoor Wi-Fi Camera

User Manual

Version 1.0.3
Revised April 4th, 2019
```

## ONVIF only supports authentication with the Admin user


After discovering that ONVIF only 'works' over port 80, I took a moment to get over my disappointment and then tried to set up a second user with minimal authority on the camera; read only and limited permission to move the camera around.

Except I was never able to get HA to successfully connect to the camera when using the secondary user. Home Assistant still failed to connect to the camera over ONVIF even after elevating the secondary user to the privilege level of an admin:


{{< figure name="admin_user" >}}


My secondary user permissions:

{{< figure name="ha_user" >}}


As Home Assistant really does not expose a ton of debugging information, I tried the secondary user credentials with the [`Onvifer`](https://play.google.com/store/apps/details?id=net.biyee.onvifer&hl=en_US&gl=US) app. The debugging logs from that app confirmed that only the `admin` credential set 'worked' with ONVIF.

After a bit more research, it turns out that I'm not the first person to discover this:


{{< figure name="support_1" >}}

{{< figure name="support_2" >}}

Now i'm embarrassed that it took me so long to discover an issue that's been documented since AT LEAST summer of 2019! 😳



### Password length

Can we *please* [stop putting ceilings on password length](https://security.stackexchange.com/questions/33470/what-technical-reasons-are-there-to-have-low-maximum-password-lengths)?

Or, at least if you're going to set an upper bound on password length, please make sure that:

- You're consistent about enforcing the length
- You tell the user when their password is longer than allowed.

The very first time the default credentials are used to sign into the camera, you're prompted to change the default password. Good! Only problem: the password generated by my password was, apparently, longer than the camera supports. The new admin password was _**silently** truncated_ as I was logged in.

I didn't realize that the password stored in the manager wouldn't let me in until after the camera applied it's FW update and rebooted. After a factory reset, I figured out what happened.

The `maxlength` property of the login form differs from the password set form. In any case, there's no warning that the input exceeds the length!

{{< figure name="pw_login" >}}

{{< figure name="pw_reset" >}}


*sigh*

## TL;DR:

- It's almost 2021, but we still have IP Cameras shipping with 2011's security issues. **Absolutely** do not expose your IP Cameras directly to the internet or otherwise allow them to contact the internet. Keep cameras on their _own_ dedicated and isolated network and consider the admin credentials for the camera to be compromised. Seriously re-consider using WiFi for your cameras.

- If you're trying to use the [Home Assistant ONVIF integration](https://www.home-assistant.io/integrations/onvif/) to control an `Amcrest IP4M-1051`, you must use the `admin` credentials and change the port from `5000` to `80`.


The first bit of documentation you see when unboxing the camera is this little guy:

{{< figure name="security_advisory" >}}

which reads:

```
IMPORTANT SECURITY NOTICE

Please make sure your device has the
latest firmware as listed on:

www.amcrest.com/firmware-subscribe

Never use the default password for
your device. Always ensure your
password contains a combination
of lowercase and uppercase
characters and numbers.
```

A for effort, though, Amcrest! 👏🌟

🙃

