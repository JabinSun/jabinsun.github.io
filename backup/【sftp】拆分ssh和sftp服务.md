### 分离SSH和SFTP服务
系统内开启ssh服务和sftp服务都是通过/usr/sbin/sshd这个后台程序监听22端⼝，⽽sftp服务作
为⼀个⼦服务，是通过/etc/ssh/sshd_config配置⽂件中的Subsystem实现的，如果没有配置
Subsystem参数，则系统是不能进⾏sftp访问的。

### 复制SSH相关⽂件，作为sftp的配置⽂件
```shell
cp /usr/lib/systemd/system/sshd.service /etc/systemd/system/sftpd.service

cp /etc/pam.d/sshd /etc/pam.d/sftpd

cp /etc/ssh/sshd_config /etc/ssh/sftpd_config

ln -sf /usr/sbin/service /usr/sbin/rcsftpd

ln -sf /usr/sbin/sshd /usr/sbin/sftpd

cp /etc/sysconfig/sshd /etc/sysconfig/sftp

cp /var/run/sshd.pid /var/run/sftpd.pid > /var/run/sftpd.pid
```

### 修改SFTP配置⽂件
```shell
vim /etc/systemd/system/sftpd.service
==========
[Unit]
Description=sftpd server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service
[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/sftp
ExecStart=/usr/sbin/sftpd -f /etc/ssh/sftpd_config
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s
[Install]
WantedBy=multi-user.target
==========
```

### 更改配置⽂件使得sftpd服务能独⽴并且拒绝其他⽤⼾使⽤
```shell
vim /etc/ssh/sftpd_config
==========
Port 22 ---> Port 2222
#PermitRootLogin yes ---> PermitRootLogin no
#PidFile /var/run/sshd.pid ---> PidFile /var/run/sftpd.pid
Subsystem sftp /usr/libexec/openssh/sftp-server ---> #Subsystem sftp /usr/libexec/openssh/sftp-server
==========
# 添加以下内容
# 指定使⽤sftp服务使⽤系统⾃带的internal-sftp
Subsystem sftp internal-sftp -l INFO -f AUTH
AllowGroups sftpusers
==========
```

### 创建⽤⼾及⽤⼾组
```shell
groupadd sftpusers
mkdir -p /home/sftp
chown root:root /home/sftp
chmod 755 /home/sftp
mkdir -p /u01/scripts
```

### ⽤⼾创建脚本
```shell
vim /u01/sftpuseradd.sh
==========
#!/bin/bash
useradd -g sftpusers -d /home/sftp/$1 $1
chown $1:sftpusers /home/sftp/$1
# ⽂件所属⽤⼾读写执⾏，属组⽤⼾、其他⽤⼾⽆权限
chmod 700 /home/sftp/$1
echo $2 | passwd --stdin $1
==========
```

### ⽤⼾创建脚本执⾏
```shell
chmod 755 /u01/sftpuseradd.sh
# ⽤⼾创建⽰例，第⼀个参数是⽤⼾名，第⼆个是密码
sh /u01/sftpuseradd.sh sftpu xxxxxxxx
```

