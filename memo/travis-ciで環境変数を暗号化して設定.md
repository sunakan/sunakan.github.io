
1. travis gemを利用して環境変数を暗号化
2. .travis.ymlに記述

```
$ gem install travis
$ travis encrypt VAR1=foo VAR2=baz
Shell completion not installed. Would you like to install it now? |y| n
Detected repository as sunakan/リポジトリ名, is this correct? |yes| yes
=> secure: "xxxxx"
```
### .travis.yml

```
env:
  global:
    secure: "xxxx"
```
