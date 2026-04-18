# rutos-zabbix-proxy

Builds a Zabbix Proxy `.ipk` package with SQLite3 support for the **RUTM30** router (MIPS/OpenWrt).

The RUTM30 runs OpenWrt on a ramips/mt7621 (mipsel_24kc) chipset. The default Zabbix Proxy package depends on PostgreSQL, which is too heavy for embedded hardware. This project patches the OpenWrt feed to swap PostgreSQL for SQLite3 and bundles a proper init.d service script.

## How it works

A GitHub Actions workflow:

1. Downloads the OpenWrt 21.02.7 SDK for ramips/mt7621
2. Patches the Zabbix Makefile to use `--with-sqlite3` instead of `--with-pgsql`
3. Injects `zabbix_proxy.init` as the package's init.d service script
4. Compiles `zabbix-proxy-openssl` for mipsel_24kc
5. Uploads the resulting `.ipk` as a GitHub Actions artifact

## Build

Triggered automatically on push to `main` (when the workflow or init script changes), or manually via **Actions → Run workflow**.

The compiled package is available as the `zabbix-proxy-sqlite3-mipsel_24kc` artifact (retained 30 days).

## Installation on the RUTM30

Download the `.ipk` artifact from GitHub Actions, then on the router:

```sh
opkg install zabbix-proxy-openssl_*.ipk
```

The init script is included in the package and installed to `/etc/init.d/zabbix_proxy`. Configure Zabbix Proxy via `/etc/zabbix_proxy.conf`, then:

```sh
/etc/init.d/zabbix_proxy enable
/etc/init.d/zabbix_proxy start
```

## Service management

The init script uses procd and will auto-respawn the process if it crashes (up to 5 retries, 3600 s cooldown).

```sh
/etc/init.d/zabbix_proxy start
/etc/init.d/zabbix_proxy stop
/etc/init.d/zabbix_proxy restart
/etc/init.d/zabbix_proxy status
```
