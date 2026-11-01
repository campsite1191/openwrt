# Linksys MX4300 OpenWrt (PPE-NSS)

Custom, automated OpenWrt production firmware builds for the **Linksys MX4300 (IPQ807x / qualcommax)** router with Qualcomm in-kernel hardware acceleration (PPE and NSS) running on modern Linux kernels.

Based on upstream work by [JuliusBairaktaris/openwrt-ppe-nss](https://github.com/JuliusBairaktaris/openwrt-ppe-nss) and [JuliusBairaktaris/Qualcommax_NSS_Builder](https://github.com/JuliusBairaktaris/Qualcommax_NSS_Builder).

---

## Key Hardware Acceleration Features

| Subsystem | Offload Engine | Implementation & Kernel Interfaces |
| :--- | :--- | :--- |
| **Routing & NAT** | **PPE** | In-kernel `fw4` hardware flowtable offload (`flow_offloading_hw`) for IPv4, IPv6, NAT, PPPoE, and 802.1Q VLANs. |
| **Bridging & Switching** | **PPE** | Linux `switchdev` hardware bridging, VLAN filtering, and IGMP/MLD snooping. |
| **SQM Traffic Shaping** | **PPE** | Hardware-accelerated tc qdisc flower offload (`hw_ppe.qos`), `qca-ppe-qos`, and `dcb`. |
| **Wi-Fi Data Path** | **NSS** | High-performance `nss-wifi-drv` host driver with `ath11k` offloaded to the NSS cores (firmware 12.5 active). |
| **Hardware Buffer Sizing** | **Dynamic 2GB** | Dynamic hardware buffer scaling for the Linksys MX4300's 2GB DDR4 RAM and Qualcomm PPE/NSS acceleration. |

---

## Included Software & Hardening

* **Toolchain Optimization**: GCC 15, Binutils 2.46, Mold linker, LTO (Link-Time Optimization), and Cortex-A53 CPU tuning (`-O2 -mcpu=cortex-a53+crc+crypto`).
* **Security & Hardening**: Strong stack protector, ASLR PIE, Format Security, Fortify Source 3, Full RELRO, and Seccomp.
* **Pure OpenSSL Stack**: `mbedtls` completely excluded in favor of `wpad-openssl`, `apk-openssl`, and `libustream-openssl`.
* **VPN**: Full **AmneziaWG** integration (`kmod-amneziawg`, `amneziawg-tools`, `luci-proto-amneziawg`) alongside standard WireGuard.
* **DNS & Routing**: **SmartDNS** (`smartdns`, `luci-app-smartdns`), `dnsmasq-full`, `ip-full`, and Policy Based Routing (`luci-app-pbr`).
* **Time Sync**: `chrony-nts` with Network Time Security enabled.
* **Network & System Tools**: BBR TCP congestion control (`kmod-tcp-bbr`), official Ookla Speedtest ARM64 CLI (`speedtest`), `iperf3`, `curl`, `btop`, `ttyd` web terminal (`luci-app-ttyd`), `qrencode`, and `rsync`.
* **Storage**: USB storage (`kmod-usb-storage`, `kmod-usb-storage-uas`) with VFAT support.

---

## Out-of-the-Box Experience

* **Pre-seeded Tri-Band Wi-Fi**:
  On a clean install, all three Linksys MX4300 radios come up enabled out of the box so you can connect wirelessly without needing an Ethernet cable:
  * **SSID**: `OpenWrt`
  * **Password**: `OpenWrt2025`
  * **Encryption**: WPA2-PSK (`psk2`)
* **Sysupgrade Protection**:
  On sysupgrade with settings preserved, your custom Wi-Fi network configurations (`/etc/config/wireless`) and system settings are retained completely without interference.

---

## Automated CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/build-ppe.yml`) runs automatically once a week on Friday mornings at 05:00 PDT / 12:00 UTC (and on demand via `workflow_dispatch`), checking for updates across:
1. Upstream OpenWrt tree (`openwrt-ppe-nss`, branch `ppe-nss`)
2. NSS Packages repository (`nss-packages`, branch `ppe-nss`)
3. Local configuration changes in this repository

### CI Highlights
* **Compiler & Download Caching**: Uses ccache and a persistent `dl/` source tarball pool to speed up builds.
* **Dynamic Kconfig Assertion**: Automatically audits generated `.config` against requested options to prevent silent symbol pruning.
* **Automated Releases**: Generates sysupgrade images, checksums (`sha256sums`), and configuration manifests, retaining the latest 7 releases.
