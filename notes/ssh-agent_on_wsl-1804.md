### WSLでssh-agentを利用して、リモート先にforwardしたい

まんまこれ

- [(2018/8/30の記事)WSL Ubuntu 18.04 の ssh-agent が動作しない問題の暫定対処法](https://www.masoo.jp/blog/2018/08/30/wsl-ssh-agent.html)
- [GithubのWSLのissue](https://github.com/Microsoft/WSL/issues/3183#issuecomment-446612748)


```
cd /tmp/
wget http://mirrors.kernel.org/ubuntu/pool/main/o/openssh/openssh-client_7.2p2-4ubuntu2.6_amd64.deb
dpkg -x openssh-client_7.2p2-4ubuntu2.6_amd64.deb /tmp/deb
sudo mv /usr/bin/ssh-agent /usr/bin/ssh-agent.18.04
# for safekeeping in case of bionic updates
sudo mv /tmp/deb/usr/bin/ssh-agent /usr/bin/ssh-agent.16.04
sudo cp /usr/bin/ssh-agent.16.04 /usr/bin/ssh-agent
sudo chown root:ssh /usr/bin/ssh-agent
```

1. 現在のssh-agentを18.04版としてとっとく
2. 古いやつを16.04版としてもってきておいとく
3. 16.04版をssh-agentコマンドとしてcpしていつでも入れ替え可能な状態に!!
