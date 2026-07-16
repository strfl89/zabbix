# Linux Kernel Version Check by Zabbix Agent

A native, lightweight Zabbix template that checks if a Linux system requires a reboot after a kernel update. It operates completely out-of-the-box using the Zabbix Active Agent without requiring any custom scripts or cronjobs deployment on the target hosts.

## Overview

The template collects the currently running kernel version and compares it against the latest installed kernel found inside the `/boot` directory. If a discrepancy is detected, an information-level alert is triggered, indicating that a system reboot is recommended to apply pending updates.

## Features

- **100% Native:** Uses standard agent keys (`vfs.file.contents` and `vfs.dir.get`).
- **No Client Footprint:** No extra Python or Bash scripts needed on your production servers.
- **Smart Sorting:** Uses embedded JavaScript preprocessing to handle natural version string sorting (equivalent to `sort -V`).
- **Active Mode:** Optimized for `ZABBIX_ACTIVE` setups to reduce server load.

## Requirements

- **Zabbix Server / Frontend:** Version 7.0 or higher
- **Zabbix Agent / Agent 2:** Configured in Active mode (`ServerActive` properly set)
- **OS:** Linux with a standard `/boot` directory structure (e.g., Debian, Ubuntu, RHEL, CentOS)

## Installation

1. Download the `template_linux_kernel_version_check.yaml` file.
2. In the Zabbix Frontend, navigate to **Data collection** -> **Templates**.
3. Click **Import** in the top right corner and select the YAML file.
4. Link the template **"Linux Kernel Version Check by Zabbix Agent"** to your desired Linux hosts.

---

## Template Documentation

### Items

| Name | Key | Type | Description |
| :--- | :--- | :--- | :--- |
| Running kernel version | `vfs.file.contents[/proc/sys/kernel/osrelease]` | Zabbix active | Reads the running kernel version directly from `/proc`. |
| Latest installed kernel version | `vfs.dir.get[/boot,"^vmlinuz-[0-9]"]` | Zabbix active | Fetches kernel files in `/boot` and extracts the newest version using JavaScript sorting. |

### Triggers

| Name | Severity | Expression | Description |
| :--- | :--- | :--- | :--- |
| Linux Kernel Version | Information | `last(/linux_kernel_version_check/vfs.file.contents[...]) <> last(/linux_kernel_version_check/vfs.dir.get[...])` | Fires if the running kernel is older than the newest installed file on disk. |

### Tags

The items and triggers are categorized with the following standard tags:
- `component: os`
- `component: linux kernel`
- `scope: notice` (Trigger only)

---

## Version History

- **1.0.0**
  - Initial community release.
  - Native active check implementation.

## Author / Vendor

Developed and maintained by **strfl89**.
