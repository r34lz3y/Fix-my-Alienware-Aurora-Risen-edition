<div align="center">
<img src="assets/hdr-toolkit.svg" alt="USB Toolkit" width="100%">
</div>

Ten tools, three groups. Group A boots the machine when Windows won't. Group B tells you whether the hardware is healthy. Group C finds what's stealing your CPU.

Everything here is free for personal use. **Download links are directly under each tool — always grab from the official source**, because diagnostic utilities are a favourite target for repackaged installers carrying adware.

[← Back to README](README.md) · [Diagnostics →](DIAGNOSTICS.md) · [Fixes →](FIXES.md)

<img src="assets/hdr-rescue.svg" alt="Rescue Media" width="100%">

Bootable images. These run *instead of* your installed Windows, which is what makes them useful — a drive too sick to boot from is still readable from here, and repairs that can't run on a live system run fine offline.

### Ventoy

The foundation. Ventoy prepares the USB drive so it can boot **multiple ISOs directly** — you install it once, then drag ISO files onto the drive like a normal flash drive. No reflashing, no one-ISO-per-stick.

> **Download:** https://www.ventoy.net/en/download.html

Grab the `ventoy-x.x.xx-windows.zip` package, run `Ventoy2Disk.exe`, select your USB drive, click Install. **This erases the drive**, so use an empty one. After that the drive has two partitions — you only ever touch the big one.

---

### Hiren's BootCD PE (x64)

A full Windows PE live environment that **boots entirely into RAM**. Once it's loaded you can pull the USB stick out and it keeps running. It ships with hundreds of pre-installed diagnostic, recovery, hardware-testing, partition and malware tools, so if you only have room for one rescue ISO, this is the one.

> **Download:** https://www.hirensbootcd.org/download/

The ISO is roughly 3 GB. Because it runs from RAM and never touches the installed Windows, it's the right place to do anything destructive — DDU driver wipes, partition work, or recovering files off a drive that's failing.

---

### Windows 11 Media Creation Tool

The official Microsoft ISO. Essential for running **offline Startup Repair**, getting to a Command Prompt outside of Windows to run repair commands against the installed system, or doing a clean reinstall when everything else has been ruled out.

> **Download:** https://www.microsoft.com/software-download/windows11

Use the tool to produce an ISO file (not a USB — you want the ISO so Ventoy can boot it alongside everything else). At the install screen, **Repair your computer → Troubleshoot → Command Prompt** is the door to the offline repairs in [FIXES.md](FIXES.md).

---

### MemTest86

Bootable RAM tester. Bad memory produces exactly the symptoms that look like software problems — random hangs, inexplicable crashes, corruption that comes back after you fix it. This is how you rule it out on the 16 GB in the machine.

> **Download:** https://www.memtest86.com/download.htm

Let it run **at least four full passes**, which realistically means overnight. One pass proving clean means very little; memory faults are often intermittent and only show up under sustained hammering. **A single error is a failure** — there's no acceptable error count. Pull one stick and retest to identify which module is bad.

<img src="assets/hdr-hardware.svg" alt="Hardware Diagnostics" width="100%">

Portable `.exe` files — no installation, no registry entries. Drop them in a `\Portable` folder on the Ventoy drive and run them straight from the stick, either inside Windows or inside the Hiren's environment.

### CrystalDiskInfo

Immediate **S.M.A.R.T. health check** for your SSD or HDD. This is the fastest way to answer the most important question in the whole process. If your boot drive has reallocated sectors or a high read-error rate, Windows will freeze continuously while it waits on I/O responses that are taking retries to complete — and no amount of software tuning touches that.

> **Download:** https://crystalmark.info/en/download/

Take the **portable** edition. The big status block at the top says `Good`, `Caution`, or `Bad`. Anything but `Good` means back up your data now — see [DIAGNOSTICS.md](DIAGNOSTICS.md#step-02--drive-health) for which attributes actually matter.

---

### HWiNFO64 (Portable)

Live sensor monitoring for every temperature, voltage, clock and fan in the machine. **Alienware Aurora cases have notoriously tight thermal constraints.** If the CPU AIO liquid cooler or its fan fails, the Ryzen 5 5600X hits 95 °C almost instantly and throttles hard enough to make the PC feel unusable — while Task Manager shows nothing wrong at all.

> **Download:** https://www.hwinfo.com/download/

Take the **portable** ZIP. Launch in **Sensors-only** mode. The columns to read are Current, Minimum and Maximum — the Maximum column is where a cooling failure confesses, because it captures the spike you weren't watching for.

---

### CrystalDiskMark

Benchmarks actual read/write throughput. Where CrystalDiskInfo reports what the drive *says* about itself, this measures what it actually delivers — which catches drives that report `Good` while performing like a decade-old spinner, whether from a failed SSD cache, thermal throttling, or a controller on its way out.

> **Download:** https://crystalmark.info/en/download/

Same download page as CrystalDiskInfo; portable edition again. Run it on a drive with at least 10 GB free, and **close everything else first** or you're benchmarking the background noise.

<img src="assets/hdr-cleaners.svg" alt="System Auditors" width="100%">

Once the hardware is cleared, these find what's actually consuming the machine.

### Sysinternals Autoruns

Task Manager's Startup tab shows you the surface. Autoruns shows you **everything** — hidden startup registry keys, scheduled tasks, services, browser extensions, codecs, boot drivers, and every other place a program can arrange to run without asking. It is the difference between the ten entries Windows admits to and the two hundred that exist.

> **Download:** https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns

Microsoft-signed, portable, no install. Turn on **Options → Hide Microsoft Entries** immediately — it cuts the list by roughly 80% and leaves only third-party code, which is where your problem lives. Uncheck to disable rather than deleting, so you can put it back.

---

### Malwarebytes AdwCleaner

Lightweight portable scanner aimed squarely at adware, potentially unwanted programs, and browser hijackers — the category conventional antivirus deliberately ignores because it's technically consented-to software. It is also the category most likely to be quietly eating your CPU.

> **Download:** https://www.malwarebytes.com/adwcleaner

Single `.exe`, no installation. Scan takes a few minutes and it will want to reboot to finish cleaning. Review the results list before you accept it — it's aggressive, and it occasionally flags something you actually use.

---

### Display Driver Uninstaller (DDU)

If the system hangs when launching processes or rendering graphics, a **corrupted AMD or NVIDIA display driver** is a strong suspect. Windows' own uninstaller leaves registry keys and files behind, and a reinstall on top of that debris reproduces the same fault. DDU wipes display drivers completely clean.

> **Download:** https://www.wagnardsoft.com/display-driver-uninstaller-ddu

**Run it in Safe Mode**, and **disconnect from the internet first** — otherwise Windows Update will helpfully reinstall the broken driver before you can install the good one. Have your replacement driver downloaded and sitting on the desktop *before* you start. Full procedure in [FIXES.md](FIXES.md#display-driver-wipe).

<img src="assets/divider.svg" alt="" width="100%">

## Quick reference

| Tool | Type | Purpose | Link |
|:-----|:-----|:--------|:-----|
| Ventoy | Utility | Multi-ISO bootable USB | [ventoy.net](https://www.ventoy.net/en/download.html) |
| Hiren's BootCD PE | ISO | Live WinPE rescue environment | [hirensbootcd.org](https://www.hirensbootcd.org/download/) |
| Windows 11 MCT | ISO | Offline repair & clean install | [microsoft.com](https://www.microsoft.com/software-download/windows11) |
| MemTest86 | ISO | RAM fault testing | [memtest86.com](https://www.memtest86.com/download.htm) |
| CrystalDiskInfo | Portable | S.M.A.R.T. drive health | [crystalmark.info](https://crystalmark.info/en/download/) |
| HWiNFO64 | Portable | Temps, voltages, fan speeds | [hwinfo.com](https://www.hwinfo.com/download/) |
| CrystalDiskMark | Portable | Drive speed benchmark | [crystalmark.info](https://crystalmark.info/en/download/) |
| Autoruns | Portable | Full startup & service audit | [learn.microsoft.com](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns) |
| AdwCleaner | Portable | Adware / PUP removal | [malwarebytes.com](https://www.malwarebytes.com/adwcleaner) |
| DDU | Portable | Clean display driver removal | [wagnardsoft.com](https://www.wagnardsoft.com/display-driver-uninstaller-ddu) |

<div align="center">

[← README](README.md) · [Diagnostics →](DIAGNOSTICS.md) · [Fixes →](FIXES.md)

`END OF LINE`

</div>
