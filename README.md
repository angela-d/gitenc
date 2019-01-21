# Gitenc - Encrypt Sensitive Data in Git

Gitenc was born out of annoyance while submitting database dumps and raw config files to private Git repositories.

- *What if the remote git host gets hacked?*

I didn't want to sit and manually encrypt sensitive files with each new commit, so Gitenc was developed to automate the process.

There are existing solutions that achieve the same (like git-crypt) or OpenSSL flags; git-crypt appears to be abandoned and I like using GPG - though I have nothing against OpenSSL for the same function.


## What does it do?
Gitenc is a simple shell script that works as a placeholder for `git add` and will parse filenames for sensitive names from `git status` and apply [GPG encryption](https://gnupg.org/) as needed (filenames matching **config**, **connection** or **sqlbackup**) while handing everything off to git.

 It essentially automates long shell commands (allowing for git automation!)

## Dependencies
- Linux-based OS (may work in Mac or BSD, but untested).  Only tested on Debian, but should work in others.
- Git `apt install git`
- GPG `apt install gpg` (may be `gpg2` on some distros)
- Bash (Pre-installed on GNU-based systems; coreutils on Mac)

## To Use

- Clone the repo and launch setup to generate the necessary config
```bash
git clone https://github.com/angela-d/gitenc/gitenc.git  ~/gitenc && cd ~/gitenc && ./gitenc setup
```

- You will be prompt to choose a password for the GPG encryption (which you'd also use to decrypt the file)

### Example usage (via cron)
```bash
00 02 * * * cd /home/user/public_html && gitenc add . && git commit -m "Adding new changes" && git push && cd
```
- If no "sensitive" files are found, Git routines proceed like normal.

### Add a symbolic link
As root/sudo:
```bash
ln -s /home/YOUR_USER/gitenc/gitenc /bin/gitenc
```
- Instead of `git add` use `gitenc add` during your normal workflow.  If you do not have root access, you can call gitenc by its full path: `/home/YOUR_USER/gitenc/gitenc add filename`

That's it!

## Decrypt a File
Via command-line
```bash
gpg --decrypt filename.gpg
```

***
### What is Gitenc's purpose?
Assuming you have sensitive data that isn't already encrypted at rest..
The alternative to using something like Gitenc is submitting plaintext sensitive data to private git repos.  Not to mention, accidental pushes, etc; since Gitenc pays attention to common filenames, there is little room for error.

**Why encrypt at all, if the repos are private?**

You cannot bank on the fact the Git host with your private repos will never be hacked.

### How secure is this?
Gitenc doesn't do any encryption of it's own, it piggybacks on the existing GPG mechanism.

One might assume if your private Git repo was involved in a hack, the system you're generating the encryption on would be an entirely separate entity - so any "risk" is close to zero.

With that said, its as secure as your chosen password (use something long, and free of patterns), running version of GPG and most importantly server security.. if your host system gets taken, the GPG password config would be yanked, too.

**Non-interactive, plaintext passwords jeapordize security**

Sure does. Though the purpose of Gitenc isn't to encrypt where the file resides, but for *remotely* hosted versions, on potentially-untrustworthy third parties like Github, Bitbucket, etc.

If you feel an attacker can sniff a GPG password from a dotfile off *your* system, from *your* user, you have bigger security concerns that command your attention.

### What cipher algorithm is used to encrypt?
Dependent on your system/preferences and version of GPG.  My Debian 9 system's default is AES256.  To see the list of ciphers available to you, under **Ciphers**:
```bash
gpg --version
```
Gitenc will use the default.  You can override by appending your choice (using CAMELLIA256 in the example):
```bash
gpg --cipher-algo CAMELLIA256 # the cipher you want to use
```
to the line in the lockdown() function from the source:
```bash
gpg --batch -c --cipher-algo CAMELLIA256 --passphrase-file "$GITENC_CONF" "$1"
```

### What happens when the current GPG algorithm used has been penetrated?
All of your previously encrypted files become penetrable.  Routinely clear the history for those files, periodically rotate the repositories with sensitive data, or pay attention to GPG release changelogs.

### Are the diffs still tracked after the encryption?
For the triggered files, no.  I've never had a reason to track (sensitive) config changes, so that's not something I felt necessary to work out a solution for.  The rest of the files are managed purely by git itself.  Gitenc only concerns itself with files whose filenames match the sensitive conditions.

### Is the original file encrypted?
No, the original file is never touched.

***
## Worth Mentioning
- If you use Gitenc to delete/re-add previously committed sensitive files, that data is still in the repo (even after deletion) for hackers to find during future breaches.  For good measure, either erase the commit history or start a fresh repository (or simply change the passwords/sensitive data!).

***
### Remove the sensitive files from being tracked (previously tracked only - on a live server) -- optional
If you're running Gitenc on a new repo (no history), mirrors or backups only (or have changed any previously committed sensitive data), you won't need to mess with any of this.
- Run Gitenc and let it do its thing
- Open `.git/config` and **remove the public/live server** repo, like so:
```bash
url = git@bitbucket.org:{user}/{repo}.git
#pushurl = ssh://{user}@{server}:{port}/home/{user}/public_html/
```
- url = The repo I'm most concerned about
- pushurl = The live server that receives mirror updates from local pushes
- The sensitive file still shows as untracked, which won't necessarily hurt anything, but I'd like to clean it up:
```bash
git rm --cached sensitivefile.conf
```
Note that the file will remain locally but will be deleted from the live server during the next push.

- Commit it (this is removing it from Bitbucket, only.. for now)
```bash
git commit -m "Untracking file" && git push
```
- Prepare the unencrypted file to go back on the server after the pushurl is restored (open your sftp or ssh client and simply restore them)
- Uncomment the `pushurl` and run `git push` -- the sensitive files are now deleted, so quickly restore any affected file.

That's it.  Now they will NOT be tracked for future changes, unless you force them through later on.  (This does not remove the file from the *history*.)

### Public access to encrypted files
- If you're going to keep the .gpg files on a public server, ensure you add regex to prevent direct access - although they're encrypted, it's best not to tempt.

Apache 2.4+ example:
```aconf
<FilesMatch "(?i)((\.yml|\.gpg))">
  Require all denied
</FilesMatch>
```

## Backup before you use Gitenc!
## Test your encrypted files (make sure you can decrypt them)!  
Different systems might handle the encryption syntax differently.. please report in the issues if yours does.
