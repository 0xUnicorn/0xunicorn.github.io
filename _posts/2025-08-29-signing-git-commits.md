---
layout: post
title:  "Signing git commits"
date:   2025-08-29
author: Unicorn
tags: git pgp
---

I have for a long time not had the usecase for pgp keys, except if I needed to move some files using an insecure method, or verifying a package repo.

Now this has changed. As I'm comitting more and more open source code, I would also like people to be able to verify the authenticity of my commits.
This is primarily done by signing my git commits using a pgp key, and register the public part of that key to the git platforms like [GitHub](https://github.com) or [Codeberg/Forgejo](https://codeberg.org).

But thats not all.

What good does it make to sign a commit using a key, if breaching my account means you can just change that key?

So that means I also need to publish my key for everyone to see and so I did, on my [Contact page]({{ site.baseurl }}/contact) and [keys.openpgp.org](https://keys.openpgp.org/search?q=DE0D713876304814).

## Generate a new key

```bash
~ ❯ gpg --full-generate-key

gpg (GnuPG) 2.4.8; Copyright (C) 2025 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (14) Existing key from card
Your selection? 9

Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 10y

Key expires at Mon 27 Aug 2035 12:56:07 PM CEST
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Unicorn
Email address: unicorn@0xunicorn.com
Comment: Demo key
You selected this USER-ID:
    "Unicorn (Demo key) <unicorn@0xunicorn.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

gpg: revocation certificate stored as '~/.gnupg/openpgp-revocs.d/7A142AEE6CEF5813559AC0F25F505AFF45D7A5DB.rev'
public and secret key created and signed.

pub   ed25519 2025-08-29 [SC] [expires: 2035-08-27]
      7A142AEE6CEF5813559AC0F25F505AFF45D7A5DB
uid                      Unicorn (Demo key) <unicorn@0xunicorn.com>
sub   cv25519 2025-08-29 [E] [expires: 2035-08-27]
```

## Sign commits

**Manual**\
To sign git commits you can use `git commit -S -m "signed commits"`.

**Automatic**\
It can also be done as the default when comitting, by using a `.gitconfig` file like so:

```ini
[user]
    name = Unicorn
    email = unicorn@0xunicorn.com
    signingkey = 5F505AFF45D7A5DB

[commit]
    gpgsign = true
```

The signing key can be identified by the last 16 characters of your long key format.
So `7A142AEE6CEF5813559AC0F25F505AFF45D7A5DB` can be identified by `5F505AFF45D7A5DB`.

## Sign commits with different emails

When working on multiple projects, it can be useful to use different emails for your code commits. This is also done by the `.gitconfig` file.

By using the `includeIf` to include a dedicated git config for a `path` you can change all your git behaviour for each projects.

This is also how to change the signing key you want to use for signing commits in each project.

I use another email when working on [BornHack](https://bornhack.org) projects, this is how it looks config wise.

**.gitconfig**
```ini
[user]
    name = Christian Henriksen
    email = unicorn@0xunicorn.com
    signingkey = DE0D713876304814

[includeIf "gitdir:~/dev/bornhack/*/"]
    path = config-bornhack
```

**config-bornhack**
```ini
[user]
    email = unicorn@bornhack.org
```

## Publish the key

When committing code to [Github](https://github.com), you need to have a verified email address associated with you account
that matches the email address you specified in you pgp key.

Then follow this [guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account#adding-a-gpg-key) for uploading your key.

If you are using [Codeberg](https://codeberg.com) or a self hosted [Forgejo](https://forgejo.org) instance, the procedure is very much alike.

Next up is adding your key to one or more public lists like [keys.openpgp.org](https://keys.openpgp.org/) and to you own website.

## Copy key to other devices

When all of this was done and working, I wanted to use my new key for signing commits from other devices aswell.

This could be done by the following steps:

- Export private/public key from current device.
- Import private/public key on new device.
- Add `ultimate` trust to my key on the new device.

```bash
~ ❯ gpg --armor --export-secret-key 5F505AFF45D7A5DB > private.key
~ ❯ gpg --armor --export 5F505AFF45D7A5DB > public.key
```

Transfer you keys safely to a new device using eg. SSH, and import them using this method

```bash
~ ❯ gpg --import private.key public.key
gpg: key 5F505AFF45D7A5DB: public key "Unicorn (Demo key) <unicorn@0xunicorn.com>" imported
gpg: key 5F505AFF45D7A5DB: secret key imported
gpg: key 5F505AFF45D7A5DB: "Unicorn (Demo key) <unicorn@0xunicorn.com>" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2035-08-19
```

After the private/public keys are imported, you need to set the trust level

```bash
~ ❯ gpg --edit-key 5F505AFF45D7A5DB
gpg (GnuPG) 2.4.8; Copyright (C) 2025 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/5F505AFF45D7A5DB
     created: 2025-08-29  expires: 2035-08-27  usage: SC
     trust: unknown       validity: unknown
ssb  cv25519/9043BC2850B2CCE5
     created: 2025-08-29  expires: 2035-08-27  usage: E
[ unknown] (1). Unicorn (Demo key) <unicorn@0xunicorn.com>

gpg> trust
sec  ed25519/5F505AFF45D7A5DB
     created: 2025-08-29  expires: 2035-08-27  usage: SC
     trust: unknown       validity: unknown
ssb  cv25519/9043BC2850B2CCE5
     created: 2025-08-29  expires: 2035-08-27  usage: E
[ unknown] (1). Unicorn (Demo key) <unicorn@0xunicorn.com>

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

sec  ed25519/5F505AFF45D7A5DB
     created: 2025-08-29  expires: 2035-08-27  usage: SC
     trust: ultimate      validity: unknown
ssb  cv25519/9043BC2850B2CCE5
     created: 2025-08-29  expires: 2035-08-27  usage: E
[ unknown] (1). Unicorn (Demo key) <unicorn@0xunicorn.com>
Please note that the shown key validity is not necessarily correct
unless you restart the program.

gpg> save
```

For verifying your changes to the trust level use:

```bash
~ ❯ gpg --list-keys --keyid-format long
---------
pub   ed25519/5F505AFF45D7A5DB 2025-08-29 [SC] [expires: 2035-08-27]
      7A142AEE6CEF5813559AC0F25F505AFF45D7A5DB
uid                 [ultimate] Unicorn (Demo key) <unicorn@0xunicorn.com>
sub   cv25519/9043BC2850B2CCE5 2025-08-29 [E] [expires: 2035-08-27]
```

