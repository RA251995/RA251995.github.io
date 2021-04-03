# Git 
## Configuration
```bash
git config --global user.name "username"
git config --global user.email "user@example.com"
git config -l
```
## Basic Commands
```bash
git init
git status
git diff file / git diff
git add . / git add file / git add -p
git diff --staged
git commit / git commit -m "Message"
git log / git log -p -2 / git log --stat / git log --graph --oneline # -p => --patch
git show <commit-id>
git commit -a # Skip staging
```
## Remove files
```bash
git rm file
git mv file newfile
```
_.gitignore_
```bash
git add .gitignore
git commit -m "Added gitignore file"
```
## Undo
```bash
git checkout file # Before staging (-p)
git reset HEAD # After staging (-p)
git commit --amend # Overwrite last commit (Don't use online)
git revert HEAD / git revert <commit-id>
```
## Branches
```bash
git branch
git branch new-branch
git checkout new-branch / git checkout -b new-branch
git branch -d new-branch # Delete branch
git checkout master
git merge new-branch
git merge --abort
```
## Remote Repository
```bash
git clone <url>
git remote add upstream <url>
git push
git config --global credential.helper cache
git pull # Fetches remote updates and merges
git remote -v # List remote repos verbosely
git remote show origin # Describes remote repo origin
git branch -r # List remote branches
git fetch # Fetches remote updates
git log origin/master
git merge origin/master
git remote update # Fetches most up-to-date objects
git push -u origin <branch-name> # Push new branch to remote
```
### Rebase
```bash
git rebase master
git rebase --continue
git push --delete origin <branch-name>

git rebase -i master # -> squash
git push -f
```
## Terminal Commands
```bash
diff file1 file2
diff -u file1 file2
vimdiff file1 file2

diff -u file1 file2 > files.diff
patch file1 < files.diff
```