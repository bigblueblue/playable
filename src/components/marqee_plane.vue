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
      firstGuid: true,
      step: 1,
      time1: null,
      rank: '',
      oldPosition: {x: 0, y: 0},
      addArr: [20, 40, 60, 80, 100, 120, 140, 160, 180, 200]
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
      const spinCopies = 6
      const slotGap = 140
      const slotH = 120, lightDegrees = -180
      let startDegrees = -90
      const tempX = Math.cos(this.d2a(startDegrees)) * (inRadius - 32)
      const tempY = Math.sin(this.d2a(startDegrees)) * (inRadius - 32) + 6  // 第一个奖品的位置
      var app, planeList = new Array(6).fill({}), handObj2, slotList = [], sceneContainer, trackIconList = [], path
      var emptyIcon = PIXI.Texture.fromImage(configMarqee.rankEmptyUrl), coinNum = 0, coinSprite
      var fullIcon = PIXI.Texture.fromImage(configMarqee.rankFillUrl), runningPlanes = [], planeTemp
      let canvas, that = this
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
    
      let group1, group2, group3
      group1 = new PIXI.display.Group(1, true) 
      group2 = new PIXI.display.Group(2, true) // 飞机层
      group3 = new PIXI.display.Group(2, true) // 手指示
      app.stage.addChild(new PIXI.display.Layer(group1))
      app.stage.addChild(new PIXI.display.Layer(group2))
      app.stage.addChild(new PIXI.display.Layer(group3))
      let pivotSprite, circleBg, lightArr1 = [], lightArr2 = [], outCircleBg, prizeList = []
      let startIndex = 1, endIndex, rounds, degrees, rotationTime, isBegin = false, currentIndex, clearTime
      let isFirst = true, beginndex
      // 背景等静态图片
      let bgSprite = new PIXI.Sprite.fromImage(configMarqee.bgUrl)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = canvasW
      bgSprite.height = canvasH
      app.stage.addChild(bgSprite)

      let marqeeContainer = new PIXI.Container()
      marqeeContainer.x = canvasW / 2
      marqeeContainer.y = canvasH / 2
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
      // circleBg.rotation = this.d2a(60)
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
      let degreeArr = [0, 60, 120, 180, 240, 300], indexArr = [4, 3, 2, 1, 0, 5]
      // prize奖品
      let  rotateDegrees = 120
      startDegrees = 0
      for (let i = 0; i < spinCopies; i++) {  // 顺时针方向 0
        let prize = new PIXI.Sprite.fromImage(configMarqee.planeList[i])
        prize.anchor.set(0.5)
        prize.scale.set(0.22)
        circleBg.addChild(prize)
        if (i == 0) {
          prize.scale.set(1)
       }
        prize.x = Math.cos(this.d2a(startDegrees + i * 60 + 30)) * (inRadius - 32)
        prize.y = Math.sin(this.d2a(startDegrees + i * 60 + 30)) * (inRadius - 32)
        prize.rotation = that.d2a(rotateDegrees)
        rotateDegrees -= 120
        prizeList.push(prize)
      }
      
      let redCont = new PIXI.Container()  // 红光
      let yellowCont = new PIXI.Container() // 黄光
      marqeeContainer.addChild(redCont)
      marqeeContainer.addChild(yellowCont)
      //红光 黄光
      let rTexture = PIXI.Texture.fromImage(configMarqee.redLight)
      let yTexture = PIXI.Texture.fromImage(configMarqee.yellowLight)
      for (let m = 0; m < 24; m++) { // 0 - 6 - 12 - 18 顺时针方向
        let rLight = new PIXI.Sprite(rTexture)
        rLight.anchor.set(0.5, 0.5)
        rLight.width = 12
        rLight.height = 12
        rLight.x = Math.sin(this.d2a(lightDegrees + m * 360 / 24)) * (outRadius - 38)
        rLight.y = Math.cos(this.d2a(lightDegrees + m * 360 / 24)) * (outRadius - 38)
        redCont.addChild(rLight)
        lightArr1.push(rLight)
      }
      path = new PIXI.tween.TweenPath()
      path.closed = true
      let gPath = new PIXI.Graphics()
      gPath.lineStyle(1, 0x000000, 1)
      gPath.drawPath(path)
      marqeeContainer.addChild(gPath)


      const circle_tween = PIXI.tweenManager.createTween(circleBg)
      let test = 10
      if (test) {
        that.rank = 4
        currentIndex = 4
        showScene2()
      }
      pivotSprite.on('pointerdown', () => { // 开始
        console.log('是否执行第二次了')
        if (!that.firstGuid && that.time1) {
          return
        } else {
          that.firstGuid = false
        }
        rounds = this.getRandomNum(2, 4)  // 圈数
        rotationTime = this.getRandomNum(1000, 4000)
        let n = Math.floor(Math.random() * 6)
        currentIndex = indexArr[n]
        degrees = degreeArr[n] // 角度
        let prizeDegree = 0 //4 3 2 1 0 5 
        console.log(currentIndex)
        
        isBegin = true
        let temp = startIndex // 先默认3个发光小球
        for (let i = 0; i < 3; i++) {
          lightArr1[temp < 0 ? 23 : temp].texture = yTexture
          that.setWidth(lightArr1[temp < 0 ? 23 : temp], 28, 28)
          --temp
        }
        clearTime = setInterval(marqeeRun,  (rotationTime - 500) / 24)
        getStart(circle_tween, rounds, degrees, rotationTime)
      })


      function getStart (tw, rounds, degrees, rotationTime) {
        tw.from({rotation: 0}).to({rotation: that.d2a(360 * rounds + degrees)})
        tw.easing = PIXI.tween.Easing.outCubic()
        tw.time = rotationTime
        tw.on('start', () => {
          marqeeContainer.removeChild(handObj.sprite)
          PIXI.tweenManager.removeTween(handObj.tween)
          console.log('I`m rotation')
        })
        
        tw.on('end', () => {
          isBegin = false
          console.log('end')
          let temp = startIndex // 先默认3个发光小球
          for (let i = 0; i < 3; i++) {
            lightArr1[temp < 0 ? 23 : temp].texture = rTexture
            that.setWidth(lightArr1[temp < 0 ? 23 : temp], 13, 13)
            --temp
          }
          console.log('%c ' + currentIndex, 'color: pink')
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
                tw.from({alpha: 1}).to({alpha: 0})
                tw.time = 800
                tw.repeat = 3
                tw.start()
              } 
            }
            if (!currentIndex) {
              that.time1 = ''
              return
            } else {
              that.time1 = new Date().getTime()
              that.rank = currentIndex
            }
            showScene2()
          }, 400)
        })
        tw.start()
      }
      
      app.ticker.add(delta => {
        PIXI.tweenManager.update()
        if (!isBegin) {
          clearInterval(clearTime)
        }
      })
     
      function marqeeRun() {// 小球运动
        if (isFirst) {
          beginndex = startIndex == 1 ? 1 : startIndex
          endIndex = startIndex == 1 ? 23 : startIndex - 1
          isFirst = false
        }
        if (beginndex < 0) {
          beginndex = 23
        }
        if (endIndex == 0) {
          endIndex = 0
          lightArr1[beginndex].texture = rTexture
          that.setWidth(lightArr1[beginndex], 13, 13)
          lightArr1[23].texture = yTexture
          that.setWidth(lightArr1[23], 28, 28)
          lightArr1[endIndex].texture = yTexture
          that.setWidth(lightArr1[endIndex], 28, 28)
          clearInterval(clearTime)
          return
        }
        lightArr1[beginndex].texture = rTexture
        that.setWidth(lightArr1[beginndex], 13, 13)
        lightArr1[endIndex - 1].texture = yTexture
        that.setWidth(lightArr1[endIndex - 1], 28, 28)
        lightArr1[endIndex].texture = yTexture
        that.setWidth(lightArr1[endIndex], 28, 28)
        endIndex--
        beginndex--
      }
      //场景二
      function showScene2 () {
        let goodSprite
        sceneContainer = new PIXI.Container()
        app.stage.addChild(sceneContainer)   
        goodSprite = new PIXI.Sprite.fromImage(configMarqee.planeList[currentIndex])
        goodSprite.anchor.set(0.5)
        goodSprite.width = prizeList[currentIndex].width
        goodSprite.height = prizeList[currentIndex].height
        goodSprite.x = tempX
        goodSprite.y = tempY
        goodSprite.parentGroup = group2
        sceneContainer.addChild(goodSprite)
        planeList[0] = goodSprite
        goodSprite.scale.set(0.22)
        const plane_tween = PIXI.tweenManager.createTween(goodSprite)
        plane_tween.from({y: tempY, scale: {x: 0.22, y: 0.22}}).to({y: tempY - 90, scale: {x: 0.4, y: 0.4}})
        plane_tween.easing = PIXI.tween.Easing.inOutBack()
        plane_tween.time = 800

        const plane_tween2 = PIXI.tweenManager.createTween(goodSprite)
        plane_tween2.from({y: tempY - 90}).to({y: tempY})
        plane_tween2.easing = PIXI.tween.Easing.inOutBack()
        plane_tween2.time = 500
        
        let bgSprite2 = new PIXI.Sprite.fromImage(configMarqee.bgUrl)
        bgSprite2.x = 0
        bgSprite2.y = 0
        bgSprite2.width = canvasW
        bgSprite2.height = canvasH
        sceneContainer.addChild(bgSprite2)
        bgSprite2.alpha = 0
        const bg_tween = PIXI.tweenManager.createTween(bgSprite2)
        bg_tween.delay = 1800
        bg_tween.from({alpha: 0}).to({alpha: 1})
        bg_tween.time = 3000
        bg_tween.easing = PIXI.tween.Easing.linear()
        bg_tween.start()
        bg_tween.chain(plane_tween).chain(plane_tween2)

             
       
        plane_tween.on('end', () => {
          drawImage(goodSprite)
        })
      }
      function drawImage (goodSprite) {
        let coin = new PIXI.Sprite.fromImage(configMarqee.coinUrl)
        coin.anchor.set(0)
        coin.width = 49
        coin.height = 47
        coin.x = 10
        coin.y = 10
        sceneContainer.addChild(coin)
        coinNum = currentIndex * 100
        coinSprite = new PIXI.Text(coinNum, {
          fontWeight: 'bold',
          fontSize: 40,
          fontFamily: 'Arial',
          fill: '0xE4A215'
        })
        coinSprite.x = 70
        coinSprite.y = 10
        sceneContainer.addChild(coinSprite)
        //logo. btn
        let logo = new PIXI.Sprite.fromImage(configMarqee.logoUrl)
        logo.anchor.set(0, 1)
        logo.width = 103
        logo.height = 73
        logo.y = canvasH - 15
        logo.x = 10
        sceneContainer.addChild(logo)
        let downBtn = new PIXI.Sprite.fromImage(configMarqee.downbtnUrl)
        downBtn.anchor.set(1, 1)
        downBtn.width = 150
        downBtn.height = 65
        downBtn.y = canvasH - 10
        downBtn.x = canvasW - 10
        sceneContainer.addChild(downBtn)

        let track = new PIXI.Sprite.fromImage(configMarqee.trackUrl)
        track.anchor.set(0.5)
        track.width = 330
        track.height = 560
        track.x = canvasW / 2
        track.y = canvasH / 2 - 20
        sceneContainer.addChild(track)

        path = new PIXI.tween.TweenPath()
        roundedRect(path, (canvasW - track.width) / 2, (canvasH - track.height)/ 2 - 20, 300, 520, 100)
        path.closed = true
  

        let keySprite = new PIXI.Sprite.fromImage(configMarqee.keyUrl)
        keySprite.anchor.set(1, 0.5)
        keySprite.width = 32
        keySprite.height = 32
        keySprite.x = canvasW - (canvasW - track.width) / 2
        keySprite.y = track.y
        sceneContainer.addChild(keySprite)

        for (let i = 0; i < 6; i++) {
          let rankIcon = new PIXI.Sprite(emptyIcon)
          rankIcon.anchor.set(0, 0)
          rankIcon.width = 32
          rankIcon.height = 30
          rankIcon.x = (canvasW - track.width) / 2
          trackIconList.push(rankIcon)
          rankIcon.y = track.y + 60 - 30* i //60
          sceneContainer.addChild(rankIcon)
        }
        // 降落块
        let slotContainer = new PIXI.Container()
        sceneContainer.addChild(slotContainer)
        for (let i = 0; i < 4; i++) {
          let slotSprite = new PIXI.Sprite.fromImage(configMarqee.slot)
          slotSprite.anchor.set(0.5)
          slotSprite.width = 100
          slotSprite.height = 70
          let x = (i % 2) * slotGap + (canvasW - slotGap) / 2
          let y = Math.floor(i / 2) * slotH +  280
          slotSprite.x = x
          slotSprite.y = y
          slotSprite.occpuied = false
          slotSprite.running = false
          slotList.push({x, y})
          slotContainer.addChild(slotSprite)
        }
        slotList[0].occpuied = true
        const plane_tween3 = PIXI.tweenManager.createTween(goodSprite)
        plane_tween3.time = 400
        plane_tween3.easing = PIXI.tween.Easing.inOutBack()
        plane_tween3.from({scale: {x: 0.4, y: 0.4}, x: tempX, y: tempY})
        plane_tween3.to({scale: {x: 0.22, y: 0.22}, x: slotList[0].x, y: slotList[0].y})
        plane_tween3.start()
        plane_tween3.on('end', () => {
          dropGift(1, that.rank)
        })
        
        
      }
      function dropGift (n, rank) {  // 掉箱子
        if (!n){
          let indexArr = slotList.filter((item, index) => {
            return !item.occpuied
          })
          if (!indexArr.length) {
            return
          }
          n = Math.floor(Math.random() * indexArr.length)
        }
        let yValue = slotList[n].y ? slotList[n].y - 6 : slotList[n].y + 6
        let gift = new PIXI.Sprite.fromImage(configMarqee.boxUrl)
        gift.anchor.set(0.5)
        gift.scale.set(0.9)
        gift.x = slotList[n].x
        sceneContainer.addChild(gift)
        gift.y = - canvasH / 2
        const gift_tween1 = PIXI.tweenManager.createTween(gift)
        gift_tween1.from({y: - canvasH / 2}).to({y:  yValue})
        gift_tween1.time = 500
        gift_tween1.easing = PIXI.tween.Easing.outBack()

        const gift_tween2 = PIXI.tweenManager.createTween(gift)
        gift_tween2.from({scale: {x: 0.9, y: 0.9}}).to({scale: {x: 1, y: 1}})
        gift_tween2.time = 400
        gift_tween2.repeat = 5
        gift_tween2.easing = PIXI.tween.Easing.inOutBack()
        gift_tween2.pingPong = true

        const gift_tween3 = PIXI.tweenManager.createTween(gift)
        gift_tween3.from({y: yValue}).to({y: yValue - 10})
        gift_tween3.time = 400
        gift_tween3.repeat = 3
        gift_tween3.easing = PIXI.tween.Easing.inBounce() 
        gift_tween3.pingPong = true
        gift_tween1.start()
        gift_tween1.chain(gift_tween2).chain(gift_tween3)
        gift_tween3.on('end', () => {
          sceneContainer.removeChild(gift)
          createPlane(rank)
          spillAction(100, 100, slotList[n].x, slotList[n].y, [configMarqee.chipUrl, configMarqee.starUrl], configMarqee.composeEmitterConfig)
          if (that.step == 1) { // 指引合并 --- 指引到跑道 --- 指引返回
            handObj2 = that.createHandTween(slotList[n].x, slotList[n].y + 20, sceneContainer)
            handObj2.sprite.parentGroup = group3
            handObj2.tween.from({x: slotList[n].x}).to({x: slotList[0].x})
            handObj2.tween.time = 1600
            handObj2.tween.start()
          }
        })
      }
      function createPlane (r, i = 1) {
        let plane = new PIXI.Sprite.fromImage(configMarqee.planeList[r])
        plane.anchor.set(0.5, 0.5)
        plane.scale.set(0.22)
        plane.pName = Math.random().toString(36).substr(2)
        plane.pRank = r
        plane.pIndex = i
        plane.x = slotList[i].x
        plane.y = slotList[i].y
        sceneContainer.addChild(plane)
        slotList[i].rank = r + 1
        slotList[i].occpuied = true
        planeList[i] = plane
        plane.interactive = true
        plane.buttonMode = true
        if (that.step == 2) {
          handObj2 = that.createHandTween(slotList[i].x, slotList[i].y + 12, sceneContainer)
          handObj2.sprite.parentGroup = group3
          handObj2.tween.from({x: slotList[i].x, y: slotList[i].y + 12}).to({x: trackIconList[0].x, y: trackIconList[0].y})
          handObj2.tween.time = 1600
          handObj2.tween.start()
        } else if (that.step == 3) {
          // handObj2 = that.createHandTween(slotList[0].x, slotList[0].y + 20, sceneContainer)
          // handObj2.tween.start()
        }
        play(plane)
      }
      function roundedRect(path,x,y,width,height,r){  // 绘制圆角矩形
        path.moveTo(x + r, y);
        path.lineTo(x + width - r, y);
        path.arc(x + width - r, y + r, r, that.d2a(270), 0, false);
        path.lineTo(x + width, y + height - r);
        path.arc(x + width - r, y + height - r, r, 0, that.d2a(90), 0, false);
        path.lineTo(x + r, y + height);
        path.arc(x + r, y + height - r, r, that.d2a(90), that.d2a(180), false);
        path.lineTo(x, y + r);
        path.arc(x + r, y + r, r, that.d2a(180), that.d2a(270), false);
        path.closed = true
      }
      function play (item) { // 合成
        item.on('pointerdown', onDragStart)
        item.on('pointerup', onDragEnd)
        item.on('pointerupoutside', onDragEnd)
        item.on('pointermove', onDragMove)

        function onDragStart (event) {
          this.data = event.data
          
            // if (that.step == 2) {
            //   //return
            // } else {
              this.alpha = 0.5
              this.dragging = true
              that.oldPosition.x = this.x
              that.oldPosition.y = this.y
            //}
          //}
        }
        function onDragEnd (event) {
          this.alpha = 1
          let moveI = this.pIndex, rank = this.pRank
          let endI = findCloser(this.x, this.y, moveI)
          if (endI > -1) {  // 有合成
            if (that.step == 1) {
              sceneContainer.removeChild(handObj2.sprite)
              that.step++
            }
            sceneContainer.removeChild(planeList[moveI])
            planeList[endI].destroy()
            planeList[moveI] = {}
            planeList[endI] = {}
            slotList[moveI].rank = 0
            slotList[moveI].occpuied = false
            spillAction(100, 100, slotList[endI].x, slotList[endI].y, [configMarqee.lightCircle, configMarqee.starUrl], configMarqee.lightEmitterConfig)
            createPlane(this.pRank + 1, endI)
          } else {
            if (Math.abs(this.x - trackIconList[0].x) < 60) { // 第二步
              // 跑
              if (that.step == 2) {
                sceneContainer.removeChild(handObj2.sprite)
              }
              this.x = trackIconList[0].x + 12
              addMoney (moveI, rank)
              if (that.step == 2) {
                dropGift(1, rank)
                that.step++
              }
            }
            if (that.step > 3) {
              this.x = that.oldPosition.x
              this.y = that.oldPosition.y
            }
          }
        }
        function onDragMove (event) {
          if (this.dragging && that.step != 3) {
            let newPosition = this.data.getLocalPosition(this.parent)
            this.x = newPosition.x
            this.y = newPosition.y
          }
        }
       
        function findCloser (x, y, i) {
          let closerI = slotList.findIndex((item, index) => {
            return i != index && Math.sqrt(Math.pow(Math.abs(item.x - x), 2) + Math.pow(Math.abs(item.y - y), 2)) < 40
          })
          return closerI
        }
      }

      function addMoney (moveI, rank) {
        trackIconList[0].texture = fullIcon
        planeTemp = new PIXI.Sprite.fromImage(configMarqee.planeList[rank])
        planeTemp.anchor.set(0.5)
        planeTemp.alpha = 0.5
        planeTemp.pRank = rank
        planeTemp.interactive = true
        planeTemp.buttonMode = true
        planeTemp.x = slotList[moveI].x
        planeTemp.y = slotList[moveI].y
        slotList[moveI].running = true
        planeTemp.width = planeList[moveI].width
        planeTemp.height = planeList[moveI].height
        sceneContainer.addChild(planeTemp)
        let arrow = new PIXI.Sprite.fromImage(configMarqee.occpuiedUrl)
        arrow.width = 24
        arrow.height = 24
        arrow.anchor.set(1, 1)
        arrow.x = slotList[moveI].x + planeTemp.width / 2
        arrow.y = slotList[moveI].y + planeTemp.height / 2
        sceneContainer.addChild(arrow)
        const tw = PIXI.tweenManager.createTween(planeList[moveI])
        tw.path = path
        tw.time = 3000
        tw.easing = PIXI.tween.Easing.linear()
        tw.loop = true
        tw.start()
        tw.on('start', () => {
          runningPlanes.push(planeList[moveI])
        })
        app.ticker.add(delta => {
          runningPlanes.forEach(item => {
            if (Math.abs(item.x - 372) <= 40 && Math.abs(item.y - 348) <= 10) {
              spillAction(100, 100, 372, 348, [configMarqee.coinUrl], configMarqee.coinEmitterConfig)
              coinNum += that.addArr[rank - 1]
              coinSprite.text = coinNum
            }
          })
        })
        planeTemp.on('pointerStart', (event) => {
          if (slotList[this.pIndex].running) {
            let i = runningPlanes.findIndex(item => {
              return item.pRank
            })
            if (i > -1) {
              if (that.step == 3) {
                handObj2.sprite.destroy()
              }
              const tw = PIXI.tweenManager.createTween(planeTemp)
              let x = planeTemp.x, y = planeTemp.y
              tw.from({
                x: x,
                y: y
              })
              tw.to({
                x: slotList[0].x,
                y: slotList[0].y
              })
              tw.time = 800
              tw.easing = PIXI.tween.Easing.linear()
              tw.start()
            } else {
            }
          }
        })

      }
       /**
       * 粒子动画
       * @param w 容器宽度
       * @param h 容器高
       * @param x 容器坐标x
       * @param y 容器坐标y
       * @param imgArr 粒子动画所有图片
       * @param config 粒子动画config
       */
      function spillAction (w = 100, h = 100, x, y, imgArr, config) {
        let coinTank = new PIXI.Container();
        coinTank.width = w;
        coinTank.height = h;
        coinTank.x = x;
        coinTank.y = y;
        sceneContainer.addChild(coinTank);
        // Create a new emitter
        let emitter = new PIXI.particles.Emitter(
          coinTank,
          [...imgArr.map(v => PIXI.Texture.fromImage(v))],
          config
        )
        // Start emitting
        emitter.emit = true;
        emitter.playOnceAndDestroy(() => {
          app.stage.removeChild(coinTank)
        })
      }
    },
    d2a(n) { //度数转弧数
      return n * Math.PI / 180;
    },
    createHandTween (x, y, parentCont, propeArr) { // 创建手
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
    },
    getPosition (x, y) {
      let x1, y1
      if (x < 0) {
        x1 = x + 20
      } else {

      }
    }
  }
}
</script>
