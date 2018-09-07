<template>
  <div id="mergePlane">
    <!-- <svg height="0" width="0">
      <clipPath>
      </clipPath>
    </svg> -->
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
export default {
  name: 'mergePlane',
  data () {
    return {
      app1: {},
      distanceX: 110,
      distanceY: 10,
      boxList: [],
      step: 1,
      logoH: 60, // logo的y
      btnH: 0, // 安装btn的y
      bthHeight: 58,
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
      downBtn: {}
    }
  },
  watch: {
    step (newval, oldval) {
      if (oldval > 4) {
        this.handSprite.visible = false
      }
    },
    total: {
      handler(newval, oldval) {
        let blurFilter = new PIXI.filters.BlurFilter()
        if (oldval != newval) {
          this.caluObj.text = newval
        }
        if (newval == 1) {
          this.downBtn.filters = [blurFilter]
          blurFilter.blur = 5 * Math.cos(0.9)
        } else {
          this.downBtn.filters = null
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
        console.log(that.isLandscape)
        let canvas = document.querySelector('#mergePlane canvas');
        canvas.parentNode.removeChild(canvas);
        let obj = {}
        obj.step = that.step
        obj.status = that.status
        
        if (!obj.status.flag) {
          that.step = 1
          that.initPIXI()
        } else if (obj.status.flag == 1) {
          let newArr = that.boxList.slice(0)
          let numObj = {}
          for (let key in that.caluObj) {
            numObj[key] = that.caluObj[key]
          }
          that.initPIXI()

          that.boxList.forEach((v, i) => {
            if (!newArr[i].rank && !newArr[i].occupied ) {
              v.rank = 0
              v.occupied = false
            } else if (newArr[i].rank) {
              that.createPlane(newArr[i].rank, i)
              that.total = 3
              that.cycleFuc()
            } else if (!newArr[i].rank && newArr[i].occupied) {
              that.total = 3
              that.cycleFuc()
            }
          })

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
      this.app1 = new PIXI.Application(414, 736, {
        antialias: true
      })
      document.getElementById('mergePlane').appendChild(this.app1.view);
      // let canvas = document.querySelector('#mergePlane canvas');
      // canvas.style.width = '100%'
      // canvas.style.height = '100%'
      this.app1.renderer.view.style.position = 'absolute'
      this.app1.renderer.view.style.display = 'block'
      this.app1.renderer.autoResize = true
      this.app1.renderer.resize(document.documentElement.clientWidth, document.documentElement.clientHeight)
      container = new PIXI.Container()
      this.app1.renderer.render(container)
      bgSprite = new PIXI.Sprite.fromImage(CONFIG.bodyBg)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = this.app1.screen.width
      bgSprite.height = this.app1.screen.height
      this.app1.stage.addChild(bgSprite)
      this.layBox()
      this.downBtn = this.installBtn({
        url: CONFIG.installBtn,
        width: 146,
        height: this.bthHeight,
        y: this.isLandscape ? this.app1.screen.height - 50 : this.boxList[3].y + 240,
        parentCont: this.app1.stage
      })
      this.downBtn.on('pointerdown', () => {
        this.linkAppStore()
      })

      logoSprite = new PIXI.Sprite.fromImage(CONFIG.logo)
      logoSprite.width = this.isLandscape ? 350 : 280
      logoSprite.height = this.isLandscape ? 50 : 40
      logoSprite.x = (this.app1.screen.width - logoSprite.width) / 2
      logoSprite.y =  this.isLandscape ? 20 : this.logoH
      this.app1.stage.addChild(logoSprite)
      // console.log(this.app1)
      if (this.step >= 3) {
        return 
      }
      // 遮罩
      this.shadeSprite = new PIXI.Graphics()
      this.shadeSprite.beginFill(0x000000, 0.5)
      this.shadeSprite.drawRect(0, 0, this.app1.screen.width, this.app1.screen.height)
      this.app1.stage.addChild(this.shadeSprite)
  
      // 文字提示
      this.hintSprite = new PIXI.Sprite.fromImage(CONFIG.iconHint)
      this.hintSprite.anchor.set(0.5, 0.5)
      this.hintSprite.width = this.isLandscape ? 300 : 330
      this.hintSprite.height = this.isLandscape ? 114 : 126
      this.app1.stage.addChild(this.hintSprite)
      this.hintSprite.x = this.app1.screen.width / 2
      this.hintSprite.y = this.btnH - this.bthHeight / 2

      this.animateTxt = new PIXI.Text('SWIPE TO PLAY!', {
        fontSize: 28,
        fontWeight: 'normal',
        fontStyle: 'italic',
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
      // this.animateTxt.width = 24
      // this.animateTxt.height = 16
      this.app1.stage.addChild(this.animateTxt)
      tween_txt.start()

      //高亮
      let roundBox = new PIXI.Graphics()
      roundBox.beginFill(0x0d95f9)
      roundBox.drawRoundedRect(0, 0, this.boxList[0].width + this.distanceX + 20, this.distanceY + this.boxList[0].height)
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
      this.handSprite.anchor.set(1, 1)
      this.handSprite.width = 62
      this.handSprite.height = 62
      this.handSprite.y = this.boxList[2].y + this.boxList[2].height / 2
      this.handSprite.x = this.boxList[1].x + this.boxList[1].width + 16
      this.app1.stage.addChild(this.handSprite)
      this.tween1 = PIXI.tweenManager.createTween(this.handSprite)
      this.tween1.from({x: this.boxList[1].x + this.boxList[1].width - 18}).to({x: this.boxList[1].x + this.boxList[1].width + 40})
      this.tween1.time = 1000
      this.tween1.loop = true
      this.tween1.easing = PIXI.tween.Easing.linear()
      this.tween1.on('start', () => {
        console.log('start')
      })
      this.tween1.start()
      this.app1.ticker.add(delta => {
        PIXI.tweenManager.update()
      })
      console.log(this.app1)
    },
    layBox () { // 降落块
      for (let i = 0; i < 6; i++) {
        let sprite = new PIXI.Sprite.fromImage(CONFIG.layBox)
        sprite.anchor.set(0.5, 0.5)
        sprite.width = this.isLandscape ? 80 : 92
        sprite.height = this.isLandscape ? 80 : 116
        this.app1.stage.addChild(sprite)
        
        let x = this.isLandscape ? this.app1.screen.width / 2 - 2 * this.distanceX - this.distanceX / 2 + i * this.distanceX : (i % 3) * this.distanceX + this.app1.screen.width / 2 - this.distanceX;
        let y = this.isLandscape ? this.logoH + 80 : Math.floor(i / 3) * (this.distanceY + sprite.height) + this.logoH + 120
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

      console.log(r, index)
      
      plane = new PIXI.Sprite.fromImage(CONFIG.planeList[r - 1])
      plane.anchor.set(0.5, 0.5)
      plane.width = 80
      plane.height = 80
      plane.pName = Math.random().toString(36).substr(2)
      plane.pRank = r
      plane.x = this.boxList[index].x
      plane.y = this.boxList[index].y - 8
      this.app1.stage.addChild(plane)
      Vue.set(this.planeList, index, plane)
      Vue.set(this.boxList[index], 'rank', r);
      this.$set(this.boxList[index],'occupied',true); // occupied 表示占位
      plane.interactive = true
      this.play(plane)
      this.step ++
    },
    randomInt (len) {
      return Math.floor(Math.random() * len)
    },
    play(obj) { // 合成升级
      let that = this, onMoveing, curIndex;
      obj.buttonMode = true
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
          //})
          that.shadeSprite.visible = false
          that.app1.stage.removeChild(that.animateTxt)
          that.app1.stage.removeChild(that.hintSprite)
          that.app1.stage.removeChild(that.handSprite)
          that.app1.stage.removeChild(that.roundBox)
          that.fixedBox.forEach(item => {
            that.app1.stage.removeChild(item)
          })
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
        console.log(that.boxList);
        if (this.dragging) {
          // that.container1.children[0].destroy()
          that.container1.destroy(true, true)
          that.app1.stage.removeChild(that.container1)
          console.log(that.container1)
          let endPos = this.data.getLocalPosition(this.parent), minArr = [], resultArr = [], equalArr = []
          let onMoveing, curIndex;
           onMoveing = that.planeList.findIndex(item => {
            return item.pName == this.pName
          })
          curIndex = DetectValid(endPos.x,endPos.y,50,onMoveing);
          console.log(`%c ${curIndex}`,'color:deeppink');

          if (curIndex > -1 && that.boxList[curIndex].rank == this.pRank) {  // 飞机级别一样
            let rk = that.boxList[curIndex].rank, obj2 = that.boxList[onMoveing]
            obj2.rank = 0
            that.createHighlight(curIndex, equalArr)
            that.planeList[onMoveing].destroy()
            that.planeList[curIndex].destroy()
            Vue.set(that.planeList, onMoveing, {})
            that.$set(that.boxList[onMoveing],'occupied',false);
            that.$set(that.boxList[onMoveing], 'rank', 0)
            clearInterval(that.clearTime)
            that.spillAction(100, 100, that.boxList[curIndex].x, that.boxList[curIndex].y, [CONFIG.circleUrl, CONFIG.starUrl], CONFIG.composeEmitterConfig)
            if (rk + 1 == 5) { // 最高级
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
            that.app1.stage.removeChild(that.container1)
          } else {
            let planeObj2 = that.planeList[onMoveing]
            console.log(planeObj2.x)
            planeObj2.x = that.oldPosition.x
            planeObj2.y = that.oldPosition.y
            console.log(planeObj2.x)
            Vue.set(that.planeList, onMoveing, planeObj2)
          }
        }
        this.data = null
        this.dragging = false
      }
      function DetectValid(x,y,r,idx){
          return that.boxList.findIndex((item,index) => {
            let tx = item.x;
            let ty = item.y;
            let dis = Math.sqrt(Math.pow(Math.abs(x - tx), 2) + Math.pow(Math.abs(y - ty), 2));
            if(dis < r && index !== idx){
              console.log(`%c tx:${x} ty:${y}`,'color:green');
              return true;
            }
          })
      }

      function onDragMove (event) {
        if (this.dragging) {
          let endPos = this.data.getLocalPosition(this.parent), minArr = [], resultArr = [], equalArr = []
          this.x = endPos.x
          this.y = endPos.y
           onMoveing = that.planeList.findIndex(item => {
            return item.pName == this.pName
          })
          curIndex = DetectValid(endPos.x,endPos.y,50,onMoveing);
          console.log(`%c ${curIndex}`,'color:deeppink');

          that.boxList.forEach((item, index) => {
            if (index != onMoveing && item.rank == this.pRank) {
              equalArr.push(index)
            }
          })
          if (curIndex > -1 && that.boxList[curIndex].rank == this.pRank) {  // 飞机级别一样
            that.createHighlight(curIndex, equalArr)
          } else {
            return
          }
        }
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

      let title = new PIXI.Sprite.fromImage(CONFIG.resultTitle)
      title.anchor.set(0.5, 0.5)
      title.width = 320
      title.height = 120
      title.x = this.app1.screen.width / 2
      title.y = this.isLandscape ? 100 : this.boxList[0].y
      resultContainer.addChild(title)

      let lightBg = new PIXI.Sprite.fromImage(CONFIG.resultLight)
      lightBg.width = 302
      lightBg.height = 302
      lightBg.anchor.set(0.5)
      lightBg.x = this.app1.screen.width / 2
      lightBg.y = title.y + lightBg.height / 2
      resultContainer.addChild(lightBg)

      this.app1.ticker.add(delta => {
        lightBg.rotation += 0.01
      })

      let plane = new PIXI.Sprite.fromImage(CONFIG.planeList[this.grade - 1])
      plane.anchor.set(0.5, 0.5)
      plane.width =  120 
      plane.height = 120 
      plane.x = lightBg.x
      plane.y = lightBg.y
      plane.interactive = true
      resultContainer.addChild(plane)
      plane.on('pointerdown', onDragStart)
      function onDragStart(event) {
        resultContainer.destroy()
        that.app1.stage.removeChild(resultContainer)
        that.createPlane(r, i)
        that.status.flag = 1
        that.cycleFuc()
      }
    },
    cycleFuc () { // 底部盒子倒计时
      let GiftScene = new PIXI.Container(), nowTime, txtList = [], inter
      this.app1.stage.addChild(GiftScene)

      let btGift = new PIXI.Sprite.fromImage(CONFIG.btGift_bottom)
      btGift.anchor.set(0.5, 0.5)
      btGift.width = 98
      btGift.height = 88
      btGift.x = this.app1.screen.width / 2
      btGift.y = this.isLandscape ? this.btnH - btGift.height + 10: this.btnH - btGift.height + 10
      GiftScene.addChild(btGift)


      // 黑色礼盒遮罩
      let blmask = new PIXI.Sprite.fromImage(CONFIG.blackMask)
      blmask.anchor.set(0.5, 0.5)
      blmask.width = 98
      blmask.height = 98
      blmask.x = this.app1.screen.width / 2
      blmask.y = btGift.y
      GiftScene.addChild(blmask)

      let texture = new PIXI.Texture.from(CONFIG.blackMask)
      let rect = new PIXI.Rectangle(0, 0, 70, 20)
      texture.frame = rect
      let newTexture = new PIXI.Texture(texture.baseTexture, texture.frame)
     
      

      // 计时盒子
      let caluTime = new PIXI.Sprite.fromImage(CONFIG.caluTime)
      caluTime.anchor.set(0.5, 0.5)
      caluTime.width = 42
      caluTime.height = 34
      caluTime.x = this.app1.screen.width / 2 - caluTime.width / 2 - caluTime.width - 10
      caluTime.y = this.btnH - 90 - 20
      GiftScene.addChild(caluTime)
      

      


      let that = this
      this.caluObj = new PIXI.Text(this.total, {
        fontSize: 24,
        fontWeight: 'normal',
        fontStyle: 'italic',
        fill: '0xffae43',
        fontFamily: 'NumFont',
        align: 'center'
      })
      this.caluObj.anchor.set(0.5, 0.5)
      this.caluObj.x = caluTime.x
      this.caluObj.y = caluTime.y
      // this.caluObj.width = 24
      // this.caluObj.height = 16
      GiftScene.addChild(this.caluObj)
      console.log(this.caluObj)
      let times = 0
      this.caluObj.text = this.total

      


      this.clearTime = setInterval(() => {
        let arr = []
        // console.table(this.boxList.map(v => v.occupied))
        this.boxList.forEach((item, index) => {
          if (!item.occupied) {
            arr.push(index)
          }
        })
        if(arr.length == 0){
          console.log('没有位置了')
          clearInterval(this.clearTime)
          clearInterval(this.inter)
          return
        }
        texture.frame.width -= 20
        this.total--
        this.caluObj.text = this.total
        // blmask.y -= blmask.height / 3
        if (this.total == 0) {
          // blmask.y = btGift.y
          this.dropGift(arr);
          // setTimeout(()=> {
          //   this.total = 3
          // }, 1000)
          this.total = 3
        }
      }, 1000)
    },
    dropGift (arr) {
      let n = arr[this.randomInt(arr.length)]
      this.$set(this.boxList[n],'occupied',true);
      let gift = new PIXI.Sprite.fromImage(CONFIG.btGift)
      this.app1.stage.addChild(gift)
      gift.anchor.set(0.5, 0.5)
      gift.width = 98
      gift.height = 88
      gift.x = this.boxList[n].x;
      console.log(gift.x)
      const tween = PIXI.tweenManager.createTween(gift)
      tween.from({y: 0}).to({y: this.boxList[n].y - 8})
      tween.easing = PIXI.tween.Easing.inOutBack()
      tween.time = 800;
      
      gift.interactive = true
      const tween2 = PIXI.tweenManager.createTween(gift)
      tween2.from({rotation: 0}).to({rotation: 0.5})
      tween2.time = 500
      tween2.easing = PIXI.tween.Easing.inOutBack()
      tween2.pingPong = true
      tween2.repeat = 2
      tween2.delay = 2000
      tween.start().chain(tween2)
      // 打开
      gift.on('pointerdown', () => {
        PIXI.tweenManager.removeTween(tween)
        this.spillAction(100, 100, gift.x, this.boxList[n].y, [CONFIG.circleUrl, CONFIG.lightCircle], CONFIG.lightEmitterConfig)
        this.app1.stage.removeChild(gift)
        this.createPlane(1, n)
      })
    },
    winSuccess (a) {
      let successContainer = new PIXI.Container(), that = this, giftList = [], tweenList = []
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

      let lightBg = new PIXI.Sprite.fromImage(CONFIG.successLight)
      lightBg.width = 414
      lightBg.height = 455
      lightBg.anchor.set(0.5)
      lightBg.x = this.app1.screen.width / 2
      lightBg.y = this.isLandscape ? this.app1.screen.height / 2 : this.boxList[0].y + 302 / 2
      successContainer.addChild(lightBg)

      this.app1.ticker.add(delta => {
        lightBg.rotation += 0.01
      })
      let plane = new PIXI.Sprite.fromImage(CONFIG.planeList[0])
      plane.anchor.set(0.5, 0.5)
      plane.width = 120
      plane.height = 120
      plane.x = lightBg.x
      plane.y = lightBg.y
      plane.interactive = true
      successContainer.addChild(plane)

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
      bigLogo.height = this.isLandscape ? 100 : 164
      successContainer.addChild(bigLogo)
      const tween_logo = PIXI.tweenManager.createTween(bigLogo)
      tween_logo.from({y: - bigLogo.height / 2}).to({y: this.isLandscape ? lightBg.y / 2 + 20: lightBg.y / 2 + bigLogo.height / 2})
      tween_logo.time = 1600
      tween_logo.easing = PIXI.tween.Easing.inBounce()


      let bigPlane = new PIXI.Sprite.fromImage(CONFIG.planeList[a - 1])
      bigPlane.anchor.set(0.5, 0.5)
      bigPlane.width = 120
      bigPlane.height = 120
      bigPlane.x = lightBg.x
      bigPlane.y = lightBg.y
      bigPlane.interactive = true


      tween_logo.start()
      tween_logo.on('end', () => {
        // 黑字  按钮
        let blackTxt = new PIXI.Text('MERGE,PLANE,AND DEVELOP\nYOUR OWN TEAM!', {
          fontFamily: 'NumFont',
          fontSize: 22,
          align: 'center',
          fill: 'black',
          stroke: '#ffffff',
          strokeThickness: 6
        })
        blackTxt.anchor.set(0.5)
        blackTxt.x = this.app1.screen.width / 2
        blackTxt.y = this.isLandscape ? lightBg.y + 90 : lightBg.y + 100
        successContainer.addChild(blackTxt)
        console.log(blackTxt.style.fontFamily)
        // 按钮
        this.installBtn({
          url: CONFIG.downBtn,
          width: this.isLandscape ? 138 : 180,
          height: this.isLandscape ? 60 : 78,
          y: this.isLandscape ? this.app1.screen.height - 50 : lightBg.y + 100 + blackTxt.height + 20,
          parentCont: successContainer
        }).on('pointerdown', () => {
          this.linkAppStore()
        })
      })

      const tween_1 = PIXI.tweenManager.createTween(plane)
      const tween_2 = PIXI.tweenManager.createTween(bigPlane)
      tween_1.time = 500
      tween_1.easing = PIXI.tween.Easing.inCubic()
      tween_1.pingPong = true
      tween_1.loop = false

      tween_2.time = 1200
      tween_2.easing = PIXI.tween.Easing.inCubic()
      
      tween_1.from({scale: {x: 1, y: 1}})
      tween_1.to({scale: {x: 0, y: 0}})
      tween_1.on('end', () => {
        PIXI.tweenManager.removeTween(tween_1)
        successContainer.removeChild(plane)
        successContainer.addChild(bigPlane)
        tween_2.start()
      })

      tween_2.from({scale: {x: 1.5, y: 1.5}})
      tween_2.to({scale: {x: 2, y: 2}})
      tween_2.repeat = 2
  
      
      tween_logo.chain(tween_1)
      this.app1.ticker.add(delta => {
        PIXI.tweenManager.update()
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
        // mraid.open(url);
        // console.log('mraid open store')
      } catch (err) {
        console.error(err);
        window.location = url;
      }
    },
    createHighlight (curIndex, arr, x) {
      let that = this
      this.container1 = new PIXI.Container()
      this.app1.stage.addChild(this.container1)
      let closerSprite = new PIXI.Sprite.fromImage(CONFIG.closerPic)
      closerSprite.anchor.set(0.5, 0.5)
      closerSprite.x = that.boxList[curIndex].x
      closerSprite.y = that.boxList[curIndex].y
      closerSprite.width = that.boxList[curIndex].width
      closerSprite.height = that.boxList[curIndex].height
      this.container1.addChild(closerSprite)
      if (!arr.length) {
        return
      }
      arr.forEach(item => {
        let dotSprite = new PIXI.Sprite.fromImage(CONFIG.dotPic)
        dotSprite.anchor.set(0.5, 0.5)
        dotSprite.x = that.boxList[item].x
        dotSprite.y = that.boxList[item].y
        dotSprite.width = that.boxList[item].width
        dotSprite.height = that.boxList[item].height
        this.container1.addChild(dotSprite)
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss" rel="stylesheet">
@font-face {
  font-family: 'NumFont';
  src: url('../assets/font/ObelixProB-cyr.woff.ttf') format('woff');
  font-style: normal;
  font-weight: bold;
}
</style>
