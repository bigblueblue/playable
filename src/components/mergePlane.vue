<template>
  <div id="mergePlane">
  </div>
</template>

<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import CONFIG from '../assets/js/config'
import tweenManager from 'pixi-tween'
import Vue from 'vue';
export default {
  name: 'mergePlane',
  data () {
    return {
      app1: {},
      distanceX: 110,
      distanceY: 10,
      boxList: [],
      step: 1,
      logoH: 104, // logo的y
      btnH: 0, // 安装btn的y
      bthHeight: 58,
      handSprite: {}, //手sprite
      planeList: new Array(6).fill({}),  // 飞机sprite
      shadeSprite: {}, // 背景遮罩
      hintSprite: {}, // 文字提示遮罩
      oldPosition: {x: 0, y: 0},
      grade: 0, // 合成飞机的等级
      fixedBox: [], // 高亮盒子
      roundBox: {}, // 高亮背景
      clearTime: ''
    }
  },
  watch: {
    step (newval, oldval) {
      if (oldval > 4) {
        this.handSprite.visible = false
      }
    }
  },
  mounted () {
    this.initPIXI()
    // this.hintSprite.visible = false
    // this.shadeSprite.visible = false
    // this.layGift()
  },
  methods: {
    initPIXI () {
      let container, bgSprite, logoSprite
      this.app1 = new PIXI.Application(300, 300, {
        antialias: true
      })
      document.getElementById('mergePlane').appendChild(this.app1.view)
      this.app1.renderer.view.style.position = 'absolute'
      this.app1.renderer.view.style.display = 'block'
      this.app1.renderer.autoResize = true
      this.app1.renderer.resize(window.innerWidth, window.innerHeight)
      container = new PIXI.Container()
      this.app1.renderer.render(container)
      bgSprite = new PIXI.Sprite.fromImage(CONFIG.bodyBg)
      bgSprite.x = 0
      bgSprite.y = 0
      bgSprite.width = window.innerWidth
      bgSprite.height = window.innerHeight
      this.app1.stage.addChild(bgSprite)
      this.layBox()
      this.installBtn()
      logoSprite = new PIXI.Sprite.fromImage(CONFIG.logo)
      logoSprite.width = 206
      logoSprite.height = 33
      logoSprite.x = (window.innerWidth - logoSprite.width) / 2
      logoSprite.y = this.logoH
      this.app1.stage.addChild(logoSprite)
      // console.log(this.app1)
      // 遮罩
      this.shadeSprite = new PIXI.Graphics()
      this.shadeSprite.beginFill(0x000000, 0.5)
      this.shadeSprite.drawRect(0, 0, window.innerWidth, window.innerHeight)
      this.app1.stage.addChild(this.shadeSprite)
  
      // 文字提示
      this.hintSprite = new PIXI.Sprite.fromImage(CONFIG.iconHint)
      this.hintSprite.anchor.set(0.5, 0.5)
      this.hintSprite.width = 330
      this.hintSprite.height = 126
      this.app1.stage.addChild(this.hintSprite)
      this.hintSprite.x = window.innerWidth / 2
      this.hintSprite.y = this.btnH - this.bthHeight / 2
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
        console.log(sprite.x, sprite.y)
        this.app1.stage.addChild(sprite)
        this.fixedBox.push(sprite)
        this.createPlane()
      }
      // 手指示
      this.handSprite = new PIXI.Sprite.fromImage(CONFIG.iconHand)
      this.handSprite.anchor.set(1, 1)
      this.handSprite.width = 62
      this.handSprite.height = 62
      console.log(this.boxList)
      this.handSprite.y = this.boxList[2].y + this.boxList[2].height / 2
      this.handSprite.x = this.boxList[1].x + this.boxList[1].width + 16
      this.app1.stage.addChild(this.handSprite)
      console.log(this.handSprite)
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
    },
    layBox () { // 降落块
      for (let i = 0; i < 6; i++) {
        let sprite = new PIXI.Sprite.fromImage(CONFIG.layBox)
        sprite.anchor.set(0.5, 0.5)
        sprite.width = 92
        sprite.height = 116
        this.app1.stage.addChild(sprite)
        
        let x = (i % 3) * this.distanceX + window.innerWidth / 2 - this.distanceX;
        let y = Math.floor(i / 3) * (this.distanceY + sprite.height) + this.logoH + 110;
        let rank = 0;
        let occupied = false;
        let width = 92, height = 116
        sprite.width = width
        sprite.height = height
        sprite.x = x;
        sprite.y = y;
  
        Vue.set(this.boxList, i, {x, y, rank, occupied, width, height})
        console.table(this.boxList)
      }

    },
    installBtn () { // 安装按钮
      let installSprite = new PIXI.Sprite.fromImage(CONFIG.installBtn)
      installSprite.anchor.set(0.5, 0.5)
      installSprite.height = this.bthHeight
      installSprite.width = 146
      installSprite.x = window.innerWidth / 2
      installSprite.y = this.boxList[3].y + 200
      this.btnH = installSprite.y
      this.app1.stage.addChild(installSprite)
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
      plane.y = this.boxList[index].y
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
      let that = this
      obj.on('pointerdown', onDragStart)
      obj.on('pointerup', onDragEnd)
      obj.on('pointerupoutside', onDragEnd)
      obj.on('pointermove', onDragMove)
      function onDragStart (event) {
        this.data = event.data
        this.dragging = true
        that.oldPosition.x = this.x
        that.oldPosition.y = this.y
        if (that.step == 3) {
          that.tween1.stop()
          that.shadeSprite.visible = false
          that.hintSprite.destroy()
          that.handSprite.destroy()
          that.roundBox.destroy()
          that.fixedBox.forEach(item => {
            item.destroy()
          })
        }
      }
      function onDragEnd (event) {
        if (!this.dragging) {
          return
        }
        let endPos = this.data.getLocalPosition(this.parent), minArr = [], resultArr = []
        let onMoveing, curIndex
        onMoveing = that.planeList.findIndex(item => {
          if (!item.x) {
            return
          }
          return item.pName == this.pName
        })
        that.boxList.forEach((item, index) => {
          minArr.push({
            i: index, 
            d: Math.pow(Math.abs(item.x - endPos.x), 2) + Math.pow(Math.abs(item.y - endPos.y), 2) 
          })
        })
        resultArr = minArr.sort((a, b) => {
          return a.d - b.d
        })
        // console.log(resultArr)
        curIndex = resultArr[0].i
        console.log(onMoveing, curIndex)
        if (onMoveing != curIndex && that.boxList[curIndex].rank == this.pRank) {  // 飞机级别一样
          let rk = that.boxList[curIndex].rank, obj2 = that.boxList[onMoveing]
          obj2.rank = 0
          that.planeList[onMoveing].destroy()
          that.planeList[curIndex].destroy()
          Vue.set(that.planeList, onMoveing, {})
          that.$set(that.boxList[onMoveing],'occupied',false);
          that.$set(that.boxList[onMoveing], 'rank', 0)
          that.grade = rk + 1
          that.passResult(rk + 1, curIndex)
        } else {
          let obj = that.planeList[onMoveing]
          obj.x = that.oldPosition.x
          obj.y = that.oldPosition.y
          Vue.set(that.planeList, onMoveing, obj)
          return
        }
        this.dragging = false
        this.data = null
      }
      function onDragMove (event) {
        if (this.dragging) {
          let newPosition = this.data.getLocalPosition(this.parent)
          this.x = newPosition.x
          this.y = newPosition.y
        }
      }
    },
    layGift () {  // 底部礼物盒 计时
      let GiftScene = new PIXI.Container(), total = 3, nowTime, txtList = [], inter
      this.app1.stage.addChild(GiftScene)

      let btGift = new PIXI.Sprite.fromImage(CONFIG.btGift)
      btGift.anchor.set(0.5, 0.5)
      btGift.width = 98
      btGift.height = 88
      btGift.x = window.innerWidth / 2
      btGift.y = this.btnH - btGift.height + 10
      GiftScene.addChild(btGift)

      // 黑色礼盒遮罩
      // let blmask = new PIXI.Sprite.fromImage(CONFIG.blackMask)
      let blmask = new PIXI.Graphics()
      // blmask.anchor.set(0.5, 0.5)
      // blmask.width = 120
      // blmask.height = 120
      // blmask.x = btGift.x
      // blmask.y = btGift.y - 20
      blmask.beginFill(0x1094f9, 0.4)
      blmask.drawRect((window.innerWidth - 120)/ 2, btGift.y - 50, 120, 100)
      GiftScene.addChild(blmask)
    
      // 计时盒子
      let caluTime = new PIXI.Sprite.fromImage(CONFIG.caluTime)
      caluTime.anchor.set(0.5, 0.5)
      caluTime.width = 42
      caluTime.height = 34
      caluTime.x = btGift.x - caluTime.width / 2 - caluTime.width - 10
      caluTime.y = this.btnH - btGift.height - 20
      GiftScene.addChild(caluTime)
      this.createNum(total, GiftScene, caluTime, txtList)
      inter = setInterval(() => {
        if (total < 1) {
          clearInterval(inter)
          this.dropGift();
          blmask.destroy()
          return
        }
        --total
        blmask.y -= blmask.height / 3
        txtList.forEach((item, index) => {
          // if (index != total) {
            item.destroy()
            txtList = []
          // }
        })
        this.createNum(total, GiftScene, caluTime, txtList)
      }, 1000)
    },
    passResult (r, i) { // 点击飞机的时候 
      let resultContainer = new PIXI.Container(), that = this
      this.app1.stage.addChild(resultContainer)
      let rect = new PIXI.Graphics()
      rect.beginFill(0x54677b, 0.91)
      rect.drawRect(0, 0, window.innerWidth, window.innerHeight)
      resultContainer.addChild(rect)

      let title = new PIXI.Sprite.fromImage(CONFIG.resultTitle)
      title.anchor.set(0.5, 0.5)
      title.width = 320
      title.height = 120
      title.x = window.innerWidth / 2
      title.y = this.boxList[0].y
      resultContainer.addChild(title)

      let lightBg = new PIXI.Sprite.fromImage(CONFIG.resultLight)
      lightBg.width = 302
      lightBg.height = 302
      lightBg.anchor.set(0.5)
      lightBg.x = window.innerWidth / 2
      lightBg.y = title.y + lightBg.height / 2
      resultContainer.addChild(lightBg)

      this.app1.ticker.add(delta => {
        lightBg.rotation += 0.01
      })

      let plane = new PIXI.Sprite.fromImage(CONFIG.planeList[this.grade - 1])
      plane.anchor.set(0.5, 0.5)
      plane.width = 100
      plane.height = 100
      plane.x = lightBg.x
      plane.y = lightBg.y
      plane.interactive = true
      resultContainer.addChild(plane)
      plane.on('pointerdown', onDragStart)
      function onDragStart(event) {
        resultContainer.destroy()
        that.createPlane(r, i)
        that.clearTime = setInterval(that.layGift, 3000)
      }
    },
    createNum (index, stage, caluTime, arr) { // 产生数字
      let numName
      numName = new PIXI.Sprite.fromImage(CONFIG.timeList[index])
      numName .anchor.set(0.5, 0.5)
      numName.nName = Math.random().toString(36).substr(2)
      numName.x = caluTime.x
      numName.y = caluTime.y
      numName.width = 16
      numName.height = 16
      stage.addChild(numName)
      arr.push(numName)
    },
    dropGift () {
      let arr = [], n;
      console.table(this.boxList.map(v => v.occupied))
      this.boxList.forEach((item, index) => {
        if (!item.occupied) {
          arr.push(index)
        }
      })
      if(arr.length == 0){
        console.log('没有位置了')
        return;
      }
      console.log(arr)
      n = arr[this.randomInt(arr.length)]
      this.$set(this.boxList[n],'occupied',true);
      // if (arr.length == 1) {
      //   n = 0
      // } else {

      // }
      console.log(n)
      let gift = new PIXI.Sprite.fromImage(CONFIG.btGift)
      this.app1.stage.addChild(gift)
      gift.interactive = true
      gift.anchor.set(0.5, 0.5)
      gift.width = 98
      gift.height = 88
      gift.x = this.boxList[n].x;
      console.log(gift.x)
      const tween = PIXI.tweenManager.createTween(gift)
      tween.from({y: 0}).to({y: this.boxList[n].y})
      tween.easing = PIXI.tween.Easing.inOutBack()
      tween.time = 1200;
      tween.start()
      // 打开
      gift.on('pointerdown', () => {
        PIXI.tweenManager.removeTween(tween)
        gift.destroy()
        this.createPlane(1, n)
        // this.layGift()
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss" rel="stylesheet">
</style>
