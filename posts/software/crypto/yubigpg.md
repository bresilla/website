+++
type = "post"
status = "published"
date = "2020-01-08"
readingtime = 14

slug = "yubi-key-gpg"
title = "Use a YubiKey for OpenPGP and SSH"
thumbnail = "thumbnail.png"
foot = "Things will not calm down, as a matter of fact they will just calm up - Teal'c (Stargate)"
description = "Move OpenPGP signing, encryption, and authentication subkeys to a YubiKey, require touch, and expose the authentication key through gpg-agent for SSH."
tags = ["yubikey", "gpg", "ssh", "smartcard", "pass"]
categories = ["LINUX"]
series = ["GPG"]
part = "2"
credits = [
    "https://developers.yubico.com/PGP/Card_edit.html",
    "https://developers.yubico.com/PGP/Importing_keys.html",
    "https://docs.yubico.com/software/yubikey/tools/ykman/OpenPGP_Commands.html",
    "https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-openpgp.html",
    "https://gnupg.org/documentation/manuals/gnupg/OpenPGP-Key-Management.html",
    "https://github.com/drduh/YubiKey-Guide",
]

punchline = "A YubiKey keeps daily OpenPGP private-key operations behind a PIN and, if configured, a physical touch. The offline primary key remains the recovery authority."
tldr = "Back up first, transfer only subkeys, change both PINs, enable touch, and test signing, decryption, and SSH before removing any local secret material."

[style]
    accent = "#5f5fff"
    theme = "dark"
+++

# What this setup does

This is the daily-use half of my OpenPGP setup. The previous article created an offline certification key, three subkeys, and tested backups. Here I move the signing, encryption, and authentication subkeys to a YubiKey.

The result looks like this:

{{< image url="media/schema_gpg.png" border="1" width="70" padding="10" >}} The primary key remains offline while the YubiKey carries replaceable subkeys. {{< /image >}}

The card does not replace a backup. Keys moved into its OpenPGP application cannot be exported back out, and `gpg` replaces the local secret key with a card stub. If the only copy goes onto one YubiKey, that key now has the reliability of one tiny object that spends its life in pockets.

This guide is my shorter, opinionated workflow. [DrDuh's YubiKey Guide](https://github.com/drduh/YubiKey-Guide) remains a useful deep reference, but the text and commands here are written around the setup from part one rather than copied from it.

# Check the card and software

On Debian-family Linux systems I install GnuPG, its smart-card daemon, PC/SC support, and YubiKey Manager:

```console
$ sudo apt update
$ sudo apt install gnupg scdaemon pcscd yubikey-manager
```

Package names and whether `pcscd` is needed vary by distribution. Before touching any keys, inspect the connected device:

```console
$ ykman info
$ ykman openpgp info
$ gpg --card-status
```

`ykman info` should show the OpenPGP application as enabled. If `gpg --card-status` cannot see the card, I fix USB permissions, smart-card service conflicts, or missing `scdaemon` before going any further.

Algorithms also depend on the device generation and firmware. The Ed25519 and Curve25519 subkeys from part one require compatible hardware. When maintaining older cards or clients, RSA may be the common denominator. `ykman openpgp info` and the device's technical manual are more trustworthy than a random compatibility table preserved from 2017.

# Start from verified backups

Work from the offline `GNUPGHOME` that holds the primary key and all three secret subkeys. Keep the machine offline while transferring them. First confirm the fingerprint and capabilities:

```console
$ gpg --fingerprint "$FPR"
$ gpg --list-options show-usage --list-secret-keys "$FPR"
```

Then verify that the encrypted full-key export restores in a separate temporary GnuPG home. If a second YubiKey will be a hot spare, prepare it from that backup too. A subkey cannot be extracted from the first card to clone it onto the second.

# Change the default PINs

The OpenPGP application has a user PIN for ordinary private-key operations and an admin PIN for configuration. Change both before loading keys:

```console
$ ykman openpgp access change-pin
$ ykman openpgp access change-admin-pin
```

The commands prompt for the current and new values without placing them in shell history. I store the user PIN and admin PIN separately from the YubiKey and from each other. Guessing is limited by on-card retry counters, so improvising after a forgotten PIN can lock the application.

A reset code is optional. It can recover a blocked user PIN, but it is another secret that needs storage and testing. If I cannot explain where I would retrieve it during a recovery, I leave it unset rather than inventing a third memorable PIN.

# Transfer the three subkeys

Open the key editor by full fingerprint:

```console
$ gpg --edit-key "$FPR"
```

At the `gpg>` prompt, use `list` and check which numbered subkey has each capability. With the creation order from part one, the signing, encryption, and authentication subkeys are normally `1`, `2`, and `3`.

Move the signing subkey to the signature slot:

```text
gpg> key 1
gpg> keytocard
Please select where to store the key:
   (1) Signature key
Your selection? 1
gpg> key 1
```

The second `key 1` deselects it. Move the encryption subkey next:

```text
gpg> key 2
gpg> keytocard
Please select where to store the key:
   (2) Encryption key
Your selection? 2
gpg> key 2
```

Finally move the authentication subkey:

```text
gpg> key 3
gpg> keytocard
Please select where to store the key:
   (3) Authentication key
Your selection? 3
gpg> key 3
gpg> save
```

The admin PIN authorizes each transfer. I read the selected key's fingerprint and capability before every `keytocard`; choosing the wrong numbered key is far easier than the solemn terminal prompt makes it feel.

After `save`, GnuPG's local secret-key entries are card stubs. The private key material is on the YubiKey and cannot be recovered from it. That is why this step comes after the restore test, not before.

# Require physical touch

YubiKey touch policy is configured independently for each OpenPGP slot. Require a touch for signing, decryption, and authentication:

```console
$ ykman openpgp keys set-touch sig on
$ ykman openpgp keys set-touch enc on
$ ykman openpgp keys set-touch aut on
$ ykman openpgp info
```

The `on` policy can be changed later with the admin PIN. A `fixed` policy cannot be disabled without deleting the private key from that slot, so I do not choose it merely because the word sounds stronger. The `cached` variants reduce repeated touches for a short period and trade some confirmation for convenience.

Touch proves that a person approved *an* operation at that moment. It does not prove a compromised computer displayed the same document or hostname that the card processed. I still check the command and destination before tapping the sensor.

# Test OpenPGP before configuring SSH

Inspect the result:

```console
$ gpg --card-status
$ gpg --list-options show-usage --list-secret-keys "$FPR"
```

Secret subkeys on a card are usually shown with `>` markers or stubs associated with the card serial number. Test signing and verification:

```console
$ printf 'yubikey test\n' > message.txt
$ gpg --local-user "$FPR" --detach-sign message.txt
$ gpg --verify message.txt.sig message.txt
```

Then test encryption to the public key and decryption with the card:

```console
$ gpg --recipient "$FPR" --encrypt message.txt
$ gpg --decrypt message.txt.gpg
```

The relevant operations should ask for the PIN when required and wait for touch. I do this before leaving the offline environment because a failure is much easier to repair while the complete backup is already available.

# Use the authentication subkey for SSH

GnuPG's agent can expose the OpenPGP authentication subkey through the SSH-agent protocol. Enable it:

```console
$ install -d -m 700 "$HOME/.gnupg"
$ printf '%s\n' enable-ssh-support >> "$HOME/.gnupg/gpg-agent.conf"
$ gpgconf --kill gpg-agent
$ gpg-connect-agent /bye
```

Point SSH at the socket created by `gpg-agent`:

```console
$ export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
$ ssh-add -L
```

Put that `SSH_AUTH_SOCK` export in the login environment, not in a script that starts a competing agent. Desktop keyrings and an existing `ssh-agent` are common reasons the right key appears in one terminal and vanishes in the next.

Copy the public key printed by `ssh-add -L` into the remote account's `~/.ssh/authorized_keys`, then test in a second terminal before closing the working session:

```console
$ ssh -v user@server.example
```

SSH should request the OpenPGP user PIN when needed and require a touch for the authentication operation. The server stores only the SSH public key; it does not need GnuPG or YubiKey software.

# Set up another computer

The second computer needs the public OpenPGP key, not a secret export:

```console
$ gpg --import public.asc
$ gpg --card-status
```

Reading the card lets GnuPG associate its serial number with the public subkeys and create the necessary stubs. Enable SSH support there in the same way. If GnuPG keeps asking for a card that is not inserted, inspect the card serial recorded in the secret-key stubs rather than deleting files at random.

# If the card is lost

A lost card is inconvenient, not fatal, provided the offline primary key and backups still exist. I use the offline primary key to revoke the subkeys that lived on that card, create fresh replacements, move them to a new card, and publish the updated public key wherever people retrieve it.

I do not revoke the entire primary key unless the primary key itself is compromised or no longer trustworthy. Subkeys exist partly so a missing daily device can be replaced without throwing away the identity and its certifications.

The final checklist is short:

1. Full secret-key and revocation backups restore successfully.
2. The online machine contains card stubs, not the primary secret key.
3. Signing, decryption, and SSH each require the expected PIN and touch.
4. A spare card or a documented replacement procedure exists.
5. The offline primary key can revoke and renew the subkeys.

Once those five things are true, the YubiKey is doing a precise job: it makes daily private-key use harder to steal remotely while leaving recovery under the offline primary key's control.
