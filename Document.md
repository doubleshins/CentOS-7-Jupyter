# 目錄
[建立虛擬機](#建立虛擬機)  
[啟用SCL](#啟用SCL)  
[CentOS7上安裝Python3](#CentOS7上安裝Python3)  
[創建虛擬環境](#創建虛擬環境)  
[CentOS上安裝Pip](#CentOS上安裝Pip)  
[安裝Jupyter](#安裝Jupyter)  
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
#### CentOS 7附帶Python 2.7.5。
#### SCL將允許您安裝較新版本的python 3.x以及默認的python v2.7.5。
```bash
# ssh dic@192.168.19.13
# python -V
Python 2.7.5

# sudo yum install -y centos-release-scl
# sudo yum install -y rh-python36 vim-enhanced

# scl enable rh-python36 bash
# python --version
Python 3.6.3
```

<a name="創建虛擬環境"/>

## 創建虛擬環境
```bash
# mkdir ~/my_project1
# cd ~/my_project1

更新pip
# pip install --upgrade pip

創建一個獨立的虛擬環境，與系統自帶的 Python 隔離開來
# pip install virtualenv
# virtualenv my_venv
# source my_venv/bin/activate

更新pip
# pip install --upgrade pip

離開環境
# deactivate
```

<a name="CentOS上安裝Pip"/>

## CentOS上安裝Pip
#### pip是一個包管理系統，它簡化了用Python編寫的軟件包的安裝和管理。
```bash
- 是一個介於IDE(Pycharm, Spider)以及Editor(text,VScode, 記事本)之間的一個讓你可以寫code的工具
# pip install jupyter

- 對網路發動請求的套件，可實作對網頁做get、post等HTTP協定的行為。
# pip install requests

- 借助網頁的結構特性來解析網頁的工具，只需要簡單的幾條指令就可以提取HTML標籤裡的元素。
# pip install beautifulsoup4

建立項目目錄
# mkdir -p /data/jupyter
# mkdir /data/jupyter/root

# pip uninstall XXX

# pip search "XXX"
```


<a name="安裝Jupyter"/>

## 安裝Jupyter
#### 把網站上面的資料複製下來，一筆資料很容易複製，那一千筆呢?，不管是圖片還是文字資料，這就是爬蟲
```bash
產生設定檔：
# jupyter notebook --generate-config --allow-root

編輯設定檔
# vim ~/.jupyter/jupyter_notebook_config.py
...
c.NotebookApp.open_brower = False #不需要再服務器打開瀏覽器
c.NotebookApp.ip = '0.0.0.0' #監聽網路
c.NotebookApp.allow_root = True
c.ContentsManager.root_dir = '/data/jupyter/root'
...

設定密碼
# jupyter notebook password

對外開放8888
# sudo firewall-cmd --zone=public --add-port=8888/tcp --permanent
# sudo systemctl restart firewalld.service

啟動Jupyter Notebook服務器：
# nohup jupyter notebook &
```
![image](https://github.com/doubleshins/CentOS-7-Python-3/blob/master/4MQ3L94.png)










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
- 7步帶你玩轉Jupyter Notebook（CentOS: https://kknews.cc/zh-tw/other/4expomq.html
- 自己架一個 jupyter remote machine: https://medium.com/@chen.ishi/%E8%87%AA%E5%B7%B1%E6%9E%B6%E4%B8%80%E5%80%8B-jupyter-remote-machine-4de7122ba272
- 連接到遠程服務器上的Jupyter: https://www.digitalocean.com/community/tutorials/how-to-install-run-connect-to-jupyter-notebook-on-remote-server
- 在 Centos7 上搭建 Jupyter Notebook 环境: https://segmentfault.com/a/1190000012731626
- 五分钟教会你建立Jupyter notebook服务器: https://python.freelycode.com/contribution/detail/846
- Day-1 Python爬蟲小人生(1): https://ithelp.ithome.com.tw/articles/10202121
- How to Install Python 3 on CentOS 7: https://linuxize.com/post/how-to-install-python-3-on-centos-7/
