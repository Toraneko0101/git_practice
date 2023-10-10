# git_practice

## 基礎
```console

# stage
git add <file_name>

# commit
git commit -m "test_commit"

# stage & commit

git commit -a -m "test_commit"
```

# Push
```console

# origin -> git remote add origin <repository_URL>

# base
git push origin <local_branch>

# push先のリモートブランチを指定
git push origin <local_branch>:<remote_branch>

```

## branch

```console

# ブランチの切り替え
git checkout <branch>

# リモートブランチの追跡リスト
git branch -r

# 追跡開始
git branch origin <branch>

# 不要になったリモートブランチを追跡削除
git branch --prune

```


