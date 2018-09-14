<template>
  <div id="mergePlane"></div>
</template>
<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import tweenManager from 'pixi-tween'
import configMarqee from '../assets/js/marqee_config'
import particles from 'pixi-particles'
import '../assets/js/pixi-display'
export default{
  name: 'mergePlane',
  data () {
    return {
      isLandscape: false,
      step: 1
    }
  },
  mounted () {
    this.initPIXI()
  },
  methods: {
    initPIXI () {
      const canvasW = this.isLandscape ? 736 : 414
      const canvasH = this.isLandscape ? 414 : 736
      const outRadius = 180 // 转盘外半径
      const inRadius = 124 // 转盘内半径
      const rotationTime = 3000 // 转盘时间
      const spinCopies = 6
      let app, canvas, that = this, startDegrees = -180, lightDegrees = -180
      app = new PIXI.Application(canvasW, canvasH)
      document.getElementById('mergePlane').appendChild(app.view)
      canvas = document.querySelector('#mergePlane canvas')
      canvas.style.width = '100%'
      canvas.style.height = '100%'
      app.renderer.view.style.position = 'absolute'
      app.renderer.view.style.display = 'block'
      app.renderer.autoResize = true
      
      app.stage = new PIXI.display.Stage()
      app.stage.group.enableSort = true

      let group1, group2
      group1 = new PIXI.display.Group(2, true) 
      group2 = new PIXI.display.Group(3, true) // 飞机层
      app.stage.addChild(new PIXI.display.Layer(group1))
      app.stage.addChild(new PIXI.display.Layer(group2))

      let pivotSprite, circleBg, lightArr1 = [], lightArr2 = [], outCircleBg, prizeList = []
      let startIndex = 0, rounds, degrees, prize, isBegin = false
      // 背景等静态图片
      let bgSprite = new PIXI.Sprite.fromImage(configMarqee.bgUrl)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = canvasW
      bgSprite.height = canvasH
      app.stage.addChild(bgSprite)
      //logo. btn
      // let logo = new PIXI.Sprite.fromImage(configMarqee.planeIconUrl)
      // logo.width = 100
      // logo.height = 30
      // logo.y = canvasH - 10
      // logo.x = 10
      // app.stage.addChild(logo)
      // let downBtn = new PIXI.Sprite.fromImage(configMarqee.buyBtnUrl)
      // downBtn.width = 100
      // downBtn.height = 30
      // downBtn.y = canvasH - 10
      // downBtn.x = 10
      // app.stage.addChild(downBtn)

      let marqeeContainer = new PIXI.Container()
      marqeeContainer.x = canvasW / 2
      marqeeContainer.y = canvasH / 2
      marqeeContainer.pivot.x = marqeeContainer.width / 2
      marqeeContainer.pivot.y = marqeeContainer.height / 2
      app.stage.addChild(marqeeContainer)

      //转盘
      outCircleBg = new PIXI.Sprite.fromImage(configMarqee.circleUrl)
      outCircleBg.anchor.set(0.5)
      outCircleBg.width = 352
      outCircleBg.height = 352
      marqeeContainer.addChild(outCircleBg)
  
      circleBg = new PIXI.Sprite.fromImage(configMarqee.smCircleUrl)
      circleBg.anchor.set(0.5)
      circleBg.x = 3
      circleBg.y = -1
      circleBg.width = 252
      circleBg.height = 246
      marqeeContainer.addChild(circleBg)

      pivotSprite = new PIXI.Sprite.fromImage(configMarqee.marqeePviot)
      pivotSprite.anchor.set(0.5)
      pivotSprite.width = 88 //120
      pivotSprite.height = 99   // 135
      pivotSprite.interactive = true
      pivotSprite.buttonMode = true
      marqeeContainer.addChild(pivotSprite)

      // 手指示
      let handObj = this.createHandTween(8, 30, marqeeContainer)
      handObj.tween.start()

      // prize奖品
      for (let i = 0; i < spinCopies; i++) {
        let prize = new PIXI.Sprite.fromImage(configMarqee.marqeePlane[i])
        prize.anchor.set(0.5)
        prize.width = 70
        prize.height = 56
        circleBg.addChild(prize)
        if (i == 5) {
          prize.width = 78
          prize.height = 20
        }
        prize.x = Math.sin(this.d2a(startDegrees + i * 360 / 6)) * (inRadius - 32)
        prize.y = Math.cos(this.d2a(startDegrees + i * 360 / 6)) * (inRadius - 32)
        prize.rotation = startDegrees + 360 / 6 / 2 - 90;
        prizeList.push(prize)
      }
      let redCont = new PIXI.Container()  // 红光
      let yellowCont = new PIXI.Container() // 黄光
      marqeeContainer.addChild(redCont)
      marqeeContainer.addChild(yellowCont)
      //红光 黄光
      let rTexture = PIXI.Texture.fromImage(configMarqee.redLight)
      let yTexture = PIXI.Texture.fromImage(configMarqee.yellowLight)
      for (let m = 0; m < 24; m++) {
        let rLight = new PIXI.Sprite(rTexture)
        rLight.anchor.set(0.5, 0.5)
        rLight.width = 12
        rLight.height = 12
        rLight.x = Math.sin(this.d2a(lightDegrees + m * 360 / 24)) * (outRadius - 38)
        // if (m > 12) {
        //   rLight.y = -Math.cos(this.d2a(m * 360 / 24)) * (outRadius - 38)
        // } else {
          rLight.y = Math.cos(this.d2a(lightDegrees + m * 360 / 24)) * (outRadius - 38)
        //}
        
        redCont.addChild(rLight)
        lightArr1.push(rLight)
      }
  

      const circle_tween = PIXI.tweenManager.createTween(circleBg)
      pivotSprite.on('pointerdown', () => { // 开始
        rounds = this.getRandomNum(2, 4)  // 圈数
        degrees = this.getRandomNum(0, 360) // 角度
        prize = spinCopies - 1 - Math.floor(degrees / (360 / spinCopies)) // 奖品
        console.log(prize)
        getStart(circle_tween, rounds, degrees)
        // isBegin = true
      })


      function getStart (tw, rounds, degrees, prize) {
        tw.from({rotation: 0}).to({rotation: 360 * rounds + degrees})
        tw.easing = PIXI.tween.Easing.outCubic()
        tw.time = rotationTime
        tw.on('start', () => {
          if (that.step == 1) {
            marqeeContainer.removeChild(handObj.sprite)
            PIXI.tweenManager.removeTween(handObj.tween)
          }
          console.log('I`m rotation')
          let temp = startIndex
          for (let n = 0;n < 3; n++) {
            lightArr1[temp].texture = yTexture
            lightArr1[temp].width = 28
            lightArr1[temp].height = 28
            temp++
          }
        })
        tw.on('end', () => {
          isBegin = false
          console.log('end')
          setTimeout(() => {
            for(let i = 0; i < 24; i++) {
              lightArr1[i].texture = rTexture
              that.setWidth(lightArr1[i], 13, 13)
              if (i % 2) {
                let yLight = new PIXI.Sprite(yTexture)
                yLight.anchor.set(0.5, 0.5)
                yLight.width = 28
                yLight.height = 28
                lightArr2.push(yLight)
                yLight.x = lightArr1[i].x
                yLight.y = lightArr1[i].y
                yellowCont.addChild(yLight)
                let tw = PIXI.tweenManager.createTween(yLight)
                tw.delay = 200
                tw.from({alpha: 1}).to({alpha: 0})
                tw.time = 800
                tw.repeat = 3
                tw.start()
              } 
            }
            showScene2()
          }, 1000)
          
          
        })
        tw.start()
      }
      
      
      let count = 0
      app.ticker.add(delta => {
        PIXI.tweenManager.update()
        if (!isBegin) {
          return
        }
        let custom = startIndex
        if (startIndex == 0) {
          startIndex = 23
        }
        lightArr1[custom].texture = rTexture
        that.setWidth(lightArr1[custom], 13, 13)
        if (custom != 0) { // 当为23的时候不减1
          startIndex--
        }
        lightArr1[startIndex].texture = yTexture
        that.setWidth(lightArr1[startIndex], 28, 28)
        count += delta * 10
      })

      //场景二
      function showScene2 () {
        prizeList[prize].parentGroup = group2
        let bgSprite2 = new PIXI.Sprite.fromImage(configMarqee.bgUrl)
        bgSprite2.x = 0
        bgSprite2.y = 0
        bgSprite2.width = canvasW
        bgSprite2.height = canvasH
        app.stage.addChild(bgSprite2)
        const bg_tween = PIXI.tweenManager.createTween(bgSprite2)
        bg_tween.from({alpha: 0}).to({alpha: 1})
        bg_tween.time = 5000
        bg_tween.easing = PIXI.tween.Easing.linear()
        let tempY = prizeList[prize].y
        const plane_tween = PIXI.tweenManager.createTween(prizeList[prize])
        PIXI.tweenManager.getTweensForTarget(bg_tween)
        plane_tween.from({y: tempY}).to({y: tempY - 20})
        bg_tween.start()
      }
    },
    d2a(n) {
      return n * Math.PI / 180;
    },
    createHandTween (x, y, parentCont) { // 创建手
      let obj = {
        sprite: '',
        tween: ''
      }
      obj.sprite = new PIXI.Sprite.fromImage(configMarqee.handUrl)
      obj.sprite.anchor.set(0.5)
      obj.sprite.width = 62
      obj.sprite.height = 62
      obj.sprite.name = 'hand1'
      parentCont.addChild(obj.sprite)
      obj.tween = PIXI.tweenManager.createTween(obj.sprite)
      obj.sprite.x = x
      obj.sprite.y = y
      obj.sprite.scale.set(0.6)
      obj.tween.from({scale: {x: 0.6,y: 0.6}})
      obj.tween.to({
        scale: {
          x: 0.8,
          y: 0.8
        }
      })
      // obj.tween.delay = 1400
      obj.tween.time = 800
      obj.tween.loop = true
      obj.tween.pingPong = true
      obj.tween.easing = PIXI.tween.Easing.linear()
      return obj
    },
    setWidth (sprite, w, h) {
      sprite.width = w
      sprite.height = h
    },
    getRandomNum (min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min)
    }
  }
}
</script>
