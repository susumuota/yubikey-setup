# YubiKey setup memo

This document describes how to set up a YubiKey.

- Generate gpg keys
- Backup the keys
- Use YubiKey for SSH
- Use YubiKey for signing GitHub commits
- Use YubiKey for 2 Factor Authentication (2FA) on (e.g. Google, Amazon, Twitter, etc.) (WIP)
- Use YubiKey for Passkey Authentication (Yahoo! JAPAN) (WIP)
- Use YubiKey for Nostr (WIP)

## References

- https://support.yubico.com/hc/en-us/articles/360013790259-Using-Your-YubiKey-with-OpenPGP
- https://github.com/drduh/YubiKey-Guide
- https://musigma.blog/2021/05/09/gpg-ssh-ed25519.html
- https://keens.github.io/blog/2021/03/23/yubikeywotsukau_openpghen/
- https://qiita.com/shun-shobon/items/96f08aa09a30c26a55b5

## Buy a YubiKey.

- YubiKey 5+ Series which supports OpenPGP.
- USB drive or SD card for key backup.

## Install

- For macOS,

```sh
brew install gnupg
brew install ykman
```

## Reset

- This commands will reset the YubiKey OpenPGP data (not FIDO2, OTP, PIV, etc.). See details [here](https://docs.yubico.com/software/yubikey/tools/ykman/OpenPGP_Commands.html#ykman-openpgp-reset-options).
- **ARE YOU SURE YOU WANT TO RESET?**

```sh
ykman openpgp reset
rm -rf ~/.gnupg      # if you already have gpg keys, backup them somewhere
```

- Default PINs are here. We will change them later.

```sh
PIN:         123456
Reset code:  NOT SET
Admin PIN:   12345678
```

## Generate a master key and subkeys

- Choose the key algorithm, RSA or ECC. I chose ECC.
- If you use RSA, choose 4096 bits. See details [here](https://github.com/drduh/YubiKey-Guide#master-key).
- If you use ECC, I recommend to use Curve 25519. See details [here](https://soatok.blog/2022/05/19/guidance-for-choosing-an-elliptic-curve-signature-algorithm-in-2022/). I chose Curve 25519.
- There are 4 types of keys: S, C, E, and A. S is for signing, C is for certification, E is for encryption, and A is for authentication.
- `gpg` default procedure will generate a master key for S and C, and a subkey for E.
- We will create a subkey for A later.

> **Note**: Email address must be set to the same email address as your GitHub account. Otherwise, you cannot sign commits with GPG. In my case, `1632335+susumuota@users.noreply.github.com`.

```sh
gpg --expert --full-gen-key

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 9

Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Susumu OTA
Email address: 1632335+susumuota@users.noreply.github.com
Comment:
You selected this USER-ID:
    "Susumu OTA <1632335+susumuota@users.noreply.github.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

# input passphrase

pub   ed25519 2023-03-17 [SC]
      KEY_FINGERPRINT_HERE
uid                      Susumu OTA <1632335+susumuota@users.noreply.github.com>
sub   cv25519 2023-03-17 [E]
```

- Now, we have a master key for S and C with Curve 25519 and a subkey for E with Curve 25519.
- Save the key ID. e.g. `3AA5C34371567BD2`.

```sh
gpg --list-secret-keys --keyid-format=long
export GPG_KEY_ID="3AA5C34371567BD2"  # replace with your key ID
```

## Add subkeys for A

- Right now, we have the keys for S, C and E.
- We need to add a subkey for A.

```sh
gpg --expert --edit-key $GPG_KEY_ID

# add a subkey for A

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 11

Possible actions for this ECC key: Sign Authenticate
Current allowed actions: Sign

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? a

Possible actions for this ECC key: Sign Authenticate
Current allowed actions: Sign Authenticate

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for this ECC key: Sign Authenticate
Current allowed actions: Authenticate

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y

# input passphrase

gpg> save
```

- Confirm the keys

```sh
gpg -k
-----------------------------
pub   ed25519 2023-03-17 [SC]
      KEY_FINGERPRINT_HERE
uid           [ultimate] Susumu OTA <1632335+susumuota@users.noreply.github.com>
sub   cv25519 2023-03-17 [E]
sub   ed25519 2023-03-17 [A]
```

- Verify the keys

```sh
brew install hopenpgp-tools
rehash
gpg --export $GPG_KEY_ID | hokey lint
```

## Change expiration time for the subkeys

- Set the expiration time for the subkeys to 1 year. Renew them every year.

```sh
gpg --expert --edit-key $GPG_KEY_ID

gpg> key 1  # select E key
gpg> expire
Changing expiration time for a subkey.
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Sun Mar 17 00:41:24 2024 JST
Is this correct? (y/N) y

# input passphrase

gpg> key 1  # unselect E key
gpg> key 2  # select A key
gpg> expire
Changing expiration time for a subkey.
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Sun Mar 17 00:41:45 2024 JST
Is this correct? (y/N) y

# input passphrase

gpg> save
```

- Confirm the keys

```sh
gpg -k
-----------------------------
pub   ed25519 2023-03-17 [SC]
      KEY_FINGERPRINT_HERE
uid           [ultimate] Susumu OTA <1632335+susumuota@users.noreply.github.com>
sub   cv25519 2023-03-17 [E] [expires: 2024-03-16]
sub   ed25519 2023-03-17 [A] [expires: 2024-03-16]
```

## Backup keys

- Follow these instructions to export your secret keys
  - https://github.com/drduh/YubiKey-Guide#export-secret-keys
  - https://keens.github.io/blog/2021/03/23/yubikeywotsukau_openpghen/
- Prepare a USB drive or a SD card or something to backup the keys.
- Create a tmpfs for temporary workspace on local machine.

```sh
mkdir -p workspace
sudo mount_tmpfs workspace  # for macOS
# sudo mount -t tmpfs tmpfs workspace  # for Linux
cd workspace
```

- Export the keys.

```sh
gpg --armor --export-secret-keys $GPG_KEY_ID > gpg_secret_keys.asc
gpg --armor --export-secret-subkeys $GPG_KEY_ID > gpg_secret_subkeys.asc
gpg --armor --export $GPG_KEY_ID > gpg_public_keys.asc
```

- Confirm the exported keys whether they can be imported.

```sh
gpg --import gpg_secret_keys.asc
gpg --import gpg_secret_subkeys.asc
gpg --import gpg_public_keys.asc
gpg -k
```

- Create the revocation certificate

```sh
gpg --output gpg_revoke.asc --gen-revoke $GPG_KEY_ID

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 1
Enter an optional description; end it with an empty line:
>
Reason for revocation: Key has been compromised
(No description given)
Is this okay? (y/N) y
```

- Copy the keys to the USB drive or the SD card.

```sh
mkdir -p /Volumes/GPGBAK1/20230317-01
mkdir -p /Volumes/GPGBAK1/20230317-02
cp -p gpg_* /Volumes/GPGBAK1/20230317-01
cp -p gpg_* /Volumes/GPGBAK1/20230317-02
mkdir -p /Volumes/GPGBAK2/20230317-01
mkdir -p /Volumes/GPGBAK2/20230317-02
cp -p gpg_* /Volumes/GPGBAK2/20230317-01
cp -p gpg_* /Volumes/GPGBAK2/20230317-02
```

## Configure YubiKey

- https://github.com/drduh/YubiKey-Guide#configure-smartcard

```sh
gpg --card-edit
gpg/card> admin
```

```sh
gpg/card> kdf-setup
gpg/card> list       # KDF setting ......: on
```

```sh
gpg/card> passwd

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

# select 1, 3, 4 then q

gpg/card> quit  # needs to quit to save PIN
```

## Set name, etc.

```sh
gpg --card-edit
gpg/card> admin
gpg/card> name
gpg/card> salutation
gpg/card> lang
gpg/card> login
# gpg/card> url   # setup this later
gpg/card> list
gpg/card> quit
```

```sh
gpg --edit-key $GPG_KEY_ID

gpg> keytocard
Really move the primary key? (y/N) y
Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1

# input passphrase and PIN

gpg> key 1  # select E key
gpg> keytocard
Please select where to store the key:
   (2) Encryption key
Your selection? 2

# input passphrase and PIN

gpg> key 1  # unselect E key
gpg> key 2  # select A key
gpg> keytocard
Please select where to store the key:
   (3) Authentication key
Your selection? 3

gpg> save
```

- Confirm the keys

```sh
gpg --card-edit

# confirm these lines

Key attributes ...: ed25519 cv25519 ed25519

Signature key ....: xxxx xxxx xxxx xxxx xxxx  xxxx xxxx xxxx xxxx xxxx
      created ....: 2023-03-17 15:30:34
Encryption key....: xxxx xxxx xxxx xxxx xxxx  xxxx xxxx xxxx xxxx xxxx
      created ....: 2023-03-17 15:30:34
Authentication key: xxxx xxxx xxxx xxxx xxxx  xxxx xxxx xxxx xxxx xxxx
      created ....: 2023-03-17 15:35:09

gpg/card> quit
```

## Test sign, encrypt, verify and decrypt

- Plug the YubiKey. Test sign, encrypt, verify and decrypt.

```sh
echo test > test.txt
gpg -sea --default-recipient-self test.txt  # encrypt and sign
gpg -d test.txt.asc                         # verify and decrypt
```

- Unplug the YubiKey. It should fail to decrypt.

```sh
gpg -d test.txt.asc
```

- Plug the YubiKey again. It should succeed to decrypt.

```sh
gpg -d test.txt.asc
```

- If you confirmed the keys are stored in the YubiKey, you can delete the temporary workspace on local machine.

```sh
cd ..
sudo umount workspace
```

## SSH

- Follow the instructions in the following link.
  - https://github.com/drduh/YubiKey-Guide#ssh
- Create `gpg-agent.conf`.

```sh
brew install pinentry-mac
rehash
which pinentry-mac  # /usr/local/bin/pinentry-mac
cd ~/.gnupg
wget https://raw.githubusercontent.com/drduh/config/master/gpg-agent.conf
# edit gpg-agent.conf, comment out default pinentry-program and enable pinentry-mac
# pinentry-program /usr/local/bin/pinentry-mac
```

- Add the following to `~/.zshrc`.

```sh
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpgconf --launch gpg-agent
```

- Save the public key.

```sh
ssh-add -L | grep "cardno" > ~/.ssh/id_ed25519_yubikey.pub
```

- Add the public key to the server's `~/.ssh/authroized_keys`.

```sh
ssh -p 10022 ota@pi3.local
emacs .ssh/authorized_keys
# add the content of id_ed25519_yubikey.pub
exit
```

- Add the following to `~/.ssh/config`.

```sh
cat << EOF >> ~/.ssh/config
Host pi3.local
    IdentitiesOnly yes
    IdentityFile ~/.ssh/id_ed25519_yubikey.pub
EOF
```

- Delete `[pi3.local]` lines in `~/.ssh/know_hosts`.

```sh
emacs ~/.ssh/known_hosts
# delete all of the [pi3.local] lines
```

- Try to login again.

```sh
ssh -p 10022 ota@pi3.local -vvv
```

## GitHub

- Follow these instructions.
  - https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification
  - https://github.com/drduh/YubiKey-Guide#github

- Backup the existing `~/.gitconfig`.

```sh
cp -p ~/.gitconfig ~/.gitconfig.bak
```

- Show public key and paste it to GitHub.
  - https://github.com/settings/gpg/new

```sh
gpg --armor --export $GPG_KEY_ID
```

- Unset `gpg.format` if it is set.

```sh
git config --global --unset gpg.format
cat ~/.gitconfig
```

- Copy the long form of the GPG key ID. e.g. `3AA5C34371567BD2`

```sh
gpg --list-secret-keys --keyid-format=long  # copy your key ID
export GPG_KEY_ID="3AA5C34371567BD2"        # replace with your key ID
git config --global user.signingkey $GPG_KEY_ID
git config --global commit.gpgsign true     # add this to omit -S option
cat ~/.gitconfig
diff -u ~/.gitconfig.bak ~/.gitconfig
```

- Commit and push something.

```sh
git add README.md
git commit -m "test"  # or git commit -S -m "test" if you didn't set commit.gpgsign true
git push origin main
```

- See the commit log on GitHub. You will see green `Verified` badge and click it to confirm `GPG key ID` is same as `$GPG_KEY_ID`.

- You can also try to unplug and plug the YubiKey to test `pinentry`.
  - You will see `pinentry` dialog when `git commit` and `git push`.

That's all.

## License

MIT

## Author

S. Ota

https://github.com/susumuota/yubikey-setup
