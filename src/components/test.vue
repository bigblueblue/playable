<template>
  <div id="test"></div>
</template>
<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import tweenManager from 'pixi-tween'
import configMarqee from '../assets/js/nail_config'
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

      let marqeeContainer = new PIXI.Container()
      let yellowCont = new PIXI.Container()
      app.stage.addChild(marqeeContainer)
      app.stage.addChild(yellowCont)

      
      var background = new PIXI.Sprite.fromImage(configMarqee.blackUrl);
          marqeeContainer.addChild(background);
          background.width = 112;
          background.height = 133
          background.anchor.set(0.5, 0)
          background.x = 0 //blFrameList[selfNailIndex].x
          background.y = 0 //blFrameList[selfNailIndex].y
          var imageToReveal = new PIXI.Sprite.fromImage(configMarqee.lightUrl)
          marqeeContainer.addChild(imageToReveal);
          imageToReveal.width = 112;
          imageToReveal.height = 133
          imageToReveal.anchor.set(0.5, 0)
          imageToReveal.x = 0 //blFrameList[selfNailIndex].x
          imageToReveal.y = 0//blFrameList[selfNailIndex].y
          var renderTexture = PIXI.RenderTexture.create(app.screen.width, app.screen.height);
          var renderTextureSprite = new PIXI.Sprite(renderTexture);
          // stage.addChild(renderTextureSprite);
          imageToReveal.mask = renderTextureSprite;

      // let lightFrame = new PIXI.Sprite.fromImage(configMarqee.lightUrl)
      //     marqeeContainer.addChild(lightFrame)
      //     lightFrame.width = 112
      //     lightFrame.height = 133
      //     lightFrame.anchor.set(0.5, 0)
      //     lightFrame.x = blFrameList[selfNailIndex].x
      //     lightFrame.y = blFrameList[selfNailIndex].y
      //     var renderTexture = PIXI.RenderTexture.create(app.screen.width, app.screen.height);

      //     var renderTextureSprite = new PIXI.Sprite(renderTexture);
      //     marqeeContainer.addChild(renderTextureSprite);
      //     lightFrame.mask = renderTextureSprite;
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
