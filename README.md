git-cheat-sheet
===============

How to do stuff with git


Splitting a repo
----------------

### Remove history of all but select files in cloned repo
```sh
$ cp -Rp old new
$ cd new
$ git clean -dfx
$ git gc --aggressive --prune=now
$ git remote rm origin
$ git filter-branch \
    --prune-empty \
    --index-filter '
        git ls-tree -r --name-only HEAD \
        | grep -v file_i_want_to_keep_1 \
        | grep -v file_i_want_to_keep_.. \
        | grep -v file_i_want_to_keep_n \
        | xargs git rm --cached -r --ignore-unmatch
    ' \
    HEAD
$ git gc --aggressive --prune=now
```

### Remove history of previously-removed (not in current tree) files
```sh
$ git log --pretty=format: --name-status \
    | awk '$0 != "" {print $2}' \
    | sort -u > /tmp/tree.old
$ git ls-tree -r --name-only HEAD > /tmp/tree.new
$ git filter-branch \
    --prune-empty \
    --index-filter '
        grep -Fvxf /tmp/tree.new /tmp/tree.old \
        | xargs git rm --cached -r --ignore-unmatch
    ' \
    HEAD
```
