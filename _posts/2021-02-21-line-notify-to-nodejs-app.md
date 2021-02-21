---
title: มาสร้างการแจ้งเตือนไปที่ Line Notify กัน
subtitle: การสร้างการแจ้งเตือนไปยัง Line ผ่านแอพพลิเคขั่นที่เขียนด้วย Node.js
layout: post-it-blog
author: Theethawat Savastham
category: it-blog
created_year: 2021
comments: true
language: Javascript
tag:
  - api
  - line
  - line-notify
---

Line Notify จะเป็นอะไร ที่ค่อนข้างจะแตกต่างกับ Service ตัวอื่น ๆ ของ line developer เพราะ line notify อาจจะไม่ได้อยู่ใน line developer platform ดังนั้น จึงไม่จำเป็นต้องมี Developer Account ก็สามารถทำได้ โดยเข้าไปที่เว็บไซต์ [https://notify-bot.line.me](https://notify-bot.line.me) แล้ว Login ด้วยบัญชี Line ของเรา

## รู้จักกับ Line Notify Service

เมื่อเข้าไปที่ https://notify-bot.line.me แล้ว เราจะเห็น

![หน้าเว็บของ Line Notify](/assets/line-notify/1.png)
**_หน้าเว็บหลังจากเราเข้าไปที่ Line Notify_**

มันจะมีหน้า My Page กับ Manage registered services สำหรับส่วนของ My Page จะเป็นสเตตัสว่า Line Notify Bot แต่ละตัว ทั้งที่เป็นของเราและไม่เป็นของเรา แต่ที่เรารับบริการอยู่ เช่น ในเช่นมี E-Online มี PostAlert ของทางไปรษณีย์ ส่วนหน้า Manage Registered Services นี่แหละ คือของจริง คือสิ่งที่เราจะสร้าง Notify Service ของเราขึ้นมา

![หน้า Manage Registered Service](/assets/line-notify/2.png)

## การสร้าง Service บน Line Notify

ทำการเลือก Add Service แล้วกรอกข้อมูลต่าง ๆ ใส่ URL ของ Service ของเรา เช่น ถ้าของเราเป็น theduckcreator.in.th ก็ใส่ไป ตรงนี้ มันไม่ได้มีผลเชิงเทคนิคมาก เป็นเพียงการให้มันขึ้นไปให้คนใช้รู้ เป็นเรื่อง Policy เฉยๆ ส่วนของ Representative คือ ตัวแทน ซึ่งสามารถเขียนชื่อเราลงไปได้

![Register for a new Service](/assets/line-notify/3.png)
**_หน้าการสร้าง Service ใหม่_**

> ส่วนที่สำคัญที่สุด คือ CallbackURL

คือหลักการทำงานโดยทั่วไปกับ API ข้างนอก ไม่ว่าจะพวก Line พวก Facebook, Firebase หรื่อ อื่นๆ จะเป็นลักษณะเป็นการยิงข้อมูลและคำสั่ง ไปทำงานที่เซิฟเวอร์ของเขา แล้วเขาจะส่งผลลัพธ์กลับมาที่เซิฟเวอร์ของเรา ซึ่งเพื่อความปลอดภัย มันจะไม่ใช่ว่า ยิงกลับมาที่ End Point ตัวไหนก็ได้ของระบบเรา แต่เราจะต้องกำหนด White List หรือ Callback URL ที่เราจะอนุญาติให้ระบบของมัน ส่ง URL ที่เป็น URL ในกลุ่มนี้เข้ามาได้ ซึ่งแต่ละที่จะกำหนดแตกต่างกัน สำหรับ Line Notify จะให้ 5 ผู้ใช้

การทำงานกับ API ที่มีการยิงกลับ จำเป็นต้องทำงานบน Server เท่านั้น นั่นหมายความว่า React ที่เป็น Single Page App จะไม่สามารถทำงานคู่กับ API ได้ จำเป็นต้องทำที่ Backend แต่ถ้าเป็น Next.js, PHP, Laravel ซึ่งเป็นการผสมระหว่าง Backend และ Frontend อาจสามารถทำได้

## สิ่งที่จะได้มาหลังจากการลงทะเบียน

เราจะได้ Client ID กับ Client Secret มา โดยคล้าย ๆ กับเป็น Public and Private Key Client ID มีสถานะเป็นสิ่งปกติ ที่ไม่เป็นความลับ แต่ Secret จะเป็นความลับ

![Client ID and Client Secret](/assets/line-notify/4.png)
**_Client ID และ Client Secret ที่ได้มา_**

## การให้ผู้ใช้เข้าไปลงทะเบียนไว้กับ Line Notify

จะเป็นการทำประกอบกับ [Official Document](https://notify-bot.line.me/doc/en/) ของ Line Notify

สร้างลิงค์โดยอาจจะใช้ Button, A หรือ GET Method ธรรมดา ที่ลิงค์ไปยัง URL ของระบบ Line Notify โดยให้มันไปที่ [https://notify-bot.line.me/oauth/authorize](https://notify-bot.line.me/oauth/authorize) โดยเราจำเป็นที่จะต้่องใส่ Request Parameter ไปด้วย

![Request Parameter ที่ Line ต้องการ](/assets/line-notify/5.png)
**_Request Parameter ที่ Line ต้องการ_**

วิธีการใส่ Request Parameter ใน GET Method นั้น เราสามารถใส่ได้หลายแบบ ถ้าเราใช้วิธีแบบ

- ถ้าใชิวิธีแบบ กดปุ่มนี้ แล้วไปเรียก Action ให้ axios ไปทำงาน ตรงนี้เราอาจจะใช้ axios อาจจะใส่เป็น params ตามวิธีการของ [Axios](https://github.com/axios/axios)
- หรือ ถ้าเป็นง่าย ๆ Request param คือ หลังจาก URL แล้ว เราใส่ ?name=value ได้เลย และถ้ามีหลาย ๆ อันก็ขึ้นด้วยเครื่องหมาย & ตัวอย่างเช่น ต้องการเข้าไปที่เว็บไซต์ theduckcreator.in.th โดยมี `parameter` ว่า `clientid` เป็น 10 `connecttype` เป็น `travel` จะเขียนแบบนี้ `https://theduckcreator.in.th?clientid=10&connecttype=travel`

ในระบบนี้ของ Line Notify ก็เช่นกัน เราต้องใส่

- **response_type** ซึ่งมันบังคับว่าให้เราใส่ว่า code ใส่ client id

- **redirect_uri** (ซึ่งต้องเป็น 1 ใน 5 URL ที่เขาให้เรากำหนด)

- **state** ในที่นี้คือ **_เราจะใส่อะไรก็ได้_** ถามว่า ทำไมต้องเป็นแบบนี้ เพราะว่า state มันจะเป็นค่าที่ Line Notify เว้นไว้ให้เราว่า เราต้องการข้อมูลอะไรรึเปล่า ที่เราตกลงกันไว้เอง เช่น เรากำหนดว่า state คือ ไอดีของ User ในแอพ หรือ อะไรก็แล้วแต่ เพราะว่า ค่าตรงนี้ มันจะส่งกลับมาด้วย เมื่อเวลามันทำงานสำเร็จแล้วตีกลับมาตาม Redirect URL ซึ่งเราสามารถเขียนโปรแกรม เช็คได้ว่า ถ้าสเตทเป็นค่านี้ให้ทำยังไง หรือพูดง่าย ๆ ว่ามันคือตัวแปรว่างๆ ให้เราจะเอาไปทำอะไร เพื่อประโยชน์ของเรา ในการพัฒนาส่วนต่อไป

ตัวอย่างเช่น

```
const url = 'https://notify-bot.line.me/oauth/authorize?response_type=code&client_id=7ZmMC5mkpC0xxxW5x&redirect_uri=https://theduckcreator.in.th/notifyredirect&scope=notify&state=' + userid
```

## การเตรียมการรับการตอบกลับจาก Line Notify

เมื่อเราให้ User ของเรา ไป Login ที่ Line Notify แล้ว เมื่อทำรายการสำเร็จ Line Notify จะยิงกลับมาที่เว็บไซต์ของเรา นั่นหมายความว่า เราต้องเตรียมไว้รอรับด้วย โดย Line จะส่ง code กับ state มา **code** จะเป็น ข้อมูลชัวคราว อันนึง เหมือนเป็นรหัสของ Transaction การที่ user ไปขอรับการ notify อันหนึ่ง แล้วนำไปใช้ในขั้นตอนถัดไป **code ไม่ใช่ access** token เรายังต้องทำขั้นตอนต่อไปอีก เพื่อให้ได้ access token

อย่างที่บอกไป เราสามารถใช้ state ช่วยได้ เราอาจจะกำหนดว่า state คือ user id ของผู้ใช้ก็ได้ แล้วพอค่ากลับมา เราก็ไปอัพเดท Record ของ user id นี้ ในตาราง user ก็ได้

### การเตรียมรับการตอบกลับใน Express.js

ถ้าเขียน Server ด้วย Nodejs Express นั้น เราต้องเตรียม route สำหรับการตอบกลับของ line notify โดยระบบของ line จะตอบกลับมาโดยใช้ GET Method สมมติว่าเราจะใช้ตอบกลับมาที่ URL https://theduckcreator.in.th/api/notifyredirect เราก็ต้องสร้าง route ของ notifyredirect เอาไว้

<script src="https://gist.github.com/theethawat/dd54324552e84441ee8318b155d3ef01.js?file=NotifyAPIRecieve.js"></script>

แล้วก็ถอดค่า request_parameter ที่ระบบได้ส่งมาด้วยนั้น ออกมา เช่น ระบบส่ง `code` และ `state` เป็น parameters มาด้วย เราก็จะใช้ `req.query` ในการดึงออกมาว่ามันคือค่าอะไร

## การยืนยันกลับไปที่ Line Notify ว่าเราได้รับ Code จาก Line Notify แล้ว

ยังไม่จบ ขึ้นสุดท้ายของการลงทะเบียนรับผู้ใช้ใหม่ของ Line Notify หลังจากเราให้เขายิงไปที่ URL ของ Line Notify แล้วนั้น Line Notify จะยิง GET Request เอา Code มาบอกเรา สุดท้าย คือเราจะต้องยืนยันว่า โอเค เรารับคนนี้เข้าระบบเรานะ และเราคือเซิฟเวอร์ที่ลงทะเบียนไว้จริง ๆ เหมือนเป็นการเซ็นยืนยัน และการยืนยันเราจะยืนยันด้วยอะไรนั้น

ข้างต้น เราได้เขียนบอกแล้วว่าเราจะได้ client id กับ client secret ซึ่ง Client Secret จะเป็นข้อมูลความลับ เราใช้ Client ID เป็นส่วนหนึ่งของ Request Parameter ที่ให้ไปบอกกับ Line แล้ว การกระทำที่เราให้ User Redirect ไปหน้าของ Line เหมือนเป็นการให้เขาส่งจดหมายไปหา Line ว่า เขาต้องการรับ Notify จากระบบนี้นะ ช่วยออกโค้ดให้ฉันหน่อย จากนั้น Line ออก Code มาให้ และให้ระบบของเรายืนยันว่า มีคนขอรับการแจ้งเตือนมา เราอนุญาติรึเปล่า ถ้าเราอนุญาติ เราก็ปั้มตราด้วย Client Secret ของเราไป

การส่งไป Verify อาจจะยากเล็กน้อย เพราะว่าเรากำลังส่งข้อมูลที่มีความลับไป ดังนั้นจึงต้องส่งเป็น POST และ มันยังเพิ่มความปลอดภัย ด้วยการให้ส่ง Data เป็น **application/x-www-form-urlencoded** เพื่อเป็นการเข้ารหัสอีกครั้ง

### การส่งการยืนยันผ่าน Server ทีเป็น Express

สำหรับระบบที่เป็น Node.js Express.js เราจะไม่ได้เขียนแบบ POST axios ปกติ เพราะเราจำเป็นต้องบอกด้วยว่า เรากำลังส่งค่าเป็น application/x-www-form-urlencoded และยังต้องส่ง data เป็นแบบ encode แล้ว เราจะใช้ `qs` เป็น 1 ตัวช่วย

Import Library qs เข้ามา ไม่แน่ใจว่าจะต้อง install ก่อนหรือไม่

```
const qs = require('qs')
```

Define Data ให้เรียบร้อย ตาม Format จาก Official Documents สังเกตุว่าเขาจะให้เราส่ง code, clientid, client secret ไปด้วย

```
 // Request to LINE Authenthication Server
  const url = 'https://notify-bot.line.me/oauth/token'
  const jsonData = {
    grant_type: 'authorization_code',
    code: accessCode,
    redirect_uri: 'https://theduckcreator.in.th/api/notifyredirect',
    client_id: config.lineNotify.clientID,
    client_secret: config.lineNotify.clientSecret,
  }
```

จากนั้นเรามาทำเป็น config หรือ options ที่จะสั่งโปรแกรมของ axios

```
 const requestOption = {
    method: 'POST',
    header: { 'content-type': 'application/x-www-form-urlencoded' },
    data: qs.stringify(jsonData),
    url,
  }
```

และเราก็ส่งการให้ axios ทำการยิง POST Request ตัวนี้

```
 axios(requestOption)
    .then(async (lineRes) => {
     ...
     })
```

สิ่งที่เราจะได้กลับมาถ้ามันถูกต้อง ก็คือ Access Token ส่วนนี้สำคัญมาก เพราะมันคือสิ่งที่เราจะเข้าถึง User คนนี้ ตัวระบบ Line เอง ไม่รู้ว่า User Id ของ User แต่ละคนเป็นยังไง

![รูปแบบของข้อมูลที่ Line Notify จะส่งกลับมา](/assets/line-notify/6.png)
**_รูปแบบของข้อมูลที่ Line Notify จะส่งกลับมา_**

เราต้องเขียนต่อใน then เลย ที่จะเช็คว่าสถานะของ Response (จากการ POST) ที่ Line ตอบกลับมาที่เราเป็นยังไง และถ้ามันเป็น 200 นั่นแสดงว่า เราได้ access_token ติดมาด้วย เราก็เก็บ access_token นี้ ติดกับ user นี้ใน Database ได้เลย เราจะต้อง code จากส่วนที่แล้วนะ

<script src="https://gist.github.com/theethawat/dd54324552e84441ee8318b155d3ef01.js?file=NotifyDatapost.js"></script>

## การแจ้งเตือนไปยังผู้ใช้ เมื่อมีอะไรเกิดขึ้้น

เมื่อสักครู่ ที่เขียนไป จะเป็นการจดทะเบียนผู้ใช้เข้าระบบ ต่อไปนี้ จะกล่าวถึงการส่ง Notify ไปที่ Line ของ User แล้ว ยังคงเหมือนเดิม คือ เราจะต้องส่งผ่าน Server ของเราที่เราได้ Register กับ line ไว้ในขั้นตอนข้างต้น ไม่สามารถส่งผ่าน Client ได้ เราจะยิง axios ไปจาก Client ไม่ได้ ถ้าต้องการ ให้ ยิงมาที่ Server ก่อนแล้วค่อยให้ Server ยิงต่อ เป็นต้น โดยเรายังต้องส่งเป็น POST และ ข้อมูลยังคงเป็น application/x-www-form-urlencoded โดยมีขั้นตอนดังต่อไปนี้

เริ่มจากการเตรียม Data ที่จะส่งไปที่ Line ของผู้ใช้ Line กำหนดว่า ข้อมูลที่เป็น Text จะเขียนไว้ใน key ที่ชื่อว่า message

```
const url = 'https://notify-api.line.me/api/notify'
      const jsonData = {
        message: `สวัสดีครับ คุณติน มีคนสั่งซื้อสินค้าในร้านเดอะดักส์ของเรา ราคารวม ${
          order.allPrice
        } บาท ครับ`,
      }
```

จากนั้้นเราไปดึงเอา Access Code ออกมา อาจจะดึงมาจาก Database หรือ ที่อื่น ในที่นี้ สมมติว่าเรามีข้อมูลอยู่แล้ว

```
      const accessCode = shopinfo.shop_notification_code
```

แล้วเราก็ทำการส่ง POST ไปที่ Server ของ Line Notify ซึ่งตรงนี้จะไม่ได้ส่งแบบเดิมอีกแล้ว เราจะส่งโดยมี **Authorization Header** ซึ่ง Line เลือกใช้เป็นแบบ Bearer Token โดยให้ส่ง access token ของเรานี่แหละ เป็น Bearer Token โดยวิธีการส่ง Header ไปผ่าน axios ก็จะคล้ายๆ กับ การส่งบอกว่า Content Type เป็นยังไง โดยเราจะสร้าง Option หรือ config ของตัว axios ดังนี้

<script src="https://gist.github.com/theethawat/dd54324552e84441ee8318b155d3ef01.js?file=RequestOption.js"></script>

จากนั้นเราก็จะให้ axios มา Run ตาม config นี้

<script src="https://gist.github.com/theethawat/dd54324552e84441ee8318b155d3ef01.js?file=NotifyInject.js"></script>

ก็จะถือว่าสิ้นสุดการส่งการแจ้งเตือนไปยังผู้ใช้งาน
