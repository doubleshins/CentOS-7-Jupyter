# 目錄
[建立虛擬機](#建立虛擬機)  
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


## 安裝
```bash

```


<a name="參考資料"/>
## 參考資料
- How to Install Python 3 on CentOS 7: https://linuxize.com/post/how-to-install-python-3-on-centos-7/
