# Git Cheatsheet

mkdir -p day-01 cheatsheets labs images → 4 separate directories
mkdir -p labs/docker/day-01 → nested parent/child directories

## Daily flow
git status -> git add . -> git commit -m "..." -> git push


## Branching and merging (Day 2)

| Command | What it does |
|---|---|
| `git branch` | List branches (the `*` marks the current one) |
| `git switch <name>` | Move to an existing branch |
| `git switch -c <name>` | Create a new branch and move to it |
| `git merge <name>` | Merge that branch into the current one |
| `git branch -d <name>` | Delete a merged branch |
| `git log --oneline --graph --all` | Show history as a graph with all branches |
| `git commit -am "msg"` | Stage tracked files and commit in one step |
| `git show --stat HEAD` | List the files in the latest commit |

## Conflict resolution steps
1. `git merge <branch>` reports a conflict
2. `git status` shows "both modified"
3. Open the file and edit it to the final result
4. Delete the `<<<<<<<`, `=======` and `>>>>>>>` lines
5. `git add <file>` then `git commit`

## Before starting any new work
`git status` then `git branch` then `git switch main` if needed
