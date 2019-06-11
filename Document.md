# 目錄
[建立虛擬機](#建立虛擬機)  
[啟用SCL](#啟用SCL)  
[CentOS7上安裝Python3](#CentOS7上安裝Python3)  
[創建虛擬環境](創建虛擬環境)  
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
<source file="/vmdisk/iso/CentOS-7-x86_64-DVD-1810.iso"/>
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
```

<a name="創建虛擬環境"/>

## 創建虛擬環境
```bash

```







<a name="參考資料"/>

## 參考資料
- How to Install Python 3 on CentOS 7: https://linuxize.com/post/how-to-install-python-3-on-centos-7/
