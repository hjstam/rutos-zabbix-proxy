# rutos-zabbix-proxy

Builds a Zabbix Proxy `.ipk` package with SQLite3 and SNMP support for the **RUTM30** router (MIPS/OpenWrt).

The RUTM30 runs OpenWrt on a ramips/mt7621 (mipsel_24kc) chipset. The default Zabbix Proxy package depends on PostgreSQL, which is too heavy for embedded hardware. This project patches the OpenWrt feed to swap PostgreSQL for SQLite3, adds SNMP support (statically linked), and bundles a proper init.d service script.

## How it works

A GitHub Actions workflow:

1. Downloads the OpenWrt 23.05.5 SDK for ramips/mt7621 (GCC 12.3.0 + musl + OpenSSL 3.x, matching RutOS 7.x)
2. Fetches the Zabbix 7.x Makefile from the `openwrt-24.10` feed branch
3. Patches the Zabbix Makefile to use SQLite3 (`--with-sqlite3`) and SNMP (`--with-net-snmp`)
4. Patches net-snmp to build with OpenSSL so SHA-2 USM auth OIDs are available
5. Statically links `libnetsnmp` into the binary (no runtime SNMP dependency on the router)
6. Injects `zabbix_proxy.init` as the package's init.d service script
7. Compiles `zabbix-proxy-openssl` for mipsel_24kc
8. Uploads the resulting `.ipk` files as a GitHub Actions artifact

## Build

Triggered automatically on push to `main` (when the workflow or init script changes), or manually via **Actions → Run workflow**.

The **Zabbix version** can be set at runtime (workflow dispatch input `zabbix_version`). The default is `7.0.26`.

The compiled packages are available as the `zabbix-proxy-sqlite3-snmp-mipsel_24kc` artifact (retained 30 days). Build logs are uploaded separately as `build-logs` (retained 7 days).

## Installation on the RUTM30

Download the artifact from GitHub Actions. It contains the Zabbix Proxy package and its runtime dependencies. Transfer all `.ipk` files to the router, then install in dependency order:

```sh
opkg install libevent2-core7_*.ipk
opkg install libevent2-7_*.ipk
opkg install libevent2-pthreads7_*.ipk
opkg install fping_*.ipk
opkg install zabbix-proxy-openssl_*.ipk
```

> **Note:** `libnetsnmp` is statically linked into the binary — no separate SNMP package is required on the router.

The init script is included in the package and installed to `/etc/init.d/zabbix_proxy`. Configure Zabbix Proxy via `/etc/zabbix_proxy.conf`, then:

```sh
/etc/init.d/zabbix_proxy enable
/etc/init.d/zabbix_proxy start
```

## Service management

The init script uses procd and will auto-respawn the process if it crashes.

```sh
/etc/init.d/zabbix_proxy start
/etc/init.d/zabbix_proxy stop
/etc/init.d/zabbix_proxy restart
/etc/init.d/zabbix_proxy status
```
