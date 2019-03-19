## list all branches
```git branch -a```
## switch to branch
```git checkout branchname```
## less info about a change in the log
```git log -p --name-only --follow somefilename```
## quick commit
```echo "commit message:"; read msg; git add -A .; git commit -m "$msg"; git push origin master```
## clean current repo
```git fetch origin master; git reset --hard FETCH_HEAD; git clean -df```
