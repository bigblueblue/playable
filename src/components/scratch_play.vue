<template>
  <div id="scratchPlay"></div>
</template>
<script>
/* eslint-disable */
import * as PIXI from 'pixi.js'
import tweenManager from 'pixi-tween'
import CONFIG from '../assets/js/scratch-config.js'
import particles from 'pixi-particles'
import '../assets/js/pixi-display'
export default{
  name: 'scratchPlay',
  data () {
    return {
      total: 3,
      coin: 0
    }
  },
  mounted () {
    this.initPIXI()
  },
  methods: {
    initPIXI () {
      const canvasW = this.isLandscape ? 736 : 414
      const canvasH = this.isLandscape ? 414 : 736
      const smH = (382 - 18) / 5
      const giftList = [
        {
          color: '5de2ff',
          num: 100
        }, 
        {
          color: 'ff952e',
          num: 200
        },
        {
          color: '8bdd2a',
          num: 1000
        },
        {
          color: 'af5ace',
          num: 500
        },
        {
          color: '7529f3',
          num: 800
        }
      ]
      var that = this, app, bgSprite
      startGame()
      function startGame() {
        app = new PIXI.Application(canvasW, canvasH)
        document.getElementById('scratchPlay').appendChild(app.view)
        let canvas = document.querySelector('#scratchPlay canvas')
        canvas.style.width = '100%'
        canvas.style.height = '100%'
        app.renderer.view.style.position = 'absolute'
        app.renderer.view.style.display = 'block'
        app.renderer.autoResize = true
        app.stage = new PIXI.display.Stage()
        app.stage.group.enableSort = true
        var textStyle = new PIXI.TextStyle({
          fontFamily: 'Arial',
          fontSize: 34,
          fill: '#ffffff',
          fontWeight: 'bold'
        })
        drawImage()
        drawScratch()
        function drawImage () {
          bgSprite = new PIXI.Sprite.fromImage(CONFIG.bgUrl)
          bgSprite.width = canvasW
          bgSprite.height = canvasH
          app.stage.addChild(bgSprite)
          let ticketSprite = new PIXI.Sprite.fromImage(CONFIG.ticketUrl)
          ticketSprite.anchor.set(0.5)
          ticketSprite.width = 56
          ticketSprite.height = 38
          ticketSprite.x = 38
          ticketSprite.y = 30
          bgSprite.addChild(ticketSprite)
          let ticketNum = new PIXI.Text(that.total, textStyle)
          ticketNum.anchor.set(0.5)
          ticketNum.x = ticketSprite.x + ticketSprite.width / 2 + 20
          ticketNum.y = ticketSprite.y
          bgSprite.addChild(ticketNum)
          let coinNum = new PIXI.Text(that.coin, textStyle)
          coinNum.anchor.set(1, 0.5)
          coinNum.x = canvasW - 10
          coinNum.y = ticketSprite.y
          bgSprite.addChild(coinNum)
          let coinSprite = new PIXI.Sprite.fromImage(CONFIG.coinUrl)
          coinSprite.anchor.set(1, 0)
          coinSprite.x = coinNum.x - 26
          coinSprite.y = 10
          coinSprite.width = 38
          coinSprite.height = 38
          bgSprite.addChild(coinSprite)
          let logoSprite = new PIXI.Sprite.fromImage(CONFIG.logoUrl)
          logoSprite.anchor.set(0, 0.5)
          logoSprite.width = 170 
          logoSprite.height = 32
          logoSprite.x = 10
          logoSprite.y = canvasH - 30
          bgSprite.addChild(logoSprite)
          let btnSprite = new PIXI.Sprite.fromImage(CONFIG.btnUrl)
          btnSprite.anchor.set(1, 0.5)
          btnSprite.width = 146 
          btnSprite.height = 40
          btnSprite.x = canvasW - 10
          btnSprite.y = canvasH - 38
          bgSprite.addChild(btnSprite)
        }
        function drawScratch () {
          let scratchContainer = new PIXI.Container()
          bgSprite.addChild(scratchContainer)
          let scratchBox = new PIXI.Sprite.fromImage(CONFIG.scratchUrl)
          scratchBox.anchor.set(1)
          scratchContainer.addChild(scratchBox)
          scratchBox.width = 376
          scratchBox.height = 382
          scratchBox.x = canvasW - (canvasW - scratchBox.width) / 2
          scratchBox.y = canvasH - 90

          let brush = new PIXI.Graphics()
          brush.beginFill(0xffffff)
          brush.drawCircle(0, 0, 30)
          brush.endFill()
          let maskSprite = new PIXI.Sprite.fromImage(CONFIG.maskUrl)
          maskSprite.anchor.set(1)
          maskSprite.width = 310
          maskSprite.height = 364
          maskSprite.x = canvasW - (canvasW - 376) / 2 - 9
          maskSprite.y = canvasH - 90 - 9
          scratchContainer.addChild(maskSprite)
          let scratchSprite = new PIXI.Sprite.fromImage(CONFIG.scratchUrl)
          scratchSprite.anchor.set(1)
          scratchContainer.addChild(scratchSprite)
          scratchSprite.width = 376
          scratchSprite.height = 382
          scratchSprite.x = canvasW - (canvasW - scratchSprite.width) / 2
          scratchSprite.y = canvasH - 90
          let renderTexture = PIXI.RenderTexture.create(500, 800)
          let renderTextureSprite = new PIXI.Sprite(renderTexture)
          scratchContainer.addChild(renderTextureSprite)
          scratchSprite.mask = renderTextureSprite
          let coinTexture = PIXI.Texture.fromImage(CONFIG.coinIcon)
          for (let i = 0;i < 5; i++) {
            let coinIcon = new PIXI.Sprite(coinTexture)
            coinIcon.anchor.set(0.5)
            coinIcon.width = 20
            coinIcon.height = 20
            coinIcon.x = scratchBox.x - 40
            coinIcon.y = scratchBox.y - smH * 5 + (smH - 30) / 2 + smH * i - 4
            let smcoinNum = new PIXI.Text(giftList[i].num, {
              fontSize: 18,
              fill: '#ffffff'
            })
            smcoinNum.anchor.set(0.5)
            smcoinNum.x = coinIcon.x
            smcoinNum.y = scratchBox.y - smH * 5 + (smH - 30) / 2 + smH * i + 20
            let roundBox = new PIXI.Graphics()
            console.log(`0x${giftList[i].color}`)
            roundBox.beginFill(`0x${giftList[i].color}`)
            roundBox.drawRoundedRect(0, 0, 30, 30, 4)
            roundBox.x = scratchBox.x - 20 - 90
            roundBox.y = (scratchBox.y - 9) - smH * 5 + (smH - 30) / 2 + smH * i
            // coinIcon.alpha = false
            // smcoinNum.alpha = false
            // roundBox.alpha = false
            scratchContainer.addChild(coinIcon)
            scratchContainer.addChild(smcoinNum)
            scratchContainer.addChild(roundBox)
          }


          scratchContainer.interactive = true
          scratchContainer.on('pointerdown', pointerDown)
          scratchContainer.on('pointerup', pointerUp)
          scratchContainer.on('pointerMove', pointerMove)
          let dragging = false
          
          function pointerMove (event) {
            if (dragging) {
              brush.position.copy(event.data.global)
              app.renderer.render(brush, renderTexture, false, null, false)
            }
          }
          function pointerDown (event) {
            dragging = true
            pointerMove(event)
          }
          function pointerUp (event) {
            dragging = false
          }
        }
        function startScrape () {
        }
      }
    }
  }
}
</script>