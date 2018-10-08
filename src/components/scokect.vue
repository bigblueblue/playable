<template>
  <div id="scratch-wrapper">
    <!--<div @click="refresh" style="color:white">click v7</div>-->
    <div class="title"></div>
    <div class="info" v-show="tipShow" @click="showWrapper">
      <div class="info-word">If your snake bet the block,and has snake balls left,you win the prize.</div>
    </div>
    <div class="claim-wrapper">
      <div class="countdown" v-show="countdownShow">Close in {{countNum}}s</div>
      <div class="clear-btn" @touchstart="clearCanvas" v-show="clearShow">Claim</div>
    </div>
    <div class="scratch-main">
      <div class="hand" v-show="handShow" @touchstart="handShow=false"></div>
      <div class="canvas-parent">
        <div class="canvas-con">
          <div class="opp">
            <div v-for="(value,index) in list"
                 :class="'opp-cell opp-cell-'+index"
            >
              <span class="span-cell" v-for="(v,i) in value.you"></span>
            </div>
          </div>
          <div class="you">
            <div v-for="(value,index) in list"
                 :class="'you-cell you-cell-'+index"
                 :style="{'backgroundColor':value.opp<4?'#03ffe6':(value.opp<8?'#08ff8c':'#ffe100')}"
            >
              <span>{{value.opp}}</span>
            </div>
          </div>
          <div class="coin">
            <div v-for="(value,index) in list" :class="'coin-cell coin-cell-'+index">
              <span class="coin-icon"></span>
              <span class="coin-num">{{value.prize}}</span>
            </div>
          </div>
          <!--<div class="crash" v-show="crashShow">-->
          <div class="crash">
            <div v-for="(value,index) in list"
                 :class='"crash-cell crash-cell-"+index'
                 v-show="crashShowArr[index]"
            >

            </div>
          </div>
        </div>
        <canvas id="canvas-item" width="618" height="728"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
  import webviewJsBridge from "./assets/js/webviewJsBridge.js";
  export default {
    name: 'app',
    data(){
      return {
        list: [],
        rewardList: [],//[5, 10, 20, 50, 100, 500, 1000]
        winNum: 10,//刮卡所得金币
        overFlag: false,
        coverImg: '',
        handShow: false,
        clearShow: false,
        tipShow: true,
        bridge: null,
        winIndex: null,
        countNum: 5,
        countdownShow: false,
        numGap: 0,
        crashShow: false,
        crashShowArr: [false, false, false, false, false]
      }
    },
    beforeMount(){
      var coverImg = new Image();
      coverImg.src = "/scratchSnaBlock_v3/static/img/cover.png";
      this.coverImg = coverImg;
    },
    mounted(){
      this.setBridge();
      this.getRandom();
      this.initCanvas();
    },
    watch: {
      overFlag(newVal, oldVal){
        if (newVal) {
          console.log("刮完了");
          this.bridge.callHandler('didEndGame', function (responseData) {
          });
        }
      }
    },
    methods: {
      setBridge: function () {
        var self = this;
        webviewJsBridge.setupWebViewJavascriptBridge(function (bridge) {
          self.bridge = bridge;
          self.bridge.registerHandler('refreshGameConfig', function (data, responseCallback) {
            self.winNum = data.reward * 1;
            self.rewardList = data.reward_list;
            self.refresh();
            if (data.shouldShowGuide) {
              self.handShow = true;
            } else {
              self.handShow = false;
            }
            var urls = [
              "/scratchSnaBlock_v3/static/img/bg.png",
              "/scratchSnaBlock_v3/static/img/canvas_bg.png",
              "/scratchSnaBlock_v3/static/img/coin.png",
              "/scratchSnaBlock_v3/static/img/hand.png",
              "/scratchSnaBlock_v3/static/img/main.png",
              "/scratchSnaBlock_v3/static/img/cover.png",
              "/scratchSnaBlock_v3/static/img/title_1.png",
              "/scratchSnaBlock_v3/static/img/title_2.png",
              "/scratchSnaBlock_v3/static/img/crash-block.gif",
            ];
            self.imageLoader(urls, function () {
              responseCallback(data);
            }, function () {
              responseCallback('error');
            })
          });
        })
      },
      refresh(){
        var self = this;
        var spanNodes = document.querySelectorAll(".span-cell")
        var spanLen = spanNodes.length;
        for (var j = 0; j < spanLen; j++) {
          spanNodes[j].className = "span-cell";
        }
        for (var i = 0; i < 5; i++) {
          var oppNode = document.querySelector(".opp-cell-" + i);
          oppNode.className = "opp-cell opp-cell-" + i;
        }
        //去除动画
        if (this.winIndex !== null) {
          var coin = document.querySelectorAll(".coin-cell-" + self.winIndex)[0];
          coin.className = "coin-cell coin-cell-" + self.winIndex;
          var youNode = document.querySelector(".you-cell-" + self.winIndex);
          youNode.className = "you-cell you-cell-" + self.winIndex;
          var spanNode = document.querySelector(".you-cell-" + self.winIndex + " span");
          spanNode.style.display = "block";
        }
        this.getRandom();
        console.log(self.list);
        this.overFlag = false;
        this.clearShow = false;
        this.tipShow = true;
        this.countNum = 5;
        this.countdownShow = false;
        this.handShow = false;
        var cardCanvas = document.querySelector("#canvas-item")
        var ctx = cardCanvas.getContext("2d");
        ctx.clearRect(0, 0, 618, 728);
        ctx.globalCompositeOperation = "destination-over";
        ctx.drawImage(self.coverImg, 0, 0, 618, 728);
        ctx.globalCompositeOperation = "destination-out";
      },
      showWrapper(){
        var self = this;
        var nodes = document.querySelectorAll(".opp-cell");
        var len = nodes.length;
        for (let k = 0; k < len; k++) {
          if (self.winIndex !== null && k == self.winIndex) {
            self.numGap = self.list[k].you - self.list[k].opp;
          }
          var node = document.querySelector(".opp-cell-" + k);
          node.className = "slideRight opp-cell opp-cell-" + k;
        }
        var ballCrashTimer=null;
        window.clearTimeout(ballCrashTimer);
        ballCrashTimer=window.setTimeout(function () {
          for (let m = 0; m < 5; m++) {
            ballCrash(".opp-cell-" + m, m);
          }
        }, 600);
        if (this.winNum != 0 && this.winIndex !== null) {
          window.setTimeout(function () {
            console.log("wave")
//            var winYouNode = document.querySelector(".you-cell-" + self.winIndex);
//            winYouNode.className = "fadeOutR you-cell you-cell-" + self.winIndex;
            var spanNodes = document.querySelectorAll(".opp-cell-" + self.winIndex + " span");
            console.log(self.numGap);
            var aNum = self.list[self.winIndex].you;//5
            var yNum = aNum - self.numGap;//1
            var coin = document.querySelector(".coin-cell-" + self.winIndex);
            coin.className = "scaleAni coin-cell coin-cell-" + self.winIndex;
            for (var m = yNum; m < aNum; m++) {
              spanNodes[m].className = "span-cell ballWave";
              spanNodes[m].style.animationDelay = (m - yNum) * 0.12 + "s";
            }
          }, 2000)
        }

        this.clearShow = false;
        this.countdownShow = true;
        var countdownTimer = null;
        window.clearInterval(countdownTimer);
        countdownTimer = window.setInterval(function () {
          self.countNum -= 1;
          console.log(self.countNum);
          if (self.countNum < 1) {
            window.clearInterval(countdownTimer);
            self.overFlag = true;
          }
        }, 1000)
        function ballCrash(className, idx) {
          self.crashShowArr.splice(idx, 1, true);
          var timer = null;
          window.clearTimeout(timer);
          timer = window.setTimeout(function () {
            self.crashShowArr.splice(idx, 1, false);
          }, 1800);
          var spanNodes = document.querySelectorAll(className + " .span-cell");
          var nLen = (spanNodes.length) > (self.list[idx].opp) ? (self.list[idx].opp) : (spanNodes.length);
          for (var n = 0; n < nLen; n++) {
            spanNodes[n].style.animationDelay = 0.2 * n + "s";
            spanNodes[n].className = "span-cell ballFadeAni";
          }
          var redNumTimer = null;
          window.clearInterval(redNumTimer);
          var gap = (self.list[idx].opp) - (self.list[idx].you);
          gap = gap > 0 ? gap : 0;
          redNumTimer = window.setInterval(function () {
            if (self.list[idx].opp >= gap + 1) {
              self.list[idx].opp -= 1;
              if (self.list[idx].opp === 0) {
                var winYouNode = document.querySelector(".you-cell-" + idx);
                var spanNode = document.querySelector(".you-cell-" + idx + " span");
                spanNode.style.display = "none";
                winYouNode.className = "fadeOutR you-cell you-cell-" + idx;
              }
            } else {
              window.clearInterval(redNumTimer);
            }
          }, 250);
        }
      },
      clearCanvas(){
        var cardCanvas = document.querySelector("#canvas-item")
        var ctx = cardCanvas.getContext("2d");
        ctx.clearRect(0, 0, 618, 728);
        this.showWrapper();
      },
      //获取随机数
      getRandom(){
        var self = this;
        var prizeArr;
        if (this.rewardList.length > 0 && this.rewardList.indexOf(0) != -1) {
          var prizeArr = this.rewardList;
          prizeArr.sort(function (a, b) {
            return a - b;
          })
          prizeArr = prizeArr.slice(1);
        } else {
          prizeArr = [5, 10, 20, 50, 100, 500, 1000];
        }
        this.list = scratchRandom(prizeArr, self.winNum, 5);
        console.log(this.list)
        console.log(this.winIndex);
        function scratchRandom(p, winP, round) {
          function randomSortN(array) {
            var arr_copy = array.slice()
            for (var i = arr_copy.length - 1; i >= 0; i--) {
              var randomIndex = Math.floor(Math.random() * (i + 1));
              var itemAtIndex = arr_copy[randomIndex];
              arr_copy[randomIndex] = arr_copy[i];
              arr_copy[i] = itemAtIndex;
            }
            return arr_copy;
          }

          function randomTwo(type) {
            var arr = [];
            for (var i = 3; i <= 10; i++) {
              arr.push(i);
            }
            var a, b;
            if (type === 'win') {
              var arr_copy = arr.slice(0, arr.length - 1);
              var a_index = Math.floor(Math.random() * arr_copy.length)
              a = arr_copy[a_index];
              arr_copy = arr.slice(a_index + 1);
              var b_index = Math.floor(Math.random() * arr_copy.length)
              b = arr_copy[b_index]
            } else {
              var arr_copy = arr.slice(0, arr.length - 1);
              var b_index = Math.floor(Math.random() * arr_copy.length)
              b = arr_copy[b_index];
              arr_copy = arr.slice(b_index + 1);
              var a_index = Math.floor(Math.random() * arr_copy.length)
              a = arr_copy[a_index]
            }
            return {opp: a, you: b}
          }

          var p_copy = p.slice();
          var index = p_copy.indexOf(winP);
          if (index > -1) {
            p_copy.splice(index, 1);
          }
          // p_copy.sort(randomSort);
          p_copy = randomSortN(p_copy)
          if (winP === 0) {
            p_copy = p_copy.slice(0, round);
          } else {
            p_copy = p_copy.slice(0, round - 1);
            p_copy.push(winP);
          }
          // p_copy.sort(randomSort);
          p_copy = randomSortN(p_copy)

          var result = []
          for (var i = 0; i < round; i++) {
            var two;
            if (p_copy[i] === winP) {
              two = randomTwo('win');
              self.winIndex = i;
            } else {
              two = randomTwo('')
            }
            result.push({opp: two.opp, you: two.you, prize: p_copy[i]})
          }
          return result
        }
      },
      imageLoader: function (urls, callback, errCallback) {
        var urls_l = urls.length;
        var imgs = [];
        var load_count = 0;
        var flag = false;
        urls.forEach(function (v, i) {
          if (flag) {
            return false;
          }
          imgs[i] = new Image();
          imgs[i].src = v;
          if (imgs[i].complete) {
            load_count += 1;
            if (load_count >= urls_l) {
              flag = true
              callback();
              return;
            }
          } else {
            imgs[i].onload = function () {
              load_count += 1;
              if (load_count >= urls_l) {
                flag = true
                callback();
                return;
              }
            }
            imgs[i].onerror = function () {
              flag = true
              errCallback(v);
              return;
            }
          }
        })
      },
      initCanvas(){
        var self = this;
        var cardCanvas = document.querySelector("#canvas-item")
        var ctx = cardCanvas.getContext("2d");
        this.canvasItem = cardCanvas;
        var canvasWidth = cardCanvas.width;
        var canvasHeight = cardCanvas.height;
        var lastPoint, p = Math.PI, isDrawing = false;
        self.coverImg.onload = function () {//先把scratch蒙版图画出来再把icon的src进行处理
          ctx.drawImage(self.coverImg, 0, 0, 618, 728);
        }
        // 获取整个card的距离屏幕的X,Y值
        function getBoxPos() {
          var box = document.querySelectorAll(".canvas-parent")[0];
          var boxRect = box.getBoundingClientRect();
          var boxX = boxRect.left;
          var boxY = boxRect.top;
          return {boxX: boxX, boxY: boxY}
        }

        // 监听canvas上的touch事件
        cardCanvas.addEventListener('touchstart', handleMouseDown, false);
        cardCanvas.addEventListener('touchmove', handleMouseMove, {passive: true});
        cardCanvas.addEventListener('touchend', handleMouseUp, false);
        // 获取当前鼠标相对于canvas的x,y位置
        function getPosite(e) {
          var canvasRect = cardCanvas.getBoundingClientRect();
          var canvasW = canvasRect.width;
          var canvasH = canvasRect.height;
          var wRatio = canvasW / canvasWidth;
          var hRatio = canvasH / canvasHeight
          var boxX = getBoxPos().boxX;
          var boxY = getBoxPos().boxY;
          var e = window.event || e;
          var x = (e.clientX || e.touches[0].clientX - boxX) / wRatio;
          var y = (e.clientY || e.touches[0].clientY - boxY) / hRatio;
          return {x: x, y: y}
        }

        // touchstart
        function handleMouseDown(e) {
          e.stopPropagation();
          e.preventDefault();
          self.handShow = false;
          lastPoint = getPosite(e);
          drawArc.call(ctx, lastPoint.x, lastPoint.y, lastPoint.x, lastPoint.y, "destination-out");
          isDrawing = true;
          moveDetection(lastPoint.x, lastPoint.y);
        }

        // touchmove
        function handleMouseMove(e) {
          if (isDrawing) {
            var currentPoint = getPosite(e);
            drawArc.call(ctx, lastPoint.x, lastPoint.y, currentPoint.x, currentPoint.y, "destination-out");
            lastPoint = currentPoint;
            moveDetection(currentPoint.x, currentPoint.y);
          }
        }

        // touchend
        function handleMouseUp(e) {
          handlePercentage(getFilledInPixels(64));
          isDrawing = false;
        }


        function moveDetection(x, y) {
          if (x > 0 && x < 115) {
            if (y >= 45 && y <= 95) {
              ballMove("opp-cell-0");
            } else if (y >= 190 && y <= 240) {
              ballMove("opp-cell-1");
            } else if (y >= 337 && y <= 395) {
              ballMove("opp-cell-2");
            } else if (y >= 476 && y <= 538) {
              ballMove("opp-cell-3");
            } else if (y >= 619 && y <= 686) {
              ballMove("opp-cell-4");
            }
          }
        }

        function ballMove(className) {
          var node = document.querySelector("." + className);
          node.className = className + " opp-cell slideRight";
//          node.style.transition = "all 1s linear";
//          node.style.left = "0";
        }

        //画刷动作
        function drawArc(x, y, ex, ey, type) {
          this.beginPath();
          this.lineCap = "round";
          this.lineWidth = 120;
          this.globalCompositeOperation = type;
          this.moveTo(x, y);
          this.lineTo(ex, ey);
          this.stroke();
          this.closePath();
        }

        // 获取当前填充的像素大小
        function getFilledInPixels(stride) {
          if (!stride || stride < 1) {
            stride = 1;
          }
          var pixels = ctx.getImageData(0, 0, canvasWidth, canvasHeight),
            pdata = pixels.data,
            l = pdata.length,
            total = (l / stride),
            count = 0;
          // Iterate over all pixels
          for (var i = count = 0; i < l; i += stride) {
            if (parseInt(pdata[i]) === 0) {
              count++;
            }
          }
          return Math.round((count / total) * 100);
        }

        // 判断画笔填充的百分比
        function handlePercentage(filledInPixels) {
          filledInPixels = filledInPixels || 0;
          if (filledInPixels > 80) {
            if (self.tipShow && !self.clearShow) {
              self.tipShow = false;
              self.clearShow = true;
            }
          }
          if (filledInPixels > 99 && self.clearShow) {
            self.showWrapper();
          }
        }
      }
    }
  }
</script>

<style lang="scss">
  @import "./assets/css/common.scss";

  @font-face {
    font-family: myFont;
    src: url("/scratchSnaBlock_v3/static/font/CenturyGothic-Bold.ttf");
  }

  html, body, #app {
    width: 100%;
    height: 100%;
    position: fixed;
    left: 0;
    top: 0;
    overflow: hidden;
    -webkit-tap-highlight-color: transparent;
    user-select: none;
    -webkit-user-select: none;
  }

  #app {
    background-image: url("/scratchSnaBlock_v3/static/img/bg.png");
    background-repeat: no-repeat;
    background-size: 100% 100%;
    background-position: left top;
    .title {
      width: wP(288);
      height: hP(100);
      position: absolute;
      left: wP(62);
      top: hP(6);
      background: url("/scratchSnaBlock_v3/static/img/title_1.png");
      @include bgSet();
      animation: changeTit 1s linear both infinite;
    }
    .info {
      width: wP(374);
      height: hP(80);
      position: absolute;
      left: wP(19);
      top: hP(232);
      border-radius: ptr(8);
      box-sizing: border-box;
      text-align: center;
      background: #8280ff;
      display: flex;
      justify-content: center;
      align-items: center;
      @include bgSet();
      .info-word {
        text-align: center;
        font-family: myFont, CenturyGothic, Arial, Tahoma, sans-serif;
        animation: changeBg 1s 0s infinite both;
        width: 86%;
        letter-spacing: ptr(0.6);
        font-size: ptr(15);
        line-height: ptr(20);
        font-weight: 600;
        color: #ffffff;
      }
    }
    .claim-wrapper {
      width: ptr(140);
      height: ptr(44);
      position: absolute;
      left: 50%;
      top: hP(260);
      margin-left: ptr(-70);
    }
    .clear-btn {
      -webkit-animation: tada 2s .2s infinite ease both;
      animation: tada 2s .2s infinite ease both;
    }
    .countdown, .clear-btn {
      width: 100%;
      height: 100%;
      background: orange;
      border: ptr(2) solid #000;
      border-radius: ptr(10);
      font-size: ptr(24);
      line-height: ptr(44);
      color: #000;
      font-weight: 700;
      text-align: center;
    }
    .scratch-main {
      width: wP(374);
      height: hP(380);
      position: absolute;
      left: wP(19);
      top: hP(335);
      background: url("/scratchSnaBlock_v3/static/img/main.png") no-repeat left top;
      background-size: 100% 100%;
      .hand {
        width: ptr(192);
        height: ptr(202);
        position: absolute;
        left: 30%;
        top: 50%;
        background: url("/scratchSnaBlock_v3/static/img/hand.png") no-repeat left top;
        background-size: 100% 100%;
        z-index: 1000;
        animation: shakeHand 2s 0.5s both linear infinite;
      }
      .canvas-parent {
        width: 82.51%;
        height: 95.79%;
        position: absolute;
        left: 15.24%;
        top: 2.11%;
        .canvas-con {
          width: 100%;
          height: 100%;
          position: absolute;
          left: 0;
          top: 0;
          background: url("/scratchSnaBlock_v3/static/img/canvas_bg.png") no-repeat left top;
          background-size: 100% 100%;
          overflow: hidden;
          .you {
            width: 11%;
            height: 100%;
            position: absolute;
            top: 0;
          }
          .opp {
            position: absolute;
            width: 67.31%;
            height: 100%;
            left: 0;
          }
          .you {
            left: ptr(208);
          }
          .you-cell {
            width: 100%;
            height: 9.34%;
            position: absolute;
            @include bgSet();
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: ptr(16);
            border-radius: ptr(3);
            background: #ffffff;
          }
          .opp-cell {
            width: 100%;
            height: 19.78%;
            position: absolute;
            left: -74%;
            display: flex;
            align-items: center;
            flex-direction: row-reverse;
            /*background-color: rgba(255,0,0,0.4);*/
            .span-cell {
              display: block;
              width: 8.65%;
              height: 25%;
              background-color: #fff400;
              border-radius: 50%;
              text-align: center;
            }
          }
          .crash {
            width: ptr(70);
            height: 100%;
            position: absolute;
            left: 50%;
            z-index: 99;
            .crash-cell {
              height: 19.8%;
              background-image: url("/scratchSnaBlock_v3/static/img/crash-block.gif");
              background-position: 20% 40%;
              background-size: 160% 160%;
            }
          }
          @function hPer($h) {
            @return $h/364*100%;
          }
          .opp-cell:nth-of-type(1), .coin-cell:nth-of-type(1) {
            top: hPer(0);
          }
          .opp-cell:nth-of-type(2), .coin-cell:nth-of-type(2) {
            top: hPer(72);
          }
          .opp-cell:nth-of-type(3), .coin-cell:nth-of-type(3) {
            top: hPer(145);
          }
          .opp-cell:nth-of-type(4), .coin-cell:nth-of-type(4) {
            top: hPer(218);
          }
          .opp-cell:nth-of-type(5), .coin-cell:nth-of-type(5) {
            top: hPer(292);
          }
          .you-cell:nth-of-type(1) {
            top: hPer(17);
          }
          .you-cell:nth-of-type(2) {
            top: hPer(91);
          }
          .you-cell:nth-of-type(3) {
            top: hPer(164);
          }
          .you-cell:nth-of-type(4) {
            top: hPer(237);
          }
          .you-cell:nth-of-type(5) {
            top: hPer(310);
          }
          .coin {
            width: 13.96%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 84%;
            .coin-cell {
              width: 100%;
              height: 19.78%;
              position: absolute;
              text-align: center;
              color: #fff;
              .coin-icon {
                display: inline-block;
                background: url("/scratchSnaBlock_v3/static/img/coin.png");
                @include bgSet();
                width: ptr(18);
                height: ptr(18);
                margin-top: 40%;
                margin-left: 10%;
              }
              .coin-num {
                width: 100%;
                text-align: center;
                display: inline-block;
                font-weight: 600;
                font-size: ptr(16);
                position: relative;
                top: -8%;
                color: #fff;
              }
            }
          }
        }
        #canvas-item {
          width: 100.2%;
          height: 100.1%;
          position: absolute;
          left: -0.01%;
          top: -0.01%;
          z-index: 999;
        }
      }

    }
  }

  .scaleAni {
    animation: wrapperAni 1s 0s infinite both;
  }

  .ballAni {
    transition: 0.6s;
    left: 0;
  }

  .ballFadeAni {
    animation: ballFadeAni 0.2s ease-in both;
  }

  .slideRight {
    animation: slideRAni 0.6s linear both;
  }

  @keyframes slideRAni {
    0% {
      left: -74%;
    }
    100% {
      left: 0;
    }
  }

  @keyframes ballFadeAni {
    0% {
      transform: scale(1);
      opacity: 1;
    }
    100% {
      display: none;
      opacity: 0;
      width: 0;
      height: 0;
      transform: scale(0);
    }
  }

  @keyframes changeTit {
    0%, 49% {
      background-image: url("/scratchSnaBlock_v3/static/img/title_1.png");
    }
    50%, 100% {
      background-image: url("/scratchSnaBlock_v3/static/img/title_2.png");
    }
  }

  @keyframes changeBg {
    0%, 50% {
      color: #2d2ba1;
    }
    60%, 100% {
      color: #ffffff;
    }
  }

  @keyframes shakeHand {
    0% {
      left: 30%;
      top: 50%;
    }
    50% {
      left: 50%;
      top: 35%;
    }
    100% {
      left: 30%;
      top: 50%;
    }
  }

  @keyframes tada {
    0% {
      transform: scale(1)
    }
    3%, 8% {
      transform: scale(0.9) rotate(-3deg)
    }
    8%, 12%, 17%, 22% {
      transform: scale(1.1) rotate(3deg)
    }
    10%, 15%, 20% {
      transform: scale(1.1) rotate(-3deg)
    }
    25% {
      transform: scale(1) rotate(0)
    }
    30%, 100% {
      transform: scale(1)
    }
  }

  @keyframes wrapperAni {
    0% {
      transform: scale(0.9);
    }
    50% {
      transform: scale(1.2);
    }
    100% {
      transform: scale(0.9);
    }
  }

  .fadeOutR {
    animation: fadeOutR 0.2s ease-in both;
  }

  @keyframes fadeOutR {
    0% {
      opacity: 1;
      transform: translateX(0);
    }
    100% {
      opacity: 0;
      transform: translateX(ptr(10));
    }
  }

  .ballWave {
    animation: ballWaveAni 1s linear infinite both;
  }

  @keyframes ballWaveAni {
    0%, 50%, 100% {
      transform: translateY(0);
    }
    25% {
      transform: translateY(ptr(10));
    }
    75% {
      transform: translateY(ptr(-10));
    }
  }
</style>
