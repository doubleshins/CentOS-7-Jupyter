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

## 啟用SCL
- CentOS 7附帶Python 2.7.5。
- SCL將允許您安裝較新版本的python 3.x以及默認的python v2.7.5。
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

## CentOS上安裝Pip
- pip是一個包管理系統，它簡化了用Python編寫的軟件包的安裝和管理。
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

## 安裝Jupyter
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
- 瀏覽器中自動開啟Jupyter notebook的頁面:

![image](https://github.com/doubleshins/CentOS-7-Python-3/blob/master/img/4MQ3L94.png)



- 點選右上角的python3新建檔案:

![image](https://github.com/doubleshins/CentOS-7-Python-3/blob/master/img/ZbQT96z.png)

## 快速入門

![image](https://github.com/doubleshins/CentOS-7-Jupyter/blob/master/img/1asf.PNG)

- (Ctrl + Enter編譯)
- 查看 Response 回應

```python
import requests
r = requests.get('https://www.youtube.com/')
print(r)
```

```python
顯示物件
print(dir(r))
```

```python
詳細資料
print(help(r))
```

- 伺服器回應的狀態碼
- 檢查狀態碼是否 OK
```python
print(r.status_code)
print(r.ok)
print(r.headers)
```

- 想要下載圖片? : https://www.penghu-nsa.gov.tw/FileDownload/Album/Big/20161012162551758864338.jpg
- 以二進制格式打開一個文件只用於寫入


```python
二進制格式
import requests
r = requests.get('https://www.penghu-nsa.gov.tw/FileDownload/Album/Big/20161012162551758864338.jpg') #變數名稱為pic
print(r.content)
```

- 圖片爬蟲
- w : 寫入檔案。游標的位置會在檔案的最前面，如果沒有檔案會自動建立檔案，如果已經有檔案，會把檔案內的資料全部清空
- b : 以二進位方式打開
```python
import requests
r = requests.get('https://www.penghu-nsa.gov.tw/FileDownload/Album/Big/20161012162551758864338.jpg') #變數名稱為pic
img2 = r.content #變數名稱命名為img2
pic_out = open('myimg.png','wb') #img1.png為預存檔的圖片名稱
pic_out.write(img2) #將get圖片存入img1.png
pic_out.close() #關閉檔案(很重要)
print("完成")
```

- 實驗(1) : https://httpbin.org/get
- Get
```python
payload = {'page': 2, 'count': 25}
r = requests.get('https://httpbin.org/get', params=payload)
print(r.text)
```

```
...
  "args": {
    "count": "25", 
    "page": "2"
  }, 
...
```

```
print(r.url)
```

- 實驗(2) : https://httpbin.org/post
- Post

```python
r = requests.post('https://httpbin.org/post', data=payload)
```

```
...
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "count": "25", 
    "page": "2"
  }, 
...
```

```python
print(r.json())
```

```python
r_dict = r.json()
print(r_dict['form'])
```

- 實驗(3) : https://httpbin.org/basic-auth/corey/testing
- 帳號 : corey
- 密碼 : testing
```python
import requests

r = requests.get('https://httpbin.org/basic-auth/corey/testing', auth=('corey','testing'))

print(r.text)
```

- 實驗(4) : https://httpbin.org/delay/1
- 延遲防止
```python
import requests

r = requests.get('https://httpbin.org/delay/1', timeout=3)

print(r)
```

## 爬蟲小人生(1)
- 把網站上面的資料複製下來，一筆資料很容易複製，那一千筆呢?，不管是圖片還是文字資料，這就是爬蟲
- 以PTTjoke版為例 : https://www.ptt.cc/bbs/joke/index.html

- 因為我想選取的是網頁裡的文章標題，所以soup.select中放的才是div.title a
```
<div class="title">
	<a href="/bbs/joke/M.1560406106.A.D32.html">[ＸＤ] 二師兄</a>		
</div>
```

- 1.先將剛剛下載的Python套件import進來
```python
import requests
from bs4 import BeautifulSoup 

```
- 2.將網頁Get下來
```python
url = "https://www.ptt.cc/bbs/joke/index.html"
r = requests.get(url) #將此頁面的HTML GET下來
print(r.text) #印出HTML
```
- 3.將抓下來的資料用Beautifulsoup4轉為HTML的parser
```python
soup = BeautifulSoup(r.text,"html.parser") #將網頁資料以html.parser
sel = soup.select("div.title a") #取HTML標中的 <div class="title"></div> 中的<a>標籤存入sel
```

- 4.最後寫一個迴圈將爬下來的文章標題印出來
```
for item in sel:
    print(item) 
```


## 藉由擷取上一頁a標籤裡的網址來GET上一頁的網頁
- 看板MobileComm : https://www.ptt.cc/bbs/MobileComm/index.html

```python
import requests
from bs4 import BeautifulSoup
r = requests.get("https://www.ptt.cc/bbs/joke/index.html")
soup = BeautifulSoup(r.text,"html.parser")
u = soup.select("div.btn-group.btn-group-paging a")#上一頁按鈕的a標籤
url = "https://www.ptt.cc"+ u[1]["href"] #組合出上一頁的網址
print(url)
```
#### 需要擷取多頁(ex:3頁)，所以需要重新撰寫程式碼，藉由迴圈來重複GET網頁
```python
import requests
from bs4 import BeautifulSoup
url = "https://www.ptt.cc/bbs/joke/index.html"
for i in range(30): #往上爬3頁
    r = requests.get(url)
    soup = BeautifulSoup(r.text,"html.parser")
    sel = soup.select("div.title a") #標題
    u = soup.select("div.btn-group.btn-group-paging a") #a標籤
    print ("本頁的URL為"+url)
    url = "https://www.ptt.cc"+ u[1]["href"] #上一頁的網址

    for s in sel: #印出網址跟標題
        print(s["href"],s.text)
```


## 
- 


```python

```






## 爬蟲小人生(2)
- 以Dcard : https://www.dcard.tw/f

1.先將剛剛下載的Python套件import進來
```
<h3 class="PostEntry_title_H5o4dj PostEntry_unread_2U217-">（#持續更新）勇敢的臺灣女孩</h3>
```

```python
import requests
from bs4 import BeautifulSoup 
r = requests.get("https://www.dcard.tw/f")
```

```python
soup = BeautifulSoup(r.text,"html.parser") #將網頁資料以html.parser
sel = soup.select("div.PostEntry_content_g2afgv h3") #取HTML標中的 <div class="title"></div> 中的<a>標籤存入sel
for index,item in enumerate(sel):
    print(item) 
```

## 爬蟲小人生(3)
- 以Dcard : https://www.dcard.tw/f

1.目標
```
<h3 class="PostEntry_title_H5o4dj PostEntry_unread_2U217-">（#持續更新）勇敢的臺灣女孩</h3>

soup.find_all('h3')
```

```python
import requests
from bs4 import BeautifulSoup
import re
url = 'https://www.dcard.tw/f'
resp = requests.get(url)
soup = BeautifulSoup(resp.text, 'html.parser')
dcard_title = soup.find_all('h3', re.compile('PostEntry_title_'))
print('Dcard 熱門前十文章標題：')
for index, item in enumerate(dcard_title[:10]):
    print("{0:2d}. {1}".format(index + 1, item.text.strip()))
```

## 爬蟲小人生(4)
- 以google圖片: https://www.google.com/search?q=%E5%91%A8%E5%AD%90%E7%91%9C&source=lnms&tbm=isch&sa=X&sqi=2&ved=0ahUKEwiVquLdr-jiAhViLH0KHfFCBecQ_AUIECgB&biw=1536&bih=750

1.目標
```
<img alt="「周子瑜」的圖片搜尋結果" height="148" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQLHKmn3kp3A6JFGKuNmRs7Gz7hdhrm_eY2gNUG20Wtq_g_BMqwHwl3cgG07A" width="148"/>
```

```python
import requests
import urllib.request
from bs4 import BeautifulSoup
import os
import time
url = 'https://www.google.com/search?q=%E5%91%A8%E5%AD%90%E7%91%9C&source=lnms&tbm=isch&sa=X&sqi=2&ved=0ahUKEwiVquLdr-jiAhViLH0KHfFCBecQ_AUIECgB&biw=1536&bih=750'
photolimit = 10
headers = {'User-Agent': 'Mozilla/5.0'}
response = requests.get(url,headers = headers) #使用header避免訪問受到限制
soup = BeautifulSoup(response.content, 'html.parser')
items = soup.find_all('img')
folder_path ='./photo/'
if (os.path.exists(folder_path) == False): #判斷資料夾是否存在
    os.makedirs(folder_path) #Create folder
for index , item in enumerate (items):
    if (item and index < photolimit ):
        html = requests.get(item.get('src')) # use 'get' to get photo link path , requests = send request
        img_name = folder_path + str(index + 1) + '.png'
        with open(img_name,'wb') as file: #以byte的形式將圖片數據寫入
            file.write(html.content)
            file.flush()
        file.close()
        print('第 %d 張' % (index + 1))
        time.sleep(1)

print("----爬蟲---結束---")
```














## Apache
```bash
# sudo yum install httpd
# sudo systemctl enable httpd
# sudo systemctl start httpd

# sudo firewall-cmd --permanent --zone=public --add-service=http
# sudo firewall-cmd --reload
```


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
- Requests库详解 : https://www.jianshu.com/p/ada99b7880a6
- python requests的content和text方法的区别 : https://blog.csdn.net/xie_0723/article/details/51361006
- 7步帶你玩轉Jupyter Notebook（CentOS : https://kknews.cc/zh-tw/other/4expomq.html
- 自己架一個 jupyter remote machine : https://medium.com/@chen.ishi/%E8%87%AA%E5%B7%B1%E6%9E%B6%E4%B8%80%E5%80%8B-jupyter-remote-machine-4de7122ba272
- 連接到遠程服務器上的Jupyter : https://www.digitalocean.com/community/tutorials/how-to-install-run-connect-to-jupyter-notebook-on-remote-server
- 在 Centos7 上搭建 Jupyter Notebook 环境 : https://segmentfault.com/a/1190000012731626
- 五分钟教会你建立Jupyter notebook服务器 : https://python.freelycode.com/contribution/detail/846
- Day-1 Python爬蟲小人生(1) : https://ithelp.ithome.com.tw/articles/10202121
- How to Install Python 3 on CentOS 7 : https://linuxize.com/post/how-to-install-python-3-on-centos-7/
## Youtube : 
- Python Requests Tutorial: Request Web Pages, Download Images, POST Data, Read JSON, and More : https://www.youtube.com/watch?v=tb8gHvYlCFs
