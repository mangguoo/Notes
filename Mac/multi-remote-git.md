# Git 多账号 SSH Key 配置

**结论**：用"一仓库一把 key"，再用 `~/.ssh/config` 里不同的 `Host`（虚拟域名别名）把不同仓库流量分流到不同私钥；Git remote URL 改成对应 Host 别名即可。

## 1. 生成多把 SSH key（按账号/仓库维度）

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh

ssh-keygen -t ed25519 -C "work-github" -f ~/.ssh/id_ed25519_github_work
ssh-keygen -t ed25519 -C "personal-github" -f ~/.ssh/id_ed25519_github_personal
ssh-keygen -t ed25519 -C "gitlab-work" -f ~/.ssh/id_ed25519_gitlab_work

chmod 600 ~/.ssh/id_ed25519_* ~/.ssh/*.pub 2>/dev/null || true
```

## 2. 写 ~/.ssh/config 做分流（核心）

```ssh
# GitHub - work
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github_work
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes

# GitHub - personal
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github_personal
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes

# GitLab - work
Host gitlab-work
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab_work
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes
```

> **要点**：
>
> - `Host` 是你自定义的"分流入口"（别名），真正域名在 `HostName`。
> - `IdentitiesOnly yes` 很关键：避免 SSH 把 agent 里一堆 key 都拿去试，导致"用错 key / 被限流 / 认证失败"。

## 3. 把公钥分别加到对应平台账号

```bash
pbcopy < ~/.ssh/id_ed25519_github_work.pub
pbcopy < ~/.ssh/id_ed25519_github_personal.pub
pbcopy < ~/.ssh/id_ed25519_gitlab_work.pub
```

然后在 GitHub/GitLab 的 SSH Keys 页面粘贴。

## 4. 给每个仓库设置 remote URL 指向别名 Host

你原来可能是：

```
git@github.com:org/repo.git
```

改成：

| Remote URL | 用途 |
|------------|------|
| `git@github-work:org/repo.git` | 走 work key |
| `git@github-personal:me/repo.git` | 走 personal key |
| `git@gitlab-work:group/repo.git` | 走 gitlab work key |

操作命令：

```bash
git remote -v
git remote set-url origin git@github-work:org/repo.git
```

## 5. 验证分流是否生效

```bash
# 基础测试
ssh -T git@github-work
ssh -T git@github-personal
ssh -T git@gitlab-work

# 调试模式（确定用了哪把 key）
ssh -vT git@github-work 2>&1 | grep -E "Offering public key|identity file"
```

## 6. macOS 的 ssh-agent / Keychain（可选但推荐）

把 key 加进 agent（并写入 keychain）：

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_github_work
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_github_personal
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_gitlab_work

ssh-add -l  # 查看已加载的 key
```

## 常见坑与排查

| 现象 | 处理 |
|------|------|
| 明明配置了，还是用错 key | 确认 remote 用的是 `git@github-work:...` 这种别名；并且 `IdentitiesOnly yes` 已写 |
| Permission denied (publickey) | `ssh -vT git@xxx` 看有没有 "Offering public key ..." 和对应的 identity file；以及公钥是否加到正确账号 |
| 同一平台多个账号，clone 时仍混乱 | 一律用别名 Host，不要用 `git@github.com:` 直连；必要时清掉 agent 里多余 key：`ssh-add -D` 后再按需 `ssh-add ...` |

---

> 如果你说下"远程仓库组合"（比如 GitHub 两个账号 + GitLab 一个账号，还是 GitHub Enterprise + GitHub.com），我可以把 `~/.ssh/config` 按你的域名和命名习惯直接给到可复制版本。
