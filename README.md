# smart-keep-active

Prevent automatic spin-down by hd-idle when SMART self-tests are running

## What does it do?

It monitors:

1. system logs of the smartmontools service for any scheduled self-tests that are starting
2. the self-test state of each hard disk, if not already spun down

If a self-test is detected, it performs a small read from that disk at regular intervals to update `/proc/diskstats` and prevent hd-idle from treating it as idle.

## Installation

1. Download the two files to `/opt/smart-keep-active` with correct permissions

	```
	sudo wget -P /opt/smart-keep-active/ https://raw.githubusercontent.com/yushiyangk/smart-keep-active/refs/heads/main/smart-keep-active
	sudo wget -P /opt/smart-keep-active/ https://raw.githubusercontent.com/yushiyangk/smart-keep-active/refs/heads/main/smart-keep-active.service
	sudo chmod 700 /opt/smart-keep-active/smart-keep-active
	```

2. Install, enable and start the service

	```
	sudo systemctl link /opt/smart-keep-active/smart-keep-active.service
	sudo systemctl enable smart-keep-active
	sudo systemctl start smart-keep-active
	```

## Configuration

Edit the `smart-keep-active` script to modify its constant parameters:

- **`SMARTMONTOOLS_UNIT`**: name of the Smartmontools service unit as seen by `journalctl` (test with <code>journalctl -u <var>unit_name</var></code>)
- **`CHECK_INTERVAL_SEC`**: probe each hard disk's self-test state at this interval (disks that are already spun down will not be spun up by this)
- **`ACTIVE_INTERVAL_SEC`**: how frequent to perform a disk read to keep it active
- **`READ_BYTES`**: size of disk read (the read value is piped to `/dev/null`)

Ensure that `CHECK_INTERVAL_SEC` and `ACTIVE_INTERVAL_SEC` are shorter than the idle time configured in hd-idle.
