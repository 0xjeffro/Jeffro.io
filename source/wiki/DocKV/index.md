---
layout: wiki
wiki: DocKV
title: 快速开始
---
在开发数据监控工具时，通常需要缓存上一次的数据采集状态。这样的场景一多，维护这些数据库就会带来较高的心智成本，因此我开发了DocKV。

你只需要在Google Doc新建一个电子表格并配置好读写权限，即可把该表格变成一个存储介质。DocKV会在该表格上维护一个键值数据库。
## 准备工作
1. 新建Google Service Account, 文档在此 👉 [Document of Service Account](https://developers.google.com/identity/protocols/oauth2/service-account#creatinganaccount) . 创建完服务账号之后，记录`Email`字段，后面会用到（图1）。
{% image /assets/wiki/DocKV/1.png 图1. Google Service Account width:650px padding:15px bg:white download:true %}
2. 点击图1中表格右侧的 {% mark Actions color:cyan %} -> {% mark Manage Keys color:cyan %} -> {% mark ADD KEY color:cyan %} -> {% mark Create New Key color:cyan %}, 此时会弹出一个对话框，这里Key type选择默认的 JSON，点击 {% mark CREATE color:cyan %}，这时会自动格下载一个 `JSON文件`，要保存好，后面会用到。
{% image /assets/wiki/DocKV/2.png 图1. Google Service Account width:650px padding:15px bg:white download:true %}
2. 进入 [Google Doc](https://docs.google.com/spreadsheets) , 新建一个电子表格。在表格的URL中找到 `SHEET_ID` 记录下来。
3. 点击电子表格右上角的 {% mark 共享 color:cyan %}，把这个表格的读写权限共享给第一步得到的 `Email` 对应的 Service Account。

## 安装
``` GO
go get -u github.com/0xjeffro/DocKV
```

## 例子

### 定义数据模式
``` GO
import (  
   DocKV "github.com/0xjeffro/DocKV"  
   "os"
)   

type Model struct {  
   Name   string `json:"name"`  
   Age    int    `json:"age"`  
   Gender bool   `json:"gender"`  
}

```

### 连接
```GO
func connect() *DocKV.DocKV {  
   clientSecret := os.Getenv("CLIENT_SECRET")  
   sheetID := os.Getenv("SHEET_ID")  
   return DocKV.NewDocKV(sheetID, Model{}, []byte(clientSecret), 60)  
}
```

其中，`CLIENT_SECRET` 对应的是“准备工作”章节中拿到的`JSON文件`中的字符串；`SHEET_ID` 同理。
代码中的`60`是缓存时间，单位为秒。由于Google API有限速机制，因此DocKV实现了一个简单的缓存，尽量避免触发限制。


## 注意事项
Google Doc API对于接口调用的频率有明确的限制，具体规则可见：[Usage limits](https://developers.google.com/docs/api/limits) 。因此，DocKV只适用于很小的项目使用，在业务逻辑中要注意缓存时间的把握以及数据规模的适当维护。尤其不适用于以下场景：
- 高频地击穿查询：频繁查询缓存中可能不存在的键，导致大量请求直接落到API的调用上，触发请求频率限制。
- 高频写入：频繁地插入新的键，触发请求频率限制。