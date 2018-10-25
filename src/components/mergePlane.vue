<template>
  <div id="mergePlane">
  </div>
</template>

<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import CONFIG from '../assets/js/config'
import tweenManager from 'pixi-tween'
import Vue from 'vue'
import FontFaceObserver from 'fontfaceobserver'
import particles from 'pixi-particles'
// import 'pixi-layers'
import '../assets/js/pixi-display'
export default {
  name: 'mergePlane',
  data () {
    return {
      isLoading: true,
      app1: {},
      distanceX: 110,
      distanceY: 10,
      boxList: [],
      step: 1,
      logoH: 120, // logo的y
      btnH: 0, // 安装btn的y
      bthHeight: 58,
      tween1: '', // 手指示的tween
      tween2: '',
      handSprite: {}, //手sprite
      planeList: new Array(6).fill({}),  // 飞机sprite
      shadeSprite: {}, // 背景遮罩
      hintSprite: {}, // 文字提示遮罩
      animateTxt: {},
      oldPosition: {x: 0, y: 0},
      grade: 0, // 合成飞机的等级
      fixedBox: [], // 绿色盒子
      roundBox: {}, // 蓝色背景
      clearTime: '', // 大循环
      smClear: '',  // 小循环
      isEmpty: '',
      total: 3,
      status: {},  //  1正常  2 小成功 3 大成功
      caluObj: '',
      iosLink: 'https://itunes.apple.com/app/id1363863879',
      androidLink: 'https://play.google.com/store/apps/details?id=com.brokenreality.planemerger.android',
      rankBox: [],  // 合成
      isLandscape: true, // 默认横屏
      container1: {},
      downBtn: {},
      group1: '',
      group2: '',
      group3: '',
      group4: '',
      texture: '',
      blmask: '',
      btGift: '',
      fullText: '',
      changeFlag: false,
      caluTime: '',
      shiningBtn: '',
      shineFlag: false,
      hand_tween2: '',
      isFlag: false,
      isEnglish: true
    }
  },
  watch: {
    step (newval, oldval) {
      if (oldval >= 1) {
        this.GiftScene.visible = true
      }
    },
    total: {
      handler(newval, oldval) {
        let filter = new PIXI.filters.BlurFilter()
        if (oldval != newval) {
          this.caluObj.text = newval
        }
      }
    }
  },
  created () {
    // 解决font-face问题
    let that = this
    let font = new FontFaceObserver('NumFont')
    font.load().then(function () {
      console.log('has loaded')
      that.isLoading = false
    })
  },
  mounted () {
    this.getSize()
    this.initPIXI()
  },
  methods: {
    getSize () {
      let that = this
      if (document.body.clientWidth <= document.body.clientHeight) {
        this.isLandscape = false
      } else {
        this.isLandscape = true
      }
      window.addEventListener('resize', onResize, false)
      function onResize () {
        // scaleTowindow(this.app1.renderer.view)
        if (document.body.clientWidth > document.body.clientHeight) {
          that.isLandscape = true
        } else {
          that.isLandscape = false
        }
        console.log(that.status, that.step)
        let canvas = document.querySelector('#mergePlane canvas');
        canvas.parentNode.removeChild(canvas);
        let obj = {}
        obj.step = that.step
        obj.status = that.status
        
        if (!obj.status.flag) {
          that.step = 1
          that.initPIXI()
        } else if (obj.status.flag == 1) { // step为1 。 step 为 2 , step 为3
          that.app1.stage.removeChild(that.newTexture)
          clearInterval(that.clearTime)
          let newArr = that.boxList.slice(0)
          let numObj = {}
          for (let key in that.caluObj) {
            numObj[key] = that.caluObj[key]
          }
          that.initPIXI()
          that.GiftScene.visible = true
          if (that.step == 2) {
            that.createPlane(2, 2)
            that.dropGift()
            that.cycleFuc()
          } else {
            if (that.step == 3) {
              that.hand_tween2 =  that.createHandTween(that.btGift.x + 20, that.btGift.y + 10) // 点击指示
              that.hand_tween2.tween.start()
            }
            let arr = []
            that.boxList.forEach((v, i) => {
              if (v.rank) {
                that.createPlane(newArr[i].rank, i)
              } else if (v.occupied) {
                that.dropGift([i])
              } else {
                arr.push(v)
              }
            })
            if (!arr.length) { // 满了
              that.cycleFuc(1)
            } else {
              that.cycleFuc()
            }
          }
        } else if (obj.status.flag == 2) {
          that.initPIXI()
          that.passResult(obj.status.rank, obj.status.i)
        } else if (obj.status.flag == 3) {
          that.initPIXI()
          that.winSuccess(obj.status.rank)
        }
      }
    },
    initPIXI () {
      let container, bgSprite, logoSprite
      const canvasW = this.isLandscape ? 736 : 414
      const canvasH = this.isLandscape ? 414 : 736
      this.app1 = new PIXI.Application(canvasW, canvasH, {
        antialias: true,
        forceCanvas: 'canvas'
      })
      document.getElementById('mergePlane').appendChild(this.app1.view);
      let canvas = document.querySelector('#mergePlane canvas');
      let actualW = document.body.clientWidth, actualH = document.body.clientHeight
      if (actualW >= canvasW * actualH / canvasH ) {
        canvas.style.height = '100%'
        canvas.style.width = canvasW * actualH / canvasH + 'px'
      } else {
        canvas.style.height = canvasH * actualW / canvasW + 'px'
        canvas.style.width = '100%'
      }
      this.app1.renderer.view.style.display = 'block'
      this.app1.renderer.autoResize = true
      container = new PIXI.Container()
      this.app1.renderer.render(container)

      this.app1.stage = new PIXI.display.Stage()
      this.app1.stage.group.enableSort = true
      this.group1 = new PIXI.display.Group(2, true) 
      this.group2 = new PIXI.display.Group(2, true) // 飞机层
      this.group3 = new PIXI.display.Group(3, true) //手指示层
      this.group4 = new PIXI.display.Group(4, true) //合成成功层
      this.app1.stage.addChild(new PIXI.display.Layer(this.group1))
      this.app1.stage.addChild(new PIXI.display.Layer(this.group2))
      this.app1.stage.addChild(new PIXI.display.Layer(this.group3))    
      this.app1.stage.addChild(new PIXI.display.Layer(this.group4))
      //背景，降落块、logo等静态====================
      bgSprite = new PIXI.Sprite.fromImage(CONFIG.bodyBg)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = this.app1.screen.width
      bgSprite.height = this.app1.screen.height
      this.app1.stage.addChild(bgSprite)
      this.layBox()
      this.downBtn = this.installBtn({
        url: this.isEnglish ? CONFIG.installBtn : CONFIG.installBtn_ch,
        width: this.isEnglish ? 148 : 212,
        height: this.isEnglish ? this.bthHeight : 118,
        y: this.isLandscape ? this.app1.screen.height - 40: this.app1.screen.height - 116,
        parentCont: this.app1.stage
      })
      this.shiningBtn = this.installBtn({
        url: this.isEnglish ? CONFIG.shiningBtn : CONFIG.shiningBtn_ch,
        width: 212,
        height: 118,
        y: this.isLandscape ? this.app1.screen.height - 40 : this.app1.screen.height - 116,
        parentCont: this.app1.stage
      })
      this.shiningBtn.alpha = 0
      const shine_tween = PIXI.tweenManager.createTween(this.shiningBtn)
      shine_tween.time = 2000
      shine_tween.pingPong = true
      shine_tween.loop = true
      shine_tween.from({
        alpha: 0
      })
      shine_tween.to({
        alpha: 1
      })
      shine_tween.easing = PIXI.tween.Easing.linear()
      shine_tween.on('start', () => {
        console.log('i`m shining')
      })
      shine_tween.start()
      this.shiningBtn.on('pointerdown', () => {
        this.linkAppStore()
      })
      let logo = this.isEnglish ? CONFIG.logo : CONFIG['logo_ch']
      logoSprite = new PIXI.Sprite.fromImage(logo)
      if (this.isEnglish) {
        logoSprite.width = this.isLandscape ? 250 : 210
        logoSprite.height = this.isLandscape ? 40 : 34
      } else {
        logoSprite.width = this.isLandscape ? 196 : 156
        logoSprite.height = this.isLandscape ? 60 : 48
      }
      logoSprite.x = (this.app1.screen.width - logoSprite.width) / 2
      logoSprite.y =  this.isLandscape ? 26 : this.logoH
      this.app1.stage.addChild(logoSprite)

      
      // 指示1
      if (this.step == 1) {
        // 遮罩
        this.shadeSprite = new PIXI.Graphics()
        this.shadeSprite.beginFill(0x000000, 0.5)
        this.shadeSprite.drawRect(0, 0, this.app1.screen.width, this.app1.screen.height)
        this.app1.stage.addChild(this.shadeSprite)
    
        // 文字提示
        let hint = this.isEnglish ? CONFIG.iconHint : CONFIG.iconHint_ch
        this.hintSprite = new PIXI.Sprite.fromImage(hint)
        this.hintSprite.anchor.set(0.5, 0.5)
        this.hintSprite.width = this.isLandscape ? 260 : 330
        this.hintSprite.height = this.isLandscape ? 100 : 126
        this.app1.stage.addChild(this.hintSprite)
        this.hintSprite.x = this.app1.screen.width / 2
        this.hintSprite.y = this.isLandscape ? canvasH - 60 : this.downBtn.y + this.downBtn.height / 2 - this.hintSprite.height / 2
        if (this.isEnglish) {
          this.animateTxt = new PIXI.Text('SWIPE TO PLAY!', {
            fontSize: 28,
            fontWeight: 'normal',
            fontStyle: 'normal',
            fill: '0x000000',
            fontFamily: 'NumFont',
            align: 'center'
          })
        } else {
          this.animateTxt = new PIXI.Sprite.fromImage(CONFIG.shineTxt_ch)
        }
        this.animateTxt.anchor.set(0.5, 0.5)
        this.animateTxt.x = this.hintSprite.x
        this.animateTxt.y = this.hintSprite.y - 30
        const tween_txt = PIXI.tweenManager.createTween(this.animateTxt)
        tween_txt.from({alpha: 1}).to({alpha: 0})
        tween_txt.loop = true
        tween_txt.time = 1000
        tween_txt.easing = PIXI.tween.Easing.linear()
        this.app1.stage.addChild(this.animateTxt)
        tween_txt.start()

        //高亮
        let roundBox = new PIXI.Graphics()
        roundBox.beginFill(0x0d95f9)
        roundBox.drawRoundedRect(0, 0, this.boxList[0].width + this.distanceX + 20, this.isLandscape ? this.distanceY + this.boxList[0].height - 4: this.distanceY + this.boxList[0].height)
        roundBox.endFill()
        roundBox.x = this.boxList[1].x - (this.distanceX - this.boxList[0].width) / 2 - this.boxList[0].width / 2
        roundBox.y = this.boxList[0].y - this.boxList[0].height / 2
        this.app1.stage.addChild(roundBox)
        this.roundBox = roundBox
        for(let i = 0; i < 2; i++) { // 1 2
          let sprite = new PIXI.Sprite.fromImage(CONFIG.layBox)
          sprite.anchor.set(0.5, 0.5)
          sprite.width = 92
          sprite.height = 116
          sprite.x = (i % 2) * this.distanceX + this.boxList[0].x + this.distanceX
          sprite.y = this.boxList[0].y
          this.app1.stage.addChild(sprite)
          this.fixedBox.push(sprite)
          this.createPlane(1, i + 1)
        }
        // 手指示
        this.handSprite = new PIXI.Sprite.fromImage(CONFIG.iconHand)
        this.handSprite.anchor.set(0.5, 0.5)
        this.handSprite.width = 62
        this.handSprite.height = 62
        this.handSprite.y = this.boxList[1].y + 30
        this.handSprite.x = this.boxList[1].x + this.boxList[1].width / 2 - 8
        this.app1.stage.addChild(this.handSprite)
        this.handSprite.parentGroup = this.group3
        this.tween1 = PIXI.tweenManager.createTween(this.handSprite)
        this.tween1.from({x: this.boxList[1].x + this.boxList[1].width / 2 - 8}).to({x: this.boxList[2].x + 10})
        this.tween1.time = 3000
        this.tween1.loop = true
        this.tween1.pingPong = true
        this.tween1.easing = PIXI.tween.Easing.linear()
        this.tween1.on('start', () => {
          console.log('start')
        })
        this.tween1.start()
      }

      //底部盒子====================
      let GiftScene = new PIXI.Container()
      this.app1.stage.addChild(GiftScene)
      
      let btGift = new PIXI.Sprite.fromImage(CONFIG.btGift_bottom)
      btGift.anchor.set(0.5, 0.5)
      btGift.width = 98
      btGift.height = 88
      btGift.x = this.app1.screen.width / 2
      btGift.y = this.isLandscape ? this.btnH - btGift.height + 16: this.btnH - btGift.height + 10
      GiftScene.addChild(btGift)
      this.btGift = btGift
      // 黑色礼盒遮罩
      this.texture = new PIXI.Texture.from(CONFIG.blackMask)  // 礼物盒遮罩
      let rect = new PIXI.Rectangle(0, 0, 98, 88)
      this.texture.frame = rect
      this.newTexture = new PIXI.Texture(this.texture.baseTexture, this.texture.frame);
      let blmask = new PIXI.Sprite.from(this.newTexture);
      blmask.x = this.app1.screen.width / 2 - 49;
      blmask.y = this.isLandscape ? this.btnH - btGift.height + 16 - 44 : this.btnH - btGift.height + 10 - 44;
      GiftScene.addChild(blmask)
      this.blmask = blmask
      this.blmask.interactive = true
      
      // 计时盒子
      let caluTime = new PIXI.Sprite.fromImage(CONFIG.caluTime)
      caluTime.anchor.set(0.5, 0.5)
      caluTime.width = 42
      caluTime.height = 34
      caluTime.x = this.app1.screen.width / 2 - caluTime.width / 2 - caluTime.width - 10
      caluTime.y = this.btnH - 90 - 16
      GiftScene.addChild(caluTime)
      this.caluTime = caluTime
      this.caluObj = new PIXI.Text(this.total, {
        fontSize: 24,
        fontWeight: 'normal',
        fill: '0xffae43',
        fontFamily: 'NumFont',
        align: 'center'
      })
      this.caluObj.anchor.set(0.5, 0.5)
      this.caluObj.x = caluTime.x
      this.caluObj.y = caluTime.y
      GiftScene.addChild(this.caluObj)
      let times = 0
      this.caluObj.text = this.total
      this.GiftScene = GiftScene
      this.GiftScene.visible = false
      this.blmask.on('pointerdown', () => {
        this.step ++
        console.log('%c 我是事件', 'color: blue')
        this.shineFlag = true
        PIXI.tweenManager.removeTween(this.hand_tween2.tween)
        this.app1.stage.removeChild(this.hand_tween2.sprite)
        if (this.total == 0) {
          this.total = 0
        } else {
          this.total--
        }
      })
      let maskCount =0, shineCount = 0, that = this, unit = 0.5
      this.app1.ticker.add(delta => {
        PIXI.tweenManager.update()
        if (that.shineFlag) {
          unit = 2 - that.total
        } else {
          unit = 0.5
        }
        if (that.changeFlag) {
          maskCount += delta * unit
          // console.log(maskCount)
          if(maskCount >= 88){
            maskCount = 88
          }
        }else{
          maskCount = 0
        }
        that.updateBlmask(88 - maskCount)
      })
    },
    updateBlmask (h){
      if(h > 88){
        h = 88
      }
      this.texture = new PIXI.Texture.from(CONFIG.blackMask)  // 礼物盒遮罩
      let rect = new PIXI.Rectangle(0, 0, 98, h)
      this.texture.frame = rect
      this.newTexture = new PIXI.Texture(this.texture.baseTexture, this.texture.frame);
      this.blmask.texture =this.newTexture
    },
    layBox () { // 降落块
      for (let i = 0; i < 6; i++) {
        let sprite = new PIXI.Sprite.fromImage(CONFIG.layBox)
        sprite.anchor.set(0.5, 0.5)
        sprite.width = this.isLandscape ? 40 : 92
        sprite.height = this.isLandscape ? 50 : 116
        this.app1.stage.addChild(sprite)
        let x = this.isLandscape ? this.app1.screen.width / 2 - 2 * this.distanceX - this.distanceX / 2 + i * this.distanceX : (i % 3) * this.distanceX + this.app1.screen.width / 2 - this.distanceX;
        let y = this.isLandscape ? this.logoH + 20 : Math.floor(i / 3) * (this.distanceY + sprite.height) + this.logoH + 114
        let rank = 0;
        let occupied = false;
        let width = 92, height = 116
        sprite.width = width
        sprite.height = height
        sprite.x = x;
        sprite.y = y;
        if (this.status.flag == 1 && this.step >= 3) {
          rank = this.boxList[i].rank
          occupied = this.boxList[i].occupied
        }
        Vue.set(this.boxList, i, {x, y, rank, occupied, width, height})
      }
    },
    installBtn (obj) { // 安装按钮
      let installSprite = new PIXI.Sprite.fromImage(obj.url)
      installSprite.anchor.set(0.5, 0.5)
      installSprite.height = obj.height
      installSprite.width = obj.width
      installSprite.x = this.app1.screen.width / 2
      installSprite.y = obj.y
      this.btnH = installSprite.y
      obj.parentCont.addChild(installSprite)
      installSprite.interactive = true
      return installSprite
    },
    createPlane (r = 1, index) { // r 为飞机等级, index 飞机在哪个方块
      let plane, that = this, obj = {}, rk
      obj = this.boxList[index]
      plane = new PIXI.Sprite.fromImage(CONFIG.planeList[r - 1])
      plane.anchor.set(0.5, 0.5)
      plane.width = 88
      plane.height = 66
      plane.pName = Math.random().toString(36).substr(2)
      plane.pRank = r
      plane.x = this.boxList[index].x
      plane.y = this.boxList[index].y - 8
      Vue.set(this.planeList, index, plane)
      Vue.set(this.boxList[index], 'rank', r);
      this.$set(this.boxList[index],'occupied',true); // occupied 表示占位
      plane.interactive = true
      plane.buttonMode = true
     
      plane.parentGroup = this.group2
      this.app1.stage.addChild(plane)
      const plane_tween = PIXI.tweenManager.createTween(plane)
      let minScale = 0.12, maxScale =  r == 3 ? 0.25 : 0.2
      plane_tween.from({
        scale: {
          x: minScale,
          y: minScale
        }
      })
      plane_tween.to({
        scale: {
          x: maxScale,
          y: maxScale
        }
      })
      plane_tween.time = 100
      plane_tween.easing = PIXI.tween.Easing.inOutBack()
      plane_tween.on('start', () => {
        console.log('hhhh')
      })
      if (this.step >= 3) {
        plane_tween.start()
        // this.step ++
      }
      this.play(plane)
    },
    randomInt (len) {
      return Math.floor(Math.random() * len)
    },
    play(obj) { // 合成升级
      let that = this, onMoveing, curIndex, dragContainer, i = 0, tinting// 绿色在哪
      obj.on('pointerdown', onDragStart)
      obj.on('pointerup', onDragEnd)
      obj.on('pointerupoutside', onDragEnd)
      obj.on('pointermove', onDragMove)
      function onDragStart (event) {
        let equalArr = [], onMoveing
        this.data = event.data
        this.dragging = true
        that.oldPosition.x = this.x
        that.oldPosition.y = this.y
        console.log(that.oldPosition.x, that.oldPosition.y)
        onMoveing = that.planeList.findIndex(item => {
          if (!item.x) {
            return
          }
          return item.pName == this.pName
        })
        that.boxList.forEach((item, index) => {
          if (item.rank == that.planeList[onMoveing].pRank && index != onMoveing) {
            equalArr.push(
              index
            )
          }
        })
        console.log(equalArr)
        // that.createHighlight('', equalArr, 2)
      }
      function onDragEnd () {
        if (this.dragging) {
          let endPos = this.data.getLocalPosition(this.parent), minArr = [], resultArr = [], equalArr = []
           onMoveing = that.planeList.findIndex(item => {
            return item.pName == this.pName
          })
          curIndex = DetectValid(endPos.x,endPos.y,50,onMoveing);
          console.log(`%c ${curIndex}`,'color:deeppink');
          console.log(that.planeList[onMoveing])
          if (dragContainer && dragContainer.width) {
            console.log(that.app1.stage)
            that.app1.stage.removeChild(dragContainer)
          }
          let planeObj2 = {}, planeObj1 = {}
          if (curIndex > -1 && that.boxList[curIndex].rank == this.pRank) {  // 飞机级别一样
            if (that.step == 1) {
              PIXI.tweenManager.removeTween(that.tween1)
              that.app1.stage.removeChild(that.handSprite)
              that.app1.stage.removeChild(that.animateTxt)
              that.app1.stage.removeChild(that.hintSprite)
              that.app1.stage.removeChild(that.shadeSprite)

              that.app1.stage.removeChild(that.roundBox)
              that.fixedBox.forEach(item => {
                that.app1.stage.removeChild(item)
              })
              that.handSprite = null
              that.step++
            }
            let rk = that.boxList[curIndex].rank, obj2 = that.boxList[onMoveing]
            obj2.rank = 0
            that.planeList[onMoveing].destroy()
            that.planeList[curIndex].destroy()
            Vue.set(that.planeList, onMoveing, {})
            that.$set(that.boxList[onMoveing],'occupied',false);
            that.$set(that.boxList[onMoveing], 'rank', 0)
            clearInterval(that.clearTime)
            that.spillAction(100, 100, that.boxList[curIndex].x, that.boxList[curIndex].y, [CONFIG.circleUrl, CONFIG.starUrl], CONFIG.lightEmitterConfig)
            if (rk + 1 == 4) { // 最高级
              that.grade = rk + 1
              that.winSuccess(that.grade)
              that.rankBox.push(that.grade)
            } else { //合并级别跟当前的比， 默认为0
              if (!that.grade) {
                that.grade = rk + 1
                that.rankBox.push(that.grade)
                that.passResult(that.grade, curIndex)
              } else {
                let n = that.rankBox.findIndex(item => {
                  return item == rk + 1
                })
                if (n > -1) {
                  that.createPlane(rk + 1, curIndex)
                  that.cycleFuc()
                  return
                } else {
                  that.grade = rk + 1
                  that.rankBox.push(that.grade)
                  that.passResult(rk + 1, curIndex)
                }
              }
            }
          } else if (curIndex > -1 && that.boxList[curIndex].rank && that.boxList[curIndex].rank != this.pRank){ // 飞机交换
            if (that.step == 3) {
              return
            }
            planeObj2 = that.planeList[onMoveing], planeObj1 = that.planeList[curIndex]
            let rank2 = this.pRank, rank1 = planeObj1.pRank
            let rankTemp = rank2
            rank2 = rank1
            rank1 = rankTemp
            console.log(rank2, rank1)
            console.log(planeObj2.x)
            planeObj2.x = that.boxList[curIndex].x
            planeObj2.y = that.boxList[curIndex].y - 8
            planeObj2.pRank = rank1
            planeObj1.x = that.boxList[onMoveing].x
            planeObj1.y = that.boxList[onMoveing].y - 8
            planeObj1.pRank = rank2
            console.log(planeObj2.x)
            Vue.set(that.planeList, onMoveing, planeObj1)
            Vue.set(that.planeList, curIndex, planeObj2)
            Vue.set(that.boxList[curIndex], 'rank', rank1)
            Vue.set(that.boxList[onMoveing], 'rank', rank2)
            i = 0
          } else if (curIndex > -1 && !that.boxList[curIndex].rank && !that.boxList[curIndex].occupied){  // 交换位置，只有一个飞机
            if (that.step == 3) {
              planeObj2 = that.planeList[onMoveing]
              console.log(planeObj2.x)
              planeObj2.x = that.oldPosition.x
              planeObj2.y = that.oldPosition.y
              console.log(planeObj2.x)
              Vue.set(that.planeList, onMoveing, planeObj2)
              i = 0
            } else {
              planeObj2 = that.planeList[onMoveing]
              console.log(planeObj2)
              planeObj2.x = that.boxList[curIndex].x
              planeObj2.y = that.boxList[curIndex].y - 8
              Vue.set(that.planeList, onMoveing, {})
              Vue.set(that.planeList, curIndex, planeObj2)
              console.log(planeObj2.pRank)
              Vue.set(that.boxList[curIndex],'rank', planeObj2.pRank)
              Vue.set(that.boxList[onMoveing],'rank',0)
              Vue.set(that.boxList[curIndex],'occupied',true)
              Vue.set(that.boxList[onMoveing],'occupied', false)
              i = 0
            }
          } else {
            planeObj2 = that.planeList[onMoveing]
            console.log(planeObj2.x)
            planeObj2.x = that.oldPosition.x
            planeObj2.y = that.oldPosition.y
            console.log(planeObj2.x)
            Vue.set(that.planeList, onMoveing, planeObj2)
            i = 0
          }
          this.data = null
          this.dragging = false
        }
        
        
      }
      function onDragMove (event) {
        if (this.dragging) {
          let endPos = this.data.getLocalPosition(this.parent), minArr = [], resultArr = [], equalArr = []
          this.x = endPos.x
          this.y = endPos.y
          onMoveing = that.planeList.findIndex(item => {
            return item.pName == this.pName
          })
          tinting = onMoveing
          curIndex = DetectValid(endPos.x,endPos.y,50,onMoveing);
          // console.log(`%c ${curIndex}`,'color:deeppink');

          that.boxList.forEach((item, index) => {
            if (index != onMoveing && item.rank == this.pRank) {
              equalArr.push(index)
            }
          })
          // 有最近的, 且级别一样， 没有或者级别不一样
          if (curIndex > -1 && that.boxList[curIndex].rank == this.pRank) {
            tinting = curIndex
          } 
          // console.log(`%c ${tinting}`, 'color: red')
          createHighlight(tinting, equalArr)
        }
      }
      function DetectValid (x,y,r,idx){
          return that.boxList.findIndex((item,index) => {
            let tx = item.x;
            let ty = item.y;
            let dis = Math.sqrt(Math.pow(Math.abs(x - tx), 2) + Math.pow(Math.abs(y - ty), 2));
            if(dis < r && index != idx){
              // console.log(`%c tx:${x} ty:${y}`,'color:green');
              return true;
            }
          })
      }
      function createHighlight (curIndex, arr) {
        if (dragContainer && dragContainer.width) {
          that.app1.stage.removeChild(dragContainer)
        }
        dragContainer = new PIXI.Container()
        that.app1.stage.addChild(dragContainer)
        let closerSprite = new PIXI.Sprite.fromImage(CONFIG.closerPic)
        closerSprite.anchor.set(0.5, 0.5)
        closerSprite.x = that.boxList[curIndex].x
        closerSprite.y = that.boxList[curIndex].y - 4
        closerSprite.width = that.boxList[curIndex].width
        closerSprite.height = that.boxList[curIndex].height - 8
        closerSprite.parentGroup = that.group1
        dragContainer.addChild(closerSprite);

        if (!arr.length) {
          return
        }
        arr.forEach(item => {
          let dotSprite = new PIXI.Sprite.fromImage(CONFIG.dotPic)
          dotSprite.x = that.boxList[item].x - that.boxList[item].width / 2 
          dotSprite.y = that.boxList[item].y - that.boxList[item].height / 2
          dotSprite.width = that.boxList[item].width
          dotSprite.height = 96
          dotSprite.parentGroup = that.group1
          dragContainer.addChild(dotSprite)
        })
      }
    },
    passResult (r, i) { // 点击飞机的时候 
      this.status = {
        flag: 2,
        rank: r,
        i: i
      }
      let resultContainer = new PIXI.Container(), that = this
      this.app1.stage.addChild(resultContainer)

      let rect = new PIXI.Graphics()
      rect.beginFill(0x54677b, 0.91)
      rect.drawRect(0, 0, this.app1.screen.width, this.app1.screen.height)
      resultContainer.addChild(rect)
      rect.parentGroup = this.group3
      let t = this.isEnglish ? CONFIG.resultTitle : CONFIG['resultTitle_ch']
      let title = new PIXI.Sprite.fromImage(t)
      title.anchor.set(0.5, 0.5)
      title.width = 320
      title.height = 120
      title.x = this.app1.screen.width / 2
      title.y = this.isLandscape ? 100 : this.boxList[0].y
      resultContainer.addChild(title)
      title.parentGroup = this.group4

      let lightBg = new PIXI.Sprite.fromImage(CONFIG.resultLight)
      lightBg.width = 400
      lightBg.height = 400
      lightBg.anchor.set(0.5)
      lightBg.x = this.app1.screen.width / 2
      lightBg.y = this.isLandscape ? title.y + 110 :  title.y + 160
      resultContainer.addChild(lightBg)
      lightBg.parentGroup = this.group3

      let tw = PIXI.tweenManager.createTween(lightBg);
      tw.from({rotation:0});
      tw.to({rotation:2*Math.PI});
      tw.time = 3000;
      tw.loop = true;
      tw.start();

      let plane = new PIXI.Sprite.fromImage(CONFIG.planeList[r - 1])
      plane.anchor.set(0.5, 0.5)
      plane.width =  196 
      plane.height = 140 
      plane.x = lightBg.x
      plane.y = lightBg.y
      plane.interactive = true
      resultContainer.addChild(plane)
      plane.parentGroup = this.group3
      const result_tween = PIXI.tweenManager.createTween(plane)
      result_tween.time = 200
      result_tween.easing = PIXI.tween.Easing.linear()
      result_tween.from({
        scale: {
          x: 0.12,
          y: 0.12
        }
      })
      result_tween.to({
        scale: {
          x: 0.36,
          y: 0.36
        }
      })
      result_tween.start()
      plane.on('pointerdown', onDragStart)
      function onDragStart(event) {
        that.app1.stage.removeChild(resultContainer)
        PIXI.tweenManager.removeTween(result_tween)
        that.createPlane(r, i)
        if (that.step == 2) {
          that.dropGift()
        }
        that.cycleFuc()
        that.status.flag = 1
      }
    },
    cycleFuc (fz) { // 底部盒子倒计时
      let firstZero = fz ? fz : '', that = this //存在表示第一次没满
      this.clearTime = setInterval(() => {
        let arr = []
        that.boxList.forEach((item, index) => {
          if (!item.occupied) {
            arr.push(index)
          }
        })
        if (arr.length) {
          firstZero = ''
        }
        if(arr.length == 0 && firstZero){
          console.log('没有位置了')
          clearInterval(this.clearTime)
          this.fullText = new PIXI.Text('FULL', {
            fontSize: 24,
            fontWeight: 'normal',
            fontStyle: 'normal',
            fill: '0xffffff',
            fontFamily: 'NumFont',
            align: 'center'
          })
          this.fullText.anchor.set(0.5, 0.5)
          this.fullText.x = this.btGift.x
          this.fullText.y = this.btGift.y
          this.app1.stage.addChild(this.fullText)
          this.caluTime.visible = false
          this.caluObj.visible = false
          this.total = 3;
          h = 88;
          this.changeFlag = false
          return
        } 
        if (that.fullText && that.fullText.x) {
          that.app1.stage.removeChild(that.fullText)
          that.caluTime.visible = true
          that.caluObj.visible = true
        }
        if (that.shineFlag) { //表示点击了
          that.total
        } else {
          that.total--
        }
        that.shineFlag = false
        that.caluObj.text = that.total
        let h = that.total / 3 * 88
        that.changeFlag = true
        console.log(that.total)
        if (!firstZero && arr.length == 0) { //当
          firstZero = 1
        }
        if (that.total == 0) {
          that.dropGift(arr);
          setTimeout(()=> {
            that.total = 3
            that.caluObj.text = that.total
            h = 88;
            that.changeFlag = false
          }, 1000)
        }
      }, 1000)
    },
    dropGift (arr) {
      //将掉箱子的作为一个数组
      let newArr = [], giftList = [], handObj, giftFlag = false
      if (this.step == 2 && !arr) { //第一次掉三个
        if (this.boxList[1].rank) {
          newArr = [2, 4, 5]
        } else {
          newArr = [1, 4, 5]
        }
      } else {
        let o = arr[this.randomInt(arr.length)]
        newArr = [o]
      }
      
      for (let m = 0; m < newArr.length; m++) {
        let n = newArr[m]
        this.$set(this.boxList[n],'occupied',true);
        let gift = new PIXI.Sprite.fromImage(CONFIG.btGift)
        this.app1.stage.addChild(gift)
        gift.anchor.set(0.5, 0.5)
        gift.width = 98
        gift.height = 88
        gift.x = this.boxList[n].x;
        gift.parentGroup = this.group3
        const tween = PIXI.tweenManager.createTween(gift)
        tween.from({y: 0}).to({y: this.boxList[n].y - 8})
        tween.easing = PIXI.tween.Easing.outBounce()
        tween.time = 500 + m * 200
        gift.interactive = true

        const tween3 = PIXI.tweenManager.createTween(gift)
        tween3.easing = PIXI.tween.Easing.linear()
        tween3.from({scale: {x: 1, y: 1}})
        tween3.to({scale: {x: 1.1, y: 1.1}})
        tween3.time = 500
        tween3.pingPong = true
        tween3.loop = false
        // 闪动动画
        tween.start().chain(tween3)
        tween.on('end', () => {
          if (newArr.length > 1 && !giftFlag) {
            handObj = this.createHandTween(this.boxList[5].x + 20, this.boxList[5].y + this.boxList[5].height / 2 - 20)
            handObj.tween.start()
            giftFlag = true
          }
        })
        let timer = null;
        clearInterval(timer)
        timer = setInterval(function(){
          const tween2 = PIXI.tweenManager.createTween(gift)
          tween2.from({rotation: 0}).to({rotation: 0.3})
          tween2.time = 160
          tween2.easing = PIXI.tween.Easing.inOutBack()
          tween2.pingPong = true
          tween2.repeat = 3
          tween2.expire = true;
          tween2.start()
        }, 5000)

        // 打开
        let that = this
        gift.on('pointerdown', () => {
          PIXI.tweenManager.removeTween(tween)
          if (handObj && handObj.tween && handObj.tween.time) {
            handObj.tween.stop()
            PIXI.tweenManager.removeTween(handObj.tween)
            that.app1.stage.removeChild(handObj.sprite)
            that.step++
          }
          that.spillAction(100, 100, gift.x, that.boxList[n].y, [CONFIG.circleUrl, CONFIG.lightCircle], CONFIG.lightEmitterConfig)
          that.app1.stage.removeChild(gift)
          that.createPlane(1, n)
          if (!that.isFlag) {
            that.hand_tween2 =  that.createHandTween(that.btGift.x + 20, that.btGift.y + 10) // 点击指示
            that.hand_tween2.tween.start()
            that.isFlag = true
          }
        })
      }
    },
    winSuccess (a) {
      let successContainer = new PIXI.Container(), that = this, giftList = [], tweenList = []
      let endInt
      successContainer.interactive = true
      successContainer.on('pointerdown', () => {
        clearInterval(endInt)
        this.linkAppStore()
      })

      this.app1.stage.addChild(successContainer)
      that.status = {
        flag: 3,
        rank: a
      }
      let bgSprite = new PIXI.Sprite.fromImage(CONFIG.bodyBg)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = this.app1.screen.width
      bgSprite.height = this.app1.screen.height
      successContainer.addChild(bgSprite)
      successContainer.parentGroup = this.group4
      let lightBg = new PIXI.Sprite.fromImage(CONFIG.successLight)
      lightBg.width = 414
      lightBg.height = 455
      lightBg.anchor.set(0.5)
      lightBg.x = this.app1.screen.width / 2
      lightBg.y = this.isLandscape ? this.app1.screen.height / 2 : this.app1.screen.height / 2 - 20
      successContainer.addChild(lightBg)
      
      const lightBg_tween = PIXI.tweenManager.createTween(lightBg)
      lightBg_tween.from({rotation:0})
      lightBg_tween.to({rotation:2 * Math.PI})
      lightBg_tween.time = 9000
      lightBg_tween.loop = true
      lightBg_tween.start()
    
      // 盒子先出来，logo出来，底部黑字和按钮出来
      let arr = [
        [{x: -20, y: 0}, {x: 76, y: 180}],
        [{x: -80, y: 180}, {x: 48, y: 500}],
        [{x: 0, y: this.app1.screen.height + 40}, {x: 120, y: this.app1.screen.height}],
        [{x: this.app1.screen.width + 200, y: this.app1.screen.height + 120}, {x: this.app1.screen.width, y: this.app1.screen.height}],
        [{x: this.app1.screen.width + 200, y: 0}, {x: this.app1.screen.width, y: this.app1.screen.height - 270}],
        [{x: this.app1.screen.width, y: 0}, {x: this.app1.screen.width, y: 300}],
        [{x: this.app1.screen.width + 200, y: -275}, {x: this.app1.screen.width - 100, y: 120}]
      ]

      for(let i = 0; i < 7; i++) {
        let gift = new PIXI.Sprite.fromImage(CONFIG.boxList[i].url)
        gift.anchor.set(1, 1)
        gift.width = CONFIG.boxList[i].w
        gift.height = CONFIG.boxList[i].h
        giftList.push(gift)
        successContainer.addChild(gift)
        let tw = PIXI.tweenManager.createTween(gift)
        tw.time = CONFIG.boxList[i].time
        tw.easing = PIXI.tween.Easing.inOutExpo()
        tweenList.push(tw)
        tw.from(arr[i][0]).to(arr[i][1])
        tw.start()
      }
      let blogo = this.isEnglish ? CONFIG.bigLogo : CONFIG['logo_ch']
      let bigLogo = new PIXI.Sprite.fromImage(blogo)
      bigLogo.anchor.set(0.5, 1)
      bigLogo.x = this.app1.screen.width / 2
      if (that.isEnglish) {
        bigLogo.width = this.isLandscape ? 138 : 226
        bigLogo.height = this.isLandscape ? 98 : 160
      } else {
        bigLogo.width = this.isLandscape ? 260 : 260
        bigLogo.height = this.isLandscape ? 80 : 80
      }
      successContainer.addChild(bigLogo)
      const tween_logo = PIXI.tweenManager.createTween(bigLogo)
      tween_logo.from({y: - bigLogo.height / 2}).to({y: this.isLandscape ? lightBg.y / 2 + 20: lightBg.y / 2 + bigLogo.height / 2})
      tween_logo.time = 2400
      tween_logo.easing = PIXI.tween.Easing.inBounce()
      tween_logo.start()
      tween_logo.on('end', () => {
        let blackTxt 
        // 黑字  按钮
        if (that.isEnglish) {
          blackTxt = new PIXI.Text('MERGE,PLANE,AND DEVELOP\nYOUR OWN TEAM!', {
            fontFamily: this.isEnglish ? 'NumFont' : 'ch_numFont',
            fontSize: this.isLandscape  ? 32 : 26,
            align: 'center',
            fill: 'black',
            lineHeight: 40,
            // letterSpacing: 4,
            stroke: '#ffffff',
            strokeThickness: 6
          })
        } else {
          blackTxt = new PIXI.Sprite.fromImage(CONFIG.blackTxt)
          blackTxt.width = 350
          blackTxt.height = 30
        }
        blackTxt.anchor.set(0.5)
        blackTxt.x = this.app1.screen.width / 2
        blackTxt.y = this.isLandscape ? lightBg.y + 86 : lightBg.y + 160
        successContainer.addChild(blackTxt)
        // 按钮
        this.installBtn({
          url: this.isEnglish ? CONFIG.continueBtn : CONFIG.continueBtn_ch,
          width: this.isLandscape ? 138 : 252,
          height: this.isLandscape ? 60 : 133,
          y: this.isLandscape ? this.app1.screen.height - 40 : this.app1.screen.height - 140,
          parentCont: successContainer
        }).on('pointerdown', () => {
          this.linkAppStore()
        })
        let shiningBtn = this.installBtn({
          url: this.isEnglish ? CONFIG.shineContinueBtn : CONFIG.shineContinueBtn_ch,
          width: this.isLandscape ? 138 : 252,
          height: this.isLandscape ? 60 : 133,
          y: this.isLandscape ? this.app1.screen.height - 40 : this.app1.screen.height - 140,
          parentCont: successContainer
        })
        shiningBtn.alpha = 0
        const shine_tween2 = PIXI.tweenManager.createTween(shiningBtn)
        shine_tween2.time = 2000
        shine_tween2.pingPong = true
        shine_tween2.loop = true
        shine_tween2.from({
          alpha: 0
        })
        shine_tween2.to({
          alpha: 1
        })
        shine_tween2.easing = PIXI.tween.Easing.linear()
        shine_tween2.start()
      })

      // const tween_1 = PIXI.tweenManager.createTween(plane)
      // tween_1.time = 400
      // tween_1.easing = PIXI.tween.Easing.linear()
      // tween_1.loop = false
      // tween_1.from({scale: {x: 0.4, y: 0.4}})
      // tween_1.to({scale: {x: 0, y: 0}})

      let timeList = []
      let planeArr = []
      
      let bigPlane = new PIXI.Sprite.fromImage(CONFIG.successPlane[0])
        bigPlane.anchor.set(0.5, 0.5)
        successContainer.addChild(bigPlane)
        bigPlane.width = 168
        bigPlane.height = 120
        bigPlane.x = lightBg.x
        bigPlane.y = lightBg.y
        bigPlane.scale.set(0);
      for (let i = 0; i < 5; i++) {
        planeArr.push(PIXI.Texture.fromImage(CONFIG.successPlane[i]))
      }
      let startScale = 0.5, endScale = 0,time = 2400,pingPong = true;
      let textureIdx = 0;
      const tween_2 = PIXI.tweenManager.createTween(bigPlane)
      tween_2.time = time;
      tween_2.easing = PIXI.tween.Easing.outCirc()
      tween_2.from({scale: {x: startScale, y: startScale}})
      tween_2.to({scale: {x: endScale, y: endScale}})
      tween_2.loop = true
      tween_2.pingPong= true
      tween_2.start()
      tween_2.on('pingpong',()=>{
        textureIdx += 1;
        let n = textureIdx%5 - 1
        n = n < 0 ? 0 : n
        planeArr[n].visible = false
        bigPlane.scale.set(endScale);
        bigPlane.texture = planeArr[textureIdx%5]   // 改变texture相当于改变图片的src
      })
    },
     /**
     * 粒子动画
     * @param w 容器宽度
     * @param h 容器高
     * @param x 容器坐标x
     * @param y 容器坐标y
     * @param imgArr 粒子动画所有图片
     * @param config 粒子动画config
     */
    spillAction (w = 100, h = 100, x, y, imgArr, config) {
      let coinTank = new PIXI.Container();
      coinTank.width = w;
      coinTank.height = h;
      coinTank.x = x;
      coinTank.y = y;
      this.app1.stage.addChild(coinTank);
      // Create a new emitter
      let emitter = new PIXI.particles.Emitter(
        coinTank,
        [...imgArr.map(v => PIXI.Texture.fromImage(v))],
        config
      )
      // Start emitting
      emitter.emit = true;
      emitter.playOnceAndDestroy(() => {
        this.app1.stage.removeChild(coinTank)
      })
    },
    linkAppStore() {
      let url = this.iosLink;
      let userAgent = navigator.userAgent || navigator.vendor;
      if (/android/i.test(userAgent)) {
        url = this.androidLink;
      }
      try {
        dapi.openStoreUrl();
        console.log('dapi open store')
        // mraid.open(url);          // ad平台【AdColony Applovin Vungle】
        // console.log('mraid open store')
      } catch (err) {
        console.error(err);
        window.location = url;
      }
    },
    createHandTween (x, y) {
      let obj = {
        sprite: '',
        tween: ''
      }
      obj.sprite = new PIXI.Sprite.fromImage(CONFIG.iconHand)
      obj.sprite.anchor.set(0.5, 0.5)
      obj.sprite.width = 62
      obj.sprite.height = 62
      obj.sprite.y = this.boxList[1].y + 30
      obj.sprite.x = this.boxList[1].x + this.boxList[1].width / 2 - 8
      this.app1.stage.addChild(obj.sprite)
      obj.sprite.parentGroup = this.group3

      obj.tween = PIXI.tweenManager.createTween(obj.sprite)
      obj.sprite.y = y //this.boxList[5].y + this.boxList[5].height / 2 - 20
      obj.sprite.x = x //this.boxList[5].x
      obj.sprite.scale.set(1.2)
      obj.tween.from({scale: {x: 1.2,y: 1.2}})
      obj.tween.to({
        scale: {
          x: 1,
          y: 1
        }
      })
      // obj.tween.delay = 1400
      obj.tween.time = 800
      obj.tween.loop = true
      obj.tween.pingPong = true
      obj.tween.easing = PIXI.tween.Easing.linear()
      return obj
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" rel="stylesheet">
#mergePlane{
  background: #000;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
