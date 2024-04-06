---
title: Airflow
description: 
aliases: 
tags: 
draft: false
date: 2024-04-05"
create: 2024-04-05 12:38
---

## 1. Intro

[Airflow 介紹 - HackMD](https://hackmd.io/@JohnCena/Hki7kBTlP) 是很好的入門介紹

裡面有放一張截圖，囊括了 Airflow 重要元素:
![](https://miro.medium.com/v2/resize:fit:828/format:webp/1*czjWSmrjiRY1goA0emv7IA.png)

ref: [Understanding Apache Airflow’s key concepts | by Dustin Stansbury | Medium](https://medium.com/@dustinstansbury/understanding-apache-airflows-key-concepts-a96efed52b1a)

- 從要件的角度來看 (初級使用者需要知道的)
	- webserver
		- 這個讓我們有漂亮的 Web UI 可以看 + 互動(使用)
	- scheduler
		- scheduler 要運行起來，才能執行 task，否則 Web UI 就只是個空架子
	- metadata database
		- 所有的記錄 (?) 都存在 metadata database 裡面
		- 預設是 sqlite
		- 可以用一般 sql語句 去查看 database 有哪些表和裡面的內容 (看看而已，但我覺得滿有趣，是另一種角度XD)
- 運行的單位
	- task
		- 對應一個python function
		- Taskflow API 中會加 decorator `@task`
	- dag
		- DAG: directed Acyclic Graph (就是 data structure 的 有向無環圖~)
		- graph的node是一個task => dag 由數個task構成
		- Taskflow API 中會加 decorator `@dag`
	- dag run
		- dag run 是 dag 的一次運行
		- 就像是 一個class 可以建出 多個class object，一個dag跑很多次 就有很多個 dag run

### 1.1. 一個 由淺入深 的範例
[LeeMeng - 一段 Airflow 與資料工程的故事：談如何用 Python 追漫畫連載](https://leemeng.tw/a-story-about-airflow-and-data-engineering-using-how-to-use-python-to-catch-up-with-latest-comics-as-an-example.html) ^c7dpkm

非常推薦! 寫的很有故事性，不過我看到 排程(scheduling) 的時候有點頭昏，這部分也確實是比較複雜的概念~ 多看幾次 + 自行實作 漸漸就能領會~

## 2. 建立環境


### 2.1. 安裝 基本款 (sqlite, SequentialExecutor)

[[Day4] Airflow 快樂安裝指北(Windows篇) - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天 (ithome.com.tw)](https://ithelp.ithome.com.tw/articles/10321260)

- 基本上照著操作，名稱可以設定自己喜歡的
	- 專案資料夾 不一定要叫 `airflow`
	- 虛擬環境名稱 不一定要叫 `airflow-env`


建立虛擬環境並且進入

install airflow with postgres (版本自選)
```bash
pip install "apache-airflow[postgres]==2.8.2" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.8.2/constraints-3.11.txt"
```

添加 environment variable: `AIRFLOW_HOME`
```bash
echo 'export AIRFLOW_HOME=$HOME/<專案資料夾>' >> ~/.bashrc
```

對 Airflow整個系統 很重要的一個檔案: `airflow.cfg`  
它掌管了幾乎全部(?) Airflow 的設定

為了把它創建出來，我們要下指令
```bash
airflow db init
```

然後 `${AIRFLOW_HOME}` 底下的目錄結構:
```
.
├── airflow.cfg
├── airflow.db
└── logs
```

`airflow.db` 是預設的 sqlite database，這被寫在 airflow.cfg 裡面的 `sql_alchemy_conn`

### 2.2. 進階=> 平行化 (mysql / postgresql, LocalExecutor)

預設的 sqlite 只建議 test，production 換成別的，例如 MySQL, PostgreSQL

這個提醒在 使用 SequentialExecutor (default in airflow.cfg)，開啟 webserver 後 Web UI 在網頁最上方會顯示

#### 2.2.1. 建立 (mysql / postgresql) database
這其實跟 Airflow 是分開的，就當作是建一般用途的 database，沒什麼不一樣

也就是說，要怎麼建 mysql 或 postgresql 資料庫，就怎麼建


> [!NOTE] 連線方式
> 除了下面介紹的 sqlalchemy，也可以用 其他connector 的連線方式
> (最初我還不熟 sqlalchemy 的時候，怕是對 url格式不熟悉而設置錯誤，所以用了 connector 給 config (dict)，確認可以連線成功)

#### 2.2.2. 設置 sql_alchemy_conn

sql_alchemy_conn 跟 python 用 sqlalchemy 怎麼連線db 是一樣的東西 (這是我學習使用 sqlalchemy 才發現的事，之前沒多想)

以 postgresql db 為例
```toml
# file: airflow.cfg
sql_alchemy_conn = postgresql+psycopg2://<user>:<password>@<host>[:<port>]/airflow_db
```
(這個被稱作 url)

`airflow_db` 是我建db時取的名，作為 Airflow 的 metadata database

還記得最一開始我們下過 `airflow db init` 嗎? 這步會依據 airflow.cfg 裡面指定好的 sql_alchemy_conn 去設置那個 db

現在用
```bash
airflow db migrate
```
來做 從 sqlite 遷移到 postgresql 的動作 (雖然現在好像也可以用  `airflow db init`，但我不確定會不會有其他影響)


> [!COMMENT] 一台機器、多個使用者 在使用 Airflow 的時候
> 使用者A 建了一個 db A 作為 metadata database；使用者B 似乎也可以把 metadata db 設成 db A (這可能是 當初 db A 使用權限沒隔離好的問題)
> 
> 不過我的建議是，不要用別人建的 metadata database，自己建自己用



> [!INFO] mysql 的driver 只推薦 `PyMySQL` or `mysql-connector-python`
> ref: [There are many options for connecting MySQL from Python, but let's use PyMySQL or mysql-connector-python for now. - DEV Community](https://dev.to/miyachin/there-are-many-options-for-connecting-mysql-from-python-but-lets-use-pymysql-or-mysql-connector-python-for-now-epe)


> [!QUESTION] 名詞辨析
> 我不是很確定 connector, driver 是否有所不同，但我這裡是當作同樣的東西
> - 建議選用
> 	- mysql -> `PyMySQL` or `mysql-connector-python`
> 	- postgresql -> psycopg2
 


#### 2.2.3. 為何我用 postgresql 而非 mysql

最一開始我是用 mysql 作為 backend db 的，airflow.cfg 設定也有成功，webserver, scheduler 都跑起來

但實際執行一個 dag 後才發現，受到限制－－XCom size limitations
一個dag中 task之間透過 XCom 傳遞 (某一個task的return 給下一個task，內容都是存在XCom裡面)，而根據 [Airflow XCOM: The Ultimate Guide - marclamberti](https://marclamberti.com/blog/airflow-xcom/) 所說，MySQL 只有 64 KB，而 PostgreSQL 有 1GB (應該是很夠了)~

[使用localexecutor进行基本生产安装的apache airflow mvp完整指南-CSDN博客](https://blog.csdn.net/weixin_26717681/article/details/108496426)
用 postgresql，可以參考~

#### 2.2.4. 更改 Executor
```toml
# file: airflow.cfg
executor = LocalExecutor
```

#### 2.2.5. 觀察
`airflow run scheduler` 後會在 terminal 印出訊息，開頭的部分會寫說用哪個 metadata database，以及用甚麼 executor

## 3. 寫dags

在寫 code之前，需要先確定好 .py 要放在哪裡，所以腦中先建立好 目錄結構的概念
### 3.1. 目錄結構
[Modules Management — Airflow Documentation (apache.org)](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/modules_management.html#typical-structure-of-packages)


> [!NOTE] 我的理解與設計
> 一台機器、一個使用者 在一個時間只能使用一個 Airflow project (`${AIRFLOW_HOME}` 在airflow.cfg 的設定)  
> 如果有多個 project (github上 多個repository) 都有使用 Airflow 的需求，可是 Airflow project 又只能用一個 (不說 "有" 一個 是因為 可以建多個 Airflow project，需要用哪個 再對 `${AIRFLOW_HOME}` 做切換)
> 我的管理方法是，在 `${AIRFLOW_HOME}/dags` 底下建 各個 repository名稱的資料夾


> [!TIP] 
> dag 的 .py檔名建議跟 dag名稱 同名
> 
> 因為 dag名稱 會顯示在 Web UI，如果dag出錯比較好找到對應的 .py  
> (這招也是我看教學看到的，但忘記出處了)



### 3.2. 如何寫dags (語法)

[Airflow 是什麼? 能吃嗎 ? 數據水管工的超級蘑菇 :: 2023 iThome 鐵人賽](https://ithelp.ithome.com.tw/users/20135427/ironman/6216)

看完這個系列的文章就可以自己寫出dags了，極力推薦
- 我列出囊括的幾個重點
	- Taskflow API 是 Airflow 2.0 後的寫法，讓 code 更像 原生的python (而不會因為 airflow 是一個package 有自己專門的語法)
	- scheduling 的設定
		- [[Airflow#^c7dpkm]] 非常詳細、生動的解釋&故事
		- 這是個大坑，最好透過實踐學習
	- debug 的方式
- 另外我補上我使用過的語法，是上面系列文沒介紹到的
	- [Create dynamic Airflow tasks | Astronomer Documentation](https://docs.astronomer.io/learn/dynamic-tasks?tab=taskflow#dynamic-task-concepts) expand() 讓流程很好處理

#### 3.2.1. import 路徑

當專案稍微複雜之後，會把一些 function 獨立到其他檔案，然後 import
所以必須確保路徑是找的到的

`${AIRFLOW_HOME}/dags`, `${AIRFLOW_HOME}/config`, `${AIRFLOW_HOME}/plugins` 會被加到 system path (至少在跑 dags 的時候)


> [!NOTE] 我怎麼看到這三個路徑的
> 在其中一個 dag的.py 裡面寫 `print(sys.path)`
> 在 terminal 下 `airflow dags list-import-errors` (應該不限於，但這確實是個debug好用的cmd)
> 

#### 3.2.2. logging format

因為 dag 是 一個 .py 檔案，我就當作是一般 .py 在開頭寫了

```python
import logging
logging.basicConfig(
    level=logging.INFO,
    # ...
    )
```
結果沒有生效

[python - Apache Airflow - customize logging format - Stack Overflow](https://stackoverflow.com/questions/42141842/apache-airflow-customize-logging-format)  
我沒試過，不過看起來可行

## 4. 運行dags

### 4.1. 檢查dags，沒問題才正式運行

ref: [airflow - Debugging Broken DAGs - Stack Overflow](https://stackoverflow.com/questions/43944741/debugging-broken-dags)

- `airflow dags list` 可以檢查dags的code有沒有問題，正如上面網址標題說的debug
- `airflow dags list-import-errors` 更好用
	- 我的觀察: `airflow dags list` 不知所云，而 `airflow dags list-import-errors` 跟 Web UI 顯示的一致
	- 我個人認為 Airflow 報錯的訊息不是很直觀，有點像是 錯誤訊息說A，但其實是更底層的原因B所導致，而看到A並不是很容易想到B，甚至會在知道B之後還在想這跟A有甚麼關係
	- 熟能生巧，大概幾天後會碰的都碰過了，建立起一種debug的直覺
- 直接用 python 運行一遍 dag 的 .py
- `airflow test <dag> <execution_date>`
	- execution_date 用 YYYY-MM-DD 的格式
- `airflow trigger <dag> <execution_date>`
	- 手動觸發


### 4.2. 正式運行

上線後我會想分成兩種情境，以「爬蟲」舉例，假設我們要爬每日新聞

一個是 排程設定好要跑的，每天都會有新聞更新；另一個是 在我們Airflow dag設定好之前的那些日子，我們可能也需要爬那時的新聞 (當然有個前提是，此時此刻還能夠看到當時的資料，反例是 Google News 有 "洗版" 的問題，舊的新聞被刷掉了)，這時就需要用到 backfill 的指令

- 跑現在的、或新的 task^[這裡用詞我有點猶豫，因為Airflow跑的一個單位是 dag run，而dag是由數個task組成。不過我還是比較想稱呼為 task]
	- 設定好 scheduling (排程)，並且跑 scheduler，如果一切都沒問題就會跑了~
	- 不過似乎一開始要進 Web UI 去把 要跑的dag "unpaused" (有個toggle的鈕)
- 回補 (backfill) 之前的 task
	- 我努力了一陣子，但 下backfill指令 時常出現 deadlock，而我找不出原因，所以想了個繞過的方法
	- 用 trigger指令，寫個 shell script 對過去要跑的所有時間點做trigger (觸發任務的意思)

#### 4.2.1. 時間的坑
- 「時區」是一大重點，似乎 Airflow 裡面 XCom 變數 都是用 UTC時間
- 在 terminal 下指令給的 `<execution_date>` ，也是 UTC時間 (沒指定時間就是 0:00)
	- 不確定可不可以在 airflow.cfg 改設定，但預設是 UTC時間
	- 我也沒有在這邊給包含時間的(只有給日期)，之後可以試試看


## 5. Outro
![[Airflow#^c7dpkm]]


> [!QUOTE] 這篇的作者說他寫的文章
> 這是一篇當初我在入門資料工程以及 Airflow 時希望有人能為我寫好的文章。

而我這篇比較像是個提綱，沒有細講觀念和具體的執行，而是記錄了我遇過的坑，以及我如何從零開始上手 Airflow－－從建立環境(包含了production要用到平行化)、如何寫 dags、正式運行前要怎麼檢查 & debug

