# Using new Lets Encrypt intermediate chain with SkyHole



If you somehow missed it, one of the certificates used by Lets Encrypt chain of trust [expired](https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/) this week.
As expected, things broke.... including my private, filtered DNS over TLS server - [SkyHole](https://github.com/kquinsland/skyhole/). Below is a condensed form of my notes to create the exact document that I wish I had while trying to triage broken DNS on my phone.

**TL;DR:** Implement solution 3 from [this](https://www.openssl.org/blog/blog/2021/09/13/LetsEncryptRootCertExpire/) post.


---


**Note:** After releasing the initial version of SkyHole, I re-factored most of the code to eliminate the dependency on Docker. This was to make the project easier to deploy on resource constrained hardware.
At the time, I was working with [SaltStack](https://saltproject.io/) a lot and took the opportunity to re-do the entire thing as a salt state for a bit of practice. The _exact_ steps and commands shown below are unique to my particular instance. Use them as *guidance* for fixing an issue with the publicly released version of SkyHole.


# Symptoms


My current daily driver runs Android 11. When configured to use a [private DNS server](https://android-developers.googleblog.com/2018/04/dns-over-tls-support-in-android-p.html), Android essentially behaves as if you've turned WiFi/Cell data off if there is any issue when talking to the DoT server.
While this 'fail private' approach is commendable, the lack of debug info in the UI is not; no details are given about the failure other than a generic 'the private dns server could not be reached' message.

I have seen this behavior once before when the certificate renewal timer failed to fire off... except I implemented email based notifications after that incident and had recently received a notification from the renewal script.

Just to be sure, I checked the `notAfter` in each certificate and they all had plenty of life left:

```shell
me@dot-host:/etc/coredns/tls# gawk 'BEGIN { pipe="openssl x509 -noout -subject -dates"} \
>   /^-+BEGIN CERT/,/^-+END CERT/ { print | pipe }
>   /^-+END CERT/                 { close(pipe); printf("\n")}  ' chain.pem
subject=CN = dot.my-test-domain.tld
notBefore=Oct  1 01:50:58 2021 GMT
notAfter=Dec 30 01:50:57 2021 GMT

subject=C = US, O = Let's Encrypt, CN = R3
notBefore=Sep  4 00:00:00 2020 GMT
notAfter=Sep 15 16:00:00 2025 GMT

subject=C = US, O = Internet Security Research Group, CN = ISRG Root X1
notBefore=Jan 20 19:14:03 2021 GMT
notAfter=Sep 30 18:14:03 2024 GMT
```
Borrowed that command from [this](https://serverfault.com/questions/541262/checking-the-issued-and-expiry-dates-for-the-certificates-involved-a-certificate) post.


Looking for more information, `adb` yielded something:

```shell
❯ adb logcat | grep resolv
09-30 21:28:12.995   962 15448 W resolv  : Validating DnsTlsServer 12.34.56.78 with mark 0xf0084
09-30 21:28:13.085   962 15448 W resolv  : SSL_connect ssl error =1, mark 0xf0084: No such file or directory
09-30 21:28:13.085   962 15448 W resolv  : TLS Handshake failed
09-30 21:28:13.085   962 15448 W resolv  : query failed
09-30 21:28:13.085   962 15448 W resolv  : validateDnsTlsServer returned 0 for 12.34.56.78
```

The `No such file or directory` message came from [here](https://android.googlesource.com/platform/packages/modules/DnsResolver/+/refs/heads/master/DnsTlsSocket.cpp).

Google uses their own fork of openSSL in Android so spent some time trying to figure out what an error code of `1` means in the openSSL project.
I _think_ `1` is [`SSL_ERROR_SSL`](https://github.com/OneSignal/openssl/blob/main/include/openssl/ssl.h#L1168) but that seems to be a relatively ['generic'](https://www.openssl.org/docs/man1.1.1/man3/SSL_get_error.html) error.
Furthermore, in context of `No such file or directory` ... it makes even less sense.

Oh well. So much for that theory. From this point on, I'm treating all of the TLS connection stuff as a black box.

# Investigating

I was able to confirm that the TLS certificates on the skyhole instance had not expired and the intermediate chain was not using any of the depreciated certificates.
I had also not made any changes to the skyhole instance in close to a year and `kdig` didn't throw any warnings when querying against the DoT server.
I could see the manual query from `kdig` in the DNS query/filter logs ... so it seemed like the problem was not in either the TLS portion or the DNS portion.

That left Android as the culprit.

But I was also fairly sure that Google hadn't changed anything on the phone w/r/t how the DoT client worked...🤔

If in doubt, turn to the wires!

I ran `tcpdump` on the skyhole instance and did notice traffic from Android that was _not_ showing up in the DNS server logs.
I compared the traffic with a working manual query from `kdig` and noticed that the traffic from the Android client stopped shortly before where the `kdig` traffic would have turned into a regular DNS query.

So the problem was happening during the TLS setup. Whatever Android was choking on was happening before any DNS queries were sent.

I quickly configured the phone to use a known good / working DNS over TLS server and it was immediately accepted. The triumphant `logcat` output confirmed that everything on the TLS layer has happy: ` W resolv  : Validation success`.

Android worked instantly when configured to use a different server but immediately failed when used with the skyhole instance. This points to a problem on the server.

Nothing about the skyhole instance had changed and the certificates that it was offering up were totally valid; other clients worked w/o issue. This points to a problem on the phone.

Even though the certificates appear fine, the timing with the recent Lets Encrypt certificate expiration is too suspicious 🤨.

Looking for another data point, I moved to a different computer with a different version of openSSL installed.

```shell
❯ openssl version
LibreSSL 2.8.3

❯ openssl s_client -connect dot.my-test-domain.tld:853 -servername dot.my-test-domain.tld |
    openssl crl2pkcs7 -nocrl -certfile /dev/stdin |
    openssl pkcs7 -print_certs -noout -text |
    egrep 'not(Before|After)'
depth=1 O = Digital Signature Trust Co., CN = DST Root CA X3
verify error:num=10:certificate has expired
notAfter=Sep 30 14:01:15 2021 GMT
verify return:0
depth=1 O = Digital Signature Trust Co., CN = DST Root CA X3
verify error:num=10:certificate has expired
notAfter=Sep 30 14:01:15 2021 GMT
verify return:0
depth=3 O = Digital Signature Trust Co., CN = DST Root CA X3
verify error:num=10:certificate has expired
notAfter=Sep 30 14:01:15 2021 GMT
verify return:0
```

Huh. That sure looks like a problem!

After a [bit](https://www.mail-archive.com/openssl-users@openssl.org/msg90068.html) of google, I found that the different versions of openSSL (and their forks...) behave differently when validating certificate chains:


> The currently recommended certificate chain as presented to Let’s Encrypt ACME clients when new certificates are issued contains an intermediate certificate (ISRG Root X1) that is signed by an old DST Root CA X3 certificate that expires on 2021-09-30. In some cases the OpenSSL 1.0.2 version will regard the certificates issued by the Let’s Encrypt CA as having an expired trust chain.
>
> Most up-to-date CA cert trusted bundles, as provided by operating systems, contain this soon-to-be-expired certificate. The current CA cert bundles also contain an ISRG Root X1 self-signed certificate. This means that clients verifying certificate chains can find the alternative non-expired path to the ISRG Root X1 self-signed certificate in their trust store.
>
> Unfortunately this does not apply to OpenSSL 1.0.2 which always prefers the untrusted chain and if that chain contains a path that leads to an expired trusted root certificate (DST Root CA X3), it will be selected for the certificate verification and the expiration will be reported.


[Source](https://www.openssl.org/blog/blog/2021/09/13/LetsEncryptRootCertExpire/).

That would certainly explain the behavior I observed when checking the skyhole certificates on the second computer.
I don't know _exactly_ what version of OpenSSL the BoringSSL in my phone is based off of, but, assuming that it's got the same bug as OpenSSL 1.0.2, that would explain everything.

The openSSL blog post pointed out three possible fixes; two of which are applied client side.
My phone is not rooted so I just assumed that I would have access to the portions of the file system needed for a 'client side' fix. That left the third solution; use a different intermediate chain.

I was not aware that there was an alternate intermediate chain for Lets Encrypt. I didn't even know that was a thing let alone _why_ somebody would do that.

# Alternate Intermediate

Turns out, it's a very clever trick meant to [prevent Lets Encrypt certificates from breaking on _older_ versions](https://letsencrypt.org/2020/12/21/extending-android-compatibility.html). Ironic that a 'newer' android device got screwed in the process 😬.

The two valid chains of trust for Lets Encrypt certificates look like this:

{{< figure name="le-compat" >}}


With the help of [this](https://github.com/certbot/certbot/issues/8577) GitHub issue, the revised CertBot `cli.ini` file becomes: 

```shell
me@dot-host:~# cat /etc/letsencrypt/cli.ini | grep chain
preferred-chain='ISRG Root X1'
```

After running certbot with the new config, `logcat` shows success: 


```shell
❯ adb logcat | grep resolv
10-01 09:41:01.190   962  7711 W resolv  : Validating DnsTlsServer 12.34.56.78 with mark 0xf0084
10-01 09:41:01.734   962  7711 W resolv  : validateDnsTlsServer returned 1 for 12.34.56.78
10-01 09:41:01.734   962  7711 W resolv  : Validation success
```

Android no longer shows an unhelpful "can't connect" message and I can see DNS queries being filtered! 

🎊

