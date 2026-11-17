+++
type = "key"
visibility = "indirect"
title = "Public key"
slug = "pgp"
+++

My OpenPGP key lives on my YubiKeys. Fingerprint:

`EB04 9116 CF70 D706 FFBF  4329 C7B3 AFBD D37B 77FC`

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEavjgIBYJKwYBBAHaRw8BAQdA+ag7NrKtSXGSx4xY8kcAEkHMk9W4j3mtEN85
Jdm8qi60KlRyaW0gQnJlc2lsbGEgPHRyaW0uYnJlc2lsbGFAYnJlc2lsbGEuY29t
PoiQBBMWCgA4FiEE6wSRFs9w1wb/v0Mpx7OvvdN7d/wFAmr44CACGwMFCwkIBwIG
FQoJCAsCBBYCAwECHgECF4AACgkQx7OvvdN7d/zujgD9EAxo5DgQHk+NtZYSGEvM
OKjqjDXSsYGyY7v2ri6tZrEBAJrCsuoNGAaBmG+4hZdr5Fg25nZsR3qKZXIXBSVA
6ooBuDgEavjgIBIKKwYBBAGXVQEFAQEHQJgac8OXBMgkv6xx67qmPRBdQZSfyiaH
ZXTNUPd6J110AwEIB4h+BBgWCgAmFiEE6wSRFs9w1wb/v0Mpx7OvvdN7d/wFAmr4
4CACGwwFCQlmAYAACgkQx7OvvdN7d/x+LgD/dw+skmcOF1rU5Lp9RGnWY5i/YlJQ
TMwAxRbo6/Rc288A/R2ANaDppAweeethnIgEl2P49Ow/T85oaUjtpT56tbABuDME
avjgIBYJKwYBBAHaRw8BAQdAP5a/IQFs9wzyrJbEhihvLrmDcuUSA/acEau5pACb
I0aIfgQYFgoAJhYhBOsEkRbPcNcG/79DKcezr73Te3f8BQJq+OAgAhsgBQkJZgGA
AAoJEMezr73Te3f8tygBANOPBy71+94WvhEb0D9BaME/9Z6X9MeZV0iZzlVNCTAG
AQDTlXGjBgfEvT/SBhJK74NMpeoEGGCyY9kk604YgG02Aw==
=hO9C
-----END PGP PUBLIC KEY BLOCK-----
```

The SSH key is the authentication subkey of the same OpenPGP key. To let me log in to a machine, add it to `~/.ssh/authorized_keys` there.

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAID+WvyEBbPcM8qyWxIYoby65g3LlEgP2nBGruaQAmyNG openpgp:0xDB94731B
```

<p><a href="/public.asc" download="bresilla.asc">Download OpenPGP key (.asc)</a> <a href="/bresilla.pub" download="bresilla.pub">Download SSH key (.pub)</a></p>
