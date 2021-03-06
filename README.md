# Bing-upyun
# 轻量必应每日一图API，支持上传到又拍云，并提供调用接口，程序本体<3Kb。

### 1.简介

##### 1.1 之前用的接口

在这之前，鄙人用的必应背景图片接口的原理是：解析官方接口再返回图片URL。虽然代码简单，但由于微软服务器问题，从请求到图片接收完成，耗时大都会超过500ms，有时甚至会请求失败。且原来的这种方法不支持图片处理等功能。

##### 1.2 其他接口

目前网络上有很多优秀的必应每日一图接口，但大都需要常驻后台运行，且附带图片展示功能（这已经脱离了图片接口的本职工作）。另外，目前几乎没有采用又拍云储存图片的同类接口。因此便有了下面的项目。

##### 1.3 本接口

本程序是轻量化的php程序，目前程序本体<3Kb。本程序可以将每日最新的必应背景图片上传到又拍云，并提供调用接口（又拍云直链）。实测从请求到图片接收完成耗时300ms左右（视网络情况而不同）。

为了追求轻量化，本程序不会常驻后台，这也就意味着用户需要每日访问一次URL，以触发程序执行。建议在服务器上设置定时任务，每日 **00:05** 访问 `/php/index.php`。强烈建议定时任务在此时间访问，如需设置其他时间，请务必修改 `/index.php`中的`$delay`为合适的值。详情见 6.3 设置定时任务。

目前本接口只能返回当天图片。后续会更新接口，以支持参数，返回处理后的图片（高斯模糊、水印等等）以及前n天的图片。

### 2.特点

1. 每天从必应搜索首页拉取最新背景图。
2. 支持将背景图上传到又拍云，不占用服务器存储空间。
3. 提供从又拍云调用当天图片的接口。
4. 不占用服务器后台，程序通过访问URL触发。

### 3.DEMO

https://abc.mcloc.cn/abc/bing/index.php

### 4.目录结构

```
    ├── php
    │   ├── bing		// 图片缓存文件夹
    │   └── index.php	// 后台图片处理程序
    └── index.php		// 图片调用接口
```

### 5.接口文档

直接访问调用URL即可。

目前不支持参数，已经在开发中，将会很快支持。

### 6.部署方式

##### 6.1 修改又拍云连接信息

请在 `/index.php` 中修改：

```
$imgurl = 'https://***.***.***/' . $image_name; //此处填又拍云加速域名。注：结尾的 / 不能省略
```

在`/php/index.php`中修改：

```php
//又拍云连接信息
$bucketName   = '********'; //你的又拍云存储库
$operatorName = '********'; //你的存储库操作员
$operatorPwd  = '********'; //你的存储库操作员密码
```

##### 6.2部署文件到服务器

放在可访问目录即可。

注意：`/php/bing`文件夹需要有写入权限。

##### 6.3 设置定时任务（重要）

本程序不会常驻后台，需要定时访问程序所在URL以触发程序执行（每天访问一次）。

定时任务访问URL： `网站根目录/php/index.php` 或 `网站根目录/php`

接口调用URL： `网站根目录/index.php` 或 `网站根目录/`

为避免时间误差引起的问题，建议**不要**将定时任务设置在每天的 00:00 ，推荐将定时任务设置在每日 **00:05** ，若如此做，则在完成以上步骤后，不需要额外设置。否则请参照以下规则：

 `/index.php` 中的 `$delay` 为延时时间，如 `$delay = 310;` 即调用310s前的图片。这也就意味着，您在每天零点的310s后，才能收到当天最新的图片。在每天的 00:00:00 至 00:05:10 之间，您调用此接口返回的仍然是前一天的图片。

注意：此延时时间需比定时任务中访问URL的时间大10s左右（和网络情况有关），否则会长时间返回前一天的图片（太大）或出现404错误（太小）。