---
title: First Time on React | Card Game BNK48
subtitle: Card-Game งานแรกที่ได้ทำกับ Javascript Library อย่าง React เข้าสู่ภาษาสมัยใหม่
layout: post
author: Theethawat Savastham
category: work
language: Javascript
git: https://github.com/theethawat/card-game
created_year: 2018
comments: true
---

คงเป็นความภูมิใจเล็ก ๆ ที่วันหนึ่งเราได้ก้าวมาสู่ภาษาสมัยใหม่ที่ได้รับความนิยมอย่าง Javascript สักที
และก็เป็นครั้งแรกที่เขียน Javascript ES6 on Framework เต็มรูปแบบ ได้เข้าใจการวาง Component,Stage,Props ถึงแม้จะเป็นการสั่งงานในห้องเรียนก็ตาม และใข้ BNK48 มาเป็นพร็อพในงานครั้งนี้

## Bug Exchange Thank You

ขอขอบคุณไอเดียบางอย่างจาก GitHub ของเพื่อนผมครับ จาก GitHub [sunisathammasoon](https://github.com/sunisathammasoon/)
หวังว่าอาจารย์คงไม่ว่าอะไรนะครับ เนื่องจากมันเป็นประโยชน์สำคัญของ GitHubโดยเอามาตรงที่

      constructor(props){
          super(props)
          this.state = prepareStateFromWord(this.props.value)
      }

ขอขอบคุณไอเดียจาก GitHub ของเพื่อนผม จาก GitHub [Ikhalas](https://github.com/Ikhalas/)
โดยเอามาตรงที่(ใน WordCard.js ครับ)

        { Array.from(this.state.chars).map(
        (c,i)=> <CharacterCard value = {c} key = {i} attempt={this.state.attempt}
                    activateHandler={this.activateHandler}/>

## Dependency Install

ถ้าระบบไม่รู้จัก \_ หรือ lodash ให้พิมพ์คำสั่ง

        npm i -g npm
        npm i --save lodash

ใน Command Prompt หรือ Terminal (ตอนนั้นชี้อยู่ที่ card-game กล่าวคือ D:/card-game แล้วพิมพ์ npm i -g npm .....)
โดยมันจะเอา lodash มาติดตั้งให้

## CSS Library ที่ใช้ได้แก่

- [Bootstrap 4.1](https://getbootstrap.com)
- [Google Fonts](https://fonts.google.com)
- [Fontawesome](https://fontawesome.io)

## Deployment and View

เข้าชมเว็บไซต์ได้ที่ [theethawat.github.io/card-game](https://theethawat.github.io/card-game) โดยการให้มัน deploy websiteใช้คำสั่ง

        npm install gh-pages --save-dev

ก่อนที่จะแก้ไขข้อมูลในไฟล์ package.json ให้มัน deploy ใน githubusername.github.io/card-game

        yarn build
        yarn run deploy

แล้วมาที่ Repository ของ GitHub อันนี้ ไปที่ Setting > GitHub Page แล้วเลือก branch gh-pages branch แล้ว save เป็นบริการฟรีของ GitHub

## Thank you

ขอบคุณที่มาจาก [react-gh-pages](https://github.com/gitname/react-gh-pages)
ขอขอบคุณภาพจาก [BNK48 Office](https://www.bnk48.com/#/Member) ทำเพื่อการศึกษา มิได้มีเจตนาละเมิดลิขสิทธิ์

Made with Love and Assigned from Theethawat Savastham | ธีร์ธวัช สวาสดิ์ธรรม CoE'28 PSU
