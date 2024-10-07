# Campus Network  
Use softether to skip campus network verification  

本教程采用Ubuntu系统  

首先查看校园网的53端口是否开放  

使用命令  


```
nslookup baidu.com  
```


![bypass-auth_20210702004828](https://github.com/user-attachments/assets/10420c55-59f9-41fb-8291-6d8c598cebe4)  

可以看到，即使未在校园网上登录账户，但在使用 nslookup 命令时，也还是会输出 baidu.com 的 IP 地址（当然这只是一个 CDN 的地址），这就表明，校园网服务端的 53 端口（默认 DNS 查询端口）是开放的，如果没能显示IP地址那么就请使用nmap等工具查看别的开放端口，然后我们就可以用到下面的工具来实现利用 ping 流量来伪装 tcp/udp 流量，进而实现访问互联网的功能。  

查看服务端防火墙的状态  


```
ufw status 
```
 

显示如下表示处于关闭状态  

root@iZbp1azy5ek8l30twb64etZ:~# ufw status  

Status: inactive  

使用SoftEther VPN Server时一定要关闭防火墙  

关闭防火墙  


```
ufw disable 
```
 

开启防火墙  


```
ufw enable  
```


安装wget  


```
apt install net-tools wget -y 
```
 

使用命令查看443端口是否被占用  


```
ss -tuln  
```


或者  


```
netstat -tuln  
```


![bypass-auth_20210702011740](https://github.com/user-attachments/assets/4fc5a5ef-d553-4953-bf89-338a8af87642)  

可以看到 443 端口并未被占用，此时我们进行下一步。  

使用lsof命令找到占用443端口的进程  


```
sudo lsof -i :443  
```


使用命令将进程杀死  

sudo kill -9 <这里是对应端口的PID>
假设找到的进程ID是1234，然后使用kill命令终止它  

sudo kill -9 1234  

服务端安装 SoftEther VPN Server  


```
wget -N --no-check-certificate "https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/releases/download/v4.28-9669-beta/softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz"  
```


国内服务器  


```
wget -N --no-check-certificate "http://47.106.20.4/softwares/softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz"  
```


解压  


```
tar -zxvf softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz  
```


```
cd vpnserver/  
```



```
make  
```


此后根据提示 所有选择的地方输入 1 并回车即可。  

![bypass-auth_20210702015328](https://github.com/user-attachments/assets/1593e964-c204-4126-8c0e-fc1f9b9c6a07)  

启用 vpnserver 服务：  


```
./vpnserver start  
```



```
./vpncmd  
```


这里输入 1，然后键入两次回车  

![bypass-auth_20210702015835](https://github.com/user-attachments/assets/8f41d81f-1bac-4d9d-8247-09d88f2e75a1)  

至此服务端配置完成。  

配置本地客户端  

先下载 SoftEtherVPN 这个软件，你可以点击下面的链接下载这个软件。  

http://47.106.20.4/softwares/softether-vpnserver_vpnbridge-v4.28-9669-beta-2018.09.11-windows-x86_x64-intel.exe  

下载完成后进行安装，选择最下面的一个进行安装（软件语言会默认匹配当前安装设备的显示语言）  

![bypass-auth_20210702021023](https://github.com/user-attachments/assets/4dbd41bb-c4b1-4a76-8728-535bd3bbb4d8)  

一路选择下一步，安装成功之后如下图所示：  

![bypass-auth_20210702021222](https://github.com/user-attachments/assets/c5efa836-24c2-442c-9b89-5453218c5a23)  

选择新设置 (N)，在弹出的窗口内请根据图片提示填写相关信息：  

![bypass-auth_20210702021705](https://github.com/user-attachments/assets/d2474176-3d7a-4916-b53f-930a92e90742)  

选中刚刚配置好的连接，选择连接，在弹出的窗口内填写管理密码（首次连接时，没有旧密码，你填写的密码就是你以后连接管理的时候要输入的密码）。  

随后，管理虚拟 HUB → 管理用户 → 新建用户，新建用户时候填写的信息是要在 OpenVPN 中要用到的。  

![bypass-auth_20210702022444](https://github.com/user-attachments/assets/48116079-43ea-4077-af48-85cbc754a73b)  


![bypass-auth_20210702022747](https://github.com/user-attachments/assets/d32377c0-5ae3-4185-9315-80cd0222458f)  

随后，在 3 处，选择启用 SecureNAT  

![bypass-auth_20210702023036](https://github.com/user-attachments/assets/39b80c71-6618-41d0-ba6b-3945e6824647)  

确认之后，在管理界面选择 OpenVPN/MS/SSTP 设置，将端口号改为 53（如果后续不行的话，请尝试修改 67,68,69 这三个端口号并重复后续操作），然后点击 OK / 确定，然后再次点击进入此界面，选择为 OpenVPN Clients 生成配置样本文件。  

![bypass-auth_20210702023533](https://github.com/user-attachments/assets/4738c009-3589-4b5c-90ad-0af793436aed)  

解压你保存的压缩包文件，提取出文件名内含有 *openvpn_remote_access* 的文件，保留备用。  

配置 OpenVPN  

先下载 OpenVPN，你可以点击下面的链接下载这个软件。  

http://47.106.20.4/softwares/openvpn-install-2.4.7-I607-Win10.exe  

安装好并打开之后在桌面托盘处找到这个软件，右键单击图标选择 选项 - 高级，复制配置文件的文件夹路径到资源管理器打开，将刚刚提取保留的文件粘贴到此文件夹内，然后在托盘处右键单击图标并开启连接即可。  

![bypass-auth_20210702024133](https://github.com/user-attachments/assets/c6c4c542-0e2e-4db5-a567-4b349a17c4df)  

成功连接到互联网！  

![bypass-auth_20210702024444](https://github.com/user-attachments/assets/d297c07e-8a09-4f9e-b45a-abde435ef99d)  

注意如果您的SoftEther DNS无法ping通那么可以在SoftEther VPN Server中修改动态DNS设置!  

使用OpenVPN时候可能遇到的问题
1.连接失败 查看日志报错：“There are no TAP-Windows adapters on this system ..... 
下载tap-windows-9.21.1.exe就可以解决

2.连接的时候显示已连接并分配ip，访问网页，无响应
Win + r 输入 services.msc
然后找到OpenVPN Interactive Service服务，再连接就行
