---
title: "Update Older Branch to Newest Commit on Master or Main"
date: 2023-09-06T12:11:48-06:00
lastmod: 
draft: false
tags: ["git", "github", "shortpost", ]
categories: ["tech"]
---

(This post serves more as a mental note to myself than anything else)

If a branch, in this example `feature1` on git/github was created in the past, but now `master/main` has some commits that are not present in `feature1`, this is what has to be done, to include to branches and get `feature1` up to date with the latest commit on `master/main`:
1. Get the latest commits for both branches:
```bash
git fetch origin master #or main
git fetch origin feature1
```

2. Change to the `feature1` branch
```bash
git checkout feature1
```

3. Merge the `master/main` branch into `feature1`
```bash
git merge master
```

4. Resolve possible merge conflicts (hopefully none).

5. Push the updated `feature1` branch to the repo
```bash
git push origin feature1
```

n.b. this push will possibly have a lot of commits: all the commits that `master/main` was ahead of `feature1`. The `feature1` branch get's an added merge commit, containing all those commits. e.g. before the merge `feature1` was 1 commit ahead of `master/main` and 42 commits behind `master/main`. After the merge `feature1` will be two commits ahead of `master/main` i.e. the one commit it was ahead before the merge + the merge commit we just did. 

Using `merge` preserves the whole history and does not rewrite commit hashes.

Hopefully that's it. 
