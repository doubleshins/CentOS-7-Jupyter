# 目錄
[建立虛擬機](#建立虛擬機)  
[啟用SCL](#啟用SCL)  
[CentOS7上安裝Python3](#CentOS7上安裝Python3)  
[創建虛擬環境](#創建虛擬環境)  
[CentOS上安裝Pip](#CentOS上安裝Pip)  
[Apache](#Apache)  
[MySQL](#MySQL)  
[參考資料](#參考資料)    

<a name="建立虛擬機"/>

## 建立虛擬機
```bash
# vim create_vm_settings.sh
...
vmnames="python_min"
...

# sh create_vm_settings.sh

# qemu-img create -f qcow2 python_min.img 50G

# vim create_vm_settings.sh
...
 <source file="/vmdisk/iso/CentOS-7-x86_64-DVD-1810.iso"/>
...

安裝-選擇『最小型安裝』，但是增加『相容性函式庫』、『開發工具』、『系統管理工具』

# vim create_vm_settings.sh
...
 <source file=""/>
...

# virsh create python_min.xml
```

<a name="啟用SCL"/>

## 啟用SCL
### CentOS 7附帶Python 2.7.5。
### SCL將允許您安裝較新版本的python 3.x以及默認的python v2.7.5。
```bash
# python -V
Python 2.7.5

#sudo yum install centos-release-scl
```

<a name="CentOS7上安裝Python3"/>

## CentOS7上安裝Python3
```bash
# sudo yum install rh-python36
# python --version
Python 2.7.5

# scl enable rh-python36 bash
# python --version
Python 3.6.3
# exit
# python -V
Python 2.7.5
```

<a name="創建虛擬環境"/>

## 創建虛擬環境
```bash
# sudo yum groupinstall "Development Tools"
紅帽的官方說7中的yum已經發生變化，需要使用特定選項來安裝，具體命令如下

# sudo yum groupinstall "Development Tools" --setopt=group_package_types=mandatory,default,optional
# sudo yum -y install vim-enhanced

# mkdir ~/mypython3
# cd ~/mypython3
# scl enable rh-python36 bash
# python -m venv my_venv
# source my_venv/bin/activate
# deactivate
```

<a name="CentOS上安裝Pip"/>

## CentOS上安裝Pip
### Pip是一個包管理系統，它簡化了用Python編寫的軟件包的安裝和管理，例如Python包索引（PyPI）中的軟件包。
```bash
# sudo yum install epel-release
# sudo yum install python-pip
# pip --version
pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)

# pip install --upgrade pip
pip 19.1.1 from /usr/lib/python2.7/site-packages/pip (python 2.7)

# sudo yum install python-devel
# pip install XXX
# pip uninstall XXX
# pip search "XXX"
```

<a name="Apache"/>

## Apache
```bash
# sudo yum install httpd
# sudo systemctl enable httpd
# sudo systemctl start httpd

# sudo firewall-cmd --permanent --zone=public --add-service=http
# sudo firewall-cmd --reload
```

<a name="MySQL"/>

## MySQL
### 最新版本的MySQL是8.0版
```bash
# sudo yum localinstall https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
# sudo yum install mysql-community-server

# sudo systemctl enable mysqld
# sudo systemctl start mysqld
# sudo systemctl status mysqld

# sudo grep 'temporary password' /var/log/mysqld.log
2019-06-11T14:14:52.208324Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: XXXXXXXXX

運行該mysql_secure_installation命令以提高MySQL安裝的安全性：
# sudo mysql_secure_installation
1.輸入root用戶的密碼：
2.請設置新密碼:
3.重新輸入新的密碼：
4.y
5.y
6.y
7.y
8.y
All done!

# mysql -u root -p
```








<a name="參考資料"/>

## 參考資料
- How to Install Python 3 on CentOS 7: https://linuxize.com/post/how-to-install-python-3-on-centos-7/
