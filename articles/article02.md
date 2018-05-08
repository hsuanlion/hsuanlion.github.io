目標：使用R連接SQLite資料並做簡易視覺化資料探索
## 使用R連接SQLite資料庫
	# 安裝RSQLire套件
	install.packages("RSQLite")
	# 載入套件
	library(RSQLite)	
	library(dplyr)

	# 檢視當前路徑並設定
	getwd()
	setwd("/Users/hsuanlee/Documents/01_佩璇Python/")
	# SQLite連線設定
	con <- dbConnect(SQLite(), dbname="test.db")
	# 檢視所有DB下的tables
	dbListTables(con)
	# 檢視Table "records"的欄位
	dbListFields(con,"records")

## 將資料從DB抓下來
	# (法1與法2效果相同)
	# 法1:
	pttRecords <- 
	  dbSendQuery(con,'select * from records') %>% 
	  fetch() %>% 
	  tbl_df()
	# 法2:
	pttRecords <-
	  dbGetQuery(con,'select * from records')
	
	# 預覽資料  
	pttRecords %>% View()
	# 檢視資料型態
	class(pttRecords)

	# 關閉與SQLite DB的連線
	dbDisconnect(con)

## 簡易資料探索：檢查某時刻下PTT排行前十的版位與人氣
	# 載入繪圖套件ggplot2
	library(ggplot2)

	# 檢視資料前十列（看一下欄位名稱）
	pttRecords %>% head(10)
	# 檢視一下不重複的時間戳記
	pttRecords %>% select(timestamp) %>% distinct()
	# 篩選特定時間戳記的資料並依據人氣由高到低排序
	tmp <- pttRecords %>% filter(timestamp == '2018-05-07 22:24:30') %>% 
	  arrange(desc(popularity))
	# 並取出資料前十列
	tmp2 <- tmp %>% head(10)

	# 直方圖(且x軸版位須依據人氣由高至低做排序)(法1與法2效果相同)
	a <- ggplot(tmp2,aes(reorder(boardname, -popularity) ,popularity))
	# 法1
	a + geom_col()
	# 法2
	a + geom_bar(stat = "identity")

即可從直方圖看出前十名人氣版位依序是哪些，與站上人數多寡：
![Top10](img/programming/ptt_pop_01.png)

完成囉！