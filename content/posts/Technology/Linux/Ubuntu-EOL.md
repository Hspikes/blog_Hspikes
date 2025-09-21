+++
date = '2025-09-19T15:08:00+08:00'
draft = false
title = 'Ubuntu EOL: 旧版本主义者的灾难'
katex = true
+++

本来想取名字叫“一次 zip 引发的血案”，但想了想还是改成现在这个名字了。

众所周知，在计算机软件领域，总有人劝你不要更新软件，什么“最新的版本都是做实验，远不如老版本稳定”，“新版本都是加一些无关紧要占内存的东西，浪费硬盘”。曾经我不屑一顾，直到我在把我的游戏本从 Win10 更新到了 Win11 后也加入了旧版本至上教派。嗯，人不如故。

秉持着只要还能用就永不更新得理念，ubuntu 无数次弹出更新提示，我刚开始还给他面子读一读信息，后来干脆连红点都不清理了，看都不看，直到酿成灾难😭

## 初现端倪

某日我心血来潮，准备把电脑上的资料打包一下，上传的网盘，一是节省电脑的空间，二是整理电脑给我一个绝佳的摸鱼理由，于是我使用 ```zip``` 命令打包。

当我正常打包了几个包后，在进行一次打包的时候发生了报错，报错信息写本文的时候已经找不到了，这里就不展示了。

本着只要肯折腾就有折不完的腾的信念，我一如既往的读报错信息发现看不懂，然后直接 Google，经过简单的查阅，总有人遇到过和你一样的问题：应该是```zip``` 版本太老了，导致目录名中有中文无法识别，最新的版本已经解决这个问题了。

那好办啊，```sudo apt install zip```，嗯，还得是我们 linux，一行命令就解决了，吗？

报错以下恐怖的错误：

```bash
Error: 无法下载 http://archive.ubuntu.com/ubuntu/pool/main/z/zip/zip_3.0-14ubuntu0.2_amd64.deb 404 Not Found [IP: 91.189.91.81 80] 。
```

尝试运行 ```apt-get update``` 也报出大量错误，包括:

```bash
Error: 无法下载 http://archive.ubuntu.com/ubuntu/pool/main/z/zip/zip_3.0-14ubuntu0.2_amd64.deb 404 Not Found [IP: 91.189.91.81 80]。
```

进入图形化界面系统更新处，点击更新弹出“下载软件仓库信息失败，检查您的网络设置”。

这里的第一想法肯定是网络问题，但我连接其他服务器都很正常，难道是 ubuntu 的服务器崩了吗？```ping -c 4 archive.ubuntu.com``` 却正常连接了：

```bash
--- archive.ubuntu.com ping statistics --- 
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
```

这下不妙了，linux 下不怕报错，怕报错原因不清晰，404 not found 没找到对应的文件，难道是服务器的文件寻址有问题？因为问题比较特殊，单纯查无法下载的话更多的都是网络配置源配置一类的问题，这种详细描述定制化的问题还是交给 GPT 吧。

## 问题溯源

GPT 先询问了我的版本，版本为 Ubuntu 24.10。然后让我告诉他我当前的更新源，我在终端上执行```cat /etc/apt/sources.list.d/ubuntu.sources```输出：

```bash
Types: deb URIs: http://archive.ubuntu.com/ubuntu/ 
Suites: oracular oracular-updates oracular-backports 
Components: main restricted universe multiverse 
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg 

Types: deb URIs: http://security.ubuntu.com/ubuntu/ 
Suites: oracular-security 
Components: main restricted universe multiverse 
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

嗯，标准的源路径，接下来 GPT 开始了长时间的思考检索，并告诉我了原因。

**Ubuntu 24.10 (“Oracular Oriole”) 在 2025 年 7 月 10 日已进入生命周期结束（EOL）阶段，官方将其软件仓库归档到旧版本仓库（old-releases）。换言之，24.10 的软件包不再保留在 archive.ubuntu.com 或 security.ubuntu.com 等主镜像上。**

报错找不到 ```zip_3.0-14ubuntu0.2_amd64.deb```，以及 GUI “下载软件仓库信息失败”，都是这种情况的典型表现。Ping 测试 ```archive.ubuntu.com``` 通畅只是说明网络正常，但对应的 24.10 仓库内容已经被移走，导致下载失败。

ubuntu.sources 文件中列出的 ```Suites``` 为 ```oracular``` 等，表示仍在查询 24.10 主仓库。由于 24.10 已过 EOL，这些源地址已不再有效，导致 ```apt update``` 和软件安装时均报 404 错误。

也就是说，对于 Ubuntu 的系统版本，当他过时的时候会从主仓库中移除，通过原来的源不再能够找到对应需要的软件。

看到这里我震惊了，用了快一年 Ubuntu 了，才知道还有这种策略。非常诡异的版本维护策略，兼容老版本不应该是新版本的一个必要条件吗！

## 问题解决

主要应该采取两种方案：

1. 升级 Ubuntu 至受支持版本：Ubuntu 官方推荐的升级路径是从 24.10 升级到 Ubuntu 25.04。升级后即可使用活跃的仓库源继续获取更新，避免 404 问题。这是长远的解决方案。
2. 修改 APT 源到旧版本仓库：如果暂时不升级系统，可以将软件源地址替换为 ```old-releases.ubuntu.com```。具体操作可以编辑 ```/etc/apt/sources.list.d/ubuntu.sources``` 文件，将所有的 ```archive.ubuntu.com``` 和 ```security.ubuntu.com``` 替换为 ```old-releases.ubuntu.com```

到这里我就非常无语了，那还能怎么办，折腾呗，了解了一下如果使用第二种方案的话软件源可能也比较过时，那还是升级我的系统吧，什么，你说重装一个 archlinux？那还是算了吧，最近实在忙，下一个电脑吧。

#### 备份

要更新/重装系统，备份是少不了的，这里给出一个备份的脚本命令，只用改一改你的外部存储设备就能直接用。要找外部设备的路径用 ```lsblk and df -h```就可以了。

```bash
# === 配置（请只修改 MOUNTPOINT 为你的实际挂载点） ===
MOUNTPOINT="/media/$USER/USB"   # <- 把这里改成你的 U盘挂载路径（例如 /media/spike/USB）
BACKUPDIR="$MOUNTPOINT/backup-$(date +%F)"
USERNAME="$USER"
HOMEDIR="$HOME"

# === 检查挂载点是否存在 ===
if [ ! -d "$MOUNTPOINT" ]; then
  echo "错误：找不到挂载点 $MOUNTPOINT 。请确认 U盘已插入并已挂载 (查看 lsblk / df -h)。"
  exit 1
fi

# === 创建备份目录 ===
sudo mkdir -p "$BACKUPDIR"
sudo chown "$USER":"$USER" "$BACKUPDIR"

# === 1) 备份 /home/<user>（排除 .cache） ===
echo "正在打包 /home/$USERNAME ..."
sudo tar -C / -czpf "$BACKUPDIR/home-$USERNAME.tar.gz" --exclude="home/$USERNAME/.cache" "home/$USERNAME"

# === 2) 备份 /etc ===
echo "正在打包 /etc ..."
sudo tar -C / -czpf "$BACKUPDIR/etc-backup.tar.gz" /etc

# === 3) 保存已安装包列表 ===
echo "导出已安装包列表..."
dpkg --get-selections > "$BACKUPDIR/installed-packages.list"

# === 4) 备份 /etc/apt 源（含 ubuntu.sources 等） ===
echo "复制 /etc/apt ..."
sudo cp -a /etc/apt "$BACKUPDIR/etc-apt-backup"
# 如果你有 ubuntu.sources 文件，单独拷贝（若不存在不会报错）
sudo cp -a /etc/apt/sources.list.d/ubuntu.sources "$BACKUPDIR" 2>/dev/null || true

# === 5) 备份 ~/.ssh 与 ~/.gnupg（以 tar 保留权限） ===
echo "打包 ~/.ssh 和 ~/.gnupg ..."
# 注意：若 ~/.gnupg 很大或有权限问题，可能需要先关闭 gpg-agent
sudo tar -C "$HOMEDIR" -czpf "$BACKUPDIR/ssh-gnupg-$USERNAME.tar.gz" .ssh .gnupg || true

# === 6) 生成校验文件（sha256） ===
echo "生成 sha256 校验..."
cd "$BACKUPDIR"
sha256sum *.tar.gz > checksums.sha256
sha256sum -c checksums.sha256 || echo "注意：校验部分文件可能失败，请检查输出。"

# === 7) 同步并安全卸载的提醒 ===
sync
echo "备份完成，文件位于：$BACKUPDIR"
echo "请在卸载 U盘前确认文件无误（例如：ls -lh $BACKUPDIR ； sha256sum -c $BACKUPDIR/checksums.sha256 在另一台机器或以后验证）"
```

当然这是 AI 生成的。注意脚本不要用 ```sudo``` 运行，这样会更改你的路径，正常的 ```bash bashup.bash``` 或者 ```chmod +x bashup.bash ./bashup.bash```就可以了。

#### 临时更改 APT 源

在完成备份以后，需要把 把 APT 源临时指向 ```old-releases.ubuntu.com```,为什么需要把源临时更改呢？

24.10 的包已被从主镜像```（archive.ubuntu.com / security.ubuntu.com）```移走 —— 这会造成 ```apt update``` 报 404，导致 ```update-manager/GUI``` 无法下载仓库信息。因此临时把 ```URIs``` 指向 ```old-releases.ubuntu.com```，能恢复索引与让你安装升级工具（不是长期方案，只为完成升级准备）。官方/社区文档也建议此法用于 EOL 版本的升级准备。

先将原来的源保存一份 ```sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak```

然后更改源列表：

```bash
sudo sed -i \
  -e 's|http://archive.ubuntu.com/ubuntu/|http://old-releases.ubuntu.com/ubuntu/|g' \
  -e 's|http://security.ubuntu.com/ubuntu/|http://old-releases.ubuntu.com/ubuntu/|g' \
  /etc/apt/sources.list.d/ubuntu.sources
```

然后更新当前系统的所有包，减少后续更新时的冲突。

```bash
sudo apt update
sudo apt upgrade -y
sudo apt full-upgrade -y   # 或 sudo apt dist-upgrade
sudo apt autoremove -y
```

#### 升级系统

安装并确认升级工具存在 ```sudo apt install update-manager-core ubuntu-release-upgrader-core -y```

接着编辑 ```/etc/update-manager/release-upgrades```，确保 ```Prompt=normal```

运行 ```sudo do-release-upgrade```，接着就是漫长的升级，重新下载加载内核、各种各样的桌面组件。经过漫长的煎熬，无时无刻不在担心奇怪的报错，所幸完成了更新。这时再运行确认版本 ```cat /etc/os-release```，输出：

```bash
PRETTY_NAME="Ubuntu 25.04"
NAME="Ubuntu"
VERSION_ID="25.04"
VERSION="25.04 (Plucky Puffin)"
VERSION_CODENAME=plucky
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=plucky
LOGO=ubuntu-logo
```

终于完成了更新😭。如果我没有记错的话，再更新完之后你本地记录更新源就会被自动的覆写掉，再次执行 ```cat /etc/apt/sources.list.d/ubuntu.sources```，输出：

```bash
Types: deb
URIs: http://archive.ubuntu.com/ubuntu
Suites: plucky plucky-updates plucky-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu
Suites: plucky-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

此时再执行一次对当前所有源更新：

```bash
sudo apt update
sudo apt upgrade -y
sudo apt full-upgrade -y   # 或 sudo apt dist-upgrade
sudo apt autoremove -y
```

就可以快乐的 ```zip```打包啦！嗯，一次 zip 引发的血案。

经历了这场灾难后，我只能说下一个电脑锁定 macOS，至少不会再是 Ubuntu，当然更不会是 Windows。还有一件事，Chatgpt 真的是我的救命恩人啊😭
