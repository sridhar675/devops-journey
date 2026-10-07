# Day-1 Devops-Journey

## Goal
Set up my Ubuntu workspace and start my DevOps learning journal on GitHub.

## DevOps tool: Git
### What is Git?  
Version control system that tracks changes to files over time.

### Why do we use it?
- History of every change (who, what, when)
- Roll back mistakes
- Many people can work on the same code using branches
- It's the starting point of every CI/CD pipeline

### What I did
1. Installed Ubuntu - WSL2
2. Installed Git and checked the version
3. Configured name, email and default branch
4. Created a repo, made my first commit
5. Pushed it to GitHub

### Key concepts
- Working directory -> staging area (`git add`) -> local repo (`git commit`) -> remote (`git push`)
- Commit = a snapshot with a message
- Branch = a separate line of work

## AWS service: IAM Roles

- What it is: IAM controls who can access your AWS resources and what they are allowed to do.

- Why a role is better than access keys: IAM Roles are better because it provide temporary, automatically managed permission without storing long-term access keys.

- Trust policy vs permission policy: Trust policy → Who is allowed to assume the role; Permission policy → What the role is allowed to do.

## What I built, step by step

Goal: let an EC2 server read files from one S3 bucket without storing any access keys on it.

1. Created an S3 bucket and uploaded a test file.
2. Wrote a custom IAM policy as JSON in my repo (`Labs/iam/s3-readonly-one-bucket.json`).
   - Statement 1: `s3:ListBucket` on the bucket ARN
   - Statement 2: `s3:GetObject` on the bucket ARN with `/*`
3. Created an IAM role for the EC2 service and attached my custom policy.
   - The trust policy allows `ec2.amazonaws.com` to perform `sts:AssumeRole`.
4. Launched an Ubuntu EC2 instance with no role attached and installed the AWS CLI.
5. Ran `aws s3 ls` on the bucket and got `Unable to locate credentials` (the "before" test).
6. Attached the role to the instance (Actions > Security > Modify IAM role).
7. Ran the same command again and it worked, with no keys configured anywhere.
8. Tore everything down: terminated the instance, then deleted the role, policy, and bucket.

## How I verified it

| Test | Result | What it proves |
|---|---|---|
| `aws s3 ls s3://<bucket>` before attaching the role | `NoCredentials` error | The server had no identity |
| Same command after attaching the role | Listed my file | The role gave the server access |
| `aws sts get-caller-identity` | ARN contained `assumed-role/RoleLab-EC2-S3Read/<INSTANCE_ID>` | The server is acting as the role, not a user |
| `aws s3 ls` (all buckets) | AccessDenied on `s3:ListAllMyBuckets` | The role can't see other buckets (least privilege) |
| `aws s3 cp test.txt s3://<bucket>/` | AccessDenied on `s3:PutObject` | The role is read-only, nothing more |

Key idea: the two denials matter as much as the success, because they prove the permissions are limited to exactly what I wrote.

Break and fix: I changed the object ARN to the wrong bucket name. Downloading failed, but listing the bucket still worked, because each statement in a policy grants permission independently. Fixing the ARN restored access.

## Problems I faced and how I fixed them

| Problem | Cause | Fix |
|---|---|---|
| `json.tool` said `No such file or directory` | I was already inside `Labs/iam`, so the path `Labs/iam/...` pointed to a folder that doesn't exist | Run `pwd` to check my location, then use the short file name. Relative paths start from the current folder. |
| Policy JSON was invalid | Used `statement` and `sid` in lowercase, left out a comma, and put a space in the Sid | JSON keys are case-sensitive (`Statement`, `Sid`). Add commas between items. Sid allows letters and numbers only. |
| Two separate JSON blocks | I wrote two policies instead of one | One policy is one object, with many statements inside the `Statement` list |
| Missing closing `}` | Forgot to close the outer brace | Count brackets: every opener needs a closer. Check with `python3 -m json.tool`. |
| Policy action not recognized | I wrote `s3:GetObjects` | The correct name is `s3:GetObject` (singular). Action names must be exact. |
| `test.pem` still visible after adding `.gitignore` | `ls` shows files on disk, `git status` shows what Git tracks | `.gitignore` doesn't delete files, it only stops Git tracking them. `git check-ignore -v` proves it. |

## Key takeaways
- Roles give temporary credentials, so there are no keys to leak.
- Trust policy answers who can assume the role. Permission policy answers what they can do.
- Bucket-level actions need the bucket ARN, and object-level actions need the `/*` ARN.
- Always test what should fail, not only what should work.

## Tomorrow
- Git branching and merging, plus S3 advanced
