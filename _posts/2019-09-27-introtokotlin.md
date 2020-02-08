---
title: Kotlin In My Note
subtitle: โน้ต Syntax ต่างๆ ของภาษาคอทลิน เพื่อเตรียมสำหรับทำโปรเจกต์ของผมเอง
layout: post
author: Theethawat Savastham
category: it-blog
language: Kotlin
git: https://github.com/theethawat/HelloKotlin2
created_year: 2019
comments: true
---

นี่คือโน้ตที่ผมเขียนไว้ตอนผมเรียนคอทลิน ประกอบกับแอนดรอยด์ไปด้วย เพื่อที่จะทำโปรเจกต์ของผม ผมเองก็โน้ตเก็บเป็น Markdown ไว้ แล้วก็มาเผยแพร่ให้ดู เป็นภาษาอังกฤษนะครับ ว่าง ๆ จะมาแปลไทยให้อ่าน

## คุยกันก่อน

Kotlin เป็นภาษาตัวหนึ่งที่รันอยู่บน Java Virtual Machine(JVM) ได้ พัฒนาโดยกลุ่มที่ได้รับการสนับสนุนจากบริษัท Jetbrains ผู้พััฒนา IntelliJ Idea และ Android Studio ซึ่งผมเองต้องเขียน Android ดังนั้นผมจึงต้องมีพื้นฐาน Kotlin หน่อย ๆ Kotlin ตอนแรกเนี่ยเขาคิดว่าจะทำมาเป็น Library ของ Java แต่ตอนหลังแล้ว พอเขาคิดว่าการทำ Library พวกนี้มันต้อง Compatible กับอะไรแบบเดิม ก็เลยเขียนภาษาขึ้นมาใหม่ดีกว่า ชื่อภาษา Kotlin เป็นที่นิยม และ ตัวที่ Google เองก็เชียร์ให้เขียน Android กับ Kotlin ด้วย ถ้าใครเขียนพวก OOP อย่าง Java, C# มาก่อน Kotlin ไม่ยากครับ ง่ายกว่าแน่นอน หรือถ้าใครไม่เคยเขียน มันก็ไม่ยากนะ แค่มันอาจจะงง ๆ หน่อย ตามประสาของภาษาโปรแกรมมิ่งสมัยใหม่

## เริ่มต้นกับคอทลิน

ถ้ามว่าจะเริ่มต้นยังไงเหรอ ลองไปดูที่ https://kotlinlang.org ก็ได้ แนะนำเลยคือ ติดตั้ง IntelliJ Idea ครับ จบ หรือถ้าใช้ Ecilipse ก็ใช้ได้นะ ตอนแรกผมก็ทำกับ Ecilipse ซึ่งอาจจะต้องลง JRE หรือ JDK หรือเปล่านั้น จำไม่ได้ เพราะตอนนั้นรู้สึกว่าเครื่องผมอ่ะพร้อมอยู่แล้ว แต่ถ้ามัน Require อะไร มันคงแจ้งเตือนเรามาเองแหละครับ แต่ส่วนใหญ่ก็ไม่ค่อยมีคนเขียน Native Kotlin นะ จะเป็นการฝึกหน่อย ๆ แล้วนำไปใช้ต่อในการเขียนพวก Android หรือเว็บแอพ อะไรอย่างนี้ครับ

โอเค เริ่มละนะ ว่างเมื่อไหร่ เดี๋ยวมาแปลเป็นภาษาไทยให้

---

# Intro To Kotlin

<b>Learn Kotlin: </b> [Official Kotlin Doc](https://kotlinlang.org/docs/reference)

This is only my note during learning and coding something may be mistake. At first I cannot typing in Thai in Ecilipse IDE Now I learn in IntelliJ IDEA it can type in Thai but I'm idle.

## Structure

1. Don't Need a Pre-processor because Kotlin is initial itself at the begining from IDE
2. Don't worry about package name, It nescessory only to show that it are in the same domain. In file system they don't show in class view but in IDE it will show you the structure of class
3. Function Initial run with main
4. No Semicolon needed in Kotlin

## Table of Content

1. [Basic Syntax](#basic-syntax)
2. [Function](#function)
3. [Variable](#variable)
4. [String Management](#string-management)
5. [Array](#array)
6. [Class](#class)

---

## Basic Syntax

### printing out the word

```
//using print or println
print("Hello World")
```

### printing out the value of variable

It can be inside the double quote with \$ sign using variable Name

```
print(" $variablename ")
//If have any calculating
print(" ${variableName1 + variableName2}")
```

---

## Function

- To call using like in C Language
- Using <b>fun</b> to initial the function

### Function Prototype

```
fun functionName(parameterName:ParameterType): Returntype{
}
```

Example for Very Basic Function (don't have return)

```
fun sum(a:Int,b:Int){
	println("a is $a  b is $b")
}
```

### Call function to use Return Value

In variable define or redefine

```
	var x = functionName(Parameter)
	var y:Int
	y = functionName(Parameter)
```

Beware Function and Properties are not the same

### Using the properties

In Properties we use to get something from variable you can using Kotlin Library properties

#### Calling Properties

```
	var testLengthVariable = "Theethawat"
	var theethawatLength ="Theethawat".length
```

#### Or calling in print statement

```
	println("Variable of testLengthVariable is ${testLengthVariable.length}")
```

---

## Variable

- Using <b>val</b> can be assign a value only once
- Using <b>var</b> can be reassign
- Can use both Type define or not, if you not define a type you must initial value in the declare variable part and Kotlin will assign the type automatically
- Type Define and value Define is optional All are correct

```
var a:Int =1
var a = 2
var a:Int
```

- This is example that <b>Incorrect</b>

```
var a
```

- Variable can define both in global and local (Private to function) if it be global it can be modify by any function without re parameter-initial like

```
var x = 0
fun incrementX() {
   x += 1
}
```

---

## String Management

### Triming Margin

Can use trimMargin using with prefix '|' only if use another prefix using trimMargin('Prefix')

```
val str3 = """ #Hello World""".trimMargin()
val str4 = """ #Hello World """.trimMargin("#")
```

Can use `.trimIndent` for trim white space before text like a white space
and Can use triple quote """ for let string in the string can be multiple line as we code

---

## Array

### Array Declaring

Can Creating Array with Initial Value , Initial Value Using Function or Uninitialized using normal form `arrayOf`
if you initial value

```
val a = intArrayOf(11,13,15,17,12)
```

or using Specific type of Array

1. BooleanArray `booleanArrayOf(true, false)`
2. ByteArray `byteArrayOf(1, 2, 3)`
3. CharArray `charArrayOf('a', 'b', 'c')`
4. DoubleArray `doubleArrayOf(1.2, 5.0)`
5. FloatArray `floatArrayOf(1.2, 5.0)`
6. IntArray `intArrayOf(1, 2, 3)`
7. LongArray `longArrayOf(1, 2, 3)`
8. ShortArray `shortArrayOf(1, 2, 3)`

or can design array by using function like

    val asc = Array(5,init = {i->i+1})
     for (b in asc){
         println(b)
     }

### Array Properties

1. Properties that can use when declare an Array
   `sortedArray(), sortedArrayDescending()`
2. Properties that can use when we using function
   `sort(), sortDescending() , min(), max(),first(), last()` it will return a view of the things
   that you want in array (some only for Int or Number Variables)
   for example
   println("List of Array B is \${b.sorted()}")

### Collection

Collection is like array but it's the tool that can easier management

- List use `Listof()` (Can replace string with other type)

      val  myList = listOf<String>("Theethawat","Songpon")

- Map use `mapOf()`

       val myMap = mapOf<Int,String>(Pair(1,"Theethawat"), Pair(2,"Theematach"))

- Set use `setOf()`

      val mySet = setOf<String>("Theethawat","Sirinuch")

  using 3 types of collection and other type of Array can be random access

---

## Class

### Basic Concept of Class

- It's like class in Java and sometime it's like stucture in C
  in my own opinion
- Use function (main or any) to call a class
- If Compare to Java it has 3 part
  - Field for variable declare and initial (Java divide into
    field and Constructor) and can use argument `also()` to make this variable call the function like `val nameInput = "The Input value is $name".also(::println)`
  - `init()` can use for initial, set value and call function in class
  - function `fun()`
- Everythings contain public, protected, private same like other OOP
- Can use dot(.) for go to the attribute like in C or Java
- For Example in file Address.kt

---

## Object

### Companion object

If your class have a Companion object show that you can implement or adding the function to that object like

    class MyClass {
    	companion object { }  // will be called "Companion"
    	}

    	fun MyClass.Companion.printCompanion() { println("theethawat") }

    	fun main() {
    	MyClass.printCompanion()
    	}

You will got an output "Theethawat", i pick it from official Document at [Object Documents in Kotlin](https://kotlinlang.org/docs/reference/object-declarations.html#companion-objects)

---

## Basic Function and library

- Kotlin will add Basic Library name kotlin.\* if you use IDE or compiler
- There include many function inside
- String Function Reference is located at [Kotlin Function Reference](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/index.html)
