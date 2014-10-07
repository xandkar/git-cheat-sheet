git-cheat-sheet
===============

How to do stuff with git


Splitting a repo
----------------

##### Remove history of all but select files in cloned repo
```sh
$ git clone old new
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

##### Remove history of previously-removed (not in current tree) files
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


Inserting a new root commit
---------------------------
```sh
git checkout --orphan $TEMP_BRANCH
git rm -rf .
git commit --allow-empty -m $INIT_COMMIT_MSG
git rebase --onto $TEMP_BRANCH --root $MAIN_BRANCH
git branch -d $TEMP_BRANCH
```


Deleting all tags, locally and remotely
---------------------------------------
```sh
for tag in `git tag`;
do
    git tag -d $tag
    git push origin :refs/tags/$tag
done
```
