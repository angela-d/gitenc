# Gitenc Usage - Clear from Git History

Remove a file from the repository *and* the history.  Effectively remove any trace of the file's existence.  Useful for purging sensitive files you've tracked before you used Gitenc.

In your local copy of the repository, run the following tasks:

Replace **sqlbackup-20190128.sql.bz2** for the filename you wish to permanently remove:
```bash
git filter-branch --force --index-filter \
'git rm --cached --ignore-unmatch sqlbackup-20190128.sql.bz2' \
--prune-empty --tag-name-filter cat -- --all
```

You'll receive output similar to:
>...
>
>Rewrite 4hf89erbdel43... (9/9) (0 seconds passed, remaining 0 predicted)    rm 'sqlbackup-20190128.sql.bz2'
>
>Ref 'refs/heads/master' was rewritten
>Ref 'refs/remotes/origin/master' was rewritten
>WARNING: Ref 'refs/remotes/origin/master' is unchanged

Push the changes into the remote repository:
```bash
git push --force --all
```
