# Unattended Upgrades

## Install

Note: As of Debian 9 and Ubuntu 18.04, `unattended-upgrades` is installed by default. If it is missing:

```console
$ apt update && apt dist-upgrade
$ apt install unattended-upgrades
```

## Configure

### Upgrade Packages

Download [config/apt/50unattended-upgrades](https://github.com/simonlenton/config/blob/main/apt/50unattended-upgrades):

```console
$ cd /etc/apt/apt.conf.d ;
    wget https://raw.githubusercontent.com/simonlenton/config/main/apt/50unattended-upgrades

$ chmod 644 50unattended-upgrades
```

### Update Frequency

Download [config/apt/20auto-upgrades](https://github.com/simonlenton/config/blob/main/apt/20auto-upgrades):

```console
$ cd /etc/apt/apt.conf.d ;
    wget https://raw.githubusercontent.com/simonlenton/config/main/apt/20auto-upgrades

$ chmod 644 20auto-upgrades
```

The above configuration will update package lists, download packages and install available upgrades daily. While the APT cache will be cleaned every 7 days.

### Test unattended-upgrades

You can test the config with a dry run. Use the following command. Refer to man page for help.

```console
$ unattended-upgrades --dry-run --debug
```
