```
$ sudo apt upadate -y
$ sudo apt install tmux wget git tig tree pstree
$ sudo apt upgrade -y
```

## anyenv
```
$ git clone https://github.com/riywo/anyenv ~/.anyenv
$ vi ~/.bashrc
```

```~/.bashrc
...
# anyenv setting
export PATH=${PATH}:${HOME}/.anyenv/bin
eval "$(anyenv init -)"
```

~/.anyenv/envs以下に色々なenvが入る


## rbenv
```
$ anyenv install rbenv
$ sudo apt install build-essential
$ sudo apt install libssl-dev libreadline-dev
$ sudo apt install -y zlib1g-dev
$ rbenv install 2.5.3
$ rbenv global 2.5.3
```

## pyenv
```
$ anyenv install pyenv
$ sudo apt install -y libffi-dev
$ pyenv install 3.7.1
$ pyenv global 3.7.1
$ pip install --upgrade pip
$ pip install ansible
```

## Vagrant
参考
[Vagrant入門 on WSL ~Dockerがブイブイいわせている今だからこそ~](https://qiita.com/konankun/items/5d734a82f4c346e557bc)

Vagrant公式でDebian用の「64bit版の最新」をDL(デフォルトではダウンロードフォルダに！)
〇〇.deb(例：vagrant_2.2.1_x86_64.deb)

WSLで

```
$ cd /mnt/c/Users/〇〇/Downloads/
$ sudo dpkg -i vagrant_〇〇_x86_64.deb
```
ちょいと時間がかかる

## vimのプラグイン管理(vim-plug)

```
$ curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

#### .vimrcの中身

```
call plug#begin()
Plug 'プラグインリポジトリ'
call plug#end()
```

vim開いて、`:PlugInstall`コマンドを叩く
