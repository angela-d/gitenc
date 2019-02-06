# Gitenc - Encrypt Sensitive Data in Git

Gitenc was born out of annoyance while submitting database dumps and raw config files to private Git repositories.

- *What if the remote git host gets hacked?*

I didn't want to sit and manually encrypt sensitive files with each new commit, so Gitenc was developed to automate the process.

There are existing solutions that achieve the same (like git-crypt) or OpenSSL flags; git-crypt appears to be abandoned and I like using GPG - though I have nothing against OpenSSL for the same function.

I also wanted behavior that I felt should be simple enough to reside in a small codebase and not require a bunch of steps or maintenance.


## What does it do?
Gitenc is a simple shell script that works as a wrapper for `git add` and will parse filenames for sensitive names from `git diff` and apply [GPG encryption](https://gnupg.org/) as needed (filenames matching **config**, **connection** or **sqlbackup**) while handing everything off to git.

 It essentially automates long shell commands (allowing for git automation!)

## Compatibility
Built for GNU/Linux-based systems.

Tested on Debian 9, Ubuntu 16.04, 18.04 and CentOS 7, but should also work in others.

## Dependencies
- Git `apt install git`
- GPG `apt install gpg` (may be `gpg2` on some distros)
- Bash (Pre-installed on GNU-based systems; coreutils on Mac)

## To Use / Install

- Clone the repo and launch setup to generate the necessary config
```bash
git clone https://github.com/angela-d/gitenc.git  ~/gitenc && cd ~/gitenc && ./gitenc setup
```

- You will be prompt to choose a password for the GPG encryption (which you'd also use to decrypt the file)

### Example usage (via cron)
```bash
00 02 * * * cd /home/user/public_html && gitenc add . && git commit -m "Adding new changes" && git push && cd
```
- If no "sensitive" files are found, Git routines proceed like normal.

### Add a symbolic link
As root/sudo (assuming of course, you installed via the installation suggested path):
```bash
sudo ln -s $HOME/gitenc/gitenc /bin/gitenc
```
- Instead of `git add` use `gitenc add` during your normal workflow.  
- If you do not have root access, you can call gitenc by its full path: `/home/YOUR_USER/gitenc/gitenc add filename`

That's it!

***
***
## Additional Gitenc Usage

- [Decrypt Encrypted Files](readme/decrypt-files.md)
- [Change Cipher Algorithm](readme/change-cipher-algo.md)

Optional Usage
- [Untrack Sensitive Files](readme/untrack-sensitive-files.md) - Previously committed files (not necessary for new repos)
- [Block Public Access](readme/block-public-access.md) - If storing encrypted files in a public web directory
- [Delete from History](readme/delete-from-git-history.md) - *Previously committed* files

***
### What is Gitenc's purpose?
Assuming you have sensitive data that isn't already encrypted at rest..
The alternative to using something like Gitenc is submitting plaintext sensitive data to private git repos.  Not to mention, accidental pushes, etc; since Gitenc pays attention to common filenames, there is little room for error.

**Why encrypt at all, if the repos are private?**

You cannot bank on the fact the Git host with your private repos will never be hacked.

**Why submit confidential files to a Git repo, at all?**

Your use case may vary - my personal reason behind building something like Gitenc is because I use private repos as remote backups, so I definitely would *like* to have copies of the config and databases, when at all possible.

Before (without manually encrypting sensitive data), you'd be placing an immense amount of trust in the Git host; now, you don't have to.  If their system gets taken, the most sensitive data inside the repos remains secure.

If you use Git repos for other purposes and load up your .gitignore with all of your sensitive files, you'll have absolutely no use for Gitenc.

### How secure is this?
Gitenc doesn't do any encryption of it's own, it utilizes the existing GPG cryptography.

One might assume if your private Git repo was involved in a hack, the system you're generating the encryption on would be an entirely separate entity - so any "risk" is close to zero.

With that said, its as secure as your chosen password (use something long, and free of patterns), running version of GPG and most importantly server security.. if your host system gets taken, the GPG password config would be yanked, too.

**Non-interactive, plaintext passwords jeapordize security**

Sure does. Though the purpose of Gitenc isn't to encrypt where the file resides, but for *remotely* hosted versions, on potentially-untrustworthy third parties like Github, Bitbucket, etc.

If you feel an attacker can sniff a GPG password from a dotfile off *your* system, from *your* user, you have bigger security concerns that command your attention.

### What happens when the current GPG algorithm used has been penetrated?
All of your previously encrypted files become penetrable.  Routinely clear the history for those files, periodically rotate the repositories with sensitive data, or pay attention to GPG release changelogs.

### Are the diffs still tracked after the encryption?
For the triggered files, no.  I've never had a reason to track (sensitive) config changes, so that's not something I felt necessary to work out a solution for.  The rest of the files are managed purely by git itself.  Gitenc only concerns itself with files whose filenames match the sensitive conditions.

### Is the original file encrypted?
No, the original file is never touched.

***
## Worth Mentioning
- If you use Gitenc to delete/re-add previously committed sensitive files, that data is still in the repo (even after deletion) for hackers to find during future breaches.  For good measure, either erase the commit history or start a fresh repository (or simply change the passwords/sensitive data!).

- Gitenc will only encrypt a triggered file if it's mtime (modified time) is within the last 24 hours.  If you have an older file that isn't already listed in .gitignore you'd like Gitenc to handle, change the mtime: `touch filename.conf`
***

## Backup before you use Gitenc!
## Test your encrypted files (make sure you can decrypt them)!  
Different systems might handle the encryption syntax differently.. please report in the issues if yours does.

### Thanks
**alfunx** - Code suggestions and improvements
