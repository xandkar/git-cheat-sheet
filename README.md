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
    git push $REMOTE_NAME :refs/tags/$tag
done
```


Get an older version of a file
------------------------------

```sh
git cat-file -p $COMMIT_DIGEST:$FILE_PATH
```


Splitting a commit
------------------

From [docs](https://git-scm.com/docs/git-rebase#_splitting_commits):

- Start an interactive rebase with `git rebase -i <commit>^`, where `<commit>`
  is the commit you want to split.
- Mark the commit you want to split with the action "edit".
- When it comes to editing that commit, execute `git reset HEAD^`. The effect
  is that the HEAD is rewound by one, and the index follows suit. However, the
  working tree stays the same.
- Now add the changes to the index that you want to have in the first commit
  (`git add`)
- Commit the now-current index with whatever commit message is appropriate now.
- Repeat the last two steps until your working tree is clean.
- Continue the rebase with `git rebase --continue`.


Merge 2 repos
-------------

```sh
$ ls -1
repo_A
repo_B
$ cd repo_B
$ git remote add repo_A ../repo_A
$ git fetch repo_A --tags
$ git merge --allow-unrelated-histories repo_A/$BRANCH
$ git remote remove repo_A
```
