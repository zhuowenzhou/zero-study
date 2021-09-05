# brew&zsh&nvm安装

## brew

### 安装

``` bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

### 卸载
``` bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"
```

### 常见问题

```
compinit:503: no such file or directory: /usr/local/share/zsh/site-functions/_brew_cask
```

执行以下命令

```
brew cleanup
```

## nvm node版本管理工具

### 安装

#### 方式1
``` bash
brew install nvm
```

#### 方式2
``` bash
git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout git describe --abbrev=0 --tags
```

### 配置

打开 ~/.zshrc文件加入以下代码

``` bash
export NVM_DIR="$HOME/.nvm"
[ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh"  # This loads nvm
[ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion
```

## oh my zsh

### 安装

``` bash
# 克隆仓库
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh

# 拷贝文件
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

# 重新加载或者重启终端
source ~/.zshrc
```
### 设置主题
1. 在.zshrc文件的修改变量 **ZSH_THEME="robbyrussell"**
2. 运行source命令，重新加载.zshrc文件

### 常见问题

#### 不安全的完成相关目录提示

1. 在.zshrc文件的第一行添加 `ZSH_DISABLE_COMPFIX=true`

2. 运行source命令，重新加载.zshrc文件

   ```bash
   source ~/.zshrc
   ```
