---
title: 处理JSON大文件的几种方案
toc: true
date: 2017-04-18 18:00:23
category: 
    - 技术贴
    - 解决方案
    - 海量数据处理
tags: 
    - json

---

# 海量数据处理


## JSON文件
工作中遇到一次这样的场景，业务上和芒果tv合作，需要进行媒资数据全量同步，对方给出的是一个zip压缩包，解压后是一个巨大的.json文件，打开一看里面是分层的媒资数据，格式如下（1.不是真正数据 2.仔细观察，接下来的方案都以这个例子做demo）
<!--more-->

```
{
    "assetList":[
        {
            "vodId":"290525",
            "vodTypeId":"1",
            "vodTypeName":"综艺",
            "name":"我是歌手 第四季",
            "englishName":"I am singer",
            "otherName":"歌手4，我是歌手，我们的歌手4",
            "director":"李克勤",
            "adaptor":"",
            "leader":"李克勤|容祖儿|张靓颖|徐佳莹|李健|关喆|苏见信|沈梦辰",
            "kind":"音乐|其他",
            "area":"内地",
            "language":"国语",
            "caption":"中文简体",
            "story":"《我是歌手》是中国首档顶级歌手音乐巅峰对决节目，前三季都带来了激烈的反响。节目由洪涛团队打造，每期邀请7位已经成名的歌手之间进行竞赛。",
            "awards":"中国首档顶级歌手音乐巅峰对决节目",
            "year":"2016",
            "duration":"110888",
            "releaseTime":"2016-01-15 00:00:00",
            "productionTime":"2016-01-08 00:00:00.000",
            "workRoom":"",
            "inital":"W",
            "fullSpell":"WoShiGeShouDiSiJi",
            "simpleSpell":"wsgsdsj",
            "serialCount":"68",
            "totalNumber":"68",
            "producerName":"湖南卫视",
            "television":"湖南卫视",
            "scores":"6.5",
            "status":"1",
            "img_v":"http://0img.mgtv.com/preview/internettv/sp_images/ott/2016/zongyi/290525/20160615181020138-new.jpg",
            "img_h":"http://0img.mgtv.com/preview/internettv/sp_images/ott/2016/zongyi/290525/20160615181023298-new.jpg",
            "img_s":"http://1img.mgtv.com/preview/internettv/sp_images/ott/2016/zongyi/290525/20160615181026765-new.jpg",
            "vipMarkOtt":"",
            "createTime":"2015-12-31 18:01:30",
            "updateTime":"2017-04-14 11:40:25",
            "parts":[
                {
                    "partId":"2959574",
                    "vodId":"290525",
                    "name":"我是歌手第四季20160122期：黄致列首唱中文 李玟颠覆演绎",
                    "otherName":"",
                    "director":"李克勤",
                    "adaptor":"",
                    "leader":"李玟|李克勤|黄致列|信|关喆|HAYA乐团|徐佳莹|赵传",
                    "kind":"音乐",
                    "area":"内地",
                    "language":"",
                    "caption":"",
                    "story":"",
                    "awards":"黄致列首唱中文金曲",
                    "year":"2016",
                    "duration":"7020",
                    "releaseTime":"2016-01-22 00:00:00",
                    "inital":"G",
                    "simpleSpell":"gs4",
                    "producerName":"",
                    "isIntact":"1",
                    "serialNo":"2",
                    "status":"1",
                    "img_v":"",
                    "img_h":"http://1img.mgtv.com/preview/sp_images/2016/zongyi/290525/2959574/20160123003222296.jpg",
                    "vipMarkOtt":"",
                    "createTime":"2016-01-22 00:00:00",
                    "updateTime":"2016-01-25 11:02:44",
                    "files":[
                        {
                            "definition":"1",
                            "definitionName":"标清"
                        },
                        {
                            "definition":"2",
                            "definitionName":"高清"
                        }
                    ]
                },
                {
                    "partId":"2967573",
                    "vodId":"290525",
                    "name":"我是歌手第四季20160129期：苏运莹玩滑板唱《野子》 野性发声单挑六强",
                    "otherName":"",
                    "director":"李克勤",
                    "adaptor":"",
                    "leader":"李玟|李克勤|黄致列|信|关喆|HAYA乐团|徐佳莹|赵传|苏运莹",
                    "kind":"音乐",
                    "area":"内地",
                    "language":"",
                    "caption":"",
                    "story":"",
                    "awards":"苏运莹玩滑板唱《野子》",
                    "year":"2016",
                    "duration":"77340",
                    "releaseTime":"2016-01-29 00:00:00",
                    "inital":"W",
                    "simpleSpell":"wsgs20160129",
                    "producerName":"",
                    "isIntact":"1",
                    "serialNo":"3",
                    "status":"1",
                    "img_v":"",
                    "img_h":"http://1img.mgtv.com/preview/sp_images/2016/zongyi/290525/2967573/20160129222109197.jpg",
                    "vipMarkOtt":"",
                    "createTime":"2016-01-29 00:00:00",
                    "updateTime":"2016-02-01 22:59:58",
                    "files":[
                        {
                            "definition":"1",
                            "definitionName":"标清"
                        },
                        {
                            "definition":"2",
                            "definitionName":"高清"
                        }
                    ]
                }
             ]
         }
    ]
}
```

选取了极小部分数据，整体文件有几百M,如果一次性加载到内存中用工具去转换成bean，可想而知。因此经过一系列分析，讨论，想到了几个办法。不过这些办法都必须遵循一个原则，用流去逐行读取，并进行解析，不一次将整个大JSON都load到内存中，接下来就详细介绍几种方案。

### 符号匹配
在网上查了一会，无果，然后我试着自己先思考，想出了这个方案，方案如下：
由于JSON中的符号都是两两匹配的，因此可以根据这个特性来进行切割，学过数据结构的童鞋应该能想到，栈这种数据结构实现两两匹配这种场景是非常给力的，大概的原理就是将读到的相同符号全部压到栈中，比如{} [],并设置计数器，左边符号{ [ 就加一，读到右边符号就减一，当计数器为0时说明是一个完整的对象，就可以对其处理了。

### fast-json
先看一下使用
```
public void ReadWithFastJson()  {  
	    String jsonString = "{\"array\":[1,2,3],\"arraylist\":[{\"a\":\"b\",\"c\":\"d\",\"e\":\"f\"},{\"a\":\"b\",\"c\":\"d\",\"e\":\"f\"},{\"a\":\"b\",\"c\":\"d\",\"e\":\"f\"}],\"object\":{\"a\":\"b\",\"c\":\"d\",\"e\":\"f\"},\"string\":\"HelloWorld\"}";  
	  
	    // 如果json数据以形式保存在文件中，用FileReader进行流读取！！  
	    // path为json数据文件路径！！  
	    // JSONReader reader = new JSONReader(new FileReader(path));  
	  
	    // 为了直观，方便运行，就用StringReader做示例！  
	    JSONReader reader = new JSONReader(new StringReader(jsonString));  
	    reader.startObject();  
	    System.out.println("start fastjson");  
	    while (reader.hasNext())  
	    {  
	        String key = reader.readString();  
	        System.out.println("key " + key);  
	        if (key.equals("array"))  
	        {  
	            reader.startArray();  
	            System.out.println("start " + key);  
	            while (reader.hasNext())  
	            {  
	                String item = reader.readString();  
	                System.out.println(item);  
	            }  
	            reader.endArray();  
	            System.out.println("end " + key);  
	        }  
	        else if (key.equals("arraylist"))  
	        {  
	            reader.startArray();  
	            System.out.println("start " + key);  
	            while (reader.hasNext())  
	            {  
	                reader.startObject();  
	                System.out.println("start arraylist item");  
	                while (reader.hasNext())  
	                {  
	                    String arrayListItemKey = reader.readString();  
	                    String arrayListItemValue = reader.readObject().toString();  
	                    System.out.println("key " + arrayListItemKey);  
	                    System.out.println("value " + arrayListItemValue);  
	                }  
	                reader.endObject();  
	                System.out.println("end arraylist item");  
	            }  
	            reader.endArray();  
	            System.out.println("end " + key);  
	        }  
	        else if (key.equals("object"))  
	        {  
	            reader.startObject();  
	            System.out.println("start object item");  
	            while (reader.hasNext())  
	            {  
	                String objectKey = reader.readString();  
	                String objectValue = reader.readObject().toString();  
	                System.out.println("key " + objectKey);  
	                System.out.println("value " + objectValue);  
	            }  
	            reader.endObject();  
	            System.out.println("end object item");  
	        }  
	        else if (key.equals("string"))  
	        {  
	            System.out.println("start string");  
	            String value = reader.readObject().toString();  
	            System.out.println("value " + value);  
	            System.out.println("end string");  
	        }  
	    }  
	    reader.endObject();  
	    System.out.println("start fastjson");  
	} 
```

仔细分析代码会发现，和第一种博主想到的方案所不同的是，使用的是json的key比较，也就是说fast-json使用者首先要对需要分析的大JSON文件进行了解，知晓其结构，通过一层层的key来判断该处理哪一层级下的数据，并通过
startXXX,endXXX方法来判断是否该层解析结束，跳出hasNext。但是具体是怎么实现的呢，来看看源码。

首先有一个很重要的类 JSONStreamContext
```
class JSONStreamContext {

    final static int                  StartObject   = 1001;
    final static int                  PropertyKey   = 1002;
    final static int                  PropertyValue = 1003;
    final static int                  StartArray    = 1004;
    final static int                  ArrayValue    = 1005;

    protected final JSONStreamContext parent;

    protected int                     state;

    public JSONStreamContext(JSONStreamContext parent, int state){
        this.parent = parent;
        this.state = state;
    }
}
```

由于json中主要的存储类型就是array和object两种，因此fast-json将json规范数值化了，这在后面等值比较会使用到，另外一点就是层次化，上层和下层就是parent与children的关系，因此通过JSONStreamContext类持有自己类的实例去将这种关系串联了起来，这有点像责任链。
继续分析

```

```


### gson

