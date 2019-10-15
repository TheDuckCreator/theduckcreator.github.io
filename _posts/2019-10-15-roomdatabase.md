---
title: การใช้ Room Database เบื้องต้นใน Android
subtitle: Room Persistance Library การเข้าถึง Database SQLite ใน Android ที่นิยมที่สุด
layout: post-it-blog
author: Theethawat Savastham
category: it-blog
git: https://github.com/TheDuckCreator/TryingRoomDB
created_year: 2019
comments: true
---

การจัดเก็บข้อมูลเป็นสิ่งสำคัญสำหรับทุก ๆ การพัฒนาแอพพลิเคชั่น สำหรับแอนดรอยด์ มันมี By Default อยู่แล้วคือ SQLite Database ซึ่งเป็นฐานข้อมูลแบบ Relational Database วันนี้ผมจะเอาประสบการณ์ที่ผมไปทำใน Code Lab มาอธิบายการทำงานคร่าว ๆ ของ Room Persistance Library หรือ Room Database กัน

## Room Persistance Library

Room Persistance Library จะเป็นตัวสร้าง Database ขึ้นมา เรียกว่า Room Database จากนั้นผู้พัฒนาแอพพลิเคชั่นไม่จำเป็นต้องติดต่อกับฐานข้อมูลโดยตรง แต่จะใช้วิธีการสร้าง Data Access Object หรือ DAO เข้ามาแทน

การใช้ Room Database นี้เราสามารถเขียนได้บน Android ทั้งภาษาจาวา และ คอทลิน แต่ที่จะทำนี้จะใช้ภาษา Kotlin เป็นหลักก่อน ที่ผมเขียนเป็นบทความ เพราะเอาไว้โน้ตของผมเองด้วยนะ กลัวจะลืม

![Room Database Architecure](/assets/android/roomdb.jpg)

หมายเหตุ ที่จะมาเล่านี้อ้างอิงจาก Android with Kotlin Fundamental Course ใน Google Codelab นะครับ สามารถเข้าไปดูและเรียนได้ ฟรี

## สร้างโปรเจกต์ และจัดการ Build.Gradle

เริ่มจากสร้างโปรเจกต์ใน Android Studio แล้วทำการเพิ่ม dependency ลงไปใน Build.gradle ใน Module "app" การเพิ่ม dependency ลงไป มันก็คล้าย ๆ กับการติดตั้ง Library ต่าง ๆ เหมือนใน Node.js ก็จะมี npm install อันนี้ก็คล้าย ๆ กัน โดยเราจะต้องมา define ก่อนว่า จะเอา roomDB เวอร์ชั่นไหน จะได้เป็นตัวแปล จะได้ไม่ต้องเขียนซ้ำหลายครั้ง

        dependencies {
            def room_version = "2.2.0"

            //Room Database
            implementation "androidx.room:room-runtime:$room_version"
            annotationProcessor "androidx.room:room-compiler:$room_version"

            //Kotlin Exentension for room
            implementation "androidx.room:room-ktx:$room_version"

            //.....Dependencies อื่น ๆ ....
        }

ก็จะเป็นการ implementation runtime worker และ annotate ตัวคอมไพเลอร์

## ไฟล์สำคัญสำหรับการสร้าง และจัดการ Database

### 1. Data Class เพื่อเตรียมไว้ใช้ในการสร้าง Table

#### เตรียมสร้าง Data Class

เริ่มจากสร้างไฟล์ Kotlin (.kt) เปล่า ๆ มาตัวหนึ่ง แล้วเราก็มีตัวแปล หรือ Entity ต่าง ๆ อยู่ในสมอง เราสามารถตั้งชื่อไฟล์นี้ แทนชื่อตารางแบบง่าย ๆ ก็ได้ เช่นผมตั้งว่า `SleepNight.kt` เราทราบอยู่แล้วว่าในภาษา Kotlin นั้นชื่อไฟล์กับชื่อคลาส ไม่จำเป็นต้องตรงกันก็ได้ ไม่เหมือนใน Java แต่ว่า ตรงกันไว้ก็ดี จากนั้น เราก็เริ่มทำการ Implement dataclass โดยใส่ตัวแปล ที่เราต้องการให้มันเป็น column ใน Database ลงไป

        data class SleepNight(
            var nightId: Long = 0L,
            val startTimeMilli:Long = System.currentTimeMillis(),
            var endtimeMilli: Long = startTimeMilli,
            var sleepQuality: Int = -1
        )

Initial ค่าไปเลยก็ดี จะเห็นว่า data class ใน Kotlin นั้น เราใส่ตัวแปลลงในส่วนของพารามิเตอร์รับค่าเลย data class ในความคิดของผม ผมมองว่าคล้าย ๆ เราสร้าง Stucture ในภาษา C หรือ อื่น ๆ

#### บอกกับแอนดรอยด์ให้รู้ว่าส่วนนี้ของ Data Class คือส่วนใดของ Table ใน Database

เราจะต้อง define บอกมันให้รู้ว่าส่วนตรงนี้คืออะไร เช่น Attribute นี้เป็น Primary Key หรืออะไรต่าง ๆ เป็นต้น

- data class ของเราเป็น entity set หรือ เป็นตารางนั้นเอง เราต้องบอกให้มันรู้ว่ามันเป็นตาราง โดยการเขียนไว้ข้างหน้าว่า

        @Entity(tableName = "daily_sleep_quality_table")
        data class SleepNight { .... }

- กำหนด Primary Key โดย เราจะบอกว่า ตัวนี้เป็น primary key หน้าตัวแปลของเรา แล้วเราจะใส่ Attribute ควบคุมต่าง ๆ ข้างในวงเล็บ เช่นในตัวอย่างนี้เป็น `autoGenerate = true`

        @PrimaryKey(autoGenerate = true)
        var nightId:Long = 0L,

- กำหนด ColumnInfo ให้กับตัวคอลัมม์อื่น ๆ ที่ไม่ใช่ Primary Key โดยใช้แทก `@ColumnInfo`

          @ColumnInfo(name="start_time_milli")
          val startTimeMilli:Long = System.currentTimeMillis(),

  อย่างน้อยที่สุดก็คือ ใส่ชื่อให้มัน

จะสังเกตุว่าตัวแปลต่าง ๆ ที่เขียนใน Java, Kotlin เราจะเขียนเป็น Camel Case อยู่แล้ว แต่ถ้ามันไปอยู่ในมุมของ Physical หรือ ในมุมที่มันจะไปสร้างใน Database เรามักจะเขียนกลับไปเป็นแบบเดิม เช่นมี Underscore หรือ อะไรต่าง ๆ คล้าย ๆ กับภาษา C หรือ รักษา SQL เอาไว้ เพราะมันอาจจะ Implement ได้ง่านกว่า ในตรงนั้น

เมื่อถึงขั้นตอนนี้ โค้ดทั้งหมด ก็จะเป็นประมาณนี้

<script src="https://gist.github.com/theethawat/f8b4840225756d51540495ab3d50cdb3.js?file=SleepNight.kt"></script>

### 2. สร้าง Data Access Object (Dao)

ตรงนี้คือการสร้างฟังก์ชั่น ขึ้นมา Reference คอมมานด์ หรือ ที่เราเรียกว่า Queries ต่าง ๆ ใน Relational Database นั่นเอง กล่าวคือ Data Access Object จะช่วยให้เราไม่ต้องเขียน Query ยาว ๆ เพียงแต่เรียกเป็นฟังก์ชันสั้น ๆ แค่นั้นเอง โดย Data Access Object จะสร้างอยู่ในรูปแบบของ Interface

ถ้าตามหลักของ OOP Interface คือ โครงสร้างของ Class อันนี้ก็เช่นเดียวกัน คือ มีความสามารถ มี Method อะไรบ้าง Return ค่ามั้ย ถ้า Return จะกลับมาเป็นอะไร รับค่ามั้ย เราจะให้พารามิเตอร์มันชื่ออะไร เป็นลักษณะแบบไหน Int, Float, Char หรือ เป็น User Define Type อย่างที่เราทำ Class, Data Class เราใส่ไปให้หมด โดยไม่ต้อง Implement Method (ที่ Kotlin เรียก fun ) ข้างใน

#### Interface Feature Listing

ต้องการฟีเจอร์ หรือ Queries อะไรกับ Database บ้าง ลองลิสต์ลงมาก่อนเลย โดยชื่อไฟล์ยังเป็นไฟล์ .kt เหมือนเดิม แต่นิยมตั้งเป็น Dao ตามหลังเช่น `SleepDatabaseDao.kt` เป็นต้น

        interface SleepDatabaseDao{
            fun insert(night:SleepNight)
            fun update(night:SleepNight)
            fun clear()
            fun get(key:Long):SleepNight?
            fun getTonight():SleepNight?
            fun getAllNight():LiveData<List<SleepNight>>
        }

#### บอกให้มันรู้ว่าตรงนี้ เป็นส่วนใดของ Query เป็นส่วนใดของ Database

เหมือนอย่างในขั้นตอนที่แล้ว เราก็จะมี Annotate หน้า Class หรือ Function ตอนนี้ก็เช่นกัน

- กำหนดว่า interface นี้ เป็น Data Access Object (Dao) นะ เราก็ใส่ `@Dao` ไปข้างบน Interface

        @Dao
        interface SleepDatabaseDao ( ...)

- การใส่ Query ให้มัน ให้ถูกต้อง ใช่เนื่องจากมันเป็น Interface เราจะไปสั่งอะไรใน Function / Method มันหละ เราจะใช้วิธีนี้ในการบอกมันก่อนเลยว่า เออ เจอคนสั่งฟังก์ชันนี้มาทำยังไง มีด้วยกัน 2 แบบ คือ ใช้ Query สามัญ ๆ ที่มันมีไว้ให้แล้ว กับเขียน Query ขึ้นเอง เริ่มด้วยสามัญ ๆ

        @insert
        fun insert(night:SleepNight)

  ต่อไปมาดูคิวรี่ที่เราสามารถเขียนไปเองได้ โดยเราเองสามารถเอาตัวแปล ที่เราประกาศเป็นตัวรับค่าในฟังก์ชันนั้น มาใช้ได้ โดยใช้เครื่องหมาย : นำหน้าชื่อตัวแปล เช่น `:key`

        @Query("SELECT * FROM daily_sleep_quality_table WHERE nightId = :key ")
        fun get(key:Long):SleepNighit?

โดยสามารถใช้ SQL Command ต่าง ๆ ได้อยู่แล้ว ถึงตอนนี้ของผมจะได้โค้ดประมาณนี้ แล้วเดี๋ยวเราจะมาสร้าง Database ลงในมือถือของเรากัน

<script src="https://gist.github.com/theethawat/f8b4840225756d51540495ab3d50cdb3.js?file=SleepDatabaseDao.kt"></script>

### 3. การเอา Database ไปสร้างขึ้นอยู่บนอุปกรณ์แอนดรอยด์

หลังจากเราวางโครงสร้างของ Database เรียบร้อยแล้ว แต่เรายังไม่สร้าง เราก็จะไปกำหนดคอนฟิก ให้มันสร้าง Database ขึ้นมา คือมันไม่จำเป็นที่จะต้องสร้าง Data base ใหม่ทุกครั้ง ไม่นั้นเครื่อง Client คงความจำเต็มตายเลย

#### Abstract Class Creation

ต่อมาเราจะสร้าง Database โดยการสร้าง Abstract Class ซึ่ง Abstract Class ใน OOP คือ Class ที่ไม่สามารถสร้าง Object ได้โดยตรง แต่สามารถให้มีการถ่ายทอดไปยัง Class ลูกก่อน แล้ว Class ลูกก็จะ Implement Abstract Method จากนั้น ก็จัดการได้เลย ใน Abstract Class จะมีอย่างน้อย 1 Abstract Method ที่ว่าง ๆ เลย ให้ Class ลูก ไปจัดการ Implement เอาเอง และ Method อื่น ๆ เราสามารถเขียนต่าง ๆ ลงใน Method ได้เลย ว่าจะให้ Method นี้ทำงานอะไร ซึ่งตรงนี้เองที่ทำให้ Abstract Class แตกต่างจาก Interface

ดังนั้นเราจะมาเริ่มสร้าง Database ของเราโดยเริ่มจากสร้าง Class ที่ extend RoomDatabase

        abstract class SleepDatabase:RoomDatabase(){
            ...
        }

#### Implement Database Class

หลังจากนั้นก็จะมีขั้นตอนต่าง ๆ ใน การ Implement Database ของเราดังนี้

- บอกว่า Class นี้คือ Database แน่นอนว่าเราจำเป็นที่จะต้อง Annotate เหมือนทุก ๆ อันที่ผ่านมา โดยครั้งนี้เราต้อง **บอก** ด้วยว่าเราจะสร้างจาก Class อะไร (ซึ่งก็คือ Class นี้) มี Properties อะไรบ้าง โดยใช้ `@Database(entities = [...] , version = ... , exportSchema = ...` ดังนี้

        @Database(entities = [SleepDatabase::class],version = 1 ,exportSchema = false)
        abstract class SleepDatabase:RoomDatabase(){

        }

- เอา Data Access Object หรือแหล่งรวม Query ในรูปแบบกำหนดเป็นชื่อฟังก์ชันของเรามาใส่ โดยเราต้องใส่ในรูปแบบของ Abstract Variable ก็ถ้า Abstract Class ไม่มีอะไรเป็น Abstract แล้วเราจะสร้างมันเป็น Abstract ทำไมหละ

        abstract val sleepDatabaseDao:SleepDatabaseDao

- สร้าง Companion Object มาใช้ในการให้ไฟล์อื่น ๆ สามารถเจ้าถึง Database หรือ จะ Initial Database ได้ โดยที่ไม่ต้องไปสร้างเป็นตัวอย่าง หรือ เป็น Instance เอง คือผมพูดถูกมั้ย คือเราสร้าง Instance ไปเลย และให่เขาเรียกใช้ได่เลย ไม่ต้องมาสร้าง Instance เองอยู่

        companion object{

        }

- สร้างตัวแปรตัวอย่างของเรา หรือ INSTANCE ของเราขึ้นมา อย่างที่บอกไว้คือ ตัวลูก ไม่ต้องสร้างเอง เราสร้างไว้ตรงนี้เลย โดยตัวแปลของ INSTANCE ของเราจะเป็นชนิด `SleepDatabase` ก็คือ เรากำลังสร้าง Database ที่มี Table SleepDatabase ที่เราเขียนโครงสร้างของตารางมาแล้วด้านบน

  โดยเราจะมีการกำหนดนิดนึงว่า ตัวแปลรัวนี้จะเป็นตัวแปรรูปแบบ `@Volatile` คือ เราจะไม่ขออยู่ใน Cache ขออยู่ใน Memory ไปเลย จะทำให้ ตัวแปร INSTANCE หรือ ตัวแปรแทนโครงสร้างตารางเนี่ย ไม่อยู่ใน Cache เราจะได้ไม่ต้องกลัวปัญหาว่า มันจะตรงกันหรือเปล่า ใน Cache กับ ใน RAM เรายอมอาจจะช้าลงนิดนึง แต่ชั่วร์

  โดยเราจะสร้างตัวแปรเป็น `Nullable` และ ตั้งค่าเริ่มต้นเป็น `null` และเป็นตัวแปรแบบเปลี่ยนค่าได้

        private var INSTANCE : SleepDatabase? = null

  ที่เราตั้งเป็น Null ก็เพราะว่า โอเค ตอนนี้ INSTANCE เทียบได้กับDatabase ที่เราจะสร้างแล้ว แต่ว่า เราจะยังไม่สร้าง เราจะตรวจสอบก่อนว่าตอนนี้ มันมีอยู่แล้ว หรือ ไม่ อย่าลืมว่า เราอาจจะไม่ได้ build หรือ Install / Run App นี้ครั้งเดียว ในระหว่าง การพัฒนา ดังนั้น เราจะเช็คก่อนว่า Database มีอยู่แล้วหรือไม่ ถ้ามีก็ไม่ต้องสร้าง ถ้าไม่มี ก็สร้าง จะได้ไม่ซับซ้อนด้วย

- Synchronyze Program แล้วดูว่า instance ของเรามีอยู่หรือไม่ เราจะใช้การ Smart Cast สร้างตัวแปลใน loop ไม่สิ ใน Function นี้อีกตัว เป็นการ Copy มาจะได้ไม่มีปัญหาค่าซ้ำซ้อนของตัวแปร แล้วค่อยส่งค่ากลับไป

        synchronized(this){
            var instance = INSTANCE

            ...

            return instance
        }

- เช็คว่า Database ถูกสร้างแล้วหรือยัง ข้างต้นนี้ คือ Database ถูกสร้างแล้ว Return มันกลับไปเลย แต่ถ้า instance เป็น null เราจะสร้าง Database มันขึ้นมา โดยสิ่งต่อไปนี้ยังอยู่ใน Scope ของ `synchronized(this){ }` นะ ก่อน `return` ด้วย

        if(instance == null){
            instance = Room.databaseBuilder(
                context.applicationContext,
                SleepDatabase::class.java,
                "sleep_history_database"
            ).fallbackToDestructiveMigration().build()
        }

  ดูจากโค้ดเราก็จะพบว่า เราจะสร้าง instance ให้เป็นค่าใดค่าหนึ่ง นั่นก็คือ เป็น Room และเรียกใช้ DatabaseBuilder โดยยึดจาก Class SleepDatabase ก็คือ Class นี้ และใช้ชื่อ `sleep_history_database` จากนั้นจึงสั่ง `fallbackToDestructiveMigration()` และ `build()` ก่อนจะ Return ออกไป เป็น Instance

จากขั้นตอนทั้งหมด ของส่วนที่ 3 นี้ จะมี Source Code ทั้งหมดเป็นดังนี้

<script src="https://gist.github.com/theethawat/f8b4840225756d51540495ab3d50cdb3.js?file=SleepDatabase.kt"></script>

ทั้งหมดนี้ ก็คือการสร้าง Database เริ่มต้นในโปรแกรมของเรา โดยสร้างผ่าน Room Database บน Android ตรงนี้ยังไม่พูดถึงการใช้ UI จากฝั่งผู้ใช้มา Connect หรือ Interact กับโปรแกรมของเรา ซึ่งตรงนี้เราก็อาจจะไปดีไซด์ Layout แล้ว ใช้รูปแบบต่าง ๆ ในการ Map View ของเรา เข้ากับ Model ของเราตรงนี้ ได้ ไม่ว่าจะเป็น Model View Controller (MVC), Model View Presenter (MVP) หรือ Model View ViewModel (MVVM) ก็ได้
