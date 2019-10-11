---
title: วิเคราะห์ High Availability และ Fault Tolerance บนกรณีศึกษา MongoDB Replica
subtitle: ลองใช้ MongoDB ดูการทำงานที่ตอบสนอง High Availability และ Fault Tolerance
layout: post-it-blog
author: Theethawat Savastham
category: it-blog
created_year: 2019
comments: true
---

ยังไม่เสร็จนะ หลังจากที่เราพูดถึง HA หรือ High Availability และ FT หรือ Fault Tolerance กันแล้ว เราจะมาทดลองสร้างระบบดูกันว่า เออการทำงานมันเป็นอย่างไร กับระบบที่มันบอกว่าตัวเองเป็น High Availability และ Fault Tolerance ว่าตรงไหนคือ HA ตรงไหนคือ FT

## MongoDB Replica

MongoDB Replica เป็นการทำ Replication ของฐานข้อมูลแบบ NoSQL อันหนึ่งก็คือ MongoDB โดยเราจะต้องเตรียมโหนดสำหรับ MongoDB ไว้ เป็นจำนวนเลขคี่ หรือ เริ่มต้นก็คือ 3 Node , 5 Node เป็นต้น เราสามารถจำลองสร้างขึ้นมาทั้งอยู่บนเซิร์ฟเวอร์เดียวกัน คนละพอร์ต อยู่คนละเซิร์ฟเวอร์กัน หรือ อย่างไรก็ได้ โดยหลักการทำงานของ MongoDB Replica นั้น Client คือผู้เขียนหรือผู้อ่านข้อมูล ในที่นี้ก็รวมถึงแอดมินที่เขียนข้อมูลลงในดาตาเบสด้วย จะ Read and Write ไปที่ Primary Node ก่อนที่ระบบมันจะทำ Replication ไปยัง Secondary Node ทั้งสอง หรือ 4 โหนด และมีการทำ Heartbeat คุยกัน

![Mongo DB Replica Working Model](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)
รูปจาก Official Documents ของ MongoDB

<!-- ### ทำไมจำนวนโหนดต้องเป็นเลขคี่ด้วยหละ

นี่คือสิ่งที่อธิบายต่อไป เนื่องจากโอเค เพื่อทำการให้ระบบของเราทนต่อความขัดข้อง หรือ จัดการเพื่อตอบสนอง Fault Tolerance โหนดที่เป็น Secondary จะมีการทำ Replication อยู่ตลอดจาก Primary อยู่แล้ว และทันทีที่ Primary Node เกิดการล่มสลายไป Secondary Node พวกนี้จะต้องทำการ Vote เพื่อเลือกว่าตัวไหน จะเป็น Primary ตัวต่อไป โดยการที่จะโหวตกันได้เนี่ยจำนวนโหนดที่เหลืออยู่ จะต้องเกินกึ่งหนึ่ง (ครึ่งหนึ่ง) ของสมาชิกทั้งหมดที่เราประกาศไว้ เช่น เราเล่น 3 Nodes มีโหนดหนึ่งตายไป เหลือ 2 Node ที่ Avaliable อยู่ ซึ่งมากกว่าครึ่งของ 3 Nodes จึงโหวตกันได้

กรณีถ้าเรามี 4 Nodes   -->

### ลองเล่น MongoDB Replica กัน บน Docker

เนื่องจากมีทรัพยากรไม่มาก ผมจะเล่น 3 Module Replica set แล้วกันนะครับ ผมขี้เกียจจะเปิด Linux มาแล้ว ผมจะเข้าไปสร้างบน Server ผม ซึ่งเป็น Linux อยู่แล้ว โดยสร้าง Docker Compose ไว้ สร้าง 3 Node ผมจะสร้าง Directory ขึ้นมา แล้วใน Directory นั้น จะสร้างไฟล์ `docker-compose.yml` ขึ้น

![Docker Compose.yml File](/assets/internetprograming/mongo1.jpg)

ในรูปผิดอยู่ เอาตาม โค้ดใน Gist ของ Github ข้างล่างนะ

ในที่นี้ผมสร้างใน IP เดียวกันก่อน ยังไม่ได้ตั้ง Private Network แต่เอามันมาลงที่คนละพอร์ต คือ Forward Default Port ของ MongoDB คือ 27017 มาที่พอร์ต 4041,4042 และ 4043 โดยที่ผมเองจะให้ Volumn กับมันไป พอร์ตละ Folder ละกัน โดย By default ของ MongoDB บน Docker Ubuntu Bionic จะอยู่ที่ `/var/lib/mongodb` ซึ่งรู้สึกว่ามันก็ต่างกันอยู่นะ กับการลง MongoDB Native บนเครื่อง Ubuntu หรือ Directory เดียวกัน อันนี้ไม่มั่นใจ แล้วเราจะสร้าง Config File เอาตามชื่อของมันแล้วกัน ก็คือ `mongod.conf` ซึ่งผมจะให้มันไปอยู่ที่ `/etc/mongod.conf` ซึ่งอันนี้รู้สึกว่าเหมือนกันทั้งค่า Default ของเครื่อง Linux และ Linux บน Docker

<script src="https://gist.github.com/theethawat/45b5c5eb98dece35a77623df23bbf836.js?file=docker-compose.yml"></script>

### สร้างไฟล์ และ โฟลเดอร์ ก่อนที่จะให้ Docker Access

แน่นอนครับ แม้ว่าเราจะมีไฟล์คอนฟิกแล้ว แต่เราบอกให้ Pass Volumn และ Pass File ไป แต่เรายังไม่ได้สร้างไฟล์ที่จะให้ Pass ไปเลย ไฟล์ที่เราจะไปกำหนดการ Replication จะอยู่ใน `mongod.conf` ครับ ถ้าใครลง MongoDB ใน Linux ตัวเองลองเปิด ไม่ว่าจะ GUI,CLI ดูก็ได้ครับ ซึ่งตอนนี้ผมไม่ได้ลง ผมเลยไป Search ดูใน Google ไฟล์ก็จะเป็นตามนี้ครับ [MongoDB conf.d Original from GitHub ](https://github.com/mongodb/mongo/blob/master/rpm/mongod.conf)
่
![Folder Making](/assets/internetprograming/mongo2.jpg)

จากเดิมที่ไม่มีอะไร เราสร้าง Directory ขึ้นมา 3 Directory ก่อนที่จะสร้างไฟล์ conf.d แล้ว copy โครงพื้นฐานของมัน จากลิงค์ที่ให้ไปข้างบนนี้มาใส่ได้เลยย เราจะได้ไฟล์ของเรามา อย่างนี้เลย

![File Making](/assets/internetprograming/mongo3.jpg)

ทีนี้ เราจะโฟกัสที่ สองตัวคือ storage ซึ่งเป็นสีขาว ๆ เลยกับ replication ซึ่งมัน comment อยู่ ดังรูปด้านบน อย่างแรกเลย คือใน docker-compose.yml เราส่ง Storage ไปที่ `/var/lib/mongodb` ใช่มั้ยหละ แต่ตรงนี้เป็น mongo โอเค แก้ให้มันเหมือนกันครับ จะแก้ที่ docker-compose file หรือ แก้ที่นี่ก็ได้เหมือนกัน ตัวผมแก้ที่นี่แล้วกันนะ

### การตั้งค่า ReplicaSet

ทีนี้คือจุดสำคัญที่เราเฝ้ารอมานานคือการตั้ง replica set เรา uncomment ที่ replication ออกได้เลย และ เราก็เขียนเป็นชื่อ `replSetName:"อะไรก็ได้"` แค่ทั้ง 3 Nodes นี่ต้องตั้งให้เหมือนกันนะ ส่วนตัวเราจะใช้เป็น rs0 แล้วกัน

![Replication Setting](/assets/internetprograming/mongo4.jpg)

<script src="https://gist.github.com/theethawat/45b5c5eb98dece35a77623df23bbf836.js?file=mongod.conf"></script>

และเราก็ทำแบบนี้กับทั้ง 3 อัน ก่อนที่เราจะมา up compose กัน

### Docker-compose up

รันคำสั่ง `sudo docker-compose up` หรือ `docker-compose up` ก็ได้ถ้าเป็น root อยู่แล้ว

![Docker Compose Up](/assets/internetprograming/mongo5.jpg)

จากนั้นจะพบว่าของผม Error เต็มเลย น่าจะผิดตรงตั้งค่า Docker อะไรซักอย่างเนี่ย เดี๋ยวมาเขียนต่อละกัน วันนี้หยุดก่อน
