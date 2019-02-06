# Gitenc Usage - Change Cipher Algorithm

The cipher algorithm used by default, is dependent on your system/preferences and version of GPG.  My Debian 9 system's default is AES256.  To see the list of ciphers available to you, under **Ciphers**; run (via CLI):
```bash
gpg --version
```
Gitenc will use the default.  You can override by appending your choice (using CAMELLIA256 in the example):
```bash
gpg --cipher-algo CAMELLIA256 # the cipher you want to use
```
to the line in the lockdown() function from the source, like so:
```bash
"$GPG_LOC" --batch -c --cipher-algo CAMELLIA256 --passphrase-file "$GITENC_CONF" "$1"
```
