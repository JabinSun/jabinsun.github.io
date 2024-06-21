### 报错 

Unable to negotiate with 1.1.1.1 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss

### 环境

CentOS版本：CentOS release 6.5 (Final)
OpenSSH版本：OpenSSH_5.3p1, OpenSSL 1.0.1e-fips 11 Feb 2013
OpenSSL版本：OpenSSL 1.0.1e-fips 11 Feb 2013

### 解决

**方法一：**

生成和使用 ECDSA 或 ED25519 主机密钥
```shell
ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''
ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''
```
编辑 `/etc/ssh/sshd_config`文件，添加生成的密钥
```shell
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
```
重启SSH服务
```shell
service sshd restart
```

**方法二：**

升级 OpenSSH 和 OpenSSL

**方法三：**

客户端配置临时解决方案
```shell
sftp -o HostKeyAlgorithms=ssh-rsa,ssh-dss username@hostname
```
