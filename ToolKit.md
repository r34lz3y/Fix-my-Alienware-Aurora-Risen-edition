<div align="center">
<img src="assets/hdr-fixes.svg" alt="Repair Procedures" width="100%">
</div>

The actual repairs. Each one assumes you got here from [DIAGNOSTICS.md](DIAGNOSTICS.md) with a measurement pointing at it — **don't apply these speculatively.** Changing things you haven't measured is how a slow computer becomes a broken one.

[← Diagnostics](DIAGNOSTICS.md) · [Toolkit](TOOLKIT.md) · [README](README.md)

<img src="assets/divider.svg" alt="" width="100%">

## Thermals

**Applies when:** HWiNFO64 shows idle temps above 70–80 °C, or the CPU hits 95 °C within seconds of load.

The Aurora chassis is tight, and its cooling has a small number of failure modes. Work them cheapest-first.

### 1. Dust

Power off, unplug, open the side panel. Compressed air through the radiator fins, the intake filter, and the PSU shroud. **Hold the fan blades still while you blow air through them** — spinning a fan with compressed air generates voltage back through the motor and can damage the header it's plugged into.

Radiator fins clog invisibly. A radiator that looks clean from the outside can be packed solid one layer in.

### 2. Verify the AIO pump is alive

This is the failure that produces the dramatic temperatures. In HWiNFO64, find the pump RPM reading (usually reported as a fan header — often `CPU_FAN` or `AIO_PUMP`).

- **`0 RPM` = dead pump.** The AIO needs replacing. Nothing else you do will help.
- **Coolant sitting at room temperature while the CPU is at 90 °C** means the pump isn't moving fluid even if it reports RPM.

You can also feel it: with the machine running, the two hoses to the radiator should be *close to the same temperature*. One hot and one cold means no circulation.

> [!CAUTION]
> A failed pump is the single most common cause of a suddenly-throttling Aurora, and it is a hardware replacement. If your machine is within warranty, **stop here and contact Dell support** — opening it further may affect coverage.

### 3. Thermal paste

Paste dries out over three to five years, and its degradation is gradual, which is exactly why the machine got "slowly slower" instead of failing outright. Repasting is cheap and reliably drops temperatures on a machine of this age.

Clean both surfaces with isopropyl alcohol (90%+), apply a pea-sized amount of fresh paste to the centre of the IHS, and remount evenly. **This is an involved job and it may void your warranty — check your coverage first.**

### 4. Fan curves

If temperatures are elevated but not critical, the fan curve may simply be too conservative. This is adjustable in the BIOS (tap `F2` at boot) under the thermal or performance profile settings. Balanced or Performance mode raises fan speed at a given temperature — it's louder, and it works.

<img src="assets/divider.svg" alt="" width="100%">

## Dell and Alienware services

**Applies when:** Autoruns or Task Manager shows Dell processes consuming CPU or disk at idle.

Alienware machines ship with **Dell SupportAssist** and **Alienware Command Center (AWCC)**, and both are known to get stuck in utilization loops that pin disk or CPU at 100% indefinitely.

### Disable them

Open `services.msc` (Win+R). Sort by Name and find every service starting with **`Dell`** or **`SupportAssist`**. For each one:

1. Right-click → **Properties**
2. **Stop** the service
3. Set **Startup type** to `Disabled`
4. **Apply**

Reboot and re-measure. If idle CPU and disk drop, you found it.

> [!TIP]
> **Disable, don't uninstall — at least at first.** Disabling is instantly reversible if something you actually use breaks. Once you've confirmed the machine is stable for a week without them, uninstall properly through Settings → Apps.

### What you give up

Be aware of the trade before you commit:

| Service | What disabling it costs you |
|:--------|:----------------------------|
| **Dell SupportAssist** | Automatic driver update notifications and Dell's hardware self-tests. You can still get drivers manually from Dell's support site. |
| **Alienware Command Center** | RGB lighting control, per-game profiles, and the AWCC overclocking presets. |
| **Dell Update** | Automated BIOS and firmware update prompts. **Check for BIOS updates manually** — don't simply stop getting them. |

If you rely on AWCC for lighting, disable everything *else* first and re-test — SupportAssist is the more frequent offender of the two.

<img src="assets/divider.svg" alt="" width="100%">

## System file repair

**Applies when:** SFC reports corruption, or Windows behaves erratically after the hardware is cleared.

Command Prompt **as Administrator**, in this order:

```dos
DISM.exe /Online /Cleanup-image /Restorehealth
sfc /scannow
```

Order matters — DISM repairs the component store, and SFC pulls its replacement files from that store. Running SFC against a damaged store fails and wastes 30 minutes.

If SFC repairs anything, **reboot and run it again** until it reports a clean pass.

### When DISM can't reach Windows Update

Point it at the Windows 11 ISO instead. Mount the ISO (right-click → Mount, note the drive letter), then:

```dos
DISM /Online /Cleanup-Image /RestoreHealth /Source:esd:E:\sources\install.esd:1 /limitaccess
```

Replace `E:` with the mounted ISO's letter. If the ISO contains `install.wim` rather than `install.esd`, use `wim:E:\sources\install.wim:1` instead.

### Offline repair

If Windows won't boot far enough to run these, boot the Windows 11 ISO from Ventoy → **Repair your computer → Troubleshoot → Command Prompt**:

```dos
sfc /scannow /offbootdir=C:\ /offwindir=C:\Windows
```

Confirm the drive letter first with `diskpart` → `list volume` → `exit`. **Recovery-environment drive letters routinely differ from the ones Windows shows you.**

<img src="assets/divider.svg" alt="" width="100%">

## Display driver wipe

**Applies when:** the system hangs launching applications or rendering graphics, or a GPU driver update went wrong.

Windows' own uninstaller leaves registry keys and files behind. Installing a new driver on top of that debris reproduces the same fault, which is why "I already reinstalled the driver" so often doesn't help. [DDU](https://www.wagnardsoft.com/display-driver-uninstaller-ddu) removes everything.

**Prepare before you start:**

1. **Download your replacement driver first** and put it on the desktop. You will not have working graphics acceleration between the wipe and the install.
2. **Disconnect from the internet** — unplug the ethernet cable. Windows Update will otherwise reinstall the broken driver automatically the moment it notices one is missing.
3. **Boot into Safe Mode.** `msconfig` → Boot tab → check **Safe boot** → Minimal → reboot. (Remember to uncheck it afterward.)

**Then:**

4. Run DDU. Select device type (GPU) and vendor (AMD or NVIDIA).
5. Choose **Clean and restart**.
6. After reboot, install the driver you downloaded in step 1.
7. Reconnect the internet.

> [!NOTE]
> DDU can also run inside the [Hiren's BootCD PE](https://www.hirensbootcd.org/download/) environment, which is useful when Windows won't boot far enough to reach Safe Mode.

<img src="assets/divider.svg" alt="" width="100%">

## Failing drive replacement

**Applies when:** CrystalDiskInfo reports `Caution` or `Bad`.

Nothing on this page fixes a failing drive. The sequence is:

1. **Back up immediately.** Right now, before anything else. If the drive is `Bad`, treat every additional hour of use as a risk to your data.
2. If the drive is too unstable to copy from in Windows, boot [Hiren's BootCD PE](https://www.hirensbootcd.org/download/) and copy the files from there — it doesn't load the failing drive's OS, which puts far less stress on it.
3. Replace the drive.
4. Clean-install from the [Windows 11 ISO](https://www.microsoft.com/software-download/windows11).

A `Caution` status can persist for months before total failure — or fail tomorrow. It is not a countdown you can read.

<img src="assets/divider.svg" alt="" width="100%">

## Restore Secure Boot

**Do this when you're finished.** Booting third-party rescue media generally requires disabling Secure Boot, and it's easy to walk away leaving it off.

Reboot, tap **`F2`** for BIOS setup → **Boot** or **Security** → set **Secure Boot** back to **Enabled** → save and exit (`F10`).

Windows 11 depends on Secure Boot for security updates and core services. Leaving it disabled is a permanent reduction in the machine's security posture in exchange for a convenience you no longer need.

> [!TIP]
> While you're in the BIOS: check that **Memory Profile / XMP / DOCP** is enabled. Auroras occasionally ship with memory running at the JEDEC default rather than its rated speed, which is free performance sitting unclaimed.

<img src="assets/divider.svg" alt="" width="100%">

## Verify the fix

Re-run [DIAGNOSTICS.md](DIAGNOSTICS.md) and compare against what you measured before. You're looking for:

- **Idle CPU temperature** back in the 30–45 °C range
- **Drive health** reporting `Good`
- **Idle CPU usage** near zero with nothing pinned
- **Boot time** reasonable and consistent
- **Secure Boot** back on

If the numbers moved and the machine feels right, you're done. If the numbers moved but it still feels slow, something else is going on — go back to Step 3 and audit again with fresh eyes.

<div align="center">

[← Diagnostics](DIAGNOSTICS.md) · [Toolkit](TOOLKIT.md) · [README](README.md)

`END OF LINE`

</div>
