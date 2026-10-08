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

---

## AWS service: S3 advanced (versioning, lifecycle, bucket policy)

### What I learned
| Feature | What it does | Why DevOps needs it |
|---|---|---|
| Versioning | Keeps every version of an object. A normal delete only adds a delete marker. | Protects Terraform state and artifacts from accidents |
| Lifecycle rules | Automatically expire old versions and clean up failed uploads | Controls storage cost |
| Bucket policy | A resource-based policy attached to the bucket | Enforces rules such as "HTTPS only" |

### Identity-based vs resource-based policies
| | Identity-based | Resource-based |
|---|---|---|
| Attached to | A user or role | The resource (bucket, role trust policy) |
| Has `Principal`? | No | Yes |
| Example | Day 1 S3 read-only policy | Bucket policy, role trust policy |

The visual editor under IAM > Policies builds identity-based policies, so it never asks for a Principal. A bucket policy must include one.

### What I built
1. Created a bucket and turned on versioning.
2. Uploaded a file twice and saw two version IDs with Show versions on.
3. Deleted the file normally and found the delete marker. Removing the marker brought the file back.
4. Created a lifecycle rule for the whole bucket that permanently deletes noncurrent versions after 1 day and cleans up incomplete multipart uploads after 7 days.
5. Wrote a bucket policy that denies every request not made over HTTPS (`aws:SecureTransport` is `false`) and saved it in `Labs/s3/deny-insecure-transport.json`.
6. Tore down the bucket with Empty, then Delete.

### Verification
| Check | Result |
|---|---|
| Policy applied to the bucket in the console | Done |
| Opening an object from the console (HTTPS) | Still worked, so the Deny didn't lock me out |
| Plain HTTP request returning 403 AccessDenied | Not tested yet: CloudShell was unavailable while my account verification was in progress |

### Problems I faced and lessons
| Problem | Cause | Lesson |
|---|---|---|
| My first uploaded file disappeared and I found no delete marker | With Show versions on, I selected a specific version and deleted it, which is a permanent delete | A normal delete (Show versions off) adds a delete marker and is recoverable. Deleting a specific version is permanent. Production teams restrict `s3:DeleteObjectVersion` on important buckets. |
| Lifecycle form showed red errors | I chose "Limit the scope using filters" without entering any filter | Choose "Apply to all objects", or provide a prefix, tag or size filter |
| Bucket policy was missing a Principal | I built it in the IAM visual editor, which makes identity-based policies | Resource-based policies need a `Principal`. `"*"` is safe with a Deny. |
| Console showed "Errors: 1" | The final closing `}` was missing after pasting | Count brackets, check the error counter, and write files with `cat > file << 'EOF'` so copying can't drop the last line |
| CloudShell would not start | Account verification still in progress (up to two days for new accounts) | Not a mistake. Retry later, and run the HTTP test then. |

### Key takeaways
- Versioning plus a delete marker means "deleted" can still be recovered. Deleting a specific version cannot be undone.
- A lifecycle rule needs a scope, and "all objects" is a deliberate choice.
- Deny always beats Allow.
- A condition is built from three layers: operator, key, value.
- Bucket-level and object-level actions need both the bucket ARN and the `/*` ARN.

### To do
- Run the HTTP test once CloudShell is available:
  `aws s3api head-object --bucket <bucket> --key <file> --endpoint-url http://s3.<region>.amazonaws.com --region <region>`
  Expected result: 403 AccessDenied.
