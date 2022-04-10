---
title: 今日校园模拟登录
date: 2021-08-02
category: project
tag: toy
---

**目标**：今日校园获取签到任务。此地无：~~自动签到~~

**实现路线**：

1. 模拟登录
2. 获取签到信息
3. ~~自动签到~~

### 模拟登录

**方式一**：账号密码登录

> http://id.fzu.edu.cn/authserver/login?service=https%3A%2F%2Ffzu.campusphere.net%2Fportal%2Flogin

**方式二**：扫码登录

> http://id.fzu.edu.cn/authserver/login?display=qrLogin&service=https%3A%2F%2Ffzu.campusphere.net%2Fportal%2Flogin

* 请求QRcode-uuid:

  > http://id.fzu.edu.cn/authserver/qrCode/get?ts=1627191854081    --> QR-oWtUtosFhn7OTvCYQR2

* 请求QRcode图片：

  > http://id.fzu.edu.cn/authserver/qrCode/code?uuid=QR-oWtUtosFhn7OTvCYQR2

* 扫码状态查询：

  http://id.fzu.edu.cn/authserver/qrCode/status?ts=1627191855083&uuid=QR-oWtUtosFhn7OTvCYQR2

  * 0 未扫
  * 1 登录成功
  * 2 已扫描
  * 3 二维码失效/过期

* 登录：当扫码状态为2时POST请求

  > http://id.fzu.edu.cn/authserver/login?display=qrLogin&service=https%3A%2F%2Ffzu.campusphere.net%2Fportal%2Flogin

  获取302跳转响应头中的Location地址，请求得到重要cookie：**MOD_AUTH_CAS**

  >  https://fzu.campusphere.net/portal/login?ticket=ST-2685077-cAT3xOG21d1HXgnAMPpY1627192330448-DaeF-cas

  2021/08/01更新：系统增加了多重跳转。

### 签到任务

#### 1. 获取签到任务

> https://fzu.campusphere.net/wec-counselor-sign-apps/stu/sign/getStuSignInfosInOneDay

响应内容：

``` json
{"code":"0",
"datas":{
     "codeRcvdTasks":[],
     "unSignedTasks":[],
     "leaveTasks":[],
     "dayInMonth":"2021-08-02",
     "signedTasks":[{"rateSignDate":"2021-08-02 (周一)","rateTaskBeginTime":"06:00","isLeave":null,"signStatus":"1","leaveMobileUrl":null,"signWid":"1017xxx","senderUserName":"xxx与xxx学院(xxx)","isMalposition":"0","signRate":"1","currentTime":"2021-08-02 10:03","taskType":"1","singleTaskEndTime":null,"singleTaskBeginTime":null,"signInstanceWid":"162xxx","rateTaskEndTime":"17:00","taskName":"健康监测—xxx---xxx","stuSignWid":"26344xxx","leavePcUrl":null}]},
"message":"SUCCESS"
}
```

关键参数：

* signWid
* signInstanceWid

#### 2. 获取任务详情

> https://fzu.campusphere.net/wec-counselor-sign-apps/stu/sign/detailSignInstance

请求参数：上一步取得的signWid、signInstanceWid

响应内容：

``` json
{
    "code": "0",
    "datas": {
        "signAddress": "xx",
        "rateTaskBeginTime": "06:00",
        "signStatus": "1",
        "latitude": "xx",
        "downloadUrl": "https://img.cpdaily.com/ldy/index.html",
        "signMode": 0,
        "signPlaceSelected": [],
        "signPhotoUrl": "",
        "taskType": "1",
        "changeActorName": "xx",
        "rateTaskEndTime": "17:00",
        "signType": 1,
        "extraFieldItemVos": [],
        "longitude": "xx",
        "rateSignDate": "2021-08-02 (周一)",
        "signTime": "2021-08-02 xx",
        "extraField": [],
        "leaveAppUrl": "/wec-counselor-leave-apps/leave/home/index.html",
        "isNeedExtra": 1,
        "isPhoto": 0,
        "isAllowUpdate": true,
        "senderUserName": "xx",
        "isMalposition": 0,
        "signedStuInfo": {},
        "signRate": "1",
        "taskDesc": "https://wecres.cpdaily.com/counselor/1018615892395240/content/8e23766b22d54953b14b7c9dc4ce70f7.html",
        "currentTime": "2021-08-02 10:xx",
        "changeTime": "2021-08-02 10:xx",
        "singleTaskEndTime": null,
        "photograph": [],
        "leftNum": null,
        "singleTaskBeginTime": null,
        "updateLimit": null,
        "signInstanceWid": "162400",
        "catQrUrl": "https://cat.cpdaily.com/erweima",
        "signCondition": 0,
        "taskName": "xx",
        "qrCodeRcvdUsers": []
    },
    "message": "SUCCESS"
}
```

 

#### ~~3. 执行签到任务~~

==【WARNING】该项内容不在本项目范围内，以下内容仅供学习参考，其内容来源于其他github项目==

**配置签到信息**

``` json
{
  "abnormalReason" : "",
  "position" : "xxx",
  "longitude" : xx,
  "isNeedExtra" : 1,
  "signVersion" : "1.0.0",
  "latitude" : xx,
  "isMalposition" : 0,
  "extraFieldItems" : [
{
   "extraFieldItemWid" : xx,
   "extraFieldItemValue" : "小于37.3度"
},
{
   "extraFieldItemWid" : xx,
   "extraFieldItemValue" : "否"
}
  ],
  "signPhotoUrl" : "",
  "uaIsCpadaily" : true,
  "signInstanceWid" : "154294"
}
```

**发送签到请求**

关键请求头参数：**Cpdaily-Extension**

来源：DES加密学号、位置、版本号信息

``` python
extension = {
    "lon": 'xx',
    "model": "PCRT00",
    "appVersion": "8.0.8",
    "systemVersion": "4.4.4",
    "userId": user['username'],
    "systemName": "android",
    "lat": 'xx',
    "deviceId": uuid.uuid1()
}
 
CpdailyInfo = DESEncrypt(json.dumps(extension))
```

其中DESEncrypt函数在currency/encrypt.py。密钥要改成b3L26XNL。

==以上内容参考==：https://github.com/ZimoLoveShuang/auto-submit/issues/33#issuecomment-756525862

### 项目结构

* Java后端：Springboot提供接口服务
  * /login    学号+密码登录
  * /qrLoginCode 获取二维码
  * /qrLoginStatus 扫码状态
  * /qrLogin 扫码登录
  * /signInfos 获取签到任务
  * /signDetails/{signWid}/{signInstanceWid} 查看签到任务详情
* Web前端：Vue框架高仿今日校园APP
  * / 首页，签到任务列表
  * /login 扫码登录页，微信或今日校园APP扫码登录
  * /signDetails 任务详情页，查看签到状态

### 页面展示

<img src="https://cdn.jsdelivr.net/gh/juaran/juaran.github.io@image/typora/b96812be8bec471687ed90bbd8a2532.jpg" alt="b96812be8bec471687ed90bbd8a2532" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/juaran/juaran.github.io@image/typora/28343d83580cd81b73a0e2d0114dc5d.jpg" alt="28343d83580cd81b73a0e2d0114dc5d" style="zoom: 25%;" />

<img src="https://cdn.jsdelivr.net/gh/juaran/juaran.github.io@image/typora/image-20210802115323960.png" alt="image-20210802115323960" style="zoom: 33%;" />

### 结术语

陆陆续续花了一周时间完成了项目雏形，主要目的是学习和巩固Springboot框架、Java网络请求、Vue前端页面设计、路由跳转等基础知识，并参考了github上相关项目。最初的愿景是想结合前后端实现全自动签到，因为其他同类项目均以Python脚本方式运行，辅以Linux-Crontab定时任务或腾讯云函数，需要手动配置相关参数，希望有缘人能够继续下一步的开发。

鉴于目前疫情日益严峻，以安卓虚拟定位派为代表的同学已“锒铛入狱”，而Python脚本派的同学依旧能够在每早6:00时刻准时登上排行榜，但殊途同归，技术的目的不是为了投机取巧、瞒报疫情、隐匿行踪，希望每一位同学认真对待日签，不要抱有侥幸心理！

项目地址：https://gitee.com/juaran/cpdaily-qd