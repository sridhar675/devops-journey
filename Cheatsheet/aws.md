# AWS Cheatsheet

## CLI commands
| Command | What it does |
|---|---|
| `aws --version` | Check the CLI is installed |
| `aws sts get-caller-identity` | Show who I am (user or assumed role) |
| `aws s3 ls` | List all buckets |
| `aws s3 ls s3://<bucket>` | List objects in a bucket |
| `aws s3 cp <src> <dest>` | Copy a file to or from S3 |
| `aws s3api list-object-versions --bucket <bucket>` | Show every version and delete marker |

## IAM policy structure
| Field | Meaning |
|---|---|
| `Version` | Always `"2012-10-17"` |
| `Statement` | A list of rules |
| `Sid` | Label, letters and numbers only |
| `Effect` | `Allow` or `Deny` (Deny always wins) |
| `Principal` | Who the rule applies to (resource-based policies only) |
| `Action` | `service:Operation`, spelled exactly (`s3:GetObject`) |
| `Resource` | The ARN the actions apply to |
| `Condition` | Operator > key > value, e.g. `Bool` > `aws:SecureTransport` > `"false"` |

## S3 ARNs
- Bucket: `arn:aws:s3:::my-bucket` (for list-type actions)
- Objects: `arn:aws:s3:::my-bucket/*` (for get/put/delete actions)

## Roles
- Trust policy: who can assume the role
- Permission policy: what the role can do
- EC2 gets the role through an instance profile and receives temporary credentials

## S3 delete behaviour
- Show versions OFF + Delete: adds a delete marker (recoverable)
- Show versions ON + delete a version row: permanent
- Delete the marker itself to bring the object back
