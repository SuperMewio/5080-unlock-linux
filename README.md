WARNING! I am *not* a programmer. I used Kagi's AI feature to assist me with making this package easier to install in ARCH via DKMS. DO read the original README first *before* installing this. This *will* increase the heat of your device.

If you want to install this for whatever reason, there is an artifact package located at https://github.com/SuperMewio/5080-unlock-linux/actions/runs/36364073084

Requirements: You will need the matching headers for your kernel and DKMS.

Just download the artifact, unzip it, and install it via pacman -U like you would any other local package.

This is a fork of https://github.com/IzzieBoopers/5080-Unlock_Linux/tree/original-gpu-unlock-only, I did not create this at all other than the above to make it easier to install on arch systems for future use.

ORIGINAL README.md:

# 5080 Unlock

HP OMEN RTX 5080 Laptop GPU power unlock helper for Linux.

This project builds and installs `omen_wmi_boost`, a small kernel module that uses HP/OMEN WMI calls to switch the GPU out of the default 80 W limp-mode state and into the laptop's high-performance GPU power state.

This does not overclock the GPU. It enables the OEM firmware performance state that the laptop already exposes, but does not enable correctly on Linux for this hardware.

On the creator's test workload, this reduced a specific AI inference workload from about 12.5 seconds to about 8 seconds. That performance increase also raised GPU thermals by roughly 25 C.

## Use At Your Own Risk

This tool intentionally changes firmware-controlled GPU power behavior. Use it only if you understand the extra heat, power draw, fan noise, and hardware stress involved.

Proper cooling matters. Keep vents clear, monitor temperatures, and strongly consider using a cooling pad or other elevated airflow setup when running heavy GPU workloads.

Tested target hardware and software:

- RTX 5080 Laptop GPU
- HP OMEN laptop
- Fedora Linux running KDE Plasma
- Recent Linux kernel with matching headers
- Recent Nvidia Linux driver

Unique board, BIOS, hostname, and local machine identifiers are intentionally omitted from this release documentation.

## How It Works

`omen_wmi_boost` uses HP's WMI firmware interface to run the same class of performance-state commands normally handled by OEM tooling:

```text
WMAA -> WHCM -> GMCF -> GC21/GC22
```

The important WMI GUID is:

```text
5FB7F034-2C63-45E9-BE91-3D44E2C707E4
```

The module reads and writes firmware GPU power flags such as `CTGP`, `DTGP`, `DBST`, and the OMEN high-performance policy gate `OGHP`. See `docs/REVERSE-ENGINEERING.md` for more detail.

## What It Installs

The installer builds the module for the running kernel, loads it at boot, and installs a verifier service.

- `/lib/modules/<kernel>/updates/omen_wmi_boost.ko`
- `/etc/modules-load.d/omen_wmi_boost.conf`
- `/etc/modprobe.d/omen_wmi_boost.conf`
- `/etc/systemd/system/omen-wmi-boost-verify.service`
- `/usr/local/sbin/omen-wmi-boost-verify`
- `/usr/local/sbin/omen-wmi-boost-rebuild`
- `/etc/kernel/install.d/zz-omen-wmi-boost.install`
- `/usr/share/doc/omen-wmi-boost/BOOT-SETUP.md`

## Requirements

Install the build tools and matching kernel headers first:

```bash
sudo dnf install gcc make kernel-devel-$(uname -r)
```

Secure Boot must either be disabled or configured to trust/sign the module. Unsigned out-of-tree modules will not load with Secure Boot enforcement.

Future Linux, kernel, Nvidia driver, or BIOS updates may break compatibility.

## Install

Inspect the current system without making changes:

```bash
cd ~/5080_Unlock
./install.sh --dry-run
```

```bash
cd ~/5080_Unlock
sudo ./install.sh
```

The installer performs a smoke test, but it does not reboot automatically. Reboot when ready:

```bash
sudo reboot
```

After reboot:

```bash
cat /sys/kernel/omen_wmi_boost/gpu_state
systemctl status omen-wmi-boost-verify.service
nvidia-smi -q -d POWER
```

Expected `gpu_state` includes:

```text
ctgp=1 ppab=1
```

## Uninstall

The uninstaller actively disables the WMI GPU boost flags before removing persistence, returning the machine to the previous limp-mode behavior.

```bash
cd ~/5080_Unlock
sudo ./install.sh --uninstall
```

It leaves the source tree in place under `/usr/src/omen_wmi_boost` and this checkout untouched.

## Manual Controls

Re-apply performance mode without reboot:

```bash
echo 1 | sudo tee /sys/kernel/omen_wmi_boost/performance
```

Disable boost while the module is loaded:

```bash
echo 0 | sudo tee /sys/kernel/omen_wmi_boost/boost
```

Reload the module manually:

```bash
sudo modprobe -r omen_wmi_boost
sudo modprobe omen_wmi_boost
```

## Troubleshooting

Check the verifier journal:

```bash
journalctl -t omen-wmi-boost -b
```

Check module state:

```bash
lsmod | grep omen_wmi_boost
cat /sys/kernel/omen_wmi_boost/gpu_state
cat /sys/kernel/omen_wmi_boost/last_error
```

Check Nvidia power policy:

```bash
journalctl -u nvidia-powerd -b
nvidia-smi -q -d POWER
```

If WMI state is enabled but power is still capped, check `journalctl -u nvidia-powerd -b` and confirm the laptop is on AC power with adequate cooling.

## Maintenance

The creator intends to maintain this program only for compatibility fixes, such as Linux, Fedora, kernel, Nvidia driver, or HP firmware updates that break the current behavior. New features, broader hardware support, and general tuning are not guaranteed.

## License

MIT. See `LICENSE`.

## Development

Build manually:

```bash
make -C omen_wmi_boost
```

Clean generated module output:

```bash
make -C omen_wmi_boost clean
```

Run the quick validation script:

```bash
sudo ./run-tests.sh
```

The repository ignores generated kernel build artifacts such as `*.ko`, `*.o`, `*.mod.c`, `Module.symvers`, and `modules.order`.

## How this started

*A candid story — not part of the technical release itself.*

I had just finished setting up RAID0 and was benchmarking the machine ahead of AI workloads, keeping an eye on thermals. I asked ChatGPT whether the cooling looked sane; it replied that either the cooling was datacenter-grade or the GPU was being throttled. That sent me down the rabbit hole, where I found the GPU stuck in so-called **limp mode** — capped around 80 W while the VBIOS advertised far more. I did not care for that.

Somewhere in there I said I wanted to feel the heat of a thousand suns under my palm, and I'd be damned if anything but the laws of thermodynamics stopped me. ChatGPT and I worked through what the kernel and firmware were actually doing. This little program is what came out of that.
