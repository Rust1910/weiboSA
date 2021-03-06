项目具体介绍：
[微博话题爬取与存储分析,实战部分](https://luzhijun.github.io/2016/10/29/微博话题爬取与存储分析/#section-2)


## 安装部署
将项目git到本地后，请先确保以下环境已经安装：

*  [scrapy](https://scrapy.org)
*  [datrie](https://github.com/pytries/datrie)
*  [pymongo](https://github.com/mongodb/mongo-python-driver)
*  [mongoDB](https://www.mongodb.org)

自己的运行环境是mac10.12，python3.5，应该向下兼容没有测试。windows的童鞋装scrapy比较麻烦，建议安装了anaconda后比较方便。

执行下面命令：

>mongod  
>cd weiboSA
>scrapy crawl mblogSpider/dbSpider

可选参数：
 > scrapy crawl mblogSpider  -a num=     -a new_url= 
 
 - num 代表爬取页面数，默认为100页。
 - new_url 默认为搜索移动端‘上海租房’返回的json文件url，如果要添加其他上海租房信息，比如浦东租房，请自行在Chrome中找到请求的json地址，例如：
 - 
http://m.weibo.cn/page/pageJson?     
containerid=&containerid=100103type%3D1%26q%3D浦东租房  
&type=all   
&queryVal=浦东租房   
&luicode=10000011   
&lfid=100103type%3D%26q%3D上海无中介租房   
&title=浦东租房   
&v_p=11   
&ext=  
&fid=100103type%3D1%26q%3D浦东租房  
&uicode=10000011  
&next_cursor=  
&page=  
如果要数据库收录‘浦东租房’历史记录信息，请将[pipelines.py](https://github.com/luzhijun/weiboSA/blob/master/weiboZ/pipelines.py)第87、88行注释掉。一般如果有‘上海租房’了就不要去搜索‘浦东租房’，因为基本上有‘浦东租房’的微博都会有@‘上海租房’，所以下面会出现插入重复记录错误。

```
➜  weiboZ git:(master) ✗ scrapy crawl mblogSpider -a num=10 -a new_url="http://m.weibo.cn/page/pageJson\?containerid\=\&containerid\=100103type%3D1%26q%3D%E6%B5%A6%E4%B8%9C%E7%A7%9F%E6%88%BF\&type\=all\&queryVal\=%E6%B5%A6%E4%B8%9C%E7%A7%9F%E6%88%BF\&luicode\=10000011\&lfid\=100103type%3D%26q%3D%E4%B8%8A%E6%B5%B7%E6%97%A0%E4%B8%AD%E4%BB%8B%E7%A7%9F%E6%88%BF\&title\=%E6%B5%A6%E4%B8%9C%E7%A7%9F%E6%88%BF\&v_p\=11\&ext\=\&fid\=100103type%3D1%26q%3D%E6%B5%A6%E4%B8%9C%E7%A7%9F%E6%88%BF\&uicode\=10000011\&next_cursor\=\&page\="
2016-10-29 14:41:11 [root] WARNING: 生成MongoPipeline对象
2016-10-29 14:41:11 [root] WARNING: 开始spider
2016-10-29 14:41:11 [root] WARNING: 允许插入数据的时间大于2016-10-29 14:15:05.875000
2016-10-29 14:41:13 [root] WARNING: do page1.
2016-10-29 14:41:13 [root] WARNING: do other pages.
2016-10-29 14:41:13 [root] ERROR: 编号为:E91f233Ds的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:Ef4ri5bC6的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:Ef3UNqMmV的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:Ef3stkA8a的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:Ef3pzmJ6i的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:Ef1OBtvQr的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:Ef03Lj54z的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:EeYLU2GQd的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:EeYlBv7bn的数据插入异常
2016-10-29 14:41:13 [root] ERROR: 编号为:EeXkop2vu的数据插入异常
2016-10-29 14:41:15 [root] WARNING: 结束spider
```
 
 更改日志显示级别请在[setting.py](https://github.com/luzhijun/weiboSA/blob/master/weiboZ/settings.py)中修改LOG_LEVEL，介意采用项目默认的WARNNING，否则信息会很多。
 
## 查询示例

查询当前时区的2016-10-20至今有在9号线附近租房房租不高于2000的信息。

```javascript
db.house.find(
{
	created_at:{$gt:new Date('2016-10-20T00:00:00')},
	$or:
		[
			{price:{$lte:2000}},
			{price:[]}
		],
	admin:'9号线',
	tag:true
},
{	
	_id:0,
	text:1,
	created_at:1,
	scheme:1
}
).hint('created_at_-1').pretty()

{
	"text" : "房子在大上海国际花园，漕宝路1555弄，距9号线合川路地铁站步行5分钟，距徐家汇站只有4站，现在转租大床，有独立卫生间，公共厨房，房租2400，平摊下来1200，有一女室友，室友宜家上班，限女生，没有物业费，包网络，水电自理@上海租房无中介 @上海租房无中介 @上海租房 @上海租房无中介联盟",
	"scheme" : "http://m.weibo.cn/1641537045/EetVm3WBV?",
	"created_at" : ISODate("2016-10-25T09:18:00Z")
}
{
	"text" : "#上海租房##上海出租#9号线松江泗泾地铁站金地自在城，12层，步行、公交或小区班车直达地铁站。精装，品牌家具家电，主卧1800RMB/月；公寓门禁出入，房东直租，电话：13816835869，或QQ：36804408。@上海租房 @互助租房 @房天下上海租房 @上海租房无中介   @应届毕业生上海租房",
	"scheme" : "http://m.weibo.cn/1641537045/Een8cAoy8?",
	"created_at" : ISODate("2016-10-24T16:00:00Z")
}
{
	"text" : "#上海租房# 个人离开上海：转租地铁9号线朝南主卧带大阳台，离地铁站两分钟！设备齐全，交通方便，随时入住。具体信息看图片～@上海租房 @上海租房无中介联盟 @魔都租房 帮转谢谢！",
	"scheme" : "http://m.weibo.cn/1641537045/EdRpfuKuH?",
	"created_at" : ISODate("2016-10-21T07:14:00Z")
}
{
	"text" : "9号线桂林路 离地铁站8分钟 招女生室友哦 @上海租房 @上海租房无中介联盟 上海·南京西路",
	"scheme" : "http://m.weibo.cn/1641537045/EdJ2U8Kv3?",
	"created_at" : ISODate("2016-10-20T09:57:00Z")
}
```

## 更新
* v1.1
	* 新增豆瓣小组爬取与存储(以标题MD5为主键防止重复)
	* 新增agent随机切换
	* 新增随机静态ip代理（爬了近万个代理ip具有时效性，找几个可行的ip放进setting中）
	* 下载延迟设定为1，延迟越小越容易403错误（被豆瓣识别为爬虫）

## QA
* Q1:怎么单独查找豆瓣的数据。  
	A1:豆瓣数据相比微博多了个title字段，只要查找title不为空的即是豆瓣数据。例如：  
	```db.house.find({title:{$ne:null}}).count()
	```
* Q2: 想要爬取历史数据，但默认数据库只增加最新数据。   
	A2: 数据库有两重机制防止重复，第一个就是根据时间，同一地址的爬虫不会爬取老的数据；第二个是主键约束，比如		豆瓣上相同作者和标题的帖子会被认为重复。要想爬取历史数据需要把[pipliens.py](https://github.com/luzhijun/weiboSA/blob/master/weiboZ/pipelines.py)中的process_item两行时间约束代码注释。
*  Q3: 代理地址不可用。  
	A3: 为防止BAN，提供静态代理地址，但有些地址具有时效性，推荐去找最新的代理地址，或者扩展为动态代理地址。随机代理开启请把下面注释去掉
	
	```python
	# 代理，默认关闭
    #'weiboZ.middlewares.DBDownloaderMiddleware.ProxyMiddleware': 543
	```

* Q4: 为什么抓了豆瓣的数据数据库中不存储。  
	A4: 确认setting设置`ITEM_PIPELINES`为`weiboZ.pipelines.dbMongoPipeline`,并且数据库中豆瓣数据不是最新的。如果还有问题，日志级别设置为INFO看下爬虫统计结果，如下：
	
	```json
 'downloader/request_bytes': 119366,
 'downloader/request_count': 100,
 'downloader/request_method_count/GET': 100,
 'downloader/response_bytes': 481768,
 'downloader/response_count': 100,
 'downloader/response_status_count/200': 100,
 'finish_reason': 'finished',
	```
	以上是完全抓取100页豆瓣的统计结果，若response_status_count/403，请设置`DOWNLOAD_DELAY `大于1，或者换ip代理。