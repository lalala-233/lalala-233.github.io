# 使用 ssh 签名 git 的提交

只是吐槽了一些遇到的问题，可能没法解决您的问题。

## 介绍

几年前，Github 上支持了提交签名。

主要原因是，你在 git 中的邮箱是可以乱填的，这就容易出现一些问题。

在 Github 上填写了签名密钥并签名你的提交后，会显示你的提交是否是「Verified」。

早年只支持 GPG 密钥，不过后来也支持使用 ssh 密钥签名了。

## 为什么

检查是否签名（用 -S 签名或在设置自动签名）：

```bash
git commit --allow-empty -S -m "Test SSH signed commit"
# 或
git commit --amend -S
# 然后
git log --show-signature -1
# 如果需要
git reset HEAD\^ # Linux 下的 ^ 要转义
```

如果签名成功，则会显示：`Good "git" signature for <邮箱> with 密钥类型 key 公钥的 SHA256`。

如果你遵循了 Github 上的建议，恭喜你，可能没法成功签名。

为什么？因为还有一步，你需要设置 SignersFile（讲个笑话，我之前一不小心写成 SingerFile 了）

```bash
git config --global gpg.ssh.allowedSignersFile 文件地址
```

内容格式如下：

```text
邮箱 密钥类型 公钥
```

具体参见这篇[博客](https://www.bhekani.com/posts/sign-git-commits-with-ssh-keys/)（英文）。
