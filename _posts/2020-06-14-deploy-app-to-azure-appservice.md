---
title: มา Deploy API Server บน Azure App Service โดยใช้ GitHub Actions กัน
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

### Content

- [PaaS ใน Node.js App](#paas-ใน-node.js-app)
- [มาเริ่มสร้าง Resource กัน](#มาเริ่มสร้าง-resource-กัน)
- [Deploy With GitHub Actions](#deploy-with-github-actions)
- [เช็คงานใน Azure](#เช็คงานใน-azure)

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

---

## มาเริ่มสร้าง Resource กัน

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

จากนั้นเราก็ Review และ Create เลย

### ดูผลงานและเตรียมการ Deploy

หลังจากที่เรา Create ได้แล้ว เราจะเข้ามาที่ Dashboard ของ Web App ของเรา เราจะเห็นข้อมูลต่าง ๆ เช่นพวก IP ที่ได้ Dynamic Allocation มา Status, Package ต่าง ๆ และเราสามารถที่จะกด Browse เพื่อที่จะไปที่ URL ที่เราสร้างขึ้นมา

![Azure Web App Dashboard](/assets/az-app-services/7-Dashboard.png)
**_หน้าสรุปข้อมูลของ App Service ที่เราสร้างขึ้นมา_**

![Basic Browse App](/assets/az-app-services/8-Browse.png)
**_หน้า Web Application ที่ขึ้นมาโดยที่ยังไม่ได้ Push อะไรลงไป_**

---

## Deploy With GitHub Actions

เราจะ Deploy มันด้วย GitHub Actions ถึงแม้มันจะเป็น Preview อยู่ใน Azure แต่เนื่องจากงานของเราไม่ได้ Production อะไรมาก เราก็จะใช้นะ ถ้าใครใช้งาน Production จริง ๆ อาจจะลองหาวิธีอื่นดูนะครับ

### Setup Deployment Source

ไปที่ Deployment Center ครับ จากนั้นเราเริ่มจากการเลือกว่า Source Code ของเรานั้นอยู่ในตัวจัดการ Repository อันไหน มีทั้ง Azure Repos, GitHub, Bitbucket หรือ Local Git ในที่นี้ผมจะเลือกเป็น GitHub นะครับ จากนั้นมาถึงส่วนสำคัญคือเลือก Build Provider เราจะเลือด **GitHub Action** ครับ

![Repository Source](/assets/az-app-services/9-Repo-Source.png)
**_การเลือกผู้ให้บริการเก็บ Repository_**

![Build Provider](/assets/az-app-services/10-Build-Provider.png)
**_เลือกวิธีในการ Build Source Code โดยที่นี้จะเลือก GitHub Actions_**

เราทำการ Config ต่าง ๆ นะครับ เช่น Repository ชื่ออะไร Build Environment จะเป็นอย่างไร จากนั้นมันจะ Generate YAML File ที่เอาไว้เป็น Workflow ให้เราครับ เราไม่ต้องคัดลอก แล้วไปใส่ใน Repository เรานะครับ มันจะจัดการให้เอง โดยจะอยู่ในโฟลเดอร์ `.github/workflows` ซึ่งต่อไปเมื่อเรา Pull Repository นั้นมา เราก็จะเอามาแก้ไขไฟล์ Workflow ใน Editor ของเราได้ หรือจะแก้ไขใน UI ของ GitHub เลยก็ได้ครับ

![GitHub Action Workflow](/assets/az-app-services/12-Workflow.png)
**_Workflow ที่เราได้เพื่อไป Build ใน GitHub Actions_**

### ตรวจงาน Build ใน GitHub

GitHub จะทำการ Build ให้เราตาม Workflow ที่เราให้ ทุก ๆ ครั้งที่เรา Commit หรือ ที่เราตั้งค่าไว้ ถ้าหากมี Log อะไรระหว่างการ Build หรือมี Error มันก็จะขึ้นมาในตัว GitHub มีให้ Badge ด้วยนะว่า Build สำเร็จหรือไม่ โดยเราจะออกจาก Azure แล้วไปที่ GitHub Repository ของเราเลย

เมื่อเราไปที่ Repository และเราทำการ Commit ใหม่ (ที่จริงการ Add Workflow ก็เป็นการ Commit แล้วนะ) มันก็จะเริ่ม Build โดยที่ มันจะขึ้นอยู่ที่แถบด้านบนของ Repository นั้น

![Workflow Running Status](/assets/az-app-services/13-Commit-Action-1.png)
**_สัญลักษณ์ที่แสดงผลว่า Workflow กำลังทำงานอยู่_**

เราสามารถคลิกตรงนั้น หรือ คลิกไปที่ **Actions** ด้านบนก็ได้ แล้วเราจะเห็นผลงานที่เราทำไว้ ใน Workflow ก่อน ๆ ใน Repository นี้ (ถ้าเริ่มทำครั้งแรกมันก็จะยังมีแค่อันเดียวนี่แหละ อันนี้ Deploy มาหลายครั้งแล้ว)

![GitHub Action Workflow Result List](/assets/az-app-services/14-Commit-Action-2.png)
**_List งานที่ GitHub Actions ทำให้เรา_**

คลิกเข้าไปที่ Event นั้น ๆ เราจะเห็นว่าตอนนี้เขากำลังทำในขึ้นตอนไหนอยู่ ผลการ Run เป็นยังไงบ้าง จากนั้นพอมันเสร็จแล้ว มันจะ Push ไปที่ Azure

![GitHub Action Working Process](/assets/az-app-services/15-Action-Running.png)
**_งานที่ GitHub Actions กำลังทำให้เราอยู่_**

---

## เช็คงานใน Azure

เมื่อเรา Build ผ่าน มีการ Push ไฟล์มาที่ Azure App Service แล้ว ทีนี้มันก็จะเอา Code ของเรานี่แหละ ไปทำการ Deploy โดยถ้ามันจะเสมือนรันคำสั่ง `npm start` กล่าวคือ มันจะทำงานตามคำสั่งที่เราเขียนให้กับ Script Start ใน `package.json` ดังนั้นเราก็ดูให้ดีนะว่าเราได้สร้าง Production Build ไว้หรือเปล่า ถ้าอยากให้มันรันจาก Production Build เราจะเขียนคำสั่งยังไง ถ้าต้องการให้มันรันแบบนั้นแบบนี้จะต้องใส่คำสั่งใด เป็นต้น

> Azure App Service จะถือเอาคำสั่งใน `package.json` ตรงที่ start เป็นคำสั่งที่จะใช้ในการดำเนินการ

### เช็คการ Commit งาน

เราสามารถเช็คสถานะการส่ง CI/CD จาก GitHub มาที่ Azure ได้โดยเปิดไปที่ Deployment Center

![Azure App Service Deployment Status](/assets/az-app-services/17-Deployment-Status.png)
**_เช็คสถานะการส่งขึ้นมา Deploy ที่ Deployment Center_**

ถ้าใครสถานะที่ดี และไม่มีการกำหนด Environment Variable อะไร ก็ลองไป Browse หน้าเว็บดูได้นะ แต่ถ้าใครที่มี Environment Variable เหมือนเรา ยังก่อน เราจะมาแก้ Environment เราก่อน

### ตั้งค่า Environment Variable

เราจะไปตั้งค่าสิ่งเดียวกับที่เราเก็บไว้ในไฟล์ `.env` บนเครื่อง โดยเราจะไปที่ **Configuration** และใส่ Application String เข้าไป

![App Service Configuration](/assets/az-app-services/18-ENV-Dash.png)
**_หน้า Config Environment Variable_**

![Environment Variables Configuration](/assets/az-app-services/19-Edit-ENV.png)
**_การใส่ หรือ แก้ไข Environment Variable โดยจะเป็นลักษณะของ Key-Value_**

จากนั้น เราสามารถไปเช็คงานเราใน Website ของเรา หรือผ่านช่องทางอื่น ๆ ซึ่งอยากบอกว่ามันไม่ได้ Take Effect ทันทีนะ หรือไม่ก็อาจจะมี Cache ของบราวเซอร์ หรือ อื่น ๆ ด้วย พอร์ตต่าง ๆ ที่ให้แสดงออกดูให้ดีด้วยนะครับ รู้สึกว่าเขาจะตั้ง Default ไว้ที่ 8080 แล้วมันจะ Mapping มาที่ 80,443 หรือวิธีหนึ่งที่จะทำให้รู้ข้อผิดพลาด คือไปที่ **Diagnose and Solve Problem** แล้วเข้าไปดู Application Log ก็ได้นะ

---

## บทส่งท้าย

ก็ถือว่าวิธีนี้เป็นวิธีหนึ่งนะครับ อาจจะมีรายละเอียดปรีกย่อยค่อนข้างมากหน่อย หรือ การจัดสเปค แพ็คเกจ ค่าใช้จ่าย ที่อาจจะมีค่ายอื่นที่เป็นพื้นที่ให้ Deploy Node.JS Application โดยตรงทำได้ได้เปรียบกว่าบ้าง (อย่างโดยส่วนตัวคิดว่า Vercel ง่ายกว่า ประสิทธิภาพดีกว่า แถมใช้ Domain ตัวเองได้ด้วย มี Cert Gen ให้อีกต่างหาก แต่เครื่องมือบางส่วนทางนี้อาจจะมีมากกว่า) แต่วิธีนี้ก็เป็นอีกวิธีหนึ่งนะครับ

มีข้อดีคือเราสามารถบริหารจัดการร่วมกับ Resource อื่น ๆ และ ใช้ Command Line ในการทำพวก Deploy Infrastructure, Scale, Autoscale หรือ อื่น ๆ ได้ และเริ่มต้นที่ไม่ต้องเสียค่าใช้จ่ายใด ๆ และใช้ได้ค่อนข้างเยอะนะ ยังไง ๆ ใครมีประสบการณ์ คำแนะนำอะไร บอกมาได้นะครับ ส่วนเราเหลือแต่ Heroku ตัวนึงที่ได้รับความนิยมมาก แต่ผมยังไม่ได้ลองใช้ ไว้จะลองเล่นดูนะครับ

สามารถใช้ API ตัวนี้ของผมได้ที่ [Theethawat API](https://theethawat-api.azurewebsites.net/) และสามารถดู Doc ของ API และ Source Code ได้ที่ [GitHub](https://github.com/theethawat/personal-data-api) Repository นี้เลยครับ
