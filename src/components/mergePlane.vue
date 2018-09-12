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
      app1: {},
      distanceX: 110,
      distanceY: 10,
      boxList: [],
      step: 1,
      logoH: 120, // logo的y
      btnH: 0, // 安装btn的y
      bthHeight: 58,
      tween1: '',
      tween2: '',
      handSprite: {}, //手sprite
      planeList: new Array(6).fill({}),  // 飞机sprite
      shadeSprite: {}, // 背景遮罩
      hintSprite: {}, // 文字提示遮罩
      animateTxt: {},
      oldPosition: {x: 0, y: 0},
      grade: 0, // 合成飞机的等级
      fixedBox: [], // 高亮盒子
      roundBox: {}, // 高亮背景
      clearTime: '', // 大循环
      smClear: '',  // 小循环
      isEmpty: '',
      total: 3,
      status: {},  // 1正常  2 小成功 3 大成功
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
      isFlag: false
    }
  },
  watch: {
    step (newval, oldval) {
      if (oldval >= 3) {
        // this.handSprite.visible = false
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
    let font = new FontFaceObserver('NumFont')
    font.load().then(function () {
      console.log('has loaded')
    })
  },
  mounted () {
    this.getSize()
    this.initPIXI()
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
        // scaleTowindow(this.app1.renderer.view)
        if (window.innerWidth > window.innerHeight) {
          that.isLandscape = true
        } else {
          that.isLandscape = false
        }
        // console.log(that.isLandscape)
        let canvas = document.querySelector('#mergePlane canvas');
        canvas.parentNode.removeChild(canvas);
        let obj = {}
        obj.step = that.step
        obj.status = that.status
        
        if (!obj.status.flag) {
          that.step = 1
          that.initPIXI()
        } else if (obj.status.flag == 1) {
          that.app1.stage.removeChild(that.newTexture)
          clearInterval(that.clearTime)
          let newArr = that.boxList.slice(0)
          let numObj = {}
          for (let key in that.caluObj) {
            numObj[key] = that.caluObj[key]
          }
          that.initPIXI()
          that.GiftScene.visible = true
          that.boxList.forEach((v, i) => {
            if (!newArr[i].rank && !newArr[i].occupied ) {
              v.rank = 0
              v.occupied = false
            } else if (newArr[i].rank) {
              that.createPlane(newArr[i].rank, i)
            } else if (!newArr[i].rank && newArr[i].occupied) {
            }
          })
          that.total = 3
          that.cycleFuc()
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
        antialias: true
      })
      document.getElementById('mergePlane').appendChild(this.app1.view);
      let canvas = document.querySelector('#mergePlane canvas');
      canvas.style.width = '100%'
      canvas.style.height = '100%'
      this.app1.renderer.view.style.position = 'absolute'
      this.app1.renderer.view.style.display = 'block'
      this.app1.renderer.autoResize = true
      // this.app1.renderer.resize(document.documentElement.clientWidth, document.documentElement.clientHeight)
      container = new PIXI.Container()
      this.app1.renderer.render(container)

      this.app1.stage = new PIXI.display.Stage()
      this.app1.stage.group.enableSort = true
      this.group1 = new PIXI.display.Group(2, true) 
      this.group2 = new PIXI.display.Group(2, true) // 飞机层
      this.group3 = new PIXI.display.Group(3, true) //合成成功层
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
        url: CONFIG.installBtn,
        width: 148,
        height: this.bthHeight,
        y: this.isLandscape ? this.app1.screen.height - 40: this.app1.screen.height - 116,
        parentCont: this.app1.stage
      })
      this.downBtn.on('pointerdown', () => {
        this.linkAppStore()
      })
      this.shiningBtn = this.installBtn({
        url: CONFIG.shiningBtn,
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
      logoSprite = new PIXI.Sprite.fromImage(CONFIG.logo)
      logoSprite.width = this.isLandscape ? 350 : 210
      logoSprite.height = this.isLandscape ? 60 : 34
      logoSprite.x = (this.app1.screen.width - logoSprite.width) / 2
      logoSprite.y =  this.isLandscape ? 20 : this.logoH
      this.app1.stage.addChild(logoSprite)

      //底部盒子====================
      let GiftScene = new PIXI.Container(), nowTime, txtList = [], inter
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
        // fontStyle: 'italic',
        fill: '0xffae43',
        fontFamily: 'NumFont',
        align: 'center'
      })
      this.caluObj.anchor.set(0.5, 0.5)
      this.caluObj.x = caluTime.x
      this.caluObj.y = caluTime.y
      GiftScene.addChild(this.caluObj)
      console.log(this.caluObj)
      let times = 0
      this.caluObj.text = this.total
      GiftScene.visible = false
      this.GiftScene = GiftScene
    
      // 第三步就不要执行下面了
      if (this.step <= 3) {
        // 遮罩
        this.shadeSprite = new PIXI.Graphics()
        this.shadeSprite.beginFill(0x000000, 0.5)
        this.shadeSprite.drawRect(0, 0, this.app1.screen.width, this.app1.screen.height)
        this.app1.stage.addChild(this.shadeSprite)
    
        // 文字提示
        this.hintSprite = new PIXI.Sprite.fromImage(CONFIG.iconHint)
        this.hintSprite.anchor.set(0.5, 0.5)
        this.hintSprite.width = this.isLandscape ? 260 : 330
        this.hintSprite.height = this.isLandscape ? 100 : 126
        this.app1.stage.addChild(this.hintSprite)
        this.hintSprite.x = this.app1.screen.width / 2
        this.hintSprite.y = this.downBtn.y + this.downBtn.height / 2 - this.hintSprite.height / 2

        this.animateTxt = new PIXI.Text('SWIPE TO PLAY!', {
          fontSize: 28,
          fontWeight: 'normal',
          fontStyle: 'normal',
          fill: '0x000000',
          fontFamily: 'NumFont',
          align: 'center'
        })
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
        for(let i = 0; i < 2; i++) {
          if (this.step > 2) {
            return
          }
          let sprite = new PIXI.Sprite.fromImage(CONFIG.layBox)
          sprite.anchor.set(0.5, 0.5)
          sprite.width = 92
          sprite.height = 116
          sprite.x = (i % 2) * this.distanceX + this.boxList[0].x + this.distanceX
          sprite.y = this.boxList[0].y
          this.app1.stage.addChild(sprite)
          this.fixedBox.push(sprite)
          this.createPlane()
        }
        // 手指示
        this.handSprite = new PIXI.Sprite.fromImage(CONFIG.iconHand)
        this.handSprite.anchor.set(0.5, 0.5)
        this.handSprite.width = 62
        this.handSprite.height = 62
        this.handSprite.y = this.boxList[1].y + 30
        this.handSprite.x = this.boxList[1].x + this.boxList[1].width / 2 - 8
        this.app1.stage.addChild(this.handSprite)
        this.handSprite.parentGroup = this.group4
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
      this.blmask.on('pointerdown', () => {
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
  
        Vue.set(this.boxList, i, {x, y, rank, occupied, width, height})
      }
      // console.table(this.boxList)
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
      if (this.step < 3) {
        index = this.step
      }
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
      }

      this.play(plane)
      this.step ++
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
        if (that.step == 3) {
          that.tween1.stop()
          //that.tween1.on('end', () => {
            // that.tween.remove()
            PIXI.tweenManager.removeTween(that.tween1)
            that.app1.stage.removeChild(that.handSprite)
          //})
          that.app1.stage.removeChild(that.animateTxt)
          that.app1.stage.removeChild(that.hintSprite)
          // that.app1.stage.removeChild(that.handSprite)
        }
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
            // dragContainer.destroy()
            that.app1.stage.removeChild(dragContainer)
          }
          let planeObj2 = {}, planeObj1 = {}
          if (curIndex > -1 && that.boxList[curIndex].rank == this.pRank) {  // 飞机级别一样
            if (that.step == 3) { // 第一次当合并的时候
              that.shadeSprite.visible = false
              that.app1.stage.removeChild(that.roundBox)
              that.fixedBox.forEach(item => {
                that.app1.stage.removeChild(item)
              })
            }
            let rk = that.boxList[curIndex].rank, obj2 = that.boxList[onMoveing]
            obj2.rank = 0
            that.planeList[onMoveing].destroy()
            that.planeList[curIndex].destroy()
            Vue.set(that.planeList, onMoveing, {})
            that.$set(that.boxList[onMoveing],'occupied',false);
            that.$set(that.boxList[onMoveing], 'rank', 0)
            clearInterval(that.clearTime)
            that.spillAction(100, 100, that.boxList[curIndex].x, that.boxList[curIndex].y, [CONFIG.circleUrl, CONFIG.starUrl], CONFIG.composeEmitterConfig)
            if (rk + 1 == 4) { // 最高级
              that.grade = rk + 1
              that.winSuccess(that.grade)
              that.rankBox.push(that.grade)
            } else {
              if (rk + 1 < that.grade) {  
                that.createPlane(rk + 1, curIndex)
                that.cycleFuc()
              } else if (rk + 1 == that.grade){
                let n = that.rankBox.findIndex(item => {
                  return item == that.grade
                }) 
                if (n > -1) {
                  that.createPlane(rk + 1, curIndex)
                  that.cycleFuc()
                  return
                } else {
                  that.passResult(that.grade, curIndex)
                }
              } else {
                that.grade = rk + 1
                that.rankBox.push(that.grade)
                that.passResult(that.grade, curIndex)
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
      let title = new PIXI.Sprite.fromImage(CONFIG.resultTitle)
      title.anchor.set(0.5, 0.5)
      title.width = 320
      title.height = 120
      title.x = this.app1.screen.width / 2
      title.y = this.isLandscape ? 100 : this.boxList[0].y
      resultContainer.addChild(title)
      title.parentGroup = this.group3

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

      let plane = new PIXI.Sprite.fromImage(CONFIG.planeList[this.grade - 1])
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
      // console.log(`%c ${plane.x}`, 'color:yellow')
      // console.log(this.app1.stage)
      plane.on('pointerdown', onDragStart)
      function onDragStart(event) {
        that.app1.stage.removeChild(resultContainer)
        PIXI.tweenManager.removeTween(result_tween)
        that.createPlane(r, i)
        that.status.flag = 1
        if (that.step == 4) {
          that.dropGift()

        }
        that.cycleFuc()
      }
    },
    cycleFuc () { // 底部盒子倒计时
      let firstZero, that = this //存在表示第一次没满
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
        // console.log(this.texture.baseTexture.height)
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
      if (this.step == 4 && !arr) { //第一次掉三个
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
            // handObj.tween.delay = 1500
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
      successContainer.parentGroup = this.group3
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
    
      // let plane = new PIXI.Sprite.fromImage(CONFIG.planeList[0])
      // plane.anchor.set(0.5, 0.5)
      // plane.width = 168
      // plane.height = 120
      // plane.x = lightBg.x
      // plane.y = lightBg.y
      // plane.interactive = true
      // successContainer.addChild(plane)

      // 盒子先出来，logo出来，底部黑字和按钮出来
      let arr = [
        [{x: -20, y: 0}, {x: 76, y: 180}],
        [{x: -80, y: 180}, {x: 48, y: 500}],
        [{x: 0, y: this.app1.screen.height + 40}, {x: 120, y: this.app1.screen.height}],
        [{x: this.app1.screen.width + 200, y: this.app1.screen.height + 120}, {x: this.app1.screen.width, y: this.app1.screen.height}],
        [{x: this.app1.screen.width + 200, y: 0}, {x: this.app1.screen.width, y: this.app1.screen.height - 270}],
        [{x: this.app1.screen.width, y: 0}, {x: this.app1.screen.width - 20, y: 200}],
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

      let bigLogo = new PIXI.Sprite.fromImage(CONFIG.bigLogo)
      bigLogo.anchor.set(0.5, 1)
      bigLogo.x = this.app1.screen.width / 2
      bigLogo.width = this.isLandscape ? 138 : 226
      bigLogo.height = this.isLandscape ? 98 : 160
      successContainer.addChild(bigLogo)
      const tween_logo = PIXI.tweenManager.createTween(bigLogo)
      tween_logo.from({y: - bigLogo.height / 2}).to({y: this.isLandscape ? lightBg.y / 2 + 20: lightBg.y / 2 + bigLogo.height / 2})
      tween_logo.time = 2400
      tween_logo.easing = PIXI.tween.Easing.inBounce()
      tween_logo.start()
      tween_logo.on('end', () => {
        // 黑字  按钮
        let blackTxt = new PIXI.Text('MERGE,PLANE,AND DEVELOP\nYOUR OWN TEAM!', {
          fontFamily: 'NumFont',
          fontSize: this.isLandscape  ? 32 : 26,
          align: 'center',
          fill: 'black',
          lineHeight: 40,
          // letterSpacing: 4,
          stroke: '#ffffff',
          strokeThickness: 6
        })
        blackTxt.anchor.set(0.5)
        blackTxt.x = this.app1.screen.width / 2
        blackTxt.y = this.isLandscape ? lightBg.y + 86 : lightBg.y + 160,
        successContainer.addChild(blackTxt)
        // 按钮
        this.installBtn({
          url: CONFIG.continueBtn,
          width: this.isLandscape ? 138 : 252,
          height: this.isLandscape ? 60 : 133,
          y: this.isLandscape ? this.app1.screen.height - 40 : this.app1.screen.height - 140,
          parentCont: successContainer
        }).on('pointerdown', () => {
          this.linkAppStore()
        })
        let shiningBtn = this.installBtn({
          url: CONFIG.shineContinueBtn,
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
        // dapi.openStoreUrl();
        // console.log('dapi open store')
        mraid.open(url);          // ad平台【AdColony Applovin Vungle】
        console.log('mraid open store')
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
      obj.sprite.parentGroup = this.group4

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
<style scoped lang="scss" rel="stylesheet">
@font-face {
  font-family: 'NumFont';
  src: url('data:font/truetype;base64, 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CWFmaWkxMDAzNAlhZmlpMTAwMzUJYWZpaTEwMDM2CWFmaWkxMDAzNwlhZmlpMTAwMzgJYWZpaTEwMDM5CWFmaWkxMDA0MAlhZmlpMTAwNDEJYWZpaTEwMDQyCWFmaWkxMDA0MwlhZmlpMTAwNDQJYWZpaTEwMDQ1CWFmaWkxMDA0NglhZmlpMTAwNDcJYWZpaTEwMDQ4CWFmaWkxMDA0OQlhZmlpMTAwNjUJYWZpaTEwMDY2CWFmaWkxMDA2NwlhZmlpMTAwNjgJYWZpaTEwMDY5CWFmaWkxMDA3MAlhZmlpMTAwNzIJYWZpaTEwMDczCWFmaWkxMDA3NAlhZmlpMTAwNzUJYWZpaTEwMDc2CWFmaWkxMDA3NwlhZmlpMTAwNzgJYWZpaTEwMDc5CWFmaWkxMDA4MAlhZmlpMTAwODEJYWZpaTEwMDgyCWFmaWkxMDA4MwlhZmlpMTAwODQJYWZpaTEwMDg1CWFmaWkxMDA4NglhZmlpMTAwODcJYWZpaTEwMDg4CWFmaWkxMDA4OQlhZmlpMTAwOTAJYWZpaTEwMDkxCWFmaWkxMDA5MglhZmlpMTAwOTMJYWZpaTEwMDk0CWFmaWkxMDA5NQlhZmlpMTAwOTYJYWZpaTEwMDk3BEV1cm8LLm5vdGRlZi4wMDEAAAAAAAAB//8AAgABAAAADAAAABYAAAACAAEAAwC9AAEABAAAAAIAAAAAAAEAAAAKABwAHgABbGF0bgAIAAQAAAAA//8AAAAAAAAAAQAAAAoAHgAsAAFsYXRuAAgABAAAAAD//wABAAAAAWtlcm4ACAAAAAEAAAABAAQAAgAAAAEACAABA1AABAAAACkAXACCAIgApgCwALoA0ADaAOAA8gD4AP4BDADaASYBMAFGAWABdgGEAZ4BpAG2AdQB+gIgAjYCRAJKAlQCXgKEAqoCsALSAuAC9gMAAwoDJAMyAAkARP/gAE3/hwBQ/+AAV/+tAFn/7ABa/94AW/+1AFz/wQBd/8YAAQAR/3wABwAQ/+YARv/0AEoACQBW//IAWf/gAFr/1QBc/94AAgAQ/74AUv/0AAIAD/+kABH/pwAFAA//egAR/3wARP/kAE3/kABYAAsAAgAP/74AEf/EAAEAEP/VAAQAEP/pAFb/4wBX/7sAXP/XAAEAEP/gAAEAEP/3AAMAD//EABH/yQBb/+4ABgAP/1cAEP/1ABH/XQBE/94ATf+QAFD/5AACAA//yQAR/9IABQAP/7gAEP+qABH/sABE/+AATf/MAAYAD/+wABD/2AAR/7sARP/PAE3/uABY//EABQAP/74AEP/gABH/uwBE/9UATf/KAAMAEP+4AFL/3QBU/90ABgAP/3MAEP+1ABH/jQBE/7gATf+LAFj/+AABABD/wQAEAKD/vgCh/9IApf/bAKj/2AAHAA//zwAR/9IAjv/0AJL/5gCZ/+wAmv/vAKH/xgAJAA//3gAR/+AAjv/0AJL/7ACU/+8Amv/1AKD/2ACh/88Ao//bAAkAD/+QABH/kACO/7gAkv+kAJn/uwCa/7sAnP/aAJ//5gCt/9IABQCc/+YAn//dAKD/vgCh/8IApf++AAMAnP/hAJ//1wCi/8cAAQCc/+YAAgCc//IAof/gAAIAnP/1AKH/0gAJAA//zwAR/9gAkv/PAJT/xwCZ/+kAmv/mAKD/7wCh/+MAo//UAAkAD/9oABH/XACO/8cAkv+cAJT/2ACZ/8EAmv/NAKP/4QCt/+wAAQCc/+MACAAP/7UAEf+wAI7/uACZ/7UAmv+7AJz/2ACf/+MArf/QAAMAD/98ABH/fwCS/5MABQAP/3cAEf95AJT/rQCh/80Ao/+1AAIAoP/bAKH/2AACAKD/1QCh/80ABgAP/+AAEf/mAJT/4ACg/7MAof+1AKP/0gADAA//1QAR/+YAof+yAAcAD//BABH/xACO/9gAkv/BAJn/2wCa/9UAof/gAAEAKQAQACIARABGAEcASQBKAE4ATwBQAFEAUgBTAFUAVgBXAFkAWgBbAFwAXQCOAI8AkACRAJIAlACYAJkAmgCcAJ4AnwCgAKEAogCkAKcAqACqAKwAAAAAAAEAAAAA0yuCHgAAAADEiGOFAAAAAMnv7TY=');
  font-style: normal;
  font-weight: bold;
}
</style>
