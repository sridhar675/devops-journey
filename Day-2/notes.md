# Day 02

## DevOps tool: Git branching, merging and conflicts

### What I learned
- A **branch** is a separate line of work inside the same repo. `main` stays stable while I experiment on a branch.
- A **merge** brings the work from one branch back into another.
- A **merge conflict** happens when two branches change the same line of the same file. Git can't choose a winner, so it stops and asks me to decide.
- Conflicts are normal. They are a sign that Git is protecting my work, not an error.

### What I did
1. Created a feature branch, added the Day 2 notes skeleton, and merged it back into `main`.
2. Switched back to `main` and confirmed the new folder wasn't there, which proved branches are separate.
3. Created a demo file in `Labs/git/conflict-demo.txt`.
4. Made two branches that changed the same line to two different tools (Git and Jenkins).
5. Merged the first branch cleanly, then merged the second one and got a conflict.
6. Opened the file, saw the conflict markers, edited it to the result I wanted, and removed the markers.
7. Ran `git add` and `git commit` to finish the merge.
8. Checked the result with `git log --oneline --graph --all`.

### How to read a conflict
- `<<<<<<< HEAD` down to `=======` is the version on my current branch.
- `=======` down to `>>>>>>> branch-name` is the incoming version.
- To resolve it: edit the file into the final text, **delete all three marker lines**, then `git add` and `git commit`.

### Reading the graph
- `*` is a commit, and `|`, `\`, `/` show lines of work splitting and rejoining.
- A merge commit has two lines flowing into it.
- `HEAD -> main` shows where I am now.
- `origin/main` shows where GitHub's copy is. Mine was several commits behind until I ran `git push`.

### Problems I faced and lessons
| Problem | Lesson |
|---|---|
| My second branch was created while I was still standing on the first one, so it carried the first branch's commit | Always run `git switch main` before creating a new branch, and check with `git branch` (the `*` marks the current branch) |
| My commit messages were inconsistent (`branch: change to git`, `conflict issue resolved`) | Use one pattern, such as `day-02: ...`, so the history explains itself |
| Local `main` was ahead of `origin/main` | Commits are saved locally first. Nothing reaches GitHub until `git push`. |

### Key takeaways
- Branch for every change, and keep `main` stable.
- Conflicts mean two people changed the same line. Resolve them by editing the file, then add and commit.
- Check `git status` and `git branch` before starting any new work.
- On a team, branches are merged through pull requests, not by hand on a laptop.
