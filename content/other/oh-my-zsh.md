# 安装Oh My Zsh

## windows安装
### 安装Wsl
```bash
# 1.安装wsl
wsl --install
# 2.查看可用发行版本列表
wsl --list --online
NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
Ubuntu-16.04    Ubuntu 16.04 LTS
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS
# 3.安装发行版本
wsl --install -d Ubuntu-20.04
```

### 安装zsh和oh my zsh
```bash
# 1.安装zsh
sudo apt install zsh -y
# 2.安装oh my zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```