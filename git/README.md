# git cheat sheet

## Table of contents

* [Resources](#resources)
* [Clone](#clone)
* [Checkout](#checkout)
* [Reset](#reset)
* [Commit](#commit)
* [Remote](#remote)
* [Branch](#branch)
* [Merge](#merge)
* [Remove](#remove)
* [Push](#push)
* [Tags](#tags)
* [Rebase](#rebase)
* [Diff](#diff)
* [Stash](#stash)
* [Blame](#blame)
* [List files](#list-files)
* [Log](#log)
* [Reflog](#reflog)
* [Purge](#purge)
* [Cherry pick](#cherry-pick)
* [Submodules](#submodules)
* [Configuration](#configuration)

### Resources

* [git-scm](http://git-scm.com)
* [Github help](https://help.github.com/)
* [git immersion](http://gitimmersion.com/lab_01.html)
* [git ready](http://gitready.com)
* [git cheat sheet](http://www.ndpsoftware.com/git-cheatsheet.html)

### Clone

```sh
# clone a repo and give it an explicit name
git clone <repo_url> <local_repo_name>
```

### Checkout

```sh
# create local branch and switch to it
git checkout -b <local_branch_name>

# switch branch
git checkout <branch_name>

# discard changes made to a file since last commit
git checkout -- <file>
```

### Reset

```sh
# reset the staged changes made to a specific file
git reset HEAD <file>

# reset the last commit and put the changes made into the staging area
git reset --soft HEAD^

# reset the last commit and delete the changes
git reset --hard HEAD^

# reset the last two commits and delete the changes
git reset --hard HEAD^^
```

### Commit

```sh
# commit changes for all tracked files
git commit -a -m "Message"

# after a fixed merge conflict this will automatically create a commit message
git commit -a

# add made changes to the last commit
git commit --amend -m "Message"
```

### Remote

```sh
# show remotes
git remote -v

# add remote
git remote add <remote_name> <address>

# show information about remote including branches
git remote show <remote_name>

# delete stale remote branches referenced locally
git remote prune <remote_name>
```

### Branch

```sh
# show local branches
git branch

# show remote branches already known locally
git branch -r

# create a local branch
git branch <local_branch_name>

# delete local branch
git branch -d <local_branch_name>

# force delete local branch
git branch -D <local_branch_name>
```

### Merge

```sh
# merge a branch into the current one
git merge <branch_name>
```

### Remove

```sh
# remove file from working directory and stages the deletion
git rm <file_name>

# stop tracking a file
git rm --cached <file_name>
```

### Move

```sh
# renames a file
git mv <original_file_name> <new_file_name>
```

### Push

```sh
# push to a remote and set it as default remote
git push -u <remote_name> <local_branch_name>

# push a local branch and track it
git push <remote_name> <local_branch_name>

# push a local branch onto an explicit remote one
git push <remote_name> <local_branch_name>:<remote_branch_name>

# delete remote branch
git push <remote_name> :<branch_name>

# push current repo and check if submodules need pushing, too
git push --recurse-submodules=check

# push current repo and all submodules
git push --recursive-submodules=on-demand
```

### Tags

```sh
# show tags
git tag

# checkout code at tag commit
git checkout <tag_name>

# add tag
git tag -a <tag_name> -m <message>

# push tags
git push --tags
```

### Rebase

```sh
# do a rebase based on <branch_name>
# get the new <branch_name> branch and put the commits of the current branch on top
git rebase <branch_name>

# options when a conflict occurs after rebase
# (1) conflict resolved
git rebase --continue
# (2) skip your changes
git rebase --skip
# (3) abort rebase
git rebase --abort

# interactively alter commits on the same branch, replay to HEAD-3
git rebase -i HEAD~3
```

### Diff

```sh
# show changes made since last commit
git diff

# show changes made since last commit that were already staged
git diff --staged

# show changes made since parent of last commit
git diff HEAD^

# ... and so on
git diff HEAD^^
git diff HEAD~5

# compare second most recent commit with most recent one
git diff HEAD^..HEAD

# compare two SHAs
git diff <sha1> <sha2>

# compare two branches
git diff <branch_name_1> <branch_name_2>

# time-based diffs, cf. git log
git diff --since=1.week.ago --until=1.minute.ago
```

### Stash

```sh
# stash changes onto the stash stack
git stash save

# only stash unstaged changes onto the stash stack
git stash save --keep-index

# also stash untracked changes onto the stash stack
git stash save --include-untracked

# apply first stash in the stack, does not drop it
git stash apply

# apply a certain stash in the stack
git stash apply <stash_name>

# list stashes on the stack
# can be used with any options of git log
git stash list

# shows first stash on the stack
# can be used with any options of git log
git stash show

# delete first stash in the stack
git stash drop

# delete all stashes in the stack
git stash clear

# runs apply and drop
git stash pop

# applies a stash to a newly created branch
git stash branch <new_branch_name> <stash_name>
```

### Blame

```sh
# show which changes were made by whom
git blame <file_name> --date=short

```

### List files

```sh
# list all ignored files
git ls-files --other --ignored --exclude-standard
```

### Log

```sh
# show version history of a file
git log --follow <file_name>

# show log as one liners
git log --pretty=oneline

# show log with custom format
git log --format="%h %ad- %s [%an]"

# show oneline log with the patch/changed lines
git log --oneline -p

# show oneline log with the number of insertions/deletions
git log --oneline --stat

# show oneline log with graph of branches
git log --oneline --graph
```

#### Date ranges

```sh
git log --until=1.minute.ago
git log --since=1.day.ago
git log --since=1.hour.ago
git log --since=1.month.ago --until=2.weeks.ago
git log --since=2015-01-01 --until=2015-12-10

```

### Reflog

The reflog is useful if, for example, a commit was deleted with reset and need to be retrieved again. Note that it is a local log.

```sh
# show the local git reflog
git reflog

# reset the HEAD to a commit of the reflog
git reset --hard <sha|shortname>

# show high-detail reflog, useful if a branch was deleted mistakingly and needs to be retrieved again
git log --walk-reflogs
```

### Purge

```sh
# go trough all commits and execute the specified command, e.g., 'rm -f passwords.txt'
git filter-branch --tree-filter <command>

# go trough all commits on all branches and execute the specified command
git filter-branch --tree-filter <command> -- --all

# go trough all commits in the staging area and execute the specified command, e.g., 'git rm --cached --ignore-unmatch passwords.txt'
# will not check out code before applying the command, hence, it is faster than tree-filter
git filter-branch --index-filter <command>

# also delete empty commits
git filter-branch --prune-empty -- --all
```

### Cherry pick

```sh
# pick a commit
git cherry-pick <sha>

# pick a commit and add source sha in commit message
git cherry-pick -x <sha>

# pick a commit and add cherry-picker in commit message
git cherry-pick --signoff <sha>

# pick a commit with different commit message
git cherry-pick --edit <sha>

# pick multiple commits and put changes in the staging area, useful when combining commit changes
git cherry-pick --no-commit <sha1> <sha2>
```

### Submodules

```sh
# add submodule to the current repo, after this also commit and push
git submodule add <repo_url>

# initialize submodule(s) in a newly cloned project
git submodule init

# update the submodule(s), most likely necessary after an init
git submodule update
```

### Configuration

```sh
# sets the name that is used for commit messages
git config --global user.name "[name]"

# enables colorization of command line output
git config --global color.ui auto
```

#### Aliases

```sh
# create alias for git log, git mylog
git config --global alias.mylog "log --pretty=format:'%h %s [%an]' --graph"

# git lol
git config --global alias.lol "log --graph --decorate --pretty=oneline --abbrev-commit --all"

git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```

#### Line endings

```sh
# can also be set on a per project basis in the git attributes file

# change CR/LF (windows) to LF (Mac) on commit
git config --global core-autocrlf input

# change LF (mac) to CR/LF (windows) on checkout
git config --global core-autocrlf true

# switch off conversion
git config --global core-autocrlf false
```
