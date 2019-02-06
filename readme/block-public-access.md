# Gitenc Usage - Block Public Access to Encrypted Files
If you're going to keep the .gpg files on a public server, ensure you add regex to prevent direct access - although they're encrypted, it's best not to tempt.

Apache 2.4+ example:
```aconf
<FilesMatch "(?i)((\.yml|\.gpg))">
  Require all denied
</FilesMatch>
```
