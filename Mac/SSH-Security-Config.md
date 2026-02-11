1. 开启 macOS SSH Server
   系统设置 → 通用 → 共享 → 打开「远程登录(Remote Login)」，并把允许登录的用户加进去。 ([Apple Support][1])

2. 安装 Google Authenticator PAM 模块（server 端）

```bash
brew install google-authenticator-libpam
```

该包提供 `pam_google_authenticator.so`。 ([Homebrew Formulae][2])

3. 在 PAM 链里启用 TOTP（server 端）
   把下面这一行加到 `/etc/pam.d/sshd`（建议加在文件最上面）：

```bash
echo "auth required $(brew --prefix)/opt/google-authenticator-libpam/lib/security/pam_google_authenticator.so" | sudo tee -a /etc/pam.d/sshd
```

（允许未初始化 TOTP 的用户先不输 OTP，用于灰度：把 `nullok` 加到末尾）

```bash
echo "auth required $(brew --prefix)/opt/google-authenticator-libpam/lib/security/pam_google_authenticator.so nullok" | sudo tee -a /etc/pam.d/sshd
```

Homebrew 公式页给的就是这个路径和写法。 ([Homebrew Formulae][2])

4. 配置 sshd 强制 “公钥 + OTP”（server 端）
   编辑 `/etc/ssh/sshd_config`，确保/新增以下项：

```conf
UsePAM yes
KbdInteractiveAuthentication yes
PubkeyAuthentication yes
PasswordAuthentication no
AuthenticationMethods publickey,keyboard-interactive
```

`AuthenticationMethods publickey,keyboard-interactive` 表示公钥通过后还必须完成 keyboard-interactive（PAM）这一段，从而触发 OTP。 ([Server Fault][3])

5. 生成 SSH 密钥对（client 端）

```bash
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_blofin -C "your_email@example.com"
```

生成后得到：
- 私钥：`~/.ssh/id_ed25519_blofin`（不要泄露）
- 公钥：`~/.ssh/id_ed25519_blofin.pub`

6. 在 client 连接时使用私钥签名认证（client 端）

```bash
# 可选：加载到 ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_blofin

# 发起连接（SSH 会用私钥对服务端 challenge 做签名）
ssh -i ~/.ssh/id_ed25519_blofin user@server_ip
```

可选（写入 `~/.ssh/config`，后续简化连接）：

```sshconfig
Host my-server
  HostName server_ip
  User user
  IdentityFile ~/.ssh/id_ed25519_blofin
  IdentitiesOnly yes
```

```bash
ssh my-server
```

7. 在 server 中信任 client 公钥（server 端）

```bash
# 在 client 执行：把本地公钥追加到 server 的 authorized_keys
cat ~/.ssh/id_ed25519_blofin.pub | ssh user@server_ip 'mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys'
```

然后确认 `/etc/ssh/sshd_config`：

```conf
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

8. 为每个要登录的 macOS 账号生成 TOTP 秘钥（server 端，每个用户各做一次）
   用该用户身份在 Mac 上执行：

```bash
google-authenticator
```

然后用手机的 Google Authenticator/1Password/Authy 扫码或手动录入。该命令会生成 `~/.google_authenticator`。 ([openSUSE Manpages][4])

9. 重启 sshd（macOS 最稳的方法）
   系统设置 → 通用 → 共享 → 把「远程登录」关掉再打开（相当于重启 sshd）。 ([Gist][5])

完成后，从 client 连接时流程应是：先用 SSH key 成功，再提示输入一次性验证码（OTP）。

