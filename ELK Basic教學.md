## 1. 檢查ELK是否安裝成功
* 本機系統 : Ubuntu 16.04
* 虛擬機系統 : CentOS7 1804
* 軟體版本 : ELK 6.3版

### 1.1 檢查Elasticsearch
``` 
# systemctl status elasticsearch
``` 
![](img/1.1.png) 
### 1.2 檢查Logstash
``` 
# systemctl status logstash
``` 
![](img/1.2.png) 
### 1.3 檢查Kibana
``` 
# systemctl status kibana
``` 
![](img/1.3.png) 

開啓瀏覽器，輸入"IP:5601"，例如"192.168.56.1:5601"
看到以下畫面即為Kibana連線成功

![](img/1.4.png) 

## 2. I/O練習

### 2.1 建立檔案
建立一個sample1的檔案
```
# vi sample1.conf
```
輸入以下程式碼，編輯後並儲存
```
input {
	stdin { }
}
output {
	stdout { }
}
```

### 2.2 檢測
使用-t檢測程式是否正常運作
```
# /usr/share/logstash/bin/logstash -f sample1.conf -t
```
輸入後即可看到"Configuration OK"的提示

### 2.3 執行
```
# /usr/share/logstash/bin/logstash -f sample1.conf
```

此時即可輸入"Hello World!"字串，並按ENTER

![](img/2.2.1.png)

看到此畫面，即為logstash運作成功

![](img/2.2.2.png)

## 3. 將log檔匯入Logstash

此範例以BiMap的log檔為例

### 3.1 將log檔匯入虛擬機
#### 3.1.1 Linux系統(使用scp指令)
首先，在"本機"中前往BiMap資料夾的位置，右鍵開啟本機終端機

![](img/3.1.1.png) 

使用```scp```將完整BiMap資料夾複製到虛擬機中的/root資料夾中

```
# sudo scp -r BiMap_sample2/ root@目的IP位置:/root
```
![](img/3.1.2.png) 

>補充說明: ```scp``` 指令跟一般的 ```cp``` 類似， 可以在不同的 Linux 主機之間複製檔案
>其語法為：
>```# sudo scp -r 來源資料夾/ [帳號@目的主機]:目的位置```

#### 3.1.2 Windows系統(使用winscp)

安裝winscp後，直接輸入VM的帳號密碼登入

![](img/3.1.3.png) 

找到要傳輸之檔案，用拖曳的方式即可將檔案複製到虛擬機

![](img/3.1.4.png) 

### 3.2 建立檔案


建立一個sample2的檔案
```
# vi sample2.conf
```
輸入以下程式碼，編輯後並儲存
```
input {
	file {
    	path => "/root/BiMap_sample2/u_ex180101.log"
    	start_position => beginning
	}
}
output {
	stdout { codec => rubydebug }
}
```

### 3.3 執行
```
# /usr/share/logstash/bin/logstash -f sample2.conf
```
看到以下畫面即為logstash讀取log檔案成功

下方的三筆資料即為log檔的前三筆

![](img/3.2.2.png) 

## 4. 將檔案利用Logstash傳入Elasticsearch

### 4.1 建立檔案
建立一個sample3的檔案
```
# vi sample3.conf
```
輸入以下程式碼，編輯後並儲存
```
input {
	file {
    	path => "/root/BiMap_sample2/u_ex180101.log"
    	start_position => beginning
    	sincedb_path => "/dev/null"
	}
}
output {
	elasticsearch {
		hosts => ["localhost:9200"]
		index => "logstash-bimap1-%{+YYYY.MM.dd}"
	 }
	stdout { codec => rubydebug }
}
```

### 4.2 將檔案傳入至Elasticsearch
執行以下指令
```
# /usr/share/logstash/bin/logstash -f sample3.conf
```
看到以下畫面即為logstash讀取log檔案成功

下方的三筆資料即為log檔的前三筆

![](img/4.0.png)

### 4.3 確認log檔資料行數
首先，開啟虛擬機的TTY2，進入BiMap_Sample2資料夾

使用Linux中的```wc```指令計算 u_ex180101.log的資料行數

> wc 可用來計算指定檔案內容的換行數（newline）、字數（word）與位元組數（byte）

```
# wc u_ex180101.log
```
可以看到資料行數為381945筆

![](img/4.1.1.png)


若是檔案中有"#"開頭的註解文字，可以用以下指令進行wc查詢
```
# cat u_ex180101.log | grep -v "^#" | wc
```

### 4.4 在Kibana建立Elasticsearch的index

#### 4.4.1前往Kibana的頁面
![](img/4.1.2.png)

按下左側Management，點選右側Kibana的Index Patterns

#### 4.4.2 Create index pattern
下方可以看到已匯入的elasteicsearch的index
![](img/4.1.3.png)

此時，設定kibana的index pattern

因為我們僅要讀取此檔案，因此pattern設為"logstash-bimap1-2018.07.11"

#### 4.4.3 設定timefilter
這邊選擇用何種時間格式去過濾資料，選擇預設的"@timestamp"

![](img/4.1.4.png)

即可創建index

![](img/4.1.5.png)

圖中我們可以看到有16個fields特徵與類型

#### 4.4.4 查看Kibana上的log呈現
點選左側欄位中的Discover，可能會發現還是無法看到匯入的資料

![](img/4.1.7.png)

因為時間區間需要設定為資料timestamp的時間

時間區間可從畫面的右上方來更改，最大可以設定為5年

![](img/4.1.9.png)

時間區間設定完成後，即可看到匯入的完整資料

點選畫面上方的資料bar，可看到資料筆數為381945筆，與wc指令查詢結果吻合

![](img/4.1.10.png)

 點選單筆資料向下展開，可以看到資料細項呈現，有table與json兩種呈現方式
 
 ![](img/4.1.11.png)
 

## 5. Grok
可以看到，單元4所輸入後的資料還是有些雜亂，那要怎麼做呢?

Logstash 在 ELK 架構中，是負責把收到的文字資料，做特定的規則處理，就可以變成指定的欄位，而建立欄位的優點是方便做搜尋與檢索。

資料建立欄位就得使用Grok filter，在grok 區塊中宣告match，當來源欄位符合Grok Patterns，即會建立指定欄位。
```
input {
  # ...
}
filter {
  grok {
      match => [ "來源欄位", "Grok Patterns" ]
  }
  # ...
}
output {
  # ...
}
```

常見的Grok Pattern產生有兩種型式:

> 1. 預設的Grok Patterns

> 2. 自行撰寫Regular Expression


### 5.1  Grok Patterns

Grok Patterns 的基本用法是： 

```%{Pattern名稱:欄位名稱:型別}```

* Pattern名稱:
Grok Patterns 只是 Grok 預先寫好的常用正規表示式，可以參考  [grok-patterns](http://t.cn/RV3BM2B)
* 欄位名稱:
欄位名稱是自訂的輸出名稱，當符合 Pattern 時，就會建立這個欄位，並把符合 Pattern 的內容填入這個欄位
* 型別:
預設型別都是字串。可以參考[Field datatype](http://t.cn/REjyfxa)
     
舉例來說，我們有一個筆 Log 如下：
```
2018-01-01 00:00:00 10.10.21.91 POST /api/BiMapCardInfo/Get - 80 - 10.10.21.120 - 200 0 0 234
```
通常面對Log解析需要field的字典檔，也就是解析後的目標

可以看到此log檔的field，根據此field進行Grok解析

![] (img/5.1.png)

|<center>Fields</center> |  <center>Value</center>| 
| :----------:  | :-----: |
|  date |  2018-01-01   |
| time |  00:00:00  | 
|  s-ip | 10.10.21.91 | 
| cs-method |  POST | 
| cs-uri-stem | /api/BiMapCardInfo/Get | 
| cs-uri-query | - | 
| s-port | 80 | 
| cs-username | - | 
| c-ip | 10.10.21.120 | 
| cs(User-Agent)  | - | 
| sc-status | 200 | 
| sc-substatus | 0 | 
| sc-win32-status | 0 | 
| time-taken | 234 | 

接著透過 message 欄位取得 Logstash 收到的 Log 資料，所以來源欄位就是 message，Patterns 如下：

```
grok {
	match => [ "message",
				"%{DATE:date} 
				%{TIME:TIME}  
				%{IPV4:s_ip} 
				%{WORD:cs_method} 
				%{URIPATH:cs_uri_stem} 
				%{NOTSPACE:cs_uri_query} 
				%{WORD:s_port} 
				%{NOTSPACE:cs_username} 
				%{IPV4:c_ip} 
				%{NOTSPACE:cs_useragent} 
				%{BASE10NUM:sc_status} 
				%{BASE10NUM:sc_substatus} 
				%{BASE10NUM:sc_win32_status} 
				%{BASE10NUM:time_taken}" 
			]
 }
```
  當 Log 符合這個 Patterns 時，就會切分出上方的 14 個欄位
  
### 5.2 Grok Regular Expression

Grok Regular Expression 的用法是：

```(?<欄位名稱>Regular Expression)```

Grok Patterns 只是 Grok 預先寫好的常用正規表示式，使用上會比較受限，所以也可以自己寫正規表示式，不一定要用 Grok Patterns 所提供的。

Log 用跟上述一樣的例子，把 Grok Patterns 的date跟time轉換成正規表示式，如下：
```
grok {
	match => [  "message", 
				"(?<date>\d{4}-\d{2}-\d{2}) 
				 (?<time>\d{2}:\d{2}:\d{2})
				 %{IPV4:s_ip} 
				 %{WORD:cs_method} 
				 %{URIPATH:cs_uri_stem} 
				 %{NOTSPACE:cs_uri_query} 
				 %{WORD:s_port} 
				 %{NOTSPACE:cs_username} 
				 %{IPV4:c_ip} 
				 %{NOTSPACE:cs_useragent} 
				 %{BASE10NUM:sc_status} 
				 %{BASE10NUM:sc_substatus}
				 %{BASE10NUM:sc_win32_status} 
				 %{BASE10NUM:time_taken}"
			 ]
 }
```
由此可知，Grok Patterns 及 Regular Expression 是可以混用的

### 5.3 Grok Debugger Tool
在撰寫 Grok Pattern 時，可以透過一些工具檢查語法對不對，是否有產出預期的欄位。常用的工具為 [Grok Debugger](https://grokdebug.herokuapp.com/) 

#### 5.3.1 Grok Debugger

![] (img/5.2.png)

第1欄是要解析的message
第2欄是要嘗試的pattern
最底下則是解析pattern的Match Output


#### 5.3.2 Grok Discover
如果對於Log的Pattern沒有那麼熟悉，也有工具可以協助[Grok Discover](https://grokdebug.herokuapp.com/discover?#)
將待解析的message放在第1欄，按下右下角的Discover， 即可在下方得到自動解析的Pattern，它是依照所呈現之格式去判定。
可以看到上方的例子，日期、時間、IP、路徑都有解析出來，剩下的就得自行解析
但是用戶需要自行檢驗此Pattern是否符合真實使用情境。

![](img/5.3.png)
 
 
 若是不想切換Kibana的Debugger去嘗試，此網站也有Debugger的功能
 
![](img/5.4.png)
  
### 5.4 Grok解析流程

* STEP1. 將1筆log資料放進[Grok Discover](https://grokdebug.herokuapp.com/discover?#)解析，參考部份結果
* STEP2. 未解析或尚未正確解析的部份，可參考 [grok-patterns](http://t.cn/RV3BM2B)找出對應Pattern
* STEP3. 若常用Pattern無法解析完全，自行撰寫Regular Expression
* STEP4. 帶入Kibana Dev Tools中的Grok Debugger進行最終確認

  
### 5.5 使用Grok後的資料呈現
建立一個sample4的檔案
```
# vi sample4.conf
```
輸入以下程式碼，編輯後並儲存

```
input {
  file {
    path => "/root/BiMap_sample2/demo01.log"
    start_position => beginning
    sincedb_path => "/dev/null"
  }
}
filter {
  if [message] =~ "^#" {
    drop {}
  }

  grok {
    match => [ "message",  "(?<date>\d{4}-\d{2}-\d{2}) (?<time>\d{2}:\d{2}:\d{2}) %{IPV4:s_ip} %{WORD:cs_method} %{URIPATH:cs_uri_stem} %{NOTSPACE:cs_uri_query} %{WORD:s_port} %{NOTSPACE:cs_username} %{IPV4:c_ip} %{NOTSPACE:cs_useragent} %{BASE10NUM:sc_status} %{BASE10NUM:sc_substatus} %{BASE10NUM:sc_win32_status} %{BASE10NUM:time_taken}" ]
  }

    mutate { 
    add_field => {"datetime" => "%{date} %{time}"}
    convert => {"time_taken" => "integer"}
  }

  date {
          match => [ "datetime","YYYY-MM-dd HH:mm:ss"]
          target => "@timestamp"
          timezone => "Asia/Taipei"
  }
    mutate {
    remove_field => ["date"]
  }
}
output {
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "kibana-bimap_demo01"
    }
  stdout { codec => rubydebug }
}
```

執行
```
# /usr/share/logstash/bin/logstash -f sample4.conf
```
接著前往瀏覽器中的Kibana頁面，可以看到匯入資料的field已經包含剛剛設置的14個fields，如下圖所示，之後就可以利用這些field去做視覺化呈現。

![](img/5.5.png)

## 6. Kibana
Kibana也就是ELK 裡的Ｋ，將Logstash過濾完並儲存到Elasticsearch的資料，透過設定 index 進行索引，接著就可以視覺化呈現囉~
### 6.1 Kibana介面介紹
在做視覺化圖表之前，先讓我們來檢視一下介面上的區塊及其所屬的功能。

![](img/6.1.png)

* **Menu**：Kibana 的主功能列表包含了以下功能
	* Discover : 用以檢視各索引下的記錄內容及總記錄筆數
	* Visualize : 將搜尋出的數據以長條圖、圓餅圖、表格等方式呈現
	* Dashboard : 將以儲存的搜尋結果或已完成的圖表組合成一份快速報表
	* Timelion : 時序性的監看 query
	* APM : 監控應用程式與服務的性能
	* Dev Tools : 提供一個在 Kibana 直接呼叫 Elasticsearch 的方式
	* Monitoring : 監視Logstash部署的運行狀況和性能的運行時指標
	* Management : 設定 Kibana 對應的 Elasticsearch index patterns，管理已經儲存好的搜尋結果物件、視覺化結果物件，及進階資料過濾設定
* **Index** : 要搜索的 Index pattern
* **Available Fields** : 搜索 index 下所包含的屬性(也就是我們在 Logstash 切出來的部分)
* **Timestamp** : 資料時間，可用於特定時間區間資料量觀察
* **Source** : 也就是我們先前所儲存的資料內容
* **Search** : 預設 ”*“ 搜尋 index 下所有紀錄

一般來說我們首次使用流程為開啟 Managment 先做 index patterns create 的動作，再以 Discover 搜尋特定的條件或特定的時間區間資料，並且至 Visualize 以圖表的方式將我們想知道的資訊視覺化，最後以 Dashboard 組合顯示出所有我們想了解或觀察的資訊。

### 6.2 Discover

Discover 頁面用於交互式探索數據，可以輸入搜索條件，過濾搜索結果，然後查看檔案內容。還可以看到相符合的檔案總數，獲取字段值的統計情況。如果field配置了時間，檔案會在畫面頂部以柱狀圖的形式展示時間分佈情況。

![](img/6.2.0.png)

### 6.2.1 設置時間

Time Filter可以限制搜尋結果在一個特定的時間區間內。預設的Time Filter設置為最近15 分鐘。你可以用畫面右上角的時間選擇器(Time Picker)來修改時間過濾器，或者選擇一個特定的時間間隔，或者直方圖的時間範圍。時間區間最小可以是15分鐘，最大可以到5年

![](img/6.2.1.png)

### 6.2.2 搜索數據

在Discover進行搜尋，你就可以搜尋符合當前索引模式的索引數據了。你可以直接輸入簡單的請求字符串，也就是用Lucene query syntax，也可以用完整的JSON格式的Elasticsearch Query DSL。

下圖以簡易的"sc_status:301"條件去搜尋，可以看到符合的條件會在下方以黃色醒目提示，搜尋結果的筆數從總筆數254筆變成9筆

![](img/6.2.2.0.png)

若想要同時搜尋兩個以上的條件，可以用左上角的```Add a filter```將所需要的篩選條件輸入，即可做多條件的搜尋。

![](img/6.2.2.1.png)

下圖呈現的是以"sc_status:301"與"cs_method:POST"為條件進行搜尋，搜尋結果 共8筆

![](img/6.2.2.2.png)

搜尋完的結果也可以用上方的```SAVE```鍵進行儲存，也可以用```OPEN```去讀取之前儲存的搜尋。若要清除當前搜索或開始一個新搜索，點擊Discover 工具欄的```New``` 按鈕。

### 6.2.3 按字段過濾

可以將搜尋結果進行過濾，只顯示在某字段中包含了特定值的資料。也可以創建反向filter，排除掉包含特定字段值的文檔。

下圖利用左側的```ADD```按鈕加入所要觀察的fields，此圖選擇的field是"c_ip"'、"cs_method"與"time_taken"

![](img/6.2.2.3.png)


### 6.3 視覺畫圖表呈現（Visualization)

Kibana的作圖類型

![](img/6.3.0.png)

### 6.3.0 做圖框架
一般視覺化可以先進行呈現架構的設計
![](img/6.3.0.1.png)

![](img/6.3.0.2.png)
### 6.3.1 總筆數

以數字呈現該批IIS log資料的總筆數。

![](img/6.3.1.png)

### 6.3.2 訪客人數

以數字呈現該批資料的訪客人數。

在訪客人數的計算上，通常IIS log會以client IP 的unique count (每個IP都只計入一次)作為計算方式。

![](img/6.3.2.png)

### 6.3.3 請求狀態分析

在IIS log中，可以根據client的請求狀態（post/get, *etc.*)，以甜甜圈圖呈現。

![](img/6.3.3.png)

### 6.3.4 回應狀態分析

根據IIS log裡的網頁回覆狀態（200/404, *etc.*)，以圓餅圖做呈現。

![](img/6.3.4.png)

### 6.3.5 回應時間分析
根據IIS log的資訊，分析網頁回應request時間。

![](img/6.3.5.png)

### 6.3.6 流量趨勢分析
若想透過IIS log分析流量變化時，可用柱狀圖做呈現。

![](img/6.3.6.png)

### 6.4 儀表板呈現 （Dashboard）

當建立好幾個視覺化的圖形並存檔後，可以點選Dashboard將這些視覺畫圖表做組合，合併呈現於同一個Dashboard。

![](img/6.4.1.png)

![](img/6.4.2.png)

### 6.5 Dev Tools

Kibana提供了Console UI來通過REST API與Elasticsearch互動，Console位於Kibana的Dev Tools欄下。Console有兩個主要區域，左邊是編輯區用來寫REST請求，右邊用來顯示執行結果。


![](img/6.5.1.png)

確認執行結果正確後，可以使用```Copy as cURL```按鈕取得CURL。由此可知，ELK的應用層面很廣，輸出部份不只有Kibana可以呈現，還可以進行API串接，將搜尋結果傳給Web UI或是BI商用軟體，進行即時呈現。

![](img/6.5.2.png)

### 6.5.1 Query DSL

- **Leaf query clauses 簡單查詢**

這種查詢可以單獨使用，針對指定的字段查詢指定的值。

- **Compound query clauses 複合查詢**

複合查詢用於組合成複雜的查詢語句，比如not, bool等，輸入格式為槽狀結構。
```
GET _search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
### 6.5.2 DSL 查詢語法實作
- Lab 1
查詢全部的index
```
Get _cat/indices
```
計算該index筆數
```
Get bimap_iis/_count
```
- Lab 2
精準查詢（全部index) sc_status 欄位中回應為200的資料
```
GET _search 
{
  "query": {
    "match": {"sc_status":"200"}
  }
}
```
```
GET  bimap_iis/_search?q=200
```
- Lab 3
查詢訪客人數

做查詢時，有一些查詢滿足不了我們的查詢條件，這時候就需要aggs(aggregation聚合函數)
```
GET bimap_iis/_search
{ 
  "size" : 0,
  "aggs" : { 
    "visitor count" : { 
      "cardinality" : { 
        "field" : "c_ip.keyword" 
      } 
    } 
  } 
}
```
- Lab 4
查詢等待時間為0~10秒的資料
```
GET bimap_iis/_search
{
    "query" : {
        "range" : {
            "time_taken" : {
                "gte" : 0,
                "lt" : 10
            }
        }
    }
}
```
- Lab 5
去找"BiMapCardInfo"字串出現在任何的欄位
```
GET bimap_iis/_search
{
  "query": {
    "query_string": {
      "query": "BiMapCardInfo"
    }
  }
}
```
- Lab 6
同時查詢多個欄位
```
GET bimap_iis/_search
{
    "query": {
        "multi_match" : {
            "query" : "POST",
            "fields": ["cs_method", "cs_uri_stem"]
        }
    }
}
```
- Lab 7
同時查詢多個欄位以及只顯示部份資料
```
GET bimap_iis/_search
{
    "query": {
        "multi_match" : {
            "query" : "POST",
            "fields": ["cs_method", "cs_uri_stem"]
        }
    },
    "_source": ["date", "cs_method", "cs_uri_stem"]
}
```
- Lab 8
萬用字元查詢（Wildcard Query)
```
GET bimap_iis/_search
{
    "query": {
        "wildcard" : {
            "cs_method" : "*t"
        }
    },
    "_source": ["date", "cs_method"]
}
```
- Lab 9
正則表達式查詢 （Regexp Query ）
```
GET bimap_iis/_search
{
    "query": {
        "regexp" : {
            "cs_method" : "[a-z][a-z]t"
        }
    },
    "_source": ["date", "cs_method"]
}
```
- Lab 10
 組合過濾（Bool Query）
 
 > AND: *must*
 
 > OR: *should*
 
 > NOT: *must_not*

```
GET bimap_iis/_search
{
    "query": {
        "bool": {
            "should": [
                      { "match": { "cs_method": "POST" }},
                      { "match": { "cs_uri_stem": "POST" }} 
            ] 
        }
     }
 } 
```












