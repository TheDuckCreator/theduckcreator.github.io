---
title: Android with Databinding in Kotlin (Note)
subtitle: Android With Kotlin Note use Databinding
layout: post
author: Theethawat Savastham
category: it-blog
language: Kotlin
git: https://github.com/theethawat/AndroidLayoutWithDatabinding
created_year: 2019
comments: true
---

Kotlin the new 1st class language in Android. And today the trend name databinding is so amazing more than using findViewById right? it handle Smoother.

While I study in this for my project, I has taked a note in Markdown and I will show you about my note but I recommend you to go to Android Codelab [Kotlin Fundamental Course](https://codelabs.developers.google.com/android-kotlin-fundamentals/) to learn it. This is only my note taking use sometimes it can help you if you cannot remember all like me

# Android with Kotlin Databinding

## Pre-Request

Active Android-X in your Android Studio

### Activatate Databinding

Enable Databinding in `build.gradle` file that app part inside `android{}` tag

    dataBinding {
        enabled = true
    }

After That Sync your app

## Basic Viewbinding

### Layout Compatible

if you using linear layout add layout `<layout> .... linear layout ..... </layout>` over there
and move layout name space like `xmlns:android="......"` to this layout and <b>Build your App</b>

### Programing

go to `MainActivity.kt` or other program file

- Active Binding Variable using `lateinit` that is predefine without value initial like
  private lateinit var binding: ActivityMainBinding
- Set Content View Using Data Binding everything using the attribute of variable that have define in the previous step
  binding = DataBindingUtil.setContentView(this,R.layout.activity_main)

`this` tell you that 'this activity' and R.layout.activity_main show that layout that you want to bind or showing

Databinding will generate A Specific Variable Name of Attribute like `nickname_show` to `nicknameShow` using Camel Case that
automatically making by an IDE

### Activity Recieving

- Using Binding Variable in this code using bind in syntax using dot method like

        binding.doneButton.setOnClickListener {
            addNickname(it)
        }

`it` in Kotlin like `this` in Java to call compiler that I want to make function on this class

### Managing Activity

- Using Binding Variable and dot syntax like
  binding.nicknameShow.text = binding.nicknameInput.text
- Using `binding.apply{...}` to easy managing code (binding is Binding Variable)
- Using `invalidateAll()` to invaluate data (Get new data) when Refresh UI
- Example on this program

         binding.apply {
            binding.nicknameShow.text = binding.nicknameInput.text
            invalidateAll() /_ to invaluate data (Get new data) when Refresh UI _/
            binding.nicknameInput.visibility = View.GONE
            binding.doneButton.visibility = View.GONE
            binding.nicknameShow.visibility = View.VISIBLE
        }

  Using Attribute As same as using findViewById

## Recommended Data-Binding

- Using Data Class (in Another Kotlin File) for keep the object type variable like the stuct in C or can use any class instead of data class
- Link The Layout to that class

        <data>
            <variable name="myName" type="com.example.aboutme.MyName"/>
        </data>

The name of the variable you can define but link to your data class or Any structure class.
