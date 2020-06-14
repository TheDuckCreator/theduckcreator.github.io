---
title: มา Deploy API Server บน Azure App Service โดยใช้ GitHub Action กัน
subtitle: มาลองดูวิธีปล่อย API ของเราบน PaaS ของ MS Azure
layout: post-it-blog
author: Theethawat Savastham
category: it-blog
git: https://github.com/theethawat/personal-data-api
created_year: 2020
comments: true
language: Javascript
tag:
  - api
  - azure
---

การ Deploy งานของเราใน Platform as a Service ค่อนข้างได้รับความนิยมขึ้นมากในโลกปัจจุบัน แทนที่ของการเอาไปลงแบบ IaaS ซึ่งจริง ๆ ถามว่าอะไรง่ายกว่าเนี่ย ก็อยู่ที่คนถนัดนะครับ เพราะตอนผมเองเริ่มต้น Deploy ใน IaaS ผมเองก็ใช้เวลาน้อยกว่าการทำบน PaaS แต่ด้วยการจัดการความคุ้มค่าต่องบประมาณ (Cost Effectiveness) และการที่เทรนด์การแยกแต่ละส่วนของระบบเป็น Microservice เพื่อการบริหารจัดการที่ง่ายกว่า ทำ CI/CD ได้ง่ายกว่า กำลังได้รับความนิยม ทำให้การปล่อยของบน PaaS ได้รับความนิยมขึ้นเรื่อย ๆ เช่นกัน

## PaaS ใน Node.js App

จริง ๆ แล้วก็มีหลายผู้ให้บริการเลย ที่เปิดพื้นที่ให้ปล่อยของกันสำหรับใครที่ทำ Node.js และต้องการปล่อยบน PaaS เช่นอย่าง [Heroku](https://www.heroku.com/) ที่จะได้รับการพูดถึงเป็นอย่างแรก ๆ แล้วผมก็จะไปลองดูและเปรียบเทียบเหมือนกัน หรืออย่างของใหม่มาแรงอย่าง [Vercel](https://vercel.com) ซึ่งผมเองก็ใช้ในการ Deploy เว็บส่วนตัวผมเหมือนกัน [Theethawat Personal Site](https://theethawat.theduckcreator.in.th) ซึ่งเดิมชื่อว่า Zeit Now ใช้ง่ายมาก ๆ คุณภาพอย่างดี นอกจากนั้นยังมีที่เราสามารถประยุกต์ไป Host App ได้ เช่น Firebase, Github Pages หรือบริการ App Service ในผู้ให้บริการ Cloud อย่างทั้ง AWS และ GCP และที่ผมกล่าวมาทั้งหมดนี้ มีบริการฟรี สำหรับคนที่จะเข้าไปเล่นระดับผมอยู่แล้ว

### Azure Web Apps

ในบทความนี้จะมาบอกถึงการใช้ Platform Azure App Service / Azure Webapp ของ Microsoft Azure ซึ่งผมเองก็ไม่ได้บอกว่ามันดี ราคาดี หรือ ง่ายกว่าที่กล่าวมานะ (อาจจะด้อยกว่าในบางเรื่องด้วยซ้ำ) แต่พอดีพอเสิร์จเรื่องนี้ บทความภาษาไทยยังมีไม่ค่อยมากเท่าไหร่ ดังนั้น เออคงมีคนอ่านบ้างแหละ บทความนี้คงมีประโยชน์บ้าง และเผื่อใครที่ใช้ Stack ของ Azure อยู่แล้ว จะหาที่ปล่อย Node.js App ให้มันอยู่ในบริการเดียวกัน

> ที่มาทำพวกนี้ เรากำลัง Cost Effective อยู่คร้าบ หลังจากเครดิต Azure เรารั่วไปกับ Application Gateway กับ Virtual Gateway ของ Azure นี่แหละ ไหลรวม ๆ 2 วัน กินไปเกือบ 25 US\$ ผมนี่รีบลบอย่างไว หมดตูดแล้ววว ตอนนี้เราเลยมาหาวิธีถูก ๆ หรือวิธีฟรี ๆ เล่นกัน

### แอพที่จะมา Deploy

ก็จะเป็น App ทำ Open REST API ธรรมดา ๆ นี่แหละครับ เขียนกับ Javascript Node.JS + Express แล้วมีการเชื่อมข้อมูลจาก MongoDB Cluster ที่รันอยู่บน Cloud ของ Mongo DB Atlas ครับ

![บางส่วนของ Code App ที่จะมา Deploy](/assets/az-app-services/appcode-preview.png)
**_ตัวอย่างโค้ดในแอพครับผม_**

แต่ผมจะมีการเอาข้อมูลที่เป็น Sensitive Data ออกมาอยู่ในไฟล์ `.env` นะครับ แล้วใช้ package `dotenv` เข้าไปดึงข้อมูลเหล่านี้มานะครับ ในเครื่องที่เรารันแอพ หรือ ทดสอบ เราใส่ไฟล์ `.env` ไปเลย แต่ใน server เราจะ deploy ผ่าน Source Code ใน GitHub จึงไม่ค่อยดีเท่าไหร่ที่เราจะปล่อยพวก Sensitive Data เหล่านี้ เราจะเก็บไว้ในบริการที่ Server ทำให้ จะกล่าวถึงต่อไปครับ

## มาเริ่ม Deploy กัน

เราจะติ๊งต่างว่าเราได้เขียนโค้ดและเทสบนเครื่องของเราเรียบร้อยแล้วนะครับ เราจะมาเริ่ม Deploy โดยเราจะให้เป็นการส่ง Continuous Delivery มาจาก [GitHub Action](https://github.com/features/actions) เราก็จะเริ่มจาก Push โค้ดที่เราเขียนแล้ว มาที่ GitHub กันเลยครับ ถ้าเป็น Public Repository ส่วนนี้จะฟรีทั้งหมดนะครับ

### สร้าง Resource ใน Azure

หลังจากนั้นเราจะเริ่มมาที่ Azure Portal หรือ ใครจะถนัด CLI ก็แล้วแต่เลยครับ เราจะมาเริ่มการสร้าง โดยไปที่่ New Resource หรือ Marketplace ก็ได้ แล้วหาคำว่า Web Apps จากนั้นเราก็มา Setting Resource ของเรากันเลย

![ค้นหาใน Marketplace](/assets/az-app-services/1-Search-Marketplace.png)
**_ค้นหา Web App หรือจะพิมพ์ App Service ในการสร้าง Resource ใหม่_**

![Config Runtime Stack](/assets/az-app-services/3-Runtime-Stack.png)
**_ตั้งค่า Name ที่จะกลายไปเป็น URL เลือก Runtime Stack, OS, ที่ตั้งของ Server, วิธีการจะ Publish ลงไป_**

ในที่นี้จะใช้ Node.js Version 12 LTS และจะเลือกรันบน Linux นะครับ เพราะว่าผมเองเคยลองเลือกรันบน Windows มาแล้ว ใช้ยากมาก Windows เอาไว้รัน Dotnet App แล้วกัน (หรือภาษาอื่น ๆ ก็ได้นะครับ) ที่ตั้งผมจะเลือกเป็น Southeast Asia และจะส่งไปเป็น Code นะครับ ใครจะตั้งเป็นอย่างไร สุดแล้วแต่เลยครับ

![Default App Size](/assets/az-app-services/4-Default-Size.png)
**_App Service Plan และ Size ของ Application เริ่มแรก_**

App Service Plan นี่เราต้องมีอยู่แล้วละนะ ถ้าใครไม่มีก็ให้มันสร้างใหม่มาแบบผมก็ได้ มันจะเอาไว้ช่วยบริหารจัดการพวกราคาต่าง ๆ ให้เรา ตัวเริ่มต้นเราไม่ต้องเสียเงินอยู่แล้วนะครับ แต่ตัวสำคัญคือ SKU and Size ครับ มีหลายอันให้เราเลือก แต่ที่ผมจะใช้ เราจะใช้แบบ Dev/Test F1 Basic ที่เป็นแบบฟรีก่อนครับ เขาจะมีแรมให้ 1GB และให้เรา Compute ได้ 60 นาทีต่อวัน

![Select wanted Size](/assets/az-app-services/5-Select-Size.png)
**_เลือก Size ของ Applicaion ตามที่เราต้องการ โดยเราสามารถเพิ่มหรือลดได้ภายหลัง_**

เขียนยังไม่เสร็จเลย Push มาก่อนอีกละ
