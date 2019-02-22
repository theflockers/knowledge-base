# Setting the bootflags on servers through ipmi

This small guide aims to show how to set the next boot.
It was tested on iDRAC enable HP servers.

## Preparing

### Loading credentials

Read the [securing-credentials](securing-credentials.md) section

## Setting boot flags

### Default available boot flags:

```
none        - default factory boot flag
force_pxe   - PXE boot
force_cdrom - CDROM boot
force_disk  - Local disk boot
force_safe  - safe boot??
force_diag  - ???
force_bios  - Boot to the BIOS setup
```
There might be more modes but it depends on the manufactor

### running the ipmi command
```
$ ipmitool -I lanplus -H <ip> -U ${USER} -P ${PASS} chassis bootparam get 5 # get boot flags
$ ipmitool -I lanplus -H <ip> -U ${USER} -P ${PASS} chassis bootparam set bootflag [none|force_[pxe|cdrom|disk|safe|diag|bios]]
```
