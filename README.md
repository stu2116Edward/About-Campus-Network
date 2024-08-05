# Campus Network
Use softether to skip campus network verification

本教程采用Ubuntu系统
首先查看服务端防火墙的状态
ufw status
显示如下表示处于关闭状态
root@iZbp1azy5ek8l30twb64etZ:~# ufw status
Status: inactive

使用SoftEther VPN Server时一定要关闭防火墙
关闭防火墙
ufw disable
开启防火墙
ufw enable

安装wget
apt install net-tools wget -y

使用命令查看443端口是否被占用
ss -tuln
或者
netstat -tuln
# 使用lsof命令找到占用443端口的进程  
sudo lsof -i :443
使用命令将进程杀死
sudo kill -9 <这里是对应端口的PID>
# 假设找到的进程ID是1234，然后使用kill命令终止它  
sudo kill -9 1234

服务端安装SoftEther VPN Server
wget -N --no-check-certificate "https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/releases/download/v4.28-9669-beta/softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz"
国内服务器
wget -N --no-check-certificate "http://47.106.20.4/softwares/softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz"
解压
tar -zxvf softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz
cd vpnserver/
make
此后根据提示 所有选择的地方输入 1 并回车即可。
启用 vpnserver 服务：
./vpnserver start
./vpncmd
这里输入 1，然后键入两次回车
至此服务端配置完成。

