# Getting CPU virtualization flags throughout ssh on iDRAC enable servers

HP and DELL (maybe others) manufactors that uses iDRAC might have the ssh option enabled by default, what makes it handy when we need to do some batch operations that would require a lot of time if doing by the web or virtual console.

This small doc shows how to get if the Virtualization flags are enabled on iDRAC powered servers.

## Preparation

### Securing credentials

Read the [securing-credentials](securing-credentials.md) section

### sshpass

`sshpass` is a tool that helps you to use `ssh` command to password enabled sites without interaction.

We will use it to connect without typing the password

[sspass](https://linux.die.net/man/1/sshpass) manual pages

### racadm

`racadm` is a utility in the iDRAC `BIOS` to manage the system configuration. Is available by connecting througout ssh on iDRAC enabled servers.

[RACADM](https://www.dell.com/support/manuals/uk/en/ukbsdt1/integrated-dell-remote-access-cntrllr-8-with-lifecycle-controller-v2.00.00.00/racadm_idrac_pub-v1/racadm-subcommand-details?guid=guid-cd4e81e6-818c-44fb-9e7a-82950425fbbb&lang=en-us) documentation


## Connecting to iDRAC throughout ssh

This following steps considers what you have already done the above steps and can connect to the server.

### Loding the credentials

Supposing that the your iDRAC user is `admin`

decrypt and load the user and password
```
$ export USER=admin
$ export PASS=$(gpg --decrypt .pass.gpg -r foo@bar)
```

### getting the info

```
$ sshpass -p${PASS} ssh -ossh -oStrictHostKeyChecking=no ${USER}@<foor-bar-host> \
  'racadm get BIOS.ProcSettings.ProcVirtualization'
```
The expected return is something like this:
```
[Key=BIOS.Setup.1-1#ProcSettings]
ProcVirtualization=Enabled

```

That's it.
