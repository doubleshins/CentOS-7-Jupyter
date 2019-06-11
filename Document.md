# 目錄

[建立虛擬機](#建立虛擬機)  
[啟用SCL](#啟用SCL)  
[CentOS7上安裝Python3](#CentOS7上安裝Python3)  
[參考資料](#參考資料)  

<a name="建立虛擬機"/>

## 建立虛擬機
```bash
$ vim create_vm_settings.sh

vmnames="python_min"

$ sh create_vm_settings.sh

$ qemu-img create -f qcow2 python_min.img 50G

光碟機可以進行開機行為
<source file="/vmdisk/iso/CentOS-7-x86_64-DVD-1810.iso"/>
```

<a name="啟用SCL"/>

## 啟用SCL
### CentOS 7附帶Python 2.7.5，這是CentOS基礎系統的關鍵部分。SCL將允許您安裝較新版本的python 3.x以及默認的python v2.7.5，以便系統工具等yum繼續正常工作。
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
shell中的默認Python版本

To access Python 3.6 you need to launch a new shell instance using the Software Collection scl tool:
scl enable rh-python36 bash
```








<a name="參考資料"/>

## 參考資料
- How to Install Python 3 on CentOS 7: https://linuxize.com/post/how-to-install-python-3-on-centos-7/
