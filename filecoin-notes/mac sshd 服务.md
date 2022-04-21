启动 sshd 服务
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist

停止 sshd 服务
sudo launchctl unload -w /System/Library/LaunchDaemons/ssh.plist

查看sshd服务是否启动
sudo launchctl list | grep ssh