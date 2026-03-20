## Rename a commit
### Most Recent
```bash
git commit --amend
```
### Older
```bash
git rebase -i COMMIT_HASH_HERE^
```
Then change the first commit to `reword`
## Drop a commit
```bash
git rebase -i COMMIT_HASH_HERE^
```
Then change the first commit to `drop`.

## Edit a Commit
```bash
git rebase -i COMMIT_HASH_HERE^
```
Then change the first commit to `edit`. Then once you're done, enter:
```bash
# make changes
git add .
git commit --amend
git rebase --continue
```
### Squash Commits
```bash
git rebase -i COMMIT_HASH_HERE^
```
Then change the commit in question to `squash` (keep both messages) or `fixup` only keep newest message. Note that This will combine this commit with the **previous one**.