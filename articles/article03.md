
利用爬網資料，視覺話PTT版位與人氣概況

## 載入常用套件
    # Install package if not exists 
    pkgs <- c('data.table', "plyr", "magrittr", 'reshape2', 'lubridate', 'stringr', 'dplyr', 'tidyr','shiny','RSQLite','tidyverse','googlesheets','openxlsx','psych','lubridate','ggplot2')

    new.pkgs <- pkgs[!(pkgs %in% installed.packages())]
    if (length(new.pkgs)) {
      install.packages(new.pkgs, repos = 'http://cran.csie.ntu.edu.tw/')
    }
    # Require packages
    ##lapply : 以迴圈方式library()啟用packages
    suppressPackageStartupMessages(lapply(pkgs, library, character.only=T, lib.loc = .libPaths()[1]))

## 載入資料＆檢查
    # 因為要讀的檔案是json file，故先載入json檔處理相關套件
    install.packages("jsonlite")
    library(jsonlite)
    install.packages("rjson")
    library(rjson)

    # 確認與設定目前目錄路徑
    getwd()
    setwd("~") # 回到母目錄

    # 將檔案下載到指定路徑folder下（一次性）
    download.file(url = "https://www.yoptt.com/ptt_pop_20180507.json" , destfile = "/Users/hsuanlee/Documents/05_佩璇R/BlogTopic/data/PTT_dat.json")

    # 開啟檔案
    data <- stream_in(file("Documents/05_佩璇R/BlogTopic/data/PTT_dat.json",open = 'r'))

    # 檢視資料類型與結構
    class(data)
    str(data)

![01](img/programming/article03_01.jpg)  

    # 移除將不需要的id欄位
    data$`_id` <- NULL

    # 將pop從chr型態轉成int ＆ 資料時間+8小時 (ts2)
    data %<>% mutate(pop = as.integer(pop),
                     ts2 = ts + hours(8))
    # 再一次檢視更改後的資料類型與結構
    str(data)
    summary(data)

![02](img/programming/article03_02.jpg) 

## 資料探索基本視覺化

## (1) 最新一次時間戳下的TOP N人氣版位
    # 指定Top N參數
    topN <- 10
    # 抓出最新一次時間戳
    maxTS <- data %>% summarise(maxTS = max(ts2)) %>% ungroup() 

    tmp <- 
      data %>% 
      filter(ts2 == maxTS$maxTS) %>%
      arrange(desc(pop)) %>% 
      head(topN)

    # x軸為版位，y軸為人氣(且版位依照人氣多到寡降冪排序)
    a <- ggplot(tmp, aes(reorder(bn,-pop),pop))

    # 列出所有色調模組
    RColorBrewer::display.brewer.all()
![03](img/programming/article03_03.jpeg) 

    # 繪圖
    a +
      geom_bar(stat = "identity",
               aes(fill = bn)) +  
      geom_text(aes(label = str_c(round(pop/1000,1),'K'),
                y = pop + 300)) + 
      scale_y_continuous(labels = scales::unit_format("k", 1e-3),
                         breaks = seq(0,20000, by=2000)) +
      labs(title = sprintf('%s\nTop%s人氣版位',maxTS$maxTS,topN),
           x = "版位", 
           y = "人氣", 
           fill = '人氣\n版位') + 
      scale_fill_brewer(palette = 'Set3') + 
      theme(text=element_text(family="黑體-繁 中黑"), # 解決中文亂碼問題
            plot.title = element_text(hjust = 0.5)) # 將title置中
![04](img/programming/article03_04.jpeg) 




## (2) 近n天八卦版的流量趨勢
    # 設定回朔天數參數
    diff <- 10
    endDate <- Sys.Date() - days(1)
    startDate <- endDate -days(diff) + days(1)

    tmp <-
      data %>% 
      mutate(tdate = as.Date(ts2)) %>% 
      # filter(tdate == '2018-05-06') %>%
      filter(ts2 >= startDate & ts2 <= endDate) %>%
      filter(bn == "Gossiping")

    # x軸為時間戳，y軸為人氣
    a <- ggplot(tmp, aes(ts2,pop))

    a +
      geom_line(aes(color = factor(tdate))) +
      scale_x_datetime(date_breaks = "1 day") + 
      scale_y_continuous(labels = scales::unit_format("k", 1e-3),
                         breaks = seq(0,20000, by=2000)) + 
      labs(title = sprintf('近%s天Gossiping版的人氣走勢',diff),
           x = "日期時間", 
           y = "人氣") + 
      theme(text=element_text(family="黑體-繁 中黑"), # 解決中文亂碼問題
            axis.text.x = element_text(angle = 90, vjust = 0.5),
            legend.position="none", # 移除legend圖示 (亦可調整left/right and top/bottom)
            plot.title = element_text(hjust = 0.5)) # 將title置中
![05](img/programming/article03_05.jpeg)     
    
      
## (3) 資料日最新一天的八卦版各時段流量
    #取出最新資料日
    maxTD <- data %>%
      mutate(tdate = as.Date(ts2)) %>% 
      summarise(maxTD = max(tdate)) %>% ungroup() 

    tmp <-
      data %>% 
      mutate(tdate = as.Date(ts2)) %>% 
      filter(tdate == maxTD$maxTD) %>%
      filter(bn == "Gossiping")
    
    # 檢查當天各時段資料      
    tmp %>% select (ts2) %>% distinct() %>%  View()

    # 繪圖
    # x軸為時間戳，y軸為人氣
    a <- ggplot(tmp, aes(ts2,pop))

    a +
      geom_line(color = 'pink') +
      geom_point(color = 'pink',size = 1.4) +
      scale_x_datetime(breaks = date_breaks("1 hour"),
                       date_labels = '%H') + 
      scale_y_continuous(
        labels = scales::unit_format("k", 1e-3),
        breaks = seq(0,20000, by=2000)) + 
      labs(title = sprintf('%s Gossiping版各時段人氣走勢',maxTD$maxTD),
           x = "時間", 
           y = "人氣") + 
      theme(text=element_text(family="黑體-繁 中黑"), # 解決中文亂碼問題
            plot.title = element_text(hjust = 0.5)) + # 將title置中
      theme(panel.background = element_blank(),
            axis.line = element_line(colour = "grey"),
            panel.grid.major = element_line(linetype = "dashed",color = "grey90" ))
![06](img/programming/article03_06.jpeg) 

完成囉！
