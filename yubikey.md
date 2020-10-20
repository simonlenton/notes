# Yubikey

- [Using keys](#using-keys)
- [SSH Configuration](#ssh-configuration)
  - [Replace agents](#replace-agents)
  - [Copy public key](#copy-public-key)
  - [(Optional) Save public key for identity file configuration](#-optional-save-public-key-for-identity-file-configuration)
  - [Connect with public key authentication](#connect-with-public-key-authentication)

## Using keys

Download [config/gpg.conf](https://github.com/simonlenton/config/blob/main/gpg.conf):

```console
$ cd ~/.gnupg ; wget https://raw.githubusercontent.com/simonlenton/config/main/gpg.conf

$ chmod 600 gpg.conf
```

Install the required packages and mount the non-encrypted volume created earlier:

### Linux

```console
$ sudo apt update && sudo apt install -y gnupg2 gnupg-agent gnupg-curl scdaemon pcscd

$ sudo mount /dev/mmcblk0p2 /mnt
```

Import the public key file:

```console
$ gpg --import /mnt/0x*txt
gpg: key 0xD70372559E92FEE7: public key "simonlenton <simonlenton@users.noreply.github.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Or download the public key from a keyserver:

```console
$ gpg --recv $KEYID
gpg: requesting key 0xD70372559E92FEE7 from hkps server hkps.pool.sks-keyservers.net
[...]
gpg: key 0xD70372559E92FEE7: public key "simonlenton <simonlenton@users.noreply.github.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Edit the master key to assign it ultimate trust by selecting `trust` and `5`:

```console
$ export KEYID=0xD70372559E92FEE7

$ gpg --edit-key $KEYID

gpg> trust
pub  4096R/0xD70372559E92FEE7  created: 2016-05-24  expires: never       usage: C
                               trust: unknown       validity: unknown
sub  4096R/0xBECFA3C1AE191D15  created: 2017-10-09  expires: 2018-10-09  usage: S
sub  4096R/0x5912A795E90DD2CF  created: 2017-10-09  expires: 2018-10-09  usage: E
sub  4096R/0x3F29127E79649A3D  created: 2017-10-09  expires: 2018-10-09  usage: A
[ unknown] (1). simonlenton <simonlenton@users.noreply.github.com>

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

pub  4096R/0xD70372559E92FEE7  created: 2016-05-24  expires: never       usage: C
                               trust: ultimate      validity: unknown
sub  4096R/0xBECFA3C1AE191D15  created: 2017-10-09  expires: 2018-10-09  usage: S
sub  4096R/0x5912A795E90DD2CF  created: 2017-10-09  expires: 2018-10-09  usage: E
sub  4096R/0x3F29127E79649A3D  created: 2017-10-09  expires: 2018-10-09  usage: A
[ unknown] (1). simonlenton <simonlenton@users.noreply.github.com>

gpg> quit
```

Remove and re-insert YubiKey and check the status:

```console
$ gpg --card-status
Reader ...........: Yubico YubiKey OTP FIDO CCID 00 00
Application ID ...: D2760001240102010006055532110000
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 05553211
Name of cardholder: simonlenton
Language prefs ...: en
Sex ..............: unspecified
URL of public key : [not set]
Login data .......: simonlenton@users.noreply.github.com
Signature PIN ....: not forced
Key attributes ...: rsa4096 rsa4096 rsa4096
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 3 3
Signature counter : 0
Signature key ....: 07AA 7735 E502 C5EB E09E  B8B0 BECF A3C1 AE19 1D15
      created ....: 2016-05-24 23:22:01
Encryption key....: 6F26 6F46 845B BEB8 BDF3  7E9B 5912 A795 E90D D2CF
      created ....: 2016-05-24 23:29:03
Authentication key: 82BE 7837 6A3F 2E7B E556  5E35 3F29 127E 7964 9A3D
      created ....: 2016-05-24 23:36:40
General key info..: pub  4096R/0xBECFA3C1AE191D15 2016-05-24 simonlenton <simonlenton@users.noreply.github.com>
sec#  4096R/0xD70372559E92FEE7  created: 2016-05-24  expires: never
ssb>  4096R/0xBECFA3C1AE191D15  created: 2017-10-09  expires: 2018-10-09
                      card-no: 0006 05553211
ssb>  4096R/0x5912A795E90DD2CF  created: 2017-10-09  expires: 2018-10-09
                      card-no: 0006 05553211
ssb>  4096R/0x3F29127E79649A3D  created: 2017-10-09  expires: 2018-10-09
                      card-no: 0006 05553211
```

`sec#` indicates master key is not available (as it should be stored encrypted offline).

**Note** If you see `General key info..: [none]` in the output instead - go back and import the public key using the previous step.

Encrypt a message to your own key (useful for storing password credentials and other data):

```console
$ echo "test message string" | gpg --encrypt --armor --recipient $KEYID -o encrypted.txt
```

To encrypt to multiple recipients (or to multiple keys):

```console
$ echo "test message string" | gpg --encrypt --armor --recipient $KEYID_0 --recipient $KEYID_1 --recipient $KEYID_2 -o encrypted.txt
```

Decrypt the message:

```console
$ gpg --decrypt --armor encrypted.txt
gpg: anonymous recipient; trying secret key 0x0000000000000000 ...
gpg: okay, we are the anonymous recipient.
gpg: encrypted with RSA key, ID 0x0000000000000000
test message string
```

Sign a message:

```console
$ echo "test message string" | gpg --armor --clearsign > signed.txt
```

Verify the signature:

```console
$ gpg --verify signed.txt
gpg: Signature made Wed 25 May 2016 00:00:00 AM UTC
gpg:                using RSA key 0xBECFA3C1AE191D15
gpg: Good signature from "simonlenton <simonlenton@users.noreply.github.com>" [ultimate]
Primary key fingerprint: 011C E16B D45B 27A5 5BA8  776D FF3E 7D88 647E BCDB
     Subkey fingerprint: 07AA 7735 E502 C5EB E09E  B8B0 BECF A3C1 AE19 1D15
```

Use a [shell function](https://github.com/drduh/config/blob/master/zshrc) to make encrypting files easier:

```bash
secret () {
        output=~/"${1}".$(date +%s).enc
        gpg --encrypt --armor --output ${output} -r 0x0000 -r 0x0001 -r 0x0002 "${1}" && echo "${1} -> ${output}"
}

reveal () {
        output=$(echo "${1}" | rev | cut -c16- | rev)
        gpg --decrypt --output ${output} "${1}" && echo "${1} -> ${output}"
}
```

```console
$ secret document.pdf
document.pdf -> document.pdf.1580000000.enc

$ reveal document.pdf.1580000000.enc
gpg: anonymous recipient; trying secret key 0xD70372559E92FEE7 ...
gpg: okay, we are the anonymous recipient.
gpg: encrypted with RSA key, ID 0x0000000000000000
document.pdf.1580000000.enc -> document.pdf
```

---

## SSH configuration

Create a hardened configuration for gpg-agent by downloading [config/gpg-agent.conf](https://github.com/simonlenton/config/blob/main/gpg-agent.conf):

```console
$ cd ~/.gnupg

$ wget https://raw.githubusercontent.com/simonlenton/config/main/gpg-agent.conf

$ grep -ve "^#" gpg-agent.conf
enable-ssh-support
default-cache-ttl 60
max-cache-ttl 120
pinentry-program /usr/bin/pinentry-curses
```

**Important** The `cache-ttl` options do **NOT** apply when using a YubiKey as a smartcard as the PIN is [cached by the smartcard itself](https://dev.gnupg.org/T3362). Therefore, in order to clear the PIN from cache (smartcard equivalent to `default-cache-ttl` and `max-cache-ttl`), you need to unplug the YubiKey.

**Tip** Set `pinentry-program /usr/bin/pinentry-gnome3` for a GUI-based prompt. If the _pinentry_ graphical dialog doesn't show and you get this error: `sign_and_send_pubkey: signing failed: agent refused operation`, you may need to install the `dbus-user-session` package and restart the computer for the `dbus` user session to be fully inherited; this is because behind the scenes, `pinentry` complains about `No $DBUS_SESSION_BUS_ADDRESS found`, falls back to `curses` but doesn't find the expected `tty`.

On macOS, use `brew install pinentry-mac` and set the program path to `pinentry-program /usr/local/bin/pinentry-mac` or `pinentry-program /usr/local/MacGPG2/libexec/pinentry-mac.app/Contents/MacOS/pinentry-mac` if using MacGPG Suite.

## Replace agents

To launch `gpg-agent` for use by SSH, use the `gpg-connect-agent /bye` or `gpgconf --launch gpg-agent` commands.

Add these to the shell `rc` file:

```console
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK="/run/user/$UID/gnupg/S.gpg-agent.ssh"
gpg-connect-agent updatestartuptty /bye > /dev/null
```

On modern systems, `gpgconf --list-dirs agent-ssh-socket` will automatically set `SSH_AUTH_SOCK` to the correct value and is better than hard-coding to `run/user/$UID/gnupg/S.gpg-agent.ssh`, if available:

```console
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpgconf --launch gpg-agent
```

Note that `SSH_AUTH_SOCK` normally only needs to be set on the *local* laptop (workstation), where the YubiKey is plugged in.  On the *remote* server that we SSH into, `ssh` will automatically set `SSH_AUTH_SOCK` to something like `/tmp/ssh-mXzCzYT2Np/agent.7541` when we connect.  We therefore do **NOT** manually set `SSH_AUTH_SOCK` on the server - doing so would break [SSH Agent Forwarding](#remote-machines-agent-forwarding).

## Copy public key

**Note** It is **not** necessary to import the corresponding GPG public key in order to use SSH.

Copy and paste the output from `ssh-add` to the server's `authorized_keys` file:

```console
$ ssh-add -L
ssh-rsa AAAAB4NzaC1yc2EAAAADAQABAAACAz[...]zreOKM+HwpkHzcy9DQcVG2Nw== cardno:000605553211
```

## (Optional) Save public key for identity file configuration

By default, SSH attempts to use all the identities available via the agent. It's often a good idea to manage exactly which keys SSH will use to connect to a server, for example to separate different roles or [to avoid being fingerprinted by untrusted ssh servers](https://blog.filippo.io/ssh-whoami-filippo-io/). To do this you'll need to use the command line argument `-i [identity_file]` or the `IdentityFile` and `IdentitiesOnly` options in `.ssh/config`.

The argument provided to `IdentityFile` is traditionally the path to the _private_ key file (for example `IdentityFile ~/.ssh/id_rsa`). For the YubiKey - indeed, in general for keys stored in an ssh agent - `IdentityFile` should point to the _public_ key file, `ssh` will select the appropriate private key from those available via the ssh agent. To prevent `ssh` from trying all keys in the agent use the `IdentitiesOnly yes` option along with one or more `-i` or `IdentityFile` options for the target host.

To reiterate, with `IdentitiesOnly yes`, `ssh` will not automatically enumerate public keys loaded into `ssh-agent` or `gpg-agent`. This means `publickey` authentication will not proceed unless explicitly named by `ssh -i [identity_file]` or in `.ssh/config` on a per-host basis.

In the case of YubiKey usage, to extract the public key from the ssh agent:

```console
ssh-add -L | grep "cardno:000605553211" > ~/.ssh/id_rsa_yubikey.pub
```

Then you can explicitly associate this YubiKey-stored key for used with a host, `github.com` for example, as follows:

```console
$ cat << EOF >> ~/.ssh/config
Host github.com
    IdentitiesOnly yes
    IdentityFile ~/.ssh/id_rsa_yubikey.pub
EOF
```

## Connect with public key authentication

```console
$ ssh git@github.com -vvv
[...]
debug2: key: cardno:000605553211 (0x1234567890),
debug1: Authentications that can continue: publickey
debug3: start over, passed a different list publickey
debug3: preferred gssapi-keyex,gssapi-with-mic,publickey,keyboard-interactive,password
debug3: authmethod_lookup publickey
debug3: remaining preferred: keyboard-interactive,password
debug3: authmethod_is_enabled publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: cardno:000605553211
debug3: send_pubkey_test
debug2: we sent a publickey packet, wait for reply
debug1: Server accepts key: pkalg ssh-rsa blen 535
debug2: input_userauth_pk_ok: fp e5:de:a5:74:b1:3e:96:9b:85:46:e7:28:53:b4:82:c3
debug3: sign_and_send_pubkey: RSA e5:de:a5:74:b1:3e:96:9b:85:46:e7:28:53:b4:82:c3
debug1: Authentication succeeded (publickey).
[...]
```

**Tip** To make multiple connections or securely transfer many files, consider using the [ControlMaster](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Multiplexing) ssh option. Also see [config/ssh_config](https://github.com/simonlenton/config/blob/main/ssh_config).

---

### Useful sites

#### Yubikey Setup

[DrDuh's YubiKey Guide](https://github.com/drduh/YubiKey-Guide)
