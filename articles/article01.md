目標：將ptt爬網資料, 直接存進Sqlite3的records table中.

## 自動化存取指定URL網頁原始碼
	import requests
	# 指定要抓取的網頁URL
	url = "https://www.ptt.cc/bbs/hotboards.html"

	# 使用requests.get() 來抓取
	r = requests.get(url)

	# request.get()回傳的是一個物件, 
	# 若抓成功, 則網頁內容(e.g, html)放在物件的.text屬性
	web_content = r.text
	#print(web_content)

## 解析HTML原始碼（爬蟲）
	# 載入BeautifulSoup套件
	from bs4 import BeautifulSoup

	# (如果檔案在local端）開啟檔案並讀取
	# web_content = open('ptt_hotboard.html').read()

	# 以 Beautiful Soup 解析 HTML 程式碼 : 
	soup = BeautifulSoup(web_content, 'lxml') #html.parser v.s. lxml v.s. html5lib
	# 找出連結標誌且屬性為"board"的資料
	ptt_hot_boards = soup.find_all('a', class_="board")

	# 用pprint印出來的東西比較漂亮整齊
	import pprint
	# 新增空data frame
	data = []

	for a in ptt_hot_boards:
		name = a.find('div', class_='board-name').text
		people = a.find('span').text
		data.append((name,people))

	# 檢驗版名與人氣資料
	# print(data)
	# pprint.pprint(data)
	# 檢驗ptt看板數
	# print(len(data))


	# 紀錄爬網當下的時間戳：
	import datetime

	# 測試: 現在時間
	current_time_string = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
	print(current_time_string) 

## 將資料存入sqlite DB
	# 載入sqlite3套件
	import sqlite3

	# 在目前的目錄下尋找一個叫test.db的資料庫案並建立連線(若不存在則會自動建立一個.)
	conn = sqlite3.connect('test.db') #可自行命名db名稱
	# 後面都用這個 cursor來做SQL操作.
	c = conn.cursor()

	# 砍掉舊資料.如要讓新的資料自動append在舊資料後 則下列兩段程式碼不需執行
	# c.execute('drop table if exists records')
	# c.execute('''create table records
	# 			(boardname text,
	# 			popularity int,	
	# 			timestamp datetime
	# 			)''')
		
	# 將資料存到records table
	for board in data:
		boardname = board[0]
		popularity = board[1]
		timestamp = current_time_string
		c.execute('insert into records VALUES (?,?,?)',(boardname, popularity, timestamp))

## 檢視存取結果
	raws = c.execute('select * from records').fetchall()

	#檢視table內資料列：
	for raw in raws:
		print(raw)

## 將所有變更寫入DB
	# 要commit, 之前的更改或新增動作才會確實寫到 test.db 
	conn.commit()

	# 不會再用到這個db的時候, 順手close它是個好習慣
	conn.close()


完成囉！
