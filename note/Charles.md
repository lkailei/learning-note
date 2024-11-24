# Charles

## 下载&破解

下载：https://www.charlesproxy.com/

破解：https://www.zzzmode.com/mytools/charles/

https://zhuanlan.zhihu.com/p/571533631

https://www.cnblogs.com/tester-ggf/p/13849716.html

https://www.52pojie.cn/thread-1600964-1-1.html

提交挂号接口

https://m.hsyuntai.com/med/hp/hospitals/100578/registration/submit230

```

{
	"patId": 30550167,
	"patName": "芦煜芮",
	"pcId": 26962594,
	"schId": 955756205929,  
	"isSendRegSms": "Y",
	"sTime": "09:30",
	"eTime": "10:00",
	"type": "0",
	"regSelectItems": {
		"ChargeType": "自费"
	},
	"vcode": "",
	"docId": "50913244",
	"hosDistId": "236",
	"deptId": "1318754",
	"districtId": "236",
	"patCardType": 6,
	"patCardName": "就诊ID号",
	"deptName": "皮肤科门诊",
	"isHealthCard": false,
	"timeout": 30000,
	"isLoading": true
}
```

schId 获取 https://m.hsyuntai.com/med/hp/hospitals/100578/registration/doctorDetails225?branchId=236&date=2023-10-27&docId=290191290&filtrate=N&isShowTime=false&outpatientId=1318754&schListType=true&type=0

```
[{
	"data": [{
		"deptId": 1318754,
		"hosDistId": 236,
		"hosDistName": "本部（原空军总医院）",
		"name": "皮肤科门诊"
	}],
	"nowTime": "2023-10-20 15:08:04",
	"result": true,
	"api_name": "hsytmed/r/100578/60050/402"
}, {
	"data": {
		"now": "2023-10-20 15:03:55",
		"count": 7
	},
	"nowTime": "2023-10-20 15:03:55",
	"result": true,
	"api_name": "hsytmed/r/100578/60050/505"
}, {
	"result": true,
	"data": [{
		"schId": 955756205778,
		"accessSchId": "20231027|甲病门诊|上午",
		"hosDistId": 236,
		"hosDistName": "本部（原空军总医院）",
		"distAddr": "北京市海淀区阜成路30号(钓鱼台)",
		"deptId": 1318754,
		"deptName": "皮肤科门诊",
		"docId": 290191290,
		"resNo": 1,
		"schDate": "10-27",
		"weekType": "星期五",
		"startTime": "08:00",
		"endTime": "12:00",
		"dayType": 1,
		"cost": 50,
		"remainNo": 15,
		"state": 1,
		"numSrcType": 3,
		"stateShown": "有号",
		"regType": 0,
		"needChooseFvSv": false
	}],
	"nowTime": "2023-10-20 15:08:03",
	"traceId": "ad8c7c9cf39342cda7af42d9e0d2ab63",
	"api_name": "hsytmed/r/100578/60060/112"
}]
```

1. 获取医生的信息接口

   https://m.hsyuntai.com/med/hp/hospitals/100578/hos/registration/doctorDetails/0/236/1318754/50913345?date=2023-10-30

## 问题

1. 同盾滑块验证码破解
2. submit信息处理

