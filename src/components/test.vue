<template>
  <div id="test"></div>
</template>
<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import tweenManager from 'pixi-tween'
import configMarqee from '../assets/js/marqee_config'
export default {
  name: 'test',
  data () {
    return {
      isLandscape: false
    }
  },
  mounted () {
    this.getPixi()
  },
  methods: {
    getPixi () {
      const canvasW = this.isLandscape ? 736 : 414
      const canvasH = this.isLandscape ? 414 : 736
      const outRadius = 180
      const inRadius = 124
      let app, canvas
      app = new PIXI.Application(canvasW, canvasH)
      document.getElementById('test').appendChild(app.view)
      canvas = document.querySelector('#test canvas')
      canvas.style.width = '100%'
      canvas.style.height = '100%'
      app.renderer.view.style.position = 'absolute'
      app.renderer.view.style.display = 'block'
      app.renderer.autoResize = true

      let redCont = new PIXI.Container()
      let yellowCont = new PIXI.Container()
      app.stage.addChild(redCont)
      app.stage.addChild(yellowCont)

      // let pivotSprite = new PIXI.Sprite.fromImage(configMarqee.marqeePviot)
      // pivotSprite.anchor.set(0.5)
      // pivotSprite.width = 88
      // pivotSprite.height = 99
      // pivotSprite.interactive = true
      // pivotSprite.buttonMode = true
      // redCont.addChild(pivotSprite)
      let lightArr1 = [], lightArr2 = []
      let rTexture = PIXI.Texture.fromImage(configMarqee.redLight)
      let yTexture = PIXI.Texture.fromImage(configMarqee.yellowLight)
      for (let m = 0; m < 24; m++) {
        let rLight = new PIXI.Sprite(rTexture)
        rLight.anchor.set(0.5, 0.5)
        rLight.width = 12
        rLight.height = 12
        rLight.x = Math.sin(this.d2a(m * 360 / 24)) * (outRadius - 38)
        rLight.y = Math.cos(this.d2a(m * 360 / 24)) * (outRadius - 38)
        redCont.addChild(rLight)
        lightArr1.push(rLight)
      }
      for (let n = 0, startIndex = 10;n < 3; n++) {
        let yLight = new PIXI.Sprite(yTexture)
        yLight.anchor.set(0.5, 0.5)
        yLight.width = 28
        yLight.height = 28
        lightArr2.push(yLight)
        yLight.x = lightArr1[startIndex].x
        yLight.y = lightArr1[startIndex].y
        yLight.otherIndex = startIndex
        yellowCont.addChild(yLight)
        startIndex++
      }
      let path = new PIXI.tween.TweenPath()
      path.arc(200, 200, 168, 0, Math.PI * 2, false)
      path.closed = true
      let gPath = new PIXI.Graphics()
      gPath.lineStyle(1, 0xfffff, 1)
      gPath.drawPath(path)
      app.stage.addChild(gPath)
      let timer = 1000
      const lightCont_tween = PIXI.tweenManager.createTween(yellowCont)
      lightCont_tween.path = path
      lightCont_tween.time = timer
      lightCont_tween.easing = PIXI.tween.Easing.outBounce()
      lightCont_tween.on('start', () => {
        console.log('light is moveing')
      })
      lightCont_tween.start()

      app.ticker.add(delta => {
        PIXI.tweenManager.update()
      })
    },
    d2a(n) {
      return n * Math.PI / 180;
    }
  }
}
</script>
