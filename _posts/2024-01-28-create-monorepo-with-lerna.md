---
title: สร้าง Monorepo ของ JavaScript ด้วย Lerna กัน
subtitle: การทำ Monorepo บน JavaScript ด้วย tools ที่ชื่อ Lerna
layout: post-it-blog
author: Theethawat Savastham
category: it-blog
created_year: 2024
comments: true
language: Javascript
tag:
  - monorepo
  - lerna
  - javascript
git: https://github.com/theethawat/lerna-demo
ogimage: /assets/lerna/Monorepo JS Using Lerna.png
---

เคยมั้ยที่รู้สึกว่างานของเราเริ่มจะมีความใหญ่เกินไป การทำงานของระบบที่เป็น Client-Server ธรรมดาอาจไม่สามารถ Handle งานได้อย่างมีประสิทธิภาพ เราอยากจะขยายโปรเจกต์ แยกออกไปเป็น Service ต่างๆ แต่เราก็ไม่อยากเขียนโค้ดหลายชุด อยากใช้อะไรร่วมกัน วันนี้จะมาแนะนำกับ Tools ตัวหนึ่งที่ช่วยงานด้านนี้ได้ ก็คือ Lerna

## Introduction

เริ่มต้นกลับมาเขียนอีกครั้งในรอบ 3 ปี แทบจะรันเว็บนี้ไม่เป็นแล้ว เนื่องจากว่าเมื่อวันเสาร์ที่ผ่านมา (27 มกราคม 2024) ผมได้มีโอกาสไปร่วม Bar Camp Songkhla ครั้งที่ 8 ที่ตึก LRC มหาวิทยาลัยสงขลานครินทร์มาครับ จริงๆ เตรียมเรื่องนี้ไว้ว่าจะไปเป็น Speaker แต่ไม่ค่อยชัวร์ว่าเรื่องมันเก่าแล้วหรือเปล่า รวมถึงไปครั้งแรก ยังทำตัวอะไรไม่ค่อยถูก เลยไม่ได้ยื่นไป แล้วเอามาใส่ไว้ในนี้แทน

## Monorepo คืออะไร

Monorepo ก็คือ Monorepository นี่แหละครับ มันคือการเขียนโค้ดหลายหลายๆ ส่วนไว้ใน Repository เดียวกัน เช่นอาจจะมีทั้ง frontend, backend และ service ตัวนู้นตัวนี้รวมไว้ด้วยกัน ถ้าเป็น JavaScript ก็จะเป็นหลายๆ Packages รวมไว้ด้วยกัน จากที่กล่าวไว้ข้างต้น ในการเขียนโปรเจกต์ที่เริ่มใหญ่ขึ้น บางทีระบบอย่างผมทำเป็น Client-server อาจจะไม่สามารถทำงานได้เต็มที่แล้ว เช่น ระบบ HTTP มันอาจจะเป็น Short Job ซึ่งชอบทำงานสั้นๆ แต่หลายๆ Request ได้ แต่บางทีงานของเราอาจจะยาวจนระบบมันไม่พร้อมให้บริการตัวอื่นๆ ดังนั้นแอพแบบ Monolith อาจไม่เพียงพอต่อการจัดการ เราอาจจะต้องมี Service ที่มาจัดการตัวที่ทำงานยาวๆ มีให้มันต่อคิวต่าง ๆ นา ๆ ดังนั้นเราจำเป็นต้องสเกลแอพให้มันเป็น Microservices มากขึ้น

การขยายเป็น Microservice ต่างๆ แล้ว เราสามารถเลือกได้ว่าเราจะขยายเป็นหลายๆ Repository สำหรับทำก็ได้นะ หรือ เราจะรวมอยู่ใน Repository เดียวก็ได้ แล้วแต่ความถนัดและขนาดของระบบเลยครับ ใน Opensource งานใหญ่ๆ ก็จะเห็นว่าเขาก็แยกเป็นหลายๆ Repository เหมือนกัน แต่ถ้างานอยู่ในระดับไม่ใหญ่มาก คนทำไม่ได้เยอะมาก แล้วอยากเขียนให้เราใช้โค้ดบางส่วนด้วยกันได้โดยบริหารจัดการไม่ยาก การทำ Monorepo ก็เป็นหนึ่งในคำตอบที่หน้าสนใจเหมือนกัน

Monorepo มี Tools หลายๆ ตัวขึ้นอยู่กับแต่ละภาษาเช่น [Gradle](https://gradle.org/) ถ้าใครเคยเขียน Android ก็จะเคยได้ยินมันอยู่ ตัวนี้เป็นของ Java, [Bazel](https://bazel.build/) จะเป็นของ Google เขียนได้หลายภาษา, ตัว JavaScript ก็จะมี [Lerna](https://lerna.js.org) ที่เป็นตัวเก่าแก่, [Nx](https://nx.dev/) ซึ่งเป็นลูก และเป็นเจ้าของ Lerna อีกที หรือ [Turborepo](https://turbo.build/repo/docs) ซึ่งผมยังไม่ได้ลอง ซึ่งทุกคนสามารถเข้าไปหา Tools อื่นๆ เกี่ยวกับ Monorepo ได้ที่ [monorepo.tools](https://monorepo.tools/)

## Lerna

Lerna เป็น Libary ที่ค่อนข้าง Original มีมานานอยู่พอสมควร สำหรับการทำ Monorepo ใน JavaScript แต่ส่วนตัว ผมชอบมันมากกว่า Nx นะ ซึ่งเป็นอีกตัวที่ตอนนี้ก็เจ้าของเดียวกัน แต่ตัวอื่นยังไม่ได้ลอง Official Document ของมันอยู้ที่ [lerna.js.org](https://lerna.js.org)

## ทดลองใช้กันเถอะ

เราจะมาทดลองใช้งานกัน โดยจะเป็น Frontend กับ Backend โดยเราจะใช้ Library Constant ที่เราสร้างร่วมกัน จะเป็นตัวอย่างง่ายๆ ในทางปฏิบัติจริงที่ผมใช้ ผมจะใช้เป็น Library หรือเป็น Code ของแต่ละ Module แยกจากกันเช่น โค้ดของ Manufacturing อยู่ Package หนึ่ง Code ของ Inventory อยู่อีก Package หนึ่งเป็นต้น แล้วก็ค่อยมีโค้ดตัวหลักที่อ่านจาก Library ตัวนี้มากอีกที เราจะมาเริ่มจากพื้นที่ปล่าวๆ กันเลย

### Initial Project

1. เราจะเริ่มจากการสร้าง Folder เปล่าๆ มาโฟลเดอร์หนึ่ง จากนั้นเราจะ Initial ด้วย `npm init -y`
2. เริ่ม Install Lerna ด้วยคำสั่ง

   ```
   npx lerna init
   ```

   หลังจากนั้น มันก็จะมี packages ที่ชื่อว่า Lerna ขึ้นมาใน dependencies

3. ระบุ Workspace ของเรา ว่าโฟลเดอร์ไหนที่เราจะให้เป็นโฟลเดอร์ที่เก็บแพคเกจต่างๆ ผมจะระบุว่าเป็นโฟลเดอร์ `packges/` โดยระบุเข้าไปใน Package.json ได้เลย

   ```
   "workspaces":[
     "packages/*
   ]
   ```

4. เราจะสร้าง Package ขึ้่นมา 2 Packages ก่อน โดยจะเป็น Frontend และ Backend โดยสองอันนี้จะไม่ได้ขึ้นต่อกันก่อนนะ โดย Frontend ผมจะใช้ React ส่วน Backend ผมจะใช้ Express.js โดยผมเข้าไปใน packages แล้วสร้าง react ด้วย vite

   ```shell
     cd packages
     npm create vite@latest frontend --template react
   ```

   เมื่อเรียบร้อยแล้วต่อด้วย`cd frontend` และ `npm install`

5. เลื่อนเคอร์เซอร์ออกมาข้างนอก แล้วเราจะสร้างแพคเกจสำหรับ Backend กัน โดยเราจะลง Express กับ Nodemon ซึ่งเป็นตัวที่ทำให้ Package ของเรารันแบบ demon ในขณะที่เรา Development นั่นเอง
   ```
     cd ..
     mkdir backend
     cd backend
     npm init -y
     npm install --save express nodemon
   ```
6. สร้างไฟล์แรก ผมจะตั้งชื่อว่า index.js ขึ้นมาสำหรับการ Handle API ธรรมดาเลย โดยผมจะใช้เป็น ES Module (แบบ import/export) นะครับ ดังนั้นผมจะตั้งใน package.json เป็น type: module ด้วย แต่ถ้าใครสะดวกจะเขียนเป็น commonjs ก็สามารถทำได้เลยนะครับ

   <script src="https://gist.github.com/theethawat/0a45f0bbb18d7739a969035b41292249.js"></script>

7. เรามาดูใน package.json เราตั้งค่า script ตรงที่ dev ให้รัน `node index.js` ที่ผมตั้งไว้ที่ dev เพราะว่าจะให้มัน Script เดียวกันกับ frontend ที่ development ของระบบไว้ที่ dev เหมือนกัน (ลองเข้าไปในดูในไฟล์ package.json ของ frontend ได้ครับ) ลืมบอกไปครับว่า ชื่อใน name จะเป็นชื่อใน package ที่เราจะเรียกใช้โดย package อื่นๆ ได้นะครับ แต่อันนี้ผมไม่ได้ตั้งใจให้มันเป็น package library

   <script src="https://gist.github.com/theethawat/5a24e3916e2d7b1a5c406ff7d89adbdb.js"></script>

8. ทีนี้กลับมาที่ package.json หลัก ใน project root ตั้งค่าสคริปต์ dev โดยให้สั่งไปที่ lerna run dev จากนั้นลองรันคำสั่ง npm run dev ดูครับ ทั้ง 2 แพ็คเกจก็จะเริ่มทำทั้งสองอันเลย

   <script src="https://gist.github.com/theethawat/25f3202553e74c7813c590c1f02d417e.js"></script>

### ทดลองสร้าง Constant package

1. เราจะมาสร้าง constant package โดยสร้างโฟลเดอร์เข้าไปใน packages
   ```
      cd packages
      mkdir constant
      cd constant
      npm init -y
   ```
2. สร้างไฟล์ Constant แรกขึ้นมา ผมจะสร้างไฟล์ชื่อ workingStatus.js แล้ว import เข้าไปใน index.js (โดยผมจะใช้เป็น ES Module เหมือนกัน)

   <script src="https://gist.github.com/theethawat/722996f9a59623f598485f9a41f59c0c.js"></script>

3. เข้าไปใน package.json ตั้งชือ package ที่เราต้องการ

   <script src="https://gist.github.com/theethawat/cfbf7c3733d8847e28e9e384f0bd560f.js"></script>

4. เราจะ import เข้าไปใน frontend และ backend เราจะใส่ package ตังนี้ที่ผมตั้งชื่อว่า `@theethawat/lerna-demo-constant` เข้าไปใน package.json ของ frontend และ backend โดยใน backend ผมจะใส่ nodemonConfig ให้มัน watch ข้อมูลบน constant ด้วย เมื่อมีการเปลี่ยนแปลงโค้ดใน constant จากนั้นเรา `npm install` อีกครั้ง (ถ้ายังไม่ได้ให้ลองเข้าไปใน frontend และ install ทีนึง backend อีกทีนึง)

   <script src="https://gist.github.com/theethawat/e4a3e64a5f36b1c185b91988d057eba2.js"></script>

5. เราลองตั้ง import จาก package ของเราเข้าไปใน backend ดูนะครับ อันนี้ผมใช้ VS Code ถ้าผมเคาะ Ctrl+Space bar ดูจะมี Hint ขึ้นมาช่วยแล้ว

   ![Hint ของ Lerna](/assets/lerna/1.png)

6. เราจะมา Log ดูว่า import ได้มั้ย

   ```js
   app.get("/", (req, res) => {
     console.log("Working Status List", WORKING_STATUS);
     res.send("Hello World!");
   });
   ```

   จะเห็นออกมาประมาณนี้ครับ

   ![Output of backend](/assets/lerna/2.png)

7. เช่นเดียวกัน เราจะลอง Import เข้าไปใน frontend ดูแล้วเราจะใช้ JSON.stringnify แสดง

   <script src="https://gist.github.com/theethawat/e5e6d7dc9b6585a3e5072630471f2318.js"></script>

   ภาพมันก็จะประมาณนี้

   ![Frontend import lerna](/assets/lerna/3.png)

นี่ก็จะเป็นคร่าวๆ ของการ Implement Lerna ที่เป็น Monorepo นะครับ จริงๆ แล้วยังมีเรื่องของการ Deploy Package ลงบน NPM หรือ Package Manager อื่นๆ อีก แต่สำหรับบทความนี้ คงจะแค่นี้ก่อน ส่วนสไลด์ที่ผมเตรียมไว้ แต่ยังไม่ได้ใช้ สามารถดาวน์โหลดสไลด์ได้ที่นี่เลย [ดาวน์โหลด Slide](/assets/lerna/Monorepo JS Using Lerna.pdf) สำหรับโค้ดที่ใช้ในบทความนี้ คลิกที่ Source Code ข้างล่างได้เลยนะครับ
