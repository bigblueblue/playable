<template>
  <div id="mergePlane">
  </div>
</template>

<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import CONFIG from '../assets/js/config'
import tweenManager from 'pixi-tween'
export default {
  name: 'mergePlane',
  data () {
    return {
      app1: {},
      distanceX: 110,
      distanceY: 20,
      boxList: [],
      step: 1,
      logoH: 124, // logo的y
      btnH: 0, // 安装btn的y
      handSprite: {}, //手sprite
      planeList: [],  // 飞机sprite
      shadeSprite: {}, // 背景遮罩
      hintSprite: {}, // 文字提示遮罩
      tween1: {}
    }
  },
  watch: {
    step (newval, oldval) {
      if (oldval > 3) {
        this.handSprite.visible = false
      }
    }
  },
  mounted () {
    this.initPIXI()
  },
  methods: {
    initPIXI () {
      let container, bgSprite, logoSprite
      this.app1 = new PIXI.Application(300, 300)
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
      console.log(this.app1)
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
      this.hintSprite.y = this.btnH

      for(let i = 0; i < 2; i++) {
        if (this.step > 2) {
          return
        }
        this.createPlane()
        this.step ++
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
    },
    layBox () {
      for (let i = 0; i < 6; i++) {
        let sprite = new PIXI.Sprite.fromImage(CONFIG.layBox)
        sprite.anchor.set(0.5, 0.5)
        sprite.width = 93
        sprite.height = 116
        sprite.x = (i % 3) * this.distanceX + window.innerWidth / 2 - this.distanceX
        sprite.y = Math.floor(i / 3) * (this.distanceY + sprite.height) + this.logoH + 110
        console.log(sprite.x, sprite.y)
        this.app1.stage.addChild(sprite)
        sprite.rank = 0
        this.boxList.push(sprite)
      }
    },
    installBtn () {
      let installSprite = new PIXI.Sprite.fromImage(CONFIG.installBtn)
      installSprite.anchor.set(0.5, 0.5)
      installSprite.height = 58
      installSprite.width = 146
      installSprite.x = window.innerWidth / 2
      installSprite.y = this.boxList[3].y + 130
      this.btnH = installSprite.y
      this.app1.stage.addChild(installSprite)
    },
    createPlane (r = 1) {
      var n, plane, that = this
      if (this.step < 3) {
        n = this.step
      } else {
        n = this.randomInt(6)
      }
      plane = new PIXI.Sprite.fromImage(CONFIG.planeList[0])
      plane.anchor.set(0.5, 0.5)
      plane.width = 80
      plane.height = 80
      plane.pName = Math.random().toString(36).subStr(2)
      plane.pRank = r
      plane.x = this.boxList[n].x
      plane.y = this.boxList[n].y
      this.app1.stage.addChild(plane)
      this.planeList.push(plane)

      plane.interactive = true
      this.play(plane)
    },
    randomInt (len) {
      return Math.floor(Math.random() * len)
    },
    play(obj) {
      obj.on('pointerdown', onDragStart)
      obj.on('pointeup', onDragEnd)
      obj.on('pointerupoutside', onDragEnd)
      obj.on('pointermove', onDragMove)
      function onDragStart (event) {
        this.data = event.data
        this.dragging = true
        if (that.step <= 3) {
          that.tween1.stop()
          that.shadeSprite.visible = false
          that.hintSprite.destroy()
          that.handSprite.destroy()
        }
      }
      function onDragEnd (event) {
        let endPos = this.data.getLocalPosition(this.parent)
        that.boxList.forEach(function(item) {
          // item.
        })
      }
      function onDragMove (event) {
        if (this.dragging) {
          let newPosition = this.data.getLocalPosition(this.parent)
          this.x = newPosition.x
          this.y = newPosition.y
        }
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss" rel="stylesheet">
</style>
