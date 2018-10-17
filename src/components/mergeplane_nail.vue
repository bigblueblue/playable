<template>
  <div id="mergeplaneNail">
  </div>
</template>
<script>
/* eslint-disable */
import configMarqee from '../assets/js/nail_config'
import * as PIXI from 'pixi.js'
import tweenManager from 'pixi-tween'
import FontFaceObserver from 'fontfaceobserver'
import particles from 'pixi-particles'
import '../assets/js/pixi-display'
// import 'pixi-svg'
export default{
  name: 'mergeplaneNail',
  data () {
    return {
      flag: true,
      iosLink: 'https://itunes.apple.com/app/id1363863879',
      androidLink: 'https://play.google.com/store/apps/details?id=com.brokenreality.planemerger.android',
      isLandscape: false,
      firstGuid: true,
      step: 1,
      time1: null,
      rank: 1,
      isMarquee: false, //为true表示切屏在转盘阶段
      oldPosition: {x: 0, y: 0},
      addArr: [20, 40, 60, 80, 100, 120, 140, 160, 180, 200],
      total: -1, // 跑道傻姑娘的总数
      isFirst: false,
      cleartimer: null,
      slotList: [{}, {}, {}, {}],
      planeList: [],
      isRun: false,
      durationTime: 3000,
      hammmerRun: true,
      isEnglish: true,
      toggleFlag: true, // 切屏的时候step为3时
      status: 0 // 0 表示转盘 1 表示跑道 2表示到成功页面了
    }
  },
  created () {
    // 解决font-face问题
    let font = new FontFaceObserver('NumFont')
    font.load().then(function () {
      console.log('has loaded')
    })
  },
  mounted () {
    this.getSize()
    this.initPIXI()
  },
  computed: {
    plane_location: function () {
      let m = 0
      if (this.step == 2 || this.step == 3) {
        m = this.slotList.findIndex(item => {
          return item.rank
        })
        m = m ? 0 : 1
      }
      return m
    }
  },
  watch: {
  },
  methods: {
    getSize () {
      let that = this
      if (window.innerWidth <= window.innerHeight) {
        this.isLandscape = false
      } else {
        this.isLandscape = true
      }
      window.addEventListener('resize', onResize, false)
      function onResize () {
        let canvas = document.querySelector('#mergeplaneNail canvas')
        canvas.parentNode.removeChild(canvas)
        if (window.innerWidth > window.innerHeight) {
          that.isLandscape = true
        } else {
          that.isLandscape = false
        }
        clearInterval(that.cleartimer)
        // console.log(that.status)
        if (!that.status) {
          that.isMarquee = true
          console.log('切换了')
          that.hammmerRun = true
          that.initPIXI()
        } else if (that.status == 1) {
          that.isMarquee = false
          clearInterval(that.cleartimer)
          that.firstGuid = true
          that.timer = null
          that.toggleFlag = false
          that.initPIXI()
        } else if (that.status == 2) {
          that.initPIXI()
        }
        
      }
    },
    initPIXI () {
      var canvasW, that = this, topH, canvasH, slotH, slotGap
      let startY
        canvasW = this.isLandscape ? 736 : 414
        canvasH = this.isLandscape ? 414 : 736
        slotH = this.isLandscape ? 100 : 140
        slotGap = this.isLandscape ? 200 : 120
        startY = this.isLandscape ? 42 : 84
        topH = this.isLandscape ? 130 : 270
      const trackW = that.isLandscape ? 560 : 330
      const trackH = that.isLandscape ? 330 : 560
      let airportW = trackW - 30
      let airportH = trackH - 34
      let airportR = 100
      const r = 300
      let startX = canvasW / 2
      const xCoord = [startX - airportW / 2, startX - airportW / 2 + airportR, airportW / 2 - airportR + startX, airportW / 2 + startX];
      //从上到下
      const yCoord = [startY, startY + airportR, airportH + startY - airportR, airportH + startY];
      const winY = startY + airportH / 2
      const nailW = that.isLandscape ? 30 : 38
      const nailH = that.isLandscape ? 110 : 140
      const closeH = that.isLandscape ? 36 : 50
      const outRadius = 180 // 转盘外半径
      const inRadius = 124 // 转盘内半径
      const spinCopies = 6
      const lightDegrees = -180, startDegrees = 0
      var app, planeList = new Array(4).fill({}), handObj2, slotList = [], sceneContainer,  path, keySprite
      var emptyIcon = PIXI.Texture.fromImage(configMarqee.rankEmptyUrl), coinNum = 0, coinSprite
      var fullIcon = PIXI.Texture.fromImage(configMarqee.rankFillUrl), runningPlanes = [] , planeTemp, trackIconList = [] //new Array(6).fill({})
      var tweenList = [], trackContainer
      var nailList = [], blFrameList = [], hammer, noNail, newTexture, hidden, giftList = [], renderTexture
      var beginY // 锤子初始y
      const leanDegree = -45
      let canvas
      // app = new PIXI.Application({
      //   width: canvasW,
      //   height: canvasH,
      //   forceCanvas: true
      // })
      app = new PIXI.Application(canvasW, canvasH)
      document.getElementById('mergeplaneNail').appendChild(app.view)
      canvas = document.querySelector('#mergePlane canvas')
      if (document.body.clientWidth >= canvasW * document.body.clientHeight / canvasH) {
        app.renderer.view.style.height = '100%'
        app.renderer.view.style.width = canvasW * document.body.clientHeight / canvasH
      } else {
        app.renderer.view.style.height = canvasH * document.body.clientHeight / canvasW
        app.renderer.view.style.width = '100%'
      }
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
      let prizeList = []
      let startIndex = 1, endIndex, rounds, degrees, rotationTime, isBegin = false, clearTime
      let isFirst = true, beginndex
      const rotaDegreeList = [0, 180, 300, 360, 60]
      let addY = that.isLandscape ? 0 : 60
      const tempX = canvasW / 2 - Math.cos(this.d2a(startDegrees + 4 * 60 + 30)) * (inRadius - 32) + 1.208
      const tempY =  canvasH / 2 - Math.sin(this.d2a(startDegrees + 4 * 60 + 30)) * (inRadius - 32) - inRadius - 48 + addY// 第一个奖品的位置
      
      // 背景等静态图片
      let bgSprite = new PIXI.Sprite.fromImage(that.isLandscape ? configMarqee.bgUrl2 : configMarqee.bgUrl)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = canvasW
      bgSprite.height = canvasH
      app.stage.addChild(bgSprite)
      let marqeeContainer = new PIXI.Container()
      app.stage.addChild(marqeeContainer)
      let desk= new PIXI.Sprite.fromImage(configMarqee.deskUrl)
      desk.anchor.set(0, 1)
      desk.x = 0
      desk.y = canvasH
      desk.width = canvasW
      desk.height = that.isLandscape ? 170 : 237
      marqeeContainer.addChild(desk)
      let brush = new PIXI.Graphics()
      brush.beginFill(0xffffff);
      brush.drawCircle(0, 0, 20);
      brush.endFill(); 

      let logoObj = that.isEnglish ? configMarqee.logoUrl : configMarqee['logoUrl_cn']
      let logo = new PIXI.Sprite.fromImage(logoObj)
      logo.anchor.set(0.5)
      logo.width = that.isEnglish ? 103 : 123
      logo.height = 73
      logo.x = that.isLandscape ? 60 : canvasW / 2
      logo.y = that.isLandscape ? 50 : 60
      marqeeContainer.addChild(logo)
      let downObj = that.isEnglish ? configMarqee.downBtnUrl : configMarqee['downBtnUrl_cn']
      let downBtn = new PIXI.Sprite.fromImage(downObj)
      downBtn.anchor.set(0.5)
      downBtn.width = 150 
      downBtn.height = that.isEnglish ? 65 : 45
      downBtn.x = that.isLandscape ? canvasW - 80 : canvasW / 2
      downBtn.y = that.isLandscape ? 55 : 136
      downBtn.interactive = true
      marqeeContainer.addChild(downBtn)
      downBtn.on('pointerdown', () => {
        that.linkAppStore()
      })
      var textureList1 = [], textureList2 = [],  currentIndex =  -1, anim //
      for (let i = 0; i < 5; i++) { // 轨道上的飞机
        let text1 = PIXI.Texture.fromImage(configMarqee.highPlaneList[i])
        let text2 = PIXI.Texture.fromImage(configMarqee.highTrackList[i])
        textureList1.push(text1)
        textureList2.push(text2)
      }
      layThings()

      app.stage.interactive = true
      app.buttonMode = true
      app.stage.on('pointerdown', () => { // 锤下去
        that.hammmerRun = false
        driveNail()
      })
      let eqFlag = false, maxX = that.isLandscape ? canvasW : canvasW + 100, minX = that.isLandscape ? 0 : 120
      let nailRun = false, nailCount = 0, nailFlag = true // 钉子是否可以动画
      let frameFlag = false, lightBoomFlag = 0
      
      

      function layThings () { // 布置钉子奖品等
        noNail = new PIXI.Texture.from(configMarqee.nailUrl)  // 礼物盒遮罩
        noNail.baseTexture.width = nailW // 设置值了就不会报错 frame 
        noNail.baseTexture.height = nailH
        console.log(noNail)
        let rect = new PIXI.Rectangle(0, 0, nailW, nailH)
        noNail.frame = rect
        newTexture = new PIXI.Texture(noNail.baseTexture, noNail.frame);
        let blFrameTexture = PIXI.Texture.fromImage(configMarqee.blackUrl) //飞机外层的黑框
        for (let m = 0; m < 3; m++) { // 钉子
          let nail = new PIXI.Sprite.from(newTexture)
          nail.x = (canvasW / 6) * (2 * m + 1) - nailW / 2// 1 3 5
          nail.y = desk.y - desk.height + closeH - nailH
          nailList.push(nail)
          marqeeContainer.addChild(nail)
          console.log(nail.x, nail.y, nail.anchor)
          let blFrame = new PIXI.Sprite(blFrameTexture)
          blFrame.width = that.isLandscape ? 100 : 112
          blFrame.height = that.isLandscape ? 110 : 133
          blFrame.anchor.set(0.5, 0)
          blFrame.x = nail.x + nailW / 2
          blFrame.y = that.isLandscape ? canvasH - blFrame.height : canvasH - blFrame.height - 10
          blFrameList.push(blFrame)
          marqeeContainer.addChild(blFrame)
          let plane = new PIXI.Sprite.fromImage(configMarqee.goodsList[m])
          plane.scale.set(0.9)
          plane.anchor.set(0.5)
          plane.x = 0
          plane.y = that.isLandscape ? blFrame.height / 2 + 20 : blFrame.height / 2 + 8
          blFrame.addChild(plane)
          giftList.push(plane)
        }
        // 锤子
        hammer = new PIXI.Sprite.fromImage(configMarqee.hammerUrl) 
        hammer.anchor.set(0.5, 1)
        hammer.x = canvasW - 100
        beginY = that.isLandscape ? nailList[0].y - 40 : nailList[0].y - 50
        hammer.y = beginY
        hammer.width = that.isLandscape ? 75 : 90
        hammer.height = that.isLandscape ? 130 : 156
        hammer.rotation = that.d2a(leanDegree)
        marqeeContainer.addChild(hammer)
      }
      function driveNail () {
        // let baseX = 0, baseY = 0
        let path = new PIXI.tween.TweenPath()
        path.arc(hammer.x, hammer.y, hammer.height, that.d2a(180 + 45), that.d2a(360 - 90 - 15),  false)
        path.arc(hammer.x, hammer.y, hammer.height, that.d2a(360 - 90 - 15), that.d2a(180),  true)
        path.lineTo(hammer.x - hammer.height, beginY + 140)
        var gPath = new PIXI.Graphics();
        gPath.lineStyle(10, 0xffffff, 1);
        gPath.drawPath(path);
        // marqeeContainer.addChild(gPath);
        const hammer_tw1 = PIXI.tweenManager.createTween(hammer)
        hammer_tw1.from({rotation: that.d2a(leanDegree)}).to({rotation: that.d2a(leanDegree + 30)})
        hammer_tw1.time = 400
        hammer_tw1.easing = PIXI.tween.Easing.outBack()
        currentIndex = nailList.findIndex(item => { // 是否有锤子钉子的
          return Math.abs(item.x - (hammer.x - hammer.height)) <= 28
        })
        that.rank = currentIndex
        const hammer_tw2 = PIXI.tweenManager.createTween(hammer)
        hammer_tw2.from({
          y: beginY,
          rotation: that.d2a(leanDegree + 30)
        }).to({
          y: beginY + nailH,
          rotation: that.d2a(-90)
        })
        hammer_tw2.time = 1000
        hammer_tw2.easing = PIXI.tween.Easing.outBack()
        hammer_tw2.pingPong = true
        hammer_tw1.start()
        hammer_tw1.chain(hammer_tw2)
        console.log('开始锤啦', currentIndex)
        if (currentIndex > -1) {
          var pathRect = new PIXI.tween.TweenPath(), r = 0
          let minW = that.isLandscape ? 6 : 7, minH = that.isLandscape ? 26 : 28
          let coord = {
            x: blFrameList[currentIndex].x, 
            y: blFrameList[currentIndex].y + minH,
            width: that.isLandscape ? 84 : 94,
            height: that.isLandscape ? 80 : 92
          }
          pathRect.moveTo(coord.x, coord.y - minH)
          pathRect.lineTo(coord.x - minW, coord.y)
          pathRect.lineTo(coord.x - (coord.width / 2 - r), coord.y)

          pathRect.lineTo(coord.x - (coord.width / 2 - r), coord.y + (coord.height - r))
          pathRect.lineTo(coord.x + (coord.width / 2 - r), coord.y + coord.height)
          pathRect.lineTo(coord.x + coord.width / 2, coord.y)
          pathRect.lineTo(coord.x + minW, coord.y)
          pathRect.lineTo(coord.x, coord.y - minH)
          var gPath1 = new PIXI.Graphics();
          gPath1.lineStyle(1, 0xffffff, 1);
          gPath1.drawPath(pathRect);
          // marqeeContainer.addChild(gPath1)
          var lightFrame = new PIXI.Sprite.fromImage(configMarqee.lightUrl)
          marqeeContainer.addChild(lightFrame)
          lightFrame.width = blFrameList[currentIndex].width
          lightFrame.height = blFrameList[currentIndex].height
          lightFrame.anchor.set(0.5, 0)
          lightFrame.x = blFrameList[currentIndex].x
          lightFrame.y = blFrameList[currentIndex].y
          renderTexture = PIXI.RenderTexture.create(canvasW, canvasH)
          var rendererTextureSprite = new PIXI.Sprite(renderTexture)
          lightFrame.mask = rendererTextureSprite

          hidden = new PIXI.Sprite(emptyIcon)
          hidden.anchor.set(0.5)
          hidden.scale.set(0)
          hidden.x = lightFrame.x
          hidden.y = lightFrame.y
          marqeeContainer.addChild(hidden)
        }
        hammer_tw2.on('end', () => {
          if (currentIndex == -1) {
            const hammer_tw3 = PIXI.tweenManager.createTween(hammer)
            hammer_tw3.from({rotation: that.d2a(leanDegree + 30)}).to({rotation: that.d2a(leanDegree)})
            hammer_tw3.time = 200
            hammer_tw3.easing = PIXI.tween.Easing.linear()
            hammer_tw3.on('end', () => {
              that.hammmerRun = true
            })
            hammer_tw3.start()
          } else {
            frameFlag = true
            const light_tw = PIXI.tweenManager.createTween(hidden)
            light_tw.path = pathRect
            light_tw.time = 800
            light_tw.easing = PIXI.tween.Easing.linear()
            const light_tw2 = PIXI.tweenManager.createTween(lightFrame)
            light_tw2.from({alpha: 1}).to({alpha: 0.3})
            light_tw2.time = 200
            light_tw2.repeat = 3
            light_tw2.pingPong = true
            light_tw.start()
            light_tw.chain(light_tw2)
            light_tw2.on('end', () => {
              getNewPlane()
            })
          }
        })

      }
      function nailAnimate (i, h){  // 钉子动画
        noNail = new PIXI.Texture.from(configMarqee.nailUrl)  // 礼物盒遮罩
        let rect1 = new PIXI.Rectangle(0, 0, nailW, h)
        noNail.frame = rect1
        newTexture = new PIXI.Texture(noNail.baseTexture, noNail.frame);
        nailList[i].texture = newTexture
      }
      function knockAnimate () {
        let textureArray = []
        for (let i = 0; i < 8; i++) {
          let texture = PIXI.Texture.fromImage(configMarqee.knockLight[i])
          textureArray.push(texture)
        }
        anim = new PIXI.extras.AnimatedSprite(textureArray)
        anim.x = that.isLandscape ? nailList[currentIndex].x + 16 : nailList[currentIndex].x + 16
        anim.y = that.isLandscape ? beginY + 140 : beginY + 180
        anim.anchor.set(0.5)
        anim.scale.set(0.5)
        anim.animationSpeed = 0.2;
        anim.play();
        anim.loop = false
        marqeeContainer.addChild(anim);
        anim.onComplete = function () {
          anim.stop()
          marqeeContainer.removeChild(anim)
        }
        lightBoomFlag = 2
      }
      app.ticker.add(delta => {
        PIXI.tweenManager.update()
        if (nailCount >= nailH - 9) {
          nailRun = false
        }
        if (that.hammmerRun) { // 锤子挪动
          if (eqFlag) { //300 ------- 100 ------ 300 默认减少
            if (hammer.x >= maxX) {
              eqFlag = false
            } else {
              hammer.x += delta * 3
            }
          } else {
            if (hammer.x <= minX) {
              eqFlag = true
            } else {
              hammer.x -= delta * 3
            }
          }
        }
        if (nailFlag) { // 钉子动画
          if (currentIndex > -1) {
            if (hammer.y >= nailList[currentIndex].y) {
              const nail_tw = PIXI.tweenManager.createTween(nailList[currentIndex])
              let y1 = nailList[currentIndex].y
              nail_tw.from({y: y1 }).to({y: y1 + nailH - 9})
              nail_tw.time = 500
              nail_tw.easing = PIXI.tween.Easing.outBack()
              nail_tw.start()
              nail_tw.on('start', () => {
                nailFlag = false
              })
              nail_tw.on('end', () => {
                lightBoomFlag = 1
              })
            }
          }
        } else { // 钉子变短
          nailCount += 8 
          if (nailCount >= nailH - 9) {
            nailCount = nailH - 9
          }
          nailAnimate(currentIndex, nailH - nailCount)
        }
        if (lightBoomFlag == 1) { //爆炸
          knockAnimate()
        }
        if (frameFlag) { //  飞机外层黑框
          brush.position.copy(hidden)
          app.renderer.render(brush, renderTexture, false, null, false)
        }
      })

      if (that.status == 1) {
        currentIndex = that.rank
        getNewPlane()
        return
      } else if (that.status == 2) {
        getSuccess()
        return
      }

    
       /* test
      marqeeContainer.visible = false
      currentIndex = 2
      that.rank = currentIndex
      that.status = 1
      // getSuccess()
      getNewPlane()
      */
      function getNewPlane () {
        let goodSprite
        sceneContainer = new PIXI.Container()
        app.stage.addChild(sceneContainer)
        trackContainer = new PIXI.Container()
        sceneContainer.addChild(trackContainer)
        goodSprite = new PIXI.Sprite(textureList1[that.rank])
        goodSprite.anchor.set(0.5)
        let pos = giftList[currentIndex].getGlobalPosition()
        app.stage.addChild(goodSprite)
        goodSprite.scale.set(0.4)
        goodSprite.x = pos.x
        goodSprite.y = pos.y

        goodSprite.interactive = true
        goodSprite.pName = Math.random().toString(36).substr(2)
        goodSprite.pRank = that.rank
        goodSprite.pIndex = 0
        planeList[0] = goodSprite
        that.$set(that.planeList, 0, goodSprite)
       
        if (that.status == 1 && !that.isMarquee) {
          showScene2(goodSprite)
          return
        }
        setTimeout(() => {
          showScene2(goodSprite)
        }, 100)
      }
      //场景二
      function showScene2 (goodSprite) {
        if (that.status == 1 && that.step >= 2) {
          app.stage.removeChild(goodSprite)
          let bgSprite2 = new PIXI.Sprite.fromImage(that.isLandscape ? configMarqee.bgUrl2 : configMarqee.bgUrl)
          bgSprite2.x = 0
          bgSprite2.y = 0
          bgSprite2.width = canvasW
          bgSprite2.height = canvasH
          // bgSprite2.alpha = 0
          // sceneContainer.parentGroup = group1
          sceneContainer.addChild(bgSprite2)
          drawImage()
          return
        }
        const plane_tween = PIXI.tweenManager.createTween(goodSprite)
        plane_tween.from({y: tempY, scale: {x: 0.22, y: 0.22}}).to({y: tempY - (that.isLandscape ? 40 : 90), scale: {x: 0.48, y: 0.48}})
        plane_tween.easing = PIXI.tween.Easing.linear()
        plane_tween.time = 300

        const plane_tween2 = PIXI.tweenManager.createTween(goodSprite)
        plane_tween2.from({y: tempY - (that.isLandscape ? 60 : 90)}).to({y: tempY})
        plane_tween2.easing = PIXI.tween.Easing.inOutBack()
        //plane_tween2.delay = 200
        plane_tween2.time = 500
        
        let bgSprite2 = new PIXI.Sprite.fromImage(that.isLandscape ? configMarqee.bgUrl2 : configMarqee.bgUrl)
        bgSprite2.x = 0
        bgSprite2.y = 0
        bgSprite2.width = canvasW
        bgSprite2.height = canvasH
        sceneContainer.addChild(bgSprite2)
        bgSprite2.alpha = 0
        const bg_tween = PIXI.tweenManager.createTween(bgSprite2)
        bg_tween.delay = 100
        bg_tween.from({alpha: 0}).to({alpha: 1})
        bg_tween.time = 200
        bg_tween.easing = PIXI.tween.Easing.linear()
        if (that.status == 1 && !that.isMarquee) { // 表示切屏过来的
          if (that.step == 2) {
            goodSprite.alpha = 0
          }
          // goodSprite.scale.set()
          bgSprite2.alpha = 1
          drawImage(goodSprite)
          return
        } else {
          bg_tween.start()
          bg_tween.chain(plane_tween).chain(plane_tween2)
        }
        plane_tween.on('end', () => {
          play(goodSprite)
          drawImage(goodSprite)
        })
      }
      function drawImage (goodSprite) {
        let coin = new PIXI.Sprite.fromImage(configMarqee.coinUrl)
        coin.anchor.set(0)
        coin.width = that.isLandscape ? 40 :49
        coin.height = that.isLandscape ? 38 : 47
        coin.x = 10
        coin.y = 10
        sceneContainer.addChild(coin)
        coinNum = currentIndex * 100
        coinSprite = new PIXI.Text(coinNum, {
          fontWeight: 'bold',
          fontSize: that.isLandscape ? 36 : 40,
          fontFamily: 'Arial',
          fill: '0xE26C33'
        })
        coinSprite.x = that.isLandscape ? 60 : 70
        coinSprite.y = 10
        sceneContainer.addChild(coinSprite)
        //logo. btn
        let logoObj = that.isEnglish ? configMarqee.logoUrl : configMarqee['logoUrl_cn']
        let logo = new PIXI.Sprite.fromImage(logoObj)
        logo.anchor.set(0, 1)
        logo.width = that.isEnglish ? 103 : 123
        logo.height = 73
        logo.y = canvasH - 15
        logo.x = 10
        sceneContainer.addChild(logo)
        let downObj = that.isEnglish ? configMarqee.downBtnUrl : configMarqee['downBtnUrl_cn']
        let downBtn = new PIXI.Sprite.fromImage(downObj)
        downBtn.width = 150 
        downBtn.height = that.isEnglish ? 65 : 45
        downBtn.anchor.set(1, 1)
        downBtn.y = canvasH - 10
        downBtn.x = canvasW - 10
        downBtn.interactive = true
        sceneContainer.addChild(downBtn)
        downBtn.on('pointerdown', () => {
          that.linkAppStore()
        })
        let track = new PIXI.Sprite.fromImage(that.isLandscape ? configMarqee.trackUrl2 :  configMarqee.trackUrl)
        track.anchor.set(0.5)
        track.width = trackW
        track.height = trackH
        track.x = canvasW / 2
        track.y = canvasH / 2 - 20
        sceneContainer.addChild(track)

        path = new PIXI.tween.TweenPath()
        path.moveTo(xCoord[0], winY);
        path.lineTo(xCoord[0], winY);
        path.arc(xCoord[1], yCoord[1], airportR, -Math.PI, -Math.PI / 2, false);
        path.lineTo(xCoord[2], yCoord[0]);
        path.arc(xCoord[2], yCoord[1], airportR, -Math.PI / 2, 0, false);
        path.lineTo(xCoord[3], winY);
        path.lineTo(xCoord[3], yCoord[2]);
        path.arc(xCoord[2], yCoord[2], airportR, 0, -3 * Math.PI / 2, false);
        path.lineTo(xCoord[1], yCoord[3]);
        path.arc(xCoord[1], yCoord[2], airportR, -3 * Math.PI / 2, -Math.PI, false);
        path.lineTo(xCoord[0], winY);
        path.closed = true
        
        let gPath = new PIXI.Graphics();
        gPath.lineStyle(9, 0xfffffff, 1);
        gPath.drawPath(path);
        // sceneContainer.addChild(gPath);

        keySprite = new PIXI.Sprite.fromImage(configMarqee.keyUrl)
        keySprite.anchor.set(0.5)
        keySprite.scale.set(0.8)
        keySprite.width = 32
        keySprite.height = 32
        keySprite.x = canvasW - (canvasW - track.width) / 2 - 15
        keySprite.y = track.y
        sceneContainer.addChild(keySprite)

        for (let i = 0; i < 4; i++) {
          let rankIcon = new PIXI.Sprite(emptyIcon)
          rankIcon.anchor.set(0, 0)
          rankIcon.width = that.isLandscape ? 24 : 32
          rankIcon.height = that.isLandscape ? 20 :30
          rankIcon.x = (canvasW - track.width) / 2
          trackIconList.push(rankIcon)
          if (that.isLandscape) {
            rankIcon.x = (canvasW - track.width) / 2 + 4
            rankIcon.y = track.y + 20 - 20* i //60
          } else {
            rankIcon.y = track.y - 30* i //60
          }
          
          sceneContainer.addChild(rankIcon)
        }
        // 降落块
        let slotContainer = new PIXI.Container()
        sceneContainer.addChild(slotContainer)
        for (let i = 0; i < 4; i++) {
          let slotSprite = new PIXI.Sprite.fromImage(configMarqee.slot)
          slotSprite.anchor.set(0.5)
          slotSprite.width = 88
          slotSprite.height = 64
          let x = (i % 2) * slotGap + (canvasW - slotGap) / 2
          let y = Math.floor(i / 2) * slotH + topH
          let rank = 0, occpuied = false, running = false
          slotSprite.x = x
          slotSprite.y = y
          if (that.step >= 2 && that.status == 1) {
            that.$set(that.slotList[i], 'x', x)
            that.$set(that.slotList[i], 'y', y)
            rank = that.slotList[i].rank
            occpuied = that.slotList[i].occpuied
            running = that.slotList[i].running
          } else {
            slotSprite.rank = rank
            slotSprite.occpuied = occpuied
            slotSprite.running = running
            that.$set(that.slotList, i, {x, y, rank, occpuied, running})
          }
          slotList.push({x, y, rank, occpuied, running})
          slotContainer.addChild(slotSprite)
        }
       
        
        const plane_tween3 = PIXI.tweenManager.createTween(goodSprite)
        plane_tween3.time = 400
        plane_tween3.easing = PIXI.tween.Easing.inOutBack()
        plane_tween3.from({scale: {x: 0.4, y: 0.4}, x: tempX, y: tempY})
        plane_tween3.to({scale: {x: 0.26, y: 0.26}, x: slotList[0].x, y: slotList[0].y})
        if (that.status == 1 && !that.isMarquee) {
          if (that.step == 1){ 
            goodSprite.scale.set(0.26)
            slotList[0].rank = that.rank
            slotList[0].occpuied = true
            that.$set(that.slotList[0], 'rank', that.rank)
            that.$set(that.slotList[0], 'occpuied', true)
            goodSprite.x = slotList[0].x
            goodSprite.y = slotList[0].y
            if (slotList[0].occpuied) {
              dropGift(1, that.rank)
            } else {
              dropGift(0, that.rank)
            }
            play(goodSprite)
          } else if (that.step == 2) {
            createPlane(that.slotList[that.plane_location ? 0 : 1].rank, that.plane_location ? 0 : 1)
          } else if (that.step == 3) {
            that.total = -1
            that.isRun = true
            createPlane(that.slotList[that.plane_location ? 0 : 1].rank, that.plane_location ? 0 : 1)
            that.toggleFlag = true
            that.isRun = false
            addMoney(that.plane_location ? 0 : 1, that.slotList[that.plane_location ? 0 : 1].rank)
            dropGift(that.plane_location, that.rank + 1)
          } else if (that.step > 3) { // 指示结束了， 几架飞机在跑道 几架在降落块上
            planeList = that.planeList
            that.durationTime = 3000
            that.total = -1
            that.slotList.forEach((item, index) => {
              if (item.rank) {
                if (item.running) {
                  that.isRun = true
                } else {
                  that.isRun = false
                }
                createPlane(item.rank, index)
                if (item.running) {
                  coinSprite.text = coinNum
                  addMoney(index, item.rank)
                  console.log(index, item.rank)
                }
              } else if (item.occpuied && !item.rank) {
                  dropGift(index, that.rank)
              }
            })
            if (that.step >= 5) {
              that.isRun = false
              that.cleartimer = setInterval(() => {
                dropGift('', that.rank)
              }, 2800)
            }
          } 
        } else {
          goodSprite.scale.set(0.26)
          slotList[0].rank = that.rank
          slotList[0].occpuied = true
          that.$set(that.slotList[0], 'rank', that.rank)
          that.$set(that.slotList[0], 'occpuied', true)
          plane_tween3.start()
          plane_tween3.on('end', () => {
            if (slotList[0].occpuied) {
              dropGift(1, that.rank)
            } else {
              dropGift(0, that.rank)
            }
            that.status = 1
          })
        }
        
        
      }
      function dropGift (n, rank) {  // 掉箱子
        // console.log('掉箱子', n, rank)
        let occpuiedArr = []
        if (typeof n != 'number'){
          slotList.forEach((item, index) => {
            if (!item.occpuied && !item.running) {
              occpuiedArr.push(index)
            }
          })
          // console.log(occpuiedArr)
          if (!occpuiedArr.length) {
            return
          }
          n = occpuiedArr[Math.floor(Math.random() * occpuiedArr.length)]
          // console.log(n)
        }
        let gift = new PIXI.Sprite.fromImage(configMarqee.boxUrl)
        gift.anchor.set(0.5)
        gift.scale.set(0.9)
        gift.x = slotList[n].x
        gift.interactive = true
        sceneContainer.addChild(gift)
        gift.y = 0
        const gift_tween1 = PIXI.tweenManager.createTween(gift)
        gift_tween1.from({y: 0}).to({y: slotList[n].y - 6})
        gift_tween1.time = 500
        gift_tween1.easing = PIXI.tween.Easing.outBack()
        slotList[n].occpuied = true
        that.$set(that.slotList[n], 'occpuied', true)
        const gift_tween2 = PIXI.tweenManager.createTween(gift)
        gift_tween2.from({scale: {x: 0.9, y: 0.9}}).to({scale: {x: 1, y: 1}})
        gift_tween2.time = 400
        gift_tween2.repeat = 2
        gift_tween2.easing = PIXI.tween.Easing.inOutBack()
        gift_tween2.pingPong = true
        const gift_tween3 = PIXI.tweenManager.createTween(gift)
        gift_tween3.from({y: slotList[n].y - 6}).to({y: slotList[n].y - 6 - 10})
        gift_tween3.time = 400
        gift_tween3.repeat = 3
        gift_tween3.easing = PIXI.tween.Easing.inBounce() 
        gift_tween3.pingPong = true
        gift_tween1.start()
        gift_tween1.chain(gift_tween2).chain(gift_tween3)
        let isOpen = true
        gift_tween3.on('end', () => {
          if (!isOpen) {return}
          sceneContainer.removeChild(gift)
          console.log(`%c ${isOpen}`,'color:green')
          isOpen = false
          createPlane(rank, n)
          console.log(`%c ${isOpen}`,'color:green')
          spillAction(100, 100, slotList[n].x, slotList[n].y, [configMarqee.chipUrl, configMarqee.starUrl], configMarqee.composeEmitterConfig)
        })

        gift.on('pointerdown', () => {
          if (!isOpen) {return}
          isOpen = false
          sceneContainer.removeChild(gift)
          PIXI.tweenManager.removeTween(gift_tween1)
          spillAction(100, 100, slotList[n].x, slotList[n].y, [configMarqee.chipUrl, configMarqee.starUrl], configMarqee.composeEmitterConfig)
          createPlane(rank, n)
        })
      }
      function createPlane (r, i = 1) { // 0 == 25, 1 == 26
        console.log(`%c 创建飞机${r}，在${i}位置, step为${that.step}`,'color:blue')
        let plane = new PIXI.Sprite(textureList1[r])
        plane.anchor.set(0.5, 0.5)
        plane.scale.set(0)
        plane.pName = Math.random().toString(36).substr(2)
        plane.pRank = r
        plane.pIndex = i
        plane.x = slotList[i].x
        plane.y = slotList[i].y
        app.stage.addChild(plane)
        slotList[i].rank = r
        slotList[i].occpuied = true
        that.$set(that.slotList[i], 'rank', r)
        that.$set(that.slotList[i], 'occpuied', true)
        planeList[i] = plane
        that.$set(that.planeList, i, plane)
        plane.interactive = true
        plane.buttonMode = true
        if (that.step >= 5 && that.status == 1 && that.isRun) {
        } else if (!that.isRun){
          // console.log('零零落落')
          const plane_tween = PIXI.tweenManager.createTween(plane)
          plane_tween.from({scale: {x: 0, y: 0}}).to({scale: {x: 0.26, y: 0.26}})
          plane_tween.loop = false
          plane_tween.time = 400
          plane_tween.easing = PIXI.tween.Easing.outBack()
          plane_tween.start()
        }
        play(plane)
        if (that.step == 1) { // 指引合并 --- 指引到跑道 --- 指引返回
          handObj2 = that.createHandTween(slotList[1].x, slotList[1].y + 35, sceneContainer)
          handObj2.sprite.parentGroup = group3
          handObj2.tween.from({x: slotList[1].x}).to({x: slotList[0].x})
          handObj2.tween.time = 1600
          handObj2.tween.start()
        } else if (that.step == 2) {
          handObj2 = that.createHandTween(slotList[i].x, slotList[i].y + 12, sceneContainer)
          handObj2.sprite.anchor.set(0.5, 0)
          handObj2.sprite.parentGroup = group3
          handObj2.tween.from({x: slotList[i].x, y: slotList[i].y + 12}).to({x: trackIconList[1].x + 40, y: trackIconList[1].y})
          handObj2.tween.time = 1200
          handObj2.tween.pingPong = false
          handObj2.tween.easing = PIXI.tween.Easing.outQuad()
          handObj2.tween.start()
        } else if (that.step == 3 && that.toggleFlag) {
          let i = 1
          if (slotList[0].running) {
            i = 0
          }
          if (handObj2 && handObj2.sprite) {
            sceneContainer.removeChild(handObj2.sprite)
            PIXI.tweenManager.removeTween(handObj2.tween)
            handObj2 = {}
          }
          handObj2 = that.createHandTween(slotList[i].x + 10, slotList[i].y + 35, sceneContainer)
          handObj2.tween.start()
        } else if (that.step == 4) {
          if (handObj2 && handObj2.sprite) {
            return
          }
          handObj2 = that.createHandTween(slotList[1].x, slotList[1].y + 35, sceneContainer)
          handObj2.sprite.parentGroup = group3
          handObj2.tween.from({x: slotList[1].x}).to({x: slotList[0].x})
          handObj2.tween.time = 1600
          handObj2.tween.start()
        }else if (that.step == 5) {
          console.log('开始定时器啦',that.step)
          that.cleartimer = setInterval(() => {
            dropGift('', that.rank)
          }, 2800)
        } else {}
      }
      

      function play (item) { // 合成
        item.on('pointerdown', onDragStart)
        item.on('pointerup', onDragEnd)
        item.on('pointerupoutside', onDragEnd)
        item.on('pointermove', onDragMove)

        function onDragStart (event) {
          this.data = event.data
          this.alpha = 0.5
          this.dragging = true
          that.oldPosition.x = this.x
          that.oldPosition.y = this.y
        }
        function onDragEnd (event) {
          if (!this.dragging) {return}
          // debugger
          this.alpha = 1
          let moveI = this.pIndex
          let endI = findCloser(this.x, this.y, 40, moveI)
          let moveRank = slotList[moveI].rank
          console.log(`%c ${moveI}`, 'color: deeppink')
          if (endI > -1) {  // 有最近的
            let endRank = slotList[endI].rank
            let planeObj2 = planeList[moveI]
            let planeObj1 = planeList[endI]
            if (endRank == moveRank && !slotList[endI].running) { // 合成
              if (that.step == 1 || that.step == 5 || that.step == 4) {
                if (handObj2 && handObj2.sprite) {
                  sceneContainer.removeChild(handObj2.sprite)
                  PIXI.tweenManager.removeTween(handObj2.tween)
                  handObj2 = {}
                }
                that.step++
              }
              app.stage.removeChild(planeList[moveI])
              app.stage.removeChild(planeList[endI])
              planeList[moveI] = {}
              planeList[endI] = {}
              slotList[moveI].rank = 0
              slotList[endI].rank = 0
              slotList[moveI].occpuied = false
              slotList[endI].occpuied = true
              that.$set(that.slotList[moveI], 'rank', 0)
              that.$set(that.slotList[moveI], 'occpuied', false)
              that.$set(that.slotList[endI], 'rank', 0)
              that.$set(that.slotList[endI], 'occpuied', true)
              that.$set(that.planeList, moveI, {})
              that.$set(that.planeList, endI, {})
              if (moveRank + 1 == that.rank + 3) {
                clearInterval(that.cleartimer)
                app.stage.removeChild(marqeeContainer)
                planeList.forEach(item => {
                  app.stage.removeChild(item)
                })
                getSuccess()
              } else {
                spillAction(100, 100, slotList[endI].x, slotList[endI].y, [configMarqee.lightCircle, configMarqee.starUrl], configMarqee.lightEmitterConfig)
                createPlane(moveRank + 1, endI)
              }
            } else if (endRank && endRank != moveRank && !slotList[endI].running) { // 交换位置
              if (that.step < 5) {
                this.x = that.oldPosition.x
                this.y = that.oldPosition.y
                return
              }
              planeObj2.x = slotList[endI].x
              planeObj2.y = slotList[endI].y
              planeObj2.pIndex = endI
              planeObj1.pIndex = moveI
              planeObj1.x = slotList[moveI].x
              planeObj1.y = slotList[moveI].y
              
              planeList[moveI] = planeObj1
              planeList[endI] = planeObj2

              slotList[endI].rank = moveRank
              slotList[moveI].rank = endRank
              that.$set(that.slotList[endI], 'rank', moveRank)
              that.$set(that.slotList[moveI], 'rank', endRank)
              that.$set(that.planeList, moveI, planeObj1)
              that.$set(that.planeList, endI, planeObj2)
              // console.log(`%c 终点飞机${endRank}，在终点${endI}位置`,'color:blue')
              // console.log(`%c 移动飞机${moveRank}，在终点${moveI}位置`,'color:blue')
              console.log(`%c 在${endI}位置, 交换后${slotList[endI].rank}`,'color:red')
              console.log(`%c 在${moveI}位置, 交换后${slotList[moveI].rank}，`,'color:red')
              console.log('=================')
              console.log(`%c 交换后${planeList[endI].pRank}}，在${planeList[endI].pIndex}位置`,'color:red')
              console.log(`%c 交换后${planeList[moveI].pRank}}，在${planeList[moveI].pIndex}位置`,'color:red')
            } else if (slotList[endI].occpuied || slotList[endI].running){ //挪不动
              this.x = that.oldPosition.x
              this.y = that.oldPosition.y
            } else {  // 挪动飞机
              if (that.step < 5) {
                this.x = that.oldPosition.x
                this.y = that.oldPosition.y
              } else {
                this.x = slotList[endI].x
                this.y = slotList[endI].y
                planeObj2.pIndex = endI
                planeList[endI] = planeObj2
                planeList[moveI] = {}
                that.$set(that.planeList, moveI, {})
                that.$set(that.planeList, endI, planeObj2)
                slotList[endI].occpuied = true
                slotList[endI].rank = moveRank
                slotList[moveI].rank = 0
                slotList[moveI].occpuied = false
                that.$set(that.slotList[endI], 'occpuied', true)
                that.$set(that.slotList[endI], 'rank', moveRank)
                that.$set(that.slotList[moveI], 'rank', 0)
                that.$set(that.slotList[moveI], 'occpuied', false)
              }
            }
          } else {
            if (Math.abs(this.x - trackIconList[3].x) < 30) { // 可以上轨道
              if (that.step != 2 && that.step < 5) { //step 为2 和 >= 5
                this.x = that.oldPosition.x
                this.y = that.oldPosition.y
              } else {
                this.x = trackIconList[0].x + 12
                // 跑
                if (handObj2 && handObj2.sprite && !that.isFirst) {
                  sceneContainer.removeChild(handObj2.sprite)
                  that.step++
                  addMoney (moveI, moveRank)
                  if (slotList[1].running) {
                    dropGift(0, moveRank)
                  } else {
                    dropGift(1, moveRank)
                  }
                  that.isFirst = true
                } else {
                  addMoney (moveI, moveRank)
                }
              }
            } else {
              this.x = that.oldPosition.x
              this.y = that.oldPosition.y
            }
          }
          this.data = null
          this.dragging = false
        }
        function onDragMove (event) {
          if (this.dragging) {
            let newPosition = this.data.getLocalPosition(this.parent)
            this.x = newPosition.x
            this.y = newPosition.y
          }
        }
       function findCloser (x,y,r,idx){
          return slotList.findIndex((item,index) => {
            let tx = item.x;
            let ty = item.y;
            let dis = Math.sqrt(Math.pow(Math.abs(x - tx), 2) + Math.pow(Math.abs(y - ty), 2));
            if(dis < r && index != idx){
              return true
            }
          })
        }
      }

      function addMoney (moveI, rank) {
        console.log('去跑道啦', moveI, rank)
        that.durationTime += 500
        spillAction(40, 60, xCoord[0], winY, [configMarqee.starUrl], configMarqee.starEmitterConfig)
        that.total++
        trackIconList[that.total].texture = fullIcon
        planeTemp = new PIXI.Container()
        sceneContainer.addChild(planeTemp)
        let planeShadow = new PIXI.Sprite.fromImage(configMarqee.highPlaneList[rank])
        planeShadow.anchor.set(0.5)
        // planeShadow.parentGroup = group1
        planeShadow.alpha = 0.7
        planeShadow.clickFlag = false
        planeShadow.pRank = rank
        planeShadow.pIndex = moveI
        planeShadow.x = slotList[moveI].x
        planeShadow.y = slotList[moveI].y
        planeShadow.scale.set(0.26)
        planeShadow.interactive = true 
        planeShadow.buttonMode = true
        planeTemp.addChild(planeShadow)
        slotList[moveI].running = true
        slotList[moveI].occpuied = true
        that.$set(that.slotList[moveI], 'occpuied', true)
        that.$set(that.slotList[moveI], 'running', true)

        let arrow = new PIXI.Sprite.fromImage(configMarqee.occpuiedUrl)
        arrow.width = 24
        arrow.height = 24
        arrow.anchor.set(1, 1)
        arrow.x = slotList[moveI].x + planeShadow.width / 2
        arrow.y = slotList[moveI].y + planeShadow.height / 2
        planeTemp.addChild(arrow)
        let noFlag = false, newIndex = 0
        planeList[moveI].texture = textureList2[planeList[moveI].pRank]
        that.$set(that.planeList[moveI], 'texture', textureList2[planeList[moveI].pRank])
        planeList[moveI].width = 80
        planeList[moveI].height = 80
        let moving_tween = PIXI.tweenManager.createTween(planeList[moveI])
        runningPlanes.push(planeList[moveI])
        tweenList.push(moving_tween)
        moving_tween.path = path
        moving_tween.time = that.durationTime
        moving_tween.easing = PIXI.tween.Easing.linear()
        moving_tween.loop = true
        moving_tween.start()
        moving_tween.on('start', () => {
          planeShadow.on('pointerdown', returnBack)
          function returnBack (event) {
            let amid = event.target.parent
            if (amid.clickFlag) {return}
            let ind = this['pIndex'], rank = this['pRank']
            let endX = slotList[ind].x
            let endY = slotList[ind].y
            if (slotList[ind].running) {
              if (handObj2 && handObj2.sprite) {
                sceneContainer.removeChild(handObj2.sprite)
                handObj2 = {}
              }
              let rIndex = runningPlanes.findIndex(v => {
                return v.pIndex == ind
              })
              console.log('回来的时候：', ind)
              const tw = PIXI.tweenManager.createTween(planeList[ind])
              tw.from({
                x: planeList[ind].x,
                y: planeList[ind].y
              })
              tw.to({
                x: endX,
                y: endY
              })
              tw.time = 300
              tw.easing = PIXI.tween.Easing.linear()
              tw.start()
              tw.loop = false
              tw.on('start', () => {
                console.log('i am back')
              })
              let repeatFlag = false
              tw.on('end', () => { //对应等级的running 回了
                if (repeatFlag) {return}
                console.log('%c 我回了', 'color: yellow')
                planeList[ind].texture = textureList1[rank]
                that.$set(that.planeList[ind], 'texture', textureList1[rank])
                planeList[ind].scale.set(0.26)
                // that.$set(that.planeList[ind], 'scale', {x: 0.26, y: 0.26})
                planeList[ind].rotation = 0
                slotList[ind].running = false
                that.$set(that.slotList[ind], 'running', false)
                // sceneContainer.removeChild(planeTemp)
                PIXI.tweenManager.removeTween(tw)
                PIXI.tweenManager.removeTween(tweenList[rIndex])
                sceneContainer.removeChild(amid)
                runningPlanes.splice(rIndex, 1)
                tweenList.splice(rIndex, 1)
                // if (that.total >= 0) {
                  trackIconList[that.total].texture = emptyIcon
                  that.total--
                // }
                repeatFlag = true
                if (that.step == 3) {
                  if (handObj2 && handObj2.sprite) {
                    return
                  }
                  handObj2 = that.createHandTween(slotList[1].x, slotList[1].y + 35, sceneContainer)
                  handObj2.sprite.parentGroup = group3
                  handObj2.tween.from({x: slotList[1].x}).to({x: slotList[0].x})
                  handObj2.tween.time = 1600
                  handObj2.tween.start()
                  that.step++
                }
              })
              amid.clickFlag = true
            }
          }
        })

        app.ticker.add(delta => {
          runningPlanes.forEach((item, index) => {
            let leftX = canvasW - (canvasW - trackW) / 2
            let leftY = canvasH / 2 - 20
            let x = parseInt(item.x)
            let y = parseInt(item.y)
            if (x === xCoord[0]) {
              item.rotation = 0;
              item.moneyFlag = false;
            } else if (x === xCoord[3]) {
              if (Math.abs(item.x - leftX) <= 40 && Math.abs(item.y - leftY) <= 30 &&!item.moneyFlag) {
                spillAction(40, 40, xCoord[3], winY, [configMarqee.coinUrl], configMarqee.coinEmitterConfig)
                item.moneyFlag = true;
                coinNum += that.addArr[item.pRank - 1]
                coinSprite.text = coinNum
                const tween = PIXI.tweenManager.createTween(keySprite)
                tween.from({
                  scale: {x: 0.8, y: 0.8}
                });
                tween.to({
                  scale: {x: 1.2, y: 1.2}
                });
                tween.pingPong = true;
                tween.time = 500;
                tween.easing = PIXI.tween.Easing.inOutBack();
                tween.loop = false;
                tween.start();
                tween.on('end', () => {
                  tween.remove();
                })
              }
              item.rotation = -Math.PI;
            } else if (y === yCoord[3]) {
              item.rotation = -Math.PI / 2;
            } else if (y === startY) {
              item.rotation = -Math.PI * 3 / 2;
            } else if (y >= (yCoord[2]) && x <= (xCoord[1])) {//300,400
              item.rotation = -Math.asin((y - (yCoord[2])) / airportR)
            } else if (y >= (yCoord[2]) && x > (xCoord[2])) {
              item.rotation = -Math.PI + Math.asin((y - (yCoord[2])) / airportR)
            } else if (y <= (yCoord[1]) && x > (xCoord[2])) {//300,200
              item.rotation = -Math.PI + Math.asin((y - (yCoord[1])) / airportR)
            } else if (y <= (yCoord[1]) && x <= (xCoord[1])) {
              item.rotation = -Math.asin((y - ((yCoord[1]))) / airportR)
            } else {}
          })
        })
      }
      function getSuccess () {
        app.stage.removeChild(sceneContainer)
        that.status = 2
        let bgSprite1 = new PIXI.Sprite.fromImage(that.isLandscape ? configMarqee.bgUrl2 : configMarqee.bgUrl)
        bgSprite1.width = canvasW
        bgSprite1.height = canvasH
        app.stage.addChild(bgSprite1)
        bgSprite1.parentGroup = group3
        let rectangle = new PIXI.Graphics()
        rectangle.beginFill(0x132127, 0.56)
        rectangle.drawRect(0, 0, canvasW, canvasH)
        bgSprite1.addChild(rectangle)
        console.log(app, app.stage.width)
        console.log(document.getElementsByTagName('canvas')[0].width)
        let lightBg = new PIXI.Sprite.fromImage(configMarqee.successLightUrl)
        lightBg.anchor.set(0.5)
        let x2 = canvasW / 2
        lightBg.x = x2
        lightBg.y = canvasH / 2 - 40
        lightBg.width = that.isPad ? 300 : 385
        lightBg.height = that.isPad ? 300 : 385
        bgSprite1.addChild(lightBg)
        const light_tween = PIXI.tweenManager.createTween(lightBg)
        light_tween.from({rotation: 0}).to({rotation: Math.PI * 2})
        light_tween.loop = true
        light_tween.time = 9000
        light_tween.easing = PIXI.tween.Easing.linear()
        light_tween.start()
        let playBtnObj = that.isEnglish ? configMarqee.playBtnUrl : configMarqee['playBtnUrl_cn']
        let downBtn = new PIXI.Sprite.fromImage(playBtnObj)
        downBtn.anchor.set(0.5)
        downBtn.x = lightBg.x
        downBtn.y = that.isLandscape ? canvasH - 60 : canvasH - 200
        downBtn.width = 208
        downBtn.height = 66
        downBtn.interactive = true
        bgSprite1.addChild(downBtn)
        downBtn.on('pointerdown', () => {
          that.linkAppStore()
        })
        let box = new PIXI.Sprite.fromImage(configMarqee.successBoxUrl)
        box.anchor.set(0.5)
        box.x = x2
        box.y = canvasH / 2 - 40
        box.width = that.isPad ? 160 : 208
        box.height = that.isPad ? 147 :191
        box.interactive = true
        box.on('pointerdown', () => {
          that.linkAppStore()
        })
        bgSprite1.addChild(box)
        const box_tween = PIXI.tweenManager.createTween(box)
        box_tween.from({rotation: 0}).to({rotation: that.d2a(10)})
        box_tween.pingPong = true
        box_tween.loop = true
        box_tween.time = 600
        box_tween.easing = PIXI.tween.Easing.linear()
        light_tween.start()
        
        let x1 = box.x + box.width / 2 - 40
        let y1 = canvasH / 2 + 40
        let hand = new PIXI.Sprite.fromImage(configMarqee.shakeHand)
        bgSprite1.addChild(hand)
        hand.width = 90
        hand.height = 89
        hand.x = x1
        hand.y = y1
        const hand_tween = PIXI.tweenManager.createTween(hand)
        hand_tween.from({x: x1, y: y1}).to({x: x1 - 40, y: y1 - 50})
        hand_tween.loop = true
        hand_tween.pingPong = true
        hand_tween.delay = 800
        hand_tween.time = 1400
        hand_tween.easing = PIXI.tween.Easing.linear()
        box_tween.start()
        box_tween.chain(hand_tween)
        hand_tween.start()
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
        app.stage.addChild(coinTank);
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
      obj.sprite = new PIXI.Sprite.fromImage(configMarqee.shakeHand)
      obj.sprite.anchor.set(0.5)
      // obj.sprite.width = 62
      // obj.sprite.height = 62
      obj.sprite.name = 'hand1'
      parentCont.addChild(obj.sprite)
      obj.tween = PIXI.tweenManager.createTween(obj.sprite)
      obj.sprite.x = x
      obj.sprite.y = y
      obj.sprite.scale.set(0.06)
      obj.tween.from({scale: {x: 0.06,y: 0.06}})
      obj.tween.to({
        scale: {
          x: 0.07,
          y: 0.07
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
    linkAppStore() {
      let url = this.iosLink;
      let userAgent = navigator.userAgent || navigator.vendor;
      if (/android/i.test(userAgent)) {
        url = this.androidLink;
      }
      try {
        // dapi.openStoreUrl();
        // console.log('dapi open store')
        mraid.open(url);          // ad平台【AdColony Applovin Vungle】
        console.log('mraid open store')
      } catch (err) {
        console.error(err);
        window.location = url;
      }
    }
  }
}
</script>
<style lang="scss">
#mergeplaneNail {
  width: 100%;
  height: 100%;
  background-color: #000;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-pack: center;
  justify-content: center;
  -ms-flex-align: center;
  align-items: center;
}
@font-face {
  font-family: 'NumFont';
  src: url('data:font/truetype;base64, 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CWFmaWkxMDAzNAlhZmlpMTAwMzUJYWZpaTEwMDM2CWFmaWkxMDAzNwlhZmlpMTAwMzgJYWZpaTEwMDM5CWFmaWkxMDA0MAlhZmlpMTAwNDEJYWZpaTEwMDQyCWFmaWkxMDA0MwlhZmlpMTAwNDQJYWZpaTEwMDQ1CWFmaWkxMDA0NglhZmlpMTAwNDcJYWZpaTEwMDQ4CWFmaWkxMDA0OQlhZmlpMTAwNjUJYWZpaTEwMDY2CWFmaWkxMDA2NwlhZmlpMTAwNjgJYWZpaTEwMDY5CWFmaWkxMDA3MAlhZmlpMTAwNzIJYWZpaTEwMDczCWFmaWkxMDA3NAlhZmlpMTAwNzUJYWZpaTEwMDc2CWFmaWkxMDA3NwlhZmlpMTAwNzgJYWZpaTEwMDc5CWFmaWkxMDA4MAlhZmlpMTAwODEJYWZpaTEwMDgyCWFmaWkxMDA4MwlhZmlpMTAwODQJYWZpaTEwMDg1CWFmaWkxMDA4NglhZmlpMTAwODcJYWZpaTEwMDg4CWFmaWkxMDA4OQlhZmlpMTAwOTAJYWZpaTEwMDkxCWFmaWkxMDA5MglhZmlpMTAwOTMJYWZpaTEwMDk0CWFmaWkxMDA5NQlhZmlpMTAwOTYJYWZpaTEwMDk3BEV1cm8LLm5vdGRlZi4wMDEAAAAAAAAB//8AAgABAAAADAAAABYAAAACAAEAAwC9AAEABAAAAAIAAAAAAAEAAAAKABwAHgABbGF0bgAIAAQAAAAA//8AAAAAAAAAAQAAAAoAHgAsAAFsYXRuAAgABAAAAAD//wABAAAAAWtlcm4ACAAAAAEAAAABAAQAAgAAAAEACAABA1AABAAAACkAXACCAIgApgCwALoA0ADaAOAA8gD4AP4BDADaASYBMAFGAWABdgGEAZ4BpAG2AdQB+gIgAjYCRAJKAlQCXgKEAqoCsALSAuAC9gMAAwoDJAMyAAkARP/gAE3/hwBQ/+AAV/+tAFn/7ABa/94AW/+1AFz/wQBd/8YAAQAR/3wABwAQ/+YARv/0AEoACQBW//IAWf/gAFr/1QBc/94AAgAQ/74AUv/0AAIAD/+kABH/pwAFAA//egAR/3wARP/kAE3/kABYAAsAAgAP/74AEf/EAAEAEP/VAAQAEP/pAFb/4wBX/7sAXP/XAAEAEP/gAAEAEP/3AAMAD//EABH/yQBb/+4ABgAP/1cAEP/1ABH/XQBE/94ATf+QAFD/5AACAA//yQAR/9IABQAP/7gAEP+qABH/sABE/+AATf/MAAYAD/+wABD/2AAR/7sARP/PAE3/uABY//EABQAP/74AEP/gABH/uwBE/9UATf/KAAMAEP+4AFL/3QBU/90ABgAP/3MAEP+1ABH/jQBE/7gATf+LAFj/+AABABD/wQAEAKD/vgCh/9IApf/bAKj/2AAHAA//zwAR/9IAjv/0AJL/5gCZ/+wAmv/vAKH/xgAJAA//3gAR/+AAjv/0AJL/7ACU/+8Amv/1AKD/2ACh/88Ao//bAAkAD/+QABH/kACO/7gAkv+kAJn/uwCa/7sAnP/aAJ//5gCt/9IABQCc/+YAn//dAKD/vgCh/8IApf++AAMAnP/hAJ//1wCi/8cAAQCc/+YAAgCc//IAof/gAAIAnP/1AKH/0gAJAA//zwAR/9gAkv/PAJT/xwCZ/+kAmv/mAKD/7wCh/+MAo//UAAkAD/9oABH/XACO/8cAkv+cAJT/2ACZ/8EAmv/NAKP/4QCt/+wAAQCc/+MACAAP/7UAEf+wAI7/uACZ/7UAmv+7AJz/2ACf/+MArf/QAAMAD/98ABH/fwCS/5MABQAP/3cAEf95AJT/rQCh/80Ao/+1AAIAoP/bAKH/2AACAKD/1QCh/80ABgAP/+AAEf/mAJT/4ACg/7MAof+1AKP/0gADAA//1QAR/+YAof+yAAcAD//BABH/xACO/9gAkv/BAJn/2wCa/9UAof/gAAEAKQAQACIARABGAEcASQBKAE4ATwBQAFEAUgBTAFUAVgBXAFkAWgBbAFwAXQCOAI8AkACRAJIAlACYAJkAmgCcAJ4AnwCgAKEAogCkAKcAqACqAKwAAAAAAAEAAAAA0yuCHgAAAADEiGOFAAAAAMnv7TY=');
  font-style: normal;
  font-weight: bold;
}
</style>

