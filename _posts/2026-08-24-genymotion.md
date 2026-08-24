---
title: How to Root Android 14.0–15.0 and Proxy Traffic to Burp Suite
date: 2025-2-04 03:05:12 +0800
categories: [Blog, Android]
tags: [rooting, genymotion, android]     # TAG names should always be lowercase
image: 
    path: /assets/blog/rooting-android-15-genymotion/img1.png
---

Intercepting traffic on modern Android is harder than it used to be. Since **Android 7**, apps stopped trusting user-installed CA certificates by default, and since **Android 14**, the trusted system CA store itself moved into an immutable APEX module. Add virtualization quirks (this walkthrough uses a Genymotion virtual device) and inconsistent proxy propagation in newer Android versions, and a task that used to be "drop a cert in a folder" now involves several interlocking pieces.

This post documents, step by step, the full process used to root an Android 15 virtual device and get its traffic flowing through Burp Suite including the dead ends, why each fix was necessary, and where the overall approach still falls short.

>Note:
>I used AI in the writing of this blog so be easy on me ;)

## Prerequisites

- Genymotion desktop app installed on the host machine (windows/mac/linux).
- Android 14+ virtual device created on Genymotion desktop application on host machine.
- Burp Suite running on a host machine reachable from the device over the network.
- `adb` installed and working on the host machine.
- Magisk (for root management and module installation) link provided

---

## Step 1 — Root the device with Magisk

Genymotion doesn't ship Android 12.0 upward devices pre-rooted. The rooting method used here was Genymotion's own documented Magisk integration:

Download the Magisk installer package from Genymotion's own tutorial: [Magisk](https://www.genymotion.com/blog/tutorial/magisk-genymotion/)

![genymotion ](/assets/blog/rooting-android-15-genymotion/img2.png)

Drag and drop the package onto the running virtual device  Genymotion's tooling handles the flashing/installation.

![genymotion ](/assets/blog/rooting-android-15-genymotion/img3.png)

Restart the device.

![genymotion ](/assets/blog/rooting-android-15-genymotion/img4.png)

After reboot, the device has both the Magisk app and a working `su` binary, confirmable with:


```shell
└─$ adb shell                                                                
vbox86p:/ $ su
```

Then go over to the Android device immediately and grant adb root request

![genymotion ](/assets/blog/rooting-android-15-genymotion/img5.png)

Confirm root access

```shell
vbox86p:/ # whoami
root
```


### Why `adb root` will lie to you

A natural next step is to try `adb root` to get a root shell without the manual `su` step. On this kind of setup it typically fails:

```shell
└─$ adb root 
adbd cannot run as root because the device is not rooted.
```

This is misleading. `adb root` restarts the `adbd` daemon itself as root, which depends on how `adbd` was compiled into the system image (`ro.secure` / `ro.debuggable` properties) — it has nothing to do with whether Magisk has granted root via `su`. Rooted images grant root through the `su` binary, not through a root-capable `adbd`. As long as `su` → `whoami` returns `root`, the device is properly rooted; `adb root` failing is a non-issue and can be ignored for the rest of this process.

---

## Step 2 — Point Burp's listener at all interfaces

By default, Burp's proxy listener on port 8080 may be bound to loopback only, meaning it only accepts connections from the same machine it's running on. A phone or virtual device on the network can't reach it at all in this state — the connection gets refused outright.

Fix in Burp:

1. **Proxy → Proxy settings** (older Burp: **Options**)
2. Select the `8080` listener → **Edit**
3. Under **Binding**, set **Bind to address** to **All interfaces**

![genymotion ](/assets/blog/rooting-android-15-genymotion/img6.png)


This alone resolves one of the most common causes of "Proxy Server Refused Connection" errors when setting up mobile interception — Burp simply wasn't listening on the interface the device could reach.

---

## Step 3 — Enable invisible proxying

This setting matters specifically because of a later step (transparent iptables redirection) rather than for a standard explicit-proxy setup. When traffic is redirected at the network layer instead of being explicitly configured as a proxy in the app or OS, it arrives at Burp without a proper HTTP `CONNECT` request — Burp has to infer the real destination from the `Host` header instead.

In Burp: **Proxy → Proxy settings → listener → Edit → Request handling** → check **"Support invisible proxying (enable only if needed)"**.

![genymotion ](/assets/blog/rooting-android-15-genymotion/img7.png)


---

## Step 4 — Install a certificate-trust Magisk module

**Android 14+** moved the trusted CA store out of the writable `/system/etc/security/cacerts/` path and into `/apex/com.android.conscrypt/cacerts/` — a read-only, immutable APEX-mounted path. A plain `mount -o remount,rw /system` does not touch this; even as root, writing directly into it fails with `Read-only file system`.

Rather than manually bind-mounting a writable directory over the APEX path every boot (which works but doesn't survive reboots), the more durable fix is a Magisk module built for this exact purpose. This walkthrough used the `MagiskTrustUserCerts` module (also available as [AlwaysTrustUserCerts](https://github.com/NVISOsecurity/AlwaysTrustUserCerts)

Download the module zip from its GitHub releases [AlwaysTrustUserCerts](https://github.com/NVISOsecurity/AlwaysTrustUserCerts).

Push it to the device:

```shell
└─$ adb push AlwaysTrustUserCerts_version.zip /sdcard/Download
AlwaysTrustUserCerts_v1.3.zip: 1 file pushed, 0 skipped. 9.6 MB/s (6808 bytes in 0.001s)
```

In the Magisk app: **Modules → Install from storage** → select the zip → reboot.


![genymotion ](/assets/blog/rooting-android-15-genymotion/img8.png)


![genymotion ](/assets/blog/rooting-android-15-genymotion/img9.png)


![genymotion ](/assets/blog/rooting-android-15-genymotion/img10.png)


This module hooks into boot and automatically promotes any user-installed CA certificate to system-trusted status — reapplying itself on every boot without further intervention.

---

## Step 5 — Export Burp's CA certificate

With Burp's listener already reachable (Step 2), the CA cert can be pulled directly from Burp's built-in cert page:

```shell
└─$ curl -o cert.der http://localhost:8080/cert
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    940 100    940   0      0   9436      0                              0


└─$ ls cert.der 
cert.der

─$ openssl x509 -inform der -in cert.der -out burpcert.pem
                                                                                                                                                                                                                                            
┌──(skywalker㉿kali)-[~/Music]
└─$ adb push burpcert.pem /sdcard/Download
burpcert.pem: 1 file pushed, 0 skipped. 14.6 MB/s (1330 bytes in 0.000s)
```

Note: the certificate is pushed in its original `.pem`/`.der` form here, **not** renamed to the OpenSSL subject-hash format (`<hash>.0`). That naming convention only matters for the legacy manual-bind-mount approach into `/system/etc/security/cacerts/`; installing through Android's own certificate-install UI doesn't need it.

---

## Step 6 — Install the certificate as a user cert

On the device: **Settings → Security → Encryption & credentials → Install a certificate → CA certificate**, then select the pushed `.pem` file.

![genymotion ](/assets/blog/rooting-android-15-genymotion/img11.png)


![genymotion ](/assets/blog/rooting-android-15-genymotion/img12.png)


Reboot the device once. On boot, the Magisk module from Step 4 detects the newly installed user cert and promotes it into the system trust store, so apps that specifically check for system-trusted certs (rather than accepting user certs) will now trust Burp's CA as well.

If you navigate to **Settings → Security → Encryption & credentials → Trusted credentials → System**  You should see **PortSwigger CA**


![genymotion ](/assets/blog/rooting-android-15-genymotion/img13.png)


---

## Step 7 — Clear stale global proxy settings

An earlier approach in this process attempted to set Android's system-wide proxy directly:

```shell
└─$ adb shell settings put global http_proxy 192.168.0.6:8080
```

This looked correct — `settings get global http_proxy` echoed back the value — but traffic still wasn't reaching Burp, and individual apps weren't connecting to the internet at all. Digging into `adb shell dumpsys connectivity` revealed why: the active network's `LinkProperties` had no proxy attached whatsoever, despite the `Settings` database holding the value. On this virtualized Wi-Fi interface, the proxy value written to `Settings.Global` was never being propagated into the live network object that `ConnectivityManager` actually hands to apps — so most apps saw no proxy configured and either connected directly (bypassing Burp) or failed depending on how they were written.

Because this setting was actively wrong rather than just unused, it was explicitly cleared before moving to a different approach:

```bash
adb shell settings put global http_proxy :0
adb shell settings put global global_http_proxy_host ""
adb shell settings put global global_http_proxy_port 0
```

---

## Step 8 — Redirect traffic at the network layer with iptables

Since the OS-level proxy setting couldn't be relied on, the fix was to bypass it entirely: redirect outbound traffic to Burp using `iptables` NAT rules on the device itself. This works regardless of whether the OS proxy setting propagates correctly, and as a side benefit, it also catches apps that ignore Android's system proxy setting altogether (a common issue independent of the bug above).

Clear any previous narrow rules first:

```shell
└─$ adb shell                                           
vbox86p:/ $ su
vbox86p:/ # iptables -t nat -F OUTPUT
vbox86p:/ # 
```

---

## Step 9 — Add the redirect rules

```bash
# Don't redirect traffic already destined for Burp's own IP/port (prevents a redirect loop)
iptables -t nat -A OUTPUT -p tcp -d HOST_LOCAL_IP --dport 8080 -j RETURN

# Don't redirect loopback traffic
iptables -t nat -A OUTPUT -p tcp -o lo -j RETURN

# Redirect everything else on TCP to Burp
iptables -t nat -A OUTPUT -p tcp -j DNAT --to-destination HOST_LOCAL_IP:8080
```

Replace `HOST_LOCAL_IP` with the Host Local IP.


![genymotion ](/assets/blog/rooting-android-15-genymotion/img14.png)


Order matters here — `iptables` evaluates rules top to bottom, so the `RETURN` exceptions must come before the catch-all `DNAT` rule, or every packet (including traffic to Burp itself) gets caught by the first matching rule and looped.

---

## Step 10 — Verify and use

```bash
vbox86p:/ # iptables -t nat -L OUTPUT -n --line-numbers
Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    RETURN     6    --  0.0.0.0/0            10.133.174.148       tcp dpt:8080
2    RETURN     6    --  0.0.0.0/0            0.0.0.0/0           
3    DNAT       6    --  0.0.0.0/0            0.0.0.0/0            to:10.133.174.148:8080

exit   # leave su
exit   # leave adb shell
```

At this point, opening a browser or app on the device generates traffic that appears directly in Burp's HTTP history — with the CA cert trusted from Step 4–6, HTTPS traffic decrypts cleanly with no certificate warnings.

You might get this `Secure Connection Failed` error when you open the browser
Click **Advanced → Accept the Risk and Continue**



![genymotion ](/assets/blog/rooting-android-15-genymotion/img15.png)


![genymotion ](/assets/blog/rooting-android-15-genymotion/img16.png)


Now app traffic can be received directly on burp suite proxy

---

## Limitations of this setup

This process works, but it's worth being explicit about where it's fragile — anyone following this for real assessment work should know the edges:

**Nothing here survives a reboot.** The Magisk cert-trust module does persist (that's its entire purpose), but the `iptables` rules are flushed on every reboot of the device. They need to be reapplied manually after every restart, or scripted into something that runs at boot (e.g. a Magisk `post-fs-data.d` script).

**Only TCP is redirected — UDP is untouched.** Any app or SDK using UDP-based protocols (QUIC/HTTP-3, some telemetry or VoIP traffic, raw UDP protocols) bypasses this setup entirely, since the `iptables` rules only target `-p tcp`. Burp's proxy listener is also fundamentally HTTP/TCP-focused and can't intercept raw UDP regardless. If an app is suspected of using QUIC over UDP/443 specifically, a common workaround is to drop outbound UDP/443 so the app's TLS stack falls back to HTTP/1.1 or HTTP/2 over TCP — but this is a blunt instrument and can break apps that have no TCP fallback.


**Certificate pinning is not addressed here.** Any app implementing its own certificate/public-key pinning will reject Burp's CA outright, regardless of whether it's trusted at the OS level. That requires separate handling (patching the app, using a Frida script to bypass pinning at runtime, etc.) and is out of scope for this setup.


**This is virtualization-specific in places.** The proxy-propagation bug in Step 7 was diagnosed specifically on a Genymotion virtual Wi-Fi interface (`wlan0`) that doesn't go through the normal `WifiConfiguration`/`WifiManager` flow a physical device would. On real hardware, setting the proxy through Android's Wi-Fi settings UI typically does correctly populate `LinkProperties`, and the iptables workaround may not be necessary at all — it's worth trying the standard per-network manual proxy setting first on physical devices before assuming this same failure mode applies.

**Root persistence depends on Magisk's own stability across Android version boundaries.** This was tested on Android 15; Magisk's compatibility with a given Android build can shift with major version updates, so the same module/steps aren't guaranteed to work identically across the entire 12.1–15.0 range without verification on each specific version.

---

## Summary

Getting Burp visibility into modern Android traffic is no longer a single step — it's three separate problems stacked together: gaining root, getting a CA certificate trusted by an OS that increasingly resists it, and getting traffic to actually reach the proxy despite settings that may silently fail to apply. Each of the steps above exists to solve one specific point of failure in that chain, and understanding _why_ each one is needed makes it much faster to diagnose when one part of the setup breaks on a different device or Android version.

