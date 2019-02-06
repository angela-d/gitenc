# Gitenc Usage - Decrypt a File

You can decrypt files that were encrypted by Gitenc using basic GPG commands.

Desktop environments also offer GUI-based options for decrypting encrypted files.


**Output contents to the command-line:**
```bash
gpg --decrypt filename.gpg
```
**Output contents to a file:**
```bash
gpg --output filename.conf --decrypt filename.gpg
```
