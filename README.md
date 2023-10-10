# git_practice

## 基礎
```console

# clone
git clone <repository_url>

# stage
git add <file_name>

# stage all
git add .

# commit
git commit -m "test_commit"

# stage & commit

git commit -a -m "test_commit"
```

## Push
```console

# origin -> git remote add origin <repository_URL>

# base
git push origin <local_branch>

# push先のリモートブランチを指定
git push origin <local_branch>:<remote_branch>

# リモートブランチを削除(push元がないので)
# Openなpullrequestに関連付けられているbranchは削除できない
git push origin :<remote_branch>

```

## Branch

```console

# ブランチの切り替え
git checkout <branch>

# 全てのブランチを表示
git branch -a

# ローカルブランチ一覧の表示
git branch

# リモートブランチ一覧の表示
git branch -r

# ローカルブランチの削除
git branch -D <branch>

# 上流ブランチの確認
git branch -vv

# 上流ブランチを指定(local_branchを省略すると、自動的に現在のブランチを指定する)
git branch <local_branch> -u <remote_branch> 

# リモートブランチの追跡リスト
git branch -r

# リモートブランチの追跡開始
git branch origin <branch>

# 不要になったリモートブランチを追跡しない
git fetch --prune


```

## Merge

```console

# ブランチのマージ時に、変更内容を知りたい場合、--no-ff(No-FastForward)オプションを付ける
# github上のpull-requestは--no-ffでmergeされる
# https://docs.github.com/ja/github-ae@latest/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github
git merge --no-ff
```

## github cli


[マニュアル](https://cli.github.com/manual/)

```

# install
winget install --id GitHub.cli

# upgrade
winget upgrade --id GitHub.cli

# login
gh auth login

# logout
gh auth logout

# repositoryの概要(README.md)
gh repo view <repository_url>

# 新しいrepositoryを対話的に作成
gh repo create

# repositoryをclone
gh repo clone <repository_url>

# issueの確認
gh issue list

# issueを閲覧
gh issue view 4

# issueの作成
gh issue create --title "I use github-cli" --body "Nothing"

# issueにコメントを追加(4 -> #4)
gh issue comment 4 --body "I add comment"

# issueを再度開く/削除する
gh issue reopen/delete 4

# issueを閉じる
gh issue close 4 --comment "task-complete Toraneko" --reason "complete"

# pull-requestの確認
gh pr list
```

