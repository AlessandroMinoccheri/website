# alessandrominoccheri.github.io

This is the source-code of [my personal GitHub page](http://alessandrominoccheri.github.io/).
I will post my thoughts about what I do here, but I also accepts any fixes to my articles/tutorials/guides.

## Add new post

```
hugo new post/new-post.md
```

## Deploy

To deploy launch from website repository

```
./deploy.sh
```

## Install from zero

Remove inside `.git/config` submodules if already exists.
Remove folder `.git/modules/*` if already exists.

Launch:

```
rm -rf alessandrominoccheri.github.io
git rm -r --cached alessandrominoccheri.github.io
rm -rf public
git rm -r --cached public
git rm -r --cached .git/modules/public
git submodule add https://github.com/AlessandroMinoccheri/alessandrominoccheri.github.io public

```

If you have problems with authentication on deploy you can regenerate a new token from git->Settings->DeveloperSettings and regenerate a new token.
Use that for password when you deploy.

