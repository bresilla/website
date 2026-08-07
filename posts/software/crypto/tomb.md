+++
type = "post"
status = "published"
date = "2020-01-08"
readingtime = 14

slug = "gpg-yubikey-tomb"
title = "Back up OpenPGP keys with Tomb and cryptsetup"
thumbnail = "thumbnail.png"
foot = "Things will not calm down, as a matter of fact they will just calm up - Teal'c (Stargate)"
categories = ["LINUX"]
series = ["GPG"]
part = "1"
tags = ["yubikey", "gpg", "tomb", "cryptsetup", "backup", "pass", "cryptosetup"]

description = "Create an offline OpenPGP primary key, daily-use subkeys, an encrypted Tomb archive, and a separate LUKS2 backup."
punchline = "A hardware token is useful only after the keys that went onto it have a tested recovery and revocation plan."
tldr = "Keep the certification key offline, use replaceable subkeys day to day, store encrypted backups in more than one place, and test a restore before trusting any of it."

credits = [
    "https://www.gnupg.org/documentation/manuals/gnupg/OpenPGP-Key-Management.html",
    "https://www.gnupg.org/documentation/manuals/gnupg/GPG-Configuration.html",
    "https://dyne.org/docs/tomb/",
    "https://gitlab.com/cryptsetup/cryptsetup/-/blob/main/FAQ.md",
    "https://github.com/drduh/YubiKey-Guide",
]

[style]
    accent = "#8e2a8b"
    theme = "light"
+++

# Start with the recovery plan

The dangerous part of an OpenPGP setup is not generating keys. It is discovering, after a lost laptop or dead YubiKey, that the only recoverable copy was never actually recoverable.

My preferred layout is simple:

- One **primary certification key** stays offline. It creates and revokes subkeys.
- Separate **signing, encryption, and authentication subkeys** do daily work.
- A YubiKey holds those daily subkeys, not the only copy of them.
- Encrypted backups live on at least two independent devices or locations.
- A revocation certificate stays private but accessible during an emergency.

This is not identity-based encryption, and piling several encryption tools on top of each other does not make a backup immortal. OpenPGP protects the exported key material; Tomb gives me a portable encrypted archive; LUKS2 encrypts the removable device. The useful part is separation and redundancy, not the number of padlocks in the diagram.

# Use a clean GnuPG home

I do this work on a trusted Linux system, preferably offline, with current packages. A live environment can reduce the amount of state left behind, but it does not make compromised firmware, a malicious image, or a shoulder surfer disappear.

Create a temporary GnuPG home with strict permissions:

```console
$ export GNUPGHOME="$HOME/openpgp-offline"
$ install -d -m 700 "$GNUPGHOME"
$ gpg --version
```

Record the GnuPG version and the hash of the operating-system image in the backup notes. Future me should not need to guess what produced the files.

# Generate the primary key and subkeys

Set the identity exactly as it should appear on the public key:

```console
$ IDENTITY='Bruce Wayne <bruce.wayne@waynecorp.example>'
$ gpg --quick-generate-key "$IDENTITY" ed25519 cert 2y
```

Use a long, unique passphrase. An expiry date is not a self-destruct timer; it is a reason to revisit the key while the offline primary key is still available.

Get the full fingerprint rather than relying on a short key ID:

```console
$ FPR="$(gpg --with-colons --list-keys "$IDENTITY" | awk -F: '$1 == "fpr" {print $10; exit}')"
$ printf '%s\n' "$FPR"
$ gpg --fingerprint "$FPR"
```

Read the fingerprint from the second command and compare it with what the variable contains. Then add one subkey for each job:

```console
$ gpg --quick-add-key "$FPR" ed25519 sign 1y
$ gpg --quick-add-key "$FPR" cv25519 encr 1y
$ gpg --quick-add-key "$FPR" ed25519 auth 1y
```

Hardware support varies by YubiKey generation and firmware. Check the device's supported algorithms before choosing them. RSA is still the compatibility escape hatch when a target card or old SSH stack cannot handle the curve keys.

Inspect the finished key:

```console
$ gpg --list-options show-usage --list-secret-keys "$FPR"
```

The primary key should show certification capability, while the subkeys cover signing, encryption, and authentication.

# Export everything needed for recovery

Make a staging directory that never enters cloud sync or version control:

```console
$ export BACKUP="$HOME/openpgp-backup-$FPR"
$ install -d -m 700 "$BACKUP"
```

Export the public key, a full secret backup, a daily-use subkey backup, and ownertrust:

```console
$ gpg --armor --export "$FPR" > "$BACKUP/public.asc"
$ gpg --armor --export-secret-keys "$FPR" > "$BACKUP/secret-primary-and-subkeys.asc"
$ gpg --armor --export-secret-subkeys "$FPR" > "$BACKUP/secret-subkeys.asc"
$ gpg --export-ownertrust > "$BACKUP/ownertrust.txt"
```

The armored secret exports are still protected by their GnuPG passphrases, but I treat them as raw secrets. Anyone holding one can attempt an offline passphrase attack forever.

Modern GnuPG normally creates a revocation certificate under `openpgp-revocs.d` when the primary key is generated. Copy it into the backup:

```console
$ cp "$GNUPGHOME/openpgp-revocs.d/$FPR.rev" "$BACKUP/revocation-certificate.rev"
```

If that file is absent, create a replacement interactively:

```console
$ gpg --armor --output "$BACKUP/revocation-certificate.asc" --generate-revocation "$FPR"
```

Never publish the revocation certificate as a casual backup. A person who obtains it can invalidate the public key even without learning the private-key passphrase.

Add a small text file containing the fingerprint, identity, creation date, algorithms, expiry policy, and recovery steps. Then create checksums to catch accidental corruption:

```console
$ cd "$BACKUP"
$ sha256sum public.asc secret-primary-and-subkeys.asc secret-subkeys.asc \
    ownertrust.txt revocation-certificate.* > SHA256SUMS
```

A checksum stored beside the files detects bit rot and copying mistakes. It does not stop an attacker who can replace both the files and the checksum.

# Put the bundle in a Tomb

[Tomb](https://dyne.org/docs/tomb/) creates a LUKS-backed encrypted file and keeps its key in a separate file. Choose a size with room for notes and future exports; `128` MiB is ample for keys:

```console
$ cd "$HOME"
$ tomb dig -s 128 openpgp-backup.tomb
$ tomb forge openpgp-backup.tomb.key
$ tomb lock openpgp-backup.tomb -k openpgp-backup.tomb.key
```

Open it, copy the staging directory, and close it:

```console
$ tomb open openpgp-backup.tomb -k openpgp-backup.tomb.key
$ cp -a "$BACKUP" /media/openpgp-backup/
$ sync
$ tomb close openpgp-backup
```

Tomb may choose a different mount path if that name is already in use. Read its output instead of assuming the path.

The `.tomb` file and `.key` file are deliberately separate. I keep one copy of the Tomb on offline storage and its key on another device, with documented recovery copies in different physical locations. If both pieces always travel together, that separation adds little.

Securely erasing the staging directory on an SSD, copy-on-write filesystem, or journaled filesystem is not something `rm` can promise. The safer pattern is to create the staging area inside an encrypted temporary volume from the beginning, then close and discard that volume.

# Add an encrypted removable drive

This section destroys the selected partition. I identify it by model, serial number, and size with `lsblk` before running anything. A pasted `/dev/sdX` from a blog is not a device-discovery strategy.

Set a stable by-id path for the partition you deliberately prepared:

```console
$ lsblk --output NAME,PATH,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
$ USB_PART=/dev/disk/by-id/usb-EXAMPLE-part1
```

Format it as LUKS2, open it, and create a filesystem:

```console
$ sudo cryptsetup luksFormat --type luks2 "$USB_PART"
$ sudo cryptsetup open "$USB_PART" openpgp_backup_usb
$ sudo mkfs.ext4 -L OPENPGP_BACKUP /dev/mapper/openpgp_backup_usb
$ sudo install -d /mnt/openpgp-backup
$ sudo mount /dev/mapper/openpgp_backup_usb /mnt/openpgp-backup
```

Copy the Tomb there. Whether the Tomb key belongs on the same volume depends on the recovery plan; I prefer a separate location:

```console
$ sudo cp -a "$HOME/openpgp-backup.tomb" /mnt/openpgp-backup/
$ sync
$ sudo umount /mnt/openpgp-backup
$ sudo cryptsetup close openpgp_backup_usb
```

LUKS metadata is required to unlock the volume. Back up its header to a different encrypted device:

```console
$ sudo cryptsetup luksHeaderBackup "$USB_PART" \
    --header-backup-file /path/on/separate-encrypted-device/openpgp-backup-usb.luks-header
```

Protect that header backup like the volume itself. It does not reveal the plaintext on its own, but it enables offline passphrase guessing and may preserve keyslots that were later removed from the live device.

# Test the restore, then remove the primary key

A backup is only a theory until it restores. On another trusted system, open a copy of the Tomb into a fresh temporary `GNUPGHOME` and run:

```console
$ export GNUPGHOME="$(mktemp -d)"
$ chmod 700 "$GNUPGHOME"
$ gpg --import secret-primary-and-subkeys.asc
$ gpg --import-ownertrust ownertrust.txt
$ gpg --fingerprint "$FPR"
```

Verify the checksums, inspect the capabilities and expiry dates, sign and verify a disposable file, then destroy that temporary environment appropriately. Also practice locating the revocation certificate without importing it.

Only after the restore works do I copy the daily subkeys to a YubiKey and remove the primary secret key from the online machine. The next article covers that transfer. If the YubiKey is lost, I can revoke only its subkeys with the offline primary key, issue replacements, and keep the identity intact.

The boring pieces are the ones that save the day: two independent backups, keys and passphrases stored separately, clear labels, and a restore drill on the calendar.
