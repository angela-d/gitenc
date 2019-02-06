# Gitenc Usage - Untrack Sensitive Files *Previously Committed*

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
