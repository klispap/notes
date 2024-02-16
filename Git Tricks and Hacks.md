# Git Tricks and Hacks

```
git pull
git submodule update --init --recursive
```
A super useful combo that I execute on every pull I make. It saved me a lot of pain over breaking changes in submodules that I was not informed about being pushed.


```
git fetch --all
git fetch --prune origin
```
This combo fetches any new feature branches and I does a bit of housekeeping at the same time since the second fetch command will remove all refs to origin branches that were deleted via merge or otherwise.


```
git add -u
```
Simple add command that stages all previously tracked files leaving untracked and new files behind. 

