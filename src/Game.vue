<template>
  <div></div>
</template>

<script>
  import { utils, Application, Loader, Sprite, stage, Container,
            RenderTexture, Texture, Rectangle, Text,
            DisplayObject} from 'pixi.js';
  export default {
    name: "Game",
    data () {
      return {

        CANVAS_WIDTH: 375,
        CANVAS_HEIGHT: 667,

        pixiApp: null,
        pixiContainer: null,

        assetsPath: 'src/assets/',
        spriteList: [
          'body.PNG', 'face-fuck.PNG', 'face-happy.PNG', 'face-normal.PNG',
          'full-sample.PNG', 'left-normal.PNG', 'left-piston-active.PNG',
          'left-piston-normal.PNG', 'left-rotate.PNG', 'place-work.PNG',
          'right-normal.PNG', 'right-piston-active.PNG', 'right-piston-normal.PNG',
          'right-rotate.PNG', 'tile-set.png'
        ],
        resourceDict: {},
        spriteDict: {},
        spriteOrder: [
          {sprite: 'body.PNG', order: 0},
          {sprite: 'face-fuck.PNG', order: 1}, {sprite: 'face-happy.PNG', order: 1}, {sprite: 'face-normal.PNG', order: 1},
          {sprite: 'left-normal.PNG', order: 2}, {sprite: 'left-rotate.PNG', order: 2},
          {sprite: 'right-normal.PNG', order: 2}, {sprite: 'right-rotate.PNG', order: 2},
          {sprite: 'place-work.PNG', order: 3},
          {sprite: 'left-piston-normal.PNG', order: 5}, {sprite: 'left-piston-active.PNG', order: 5},
          {sprite: 'right-piston-normal.PNG', order: 5}, {sprite: 'right-piston-active.PNG', order: 5},
          {sprite: 'black', order: 6}, {sprite: 'white', order: 6}
        ],

        faceMap: {
          'happy': 'face-happy.PNG',
          'fuck': 'face-fuck.PNG',
          'normal': 'face-normal.PNG'
        },

        player: {
          face: 'normal',
          left: false,
          right: false
        },

        workLine: [],
        workLineData: [],
        workLineScale: 6,
        workLineGap: 10,

        combo: 0,
        score: 0,

        textDict: {},
        time: 30,

        countDown: 6,

        isGameStarted: false
      }
    },
    beforeCreate: function() {
      let type = 'WebGL';
      if (!utils.isWebGLSupported()) {
        type = 'canvas'
      }
      utils.sayHello(type);
    },
    mounted: function() {
      this.workLineInit();
      this.initPixiJS();
      this.keyInit();
      this.spriteLoader()
        .then(result => {
          console.log('result', result)
          result.forEach(i => {
            if (i.status === 'fulfilled') {
              this.resourceDict[i.value.path] = i.value.res;
            }
          })
          console.log('spdict', this.resourceDict)
          if (result.filter(i => i.status !== 'fulfilled').length > 0) {
            alert('Some sprites failed to load.');
          } else {
            console.log('all sprites are ready');
            this.spriteInit(this.resourceDict);
            console.log('sprite dict', this.spriteDict)
            this.addSpritesToStage(this.spriteDict, this.workLine, this.spriteOrder);
            this.addDisplayToStage();
            this.addTextToStage();
          }
        })
        .catch(err => {
          console.log('err', err)
          alert('Error while load sprites.')
        });


      let lastSec = 0;
      let timerSec = 0;
      this.pixiApp.ticker.add((delta) => {
        this.hideSpritesAll(this.spriteDict);
        this.renderCurrentPlayer(this.spriteDict, this.player);
        this.renderCurrentDoll(this.spriteDict, this.workLine, this.workLineData);
        lastSec += delta;
        timerSec += delta;
        // console.log(`la ${lastSec} de ${delta}`)
        if (lastSec >= 24) {
          lastSec = 0;
          this.renderFlip(this.spriteDict, this.workLine);
        }

        if (timerSec >= 60) {
          timerSec = 0;
          if (this.countDown <= 0 && this.time > 0 && this.textDict.hasOwnProperty('time')) {
            this.time --;
            this.textDict['time'].text = `${this.time}`
          }

          // console.log(this.countDown, this.textDict.hasOwnProperty('countDown'))
          if (this.countDown > 1 && this.textDict.hasOwnProperty('countDown')) {
            this.countDown --;
            this.textDict['countDown'].text = `${this.countDown}`
          } else if (this.countDown === 1) {
            this.countDown --;
            this.textDict['countDown'].text = `START`
            this.isGameStarted = true;
          } else if (this.countDown === 0) {
            this.countDown --;
            this.textDict['countDown'].text = ``
          } else if (this.time <= 0 && this.isGameStarted) {
            this.isGameStarted = false;
            this.textDict['countDown'].text = `GAME OVER`;
          }
        }
      });
    },

    beforeDestroy: function () {
      // window.removeEventListener('resize', this.handleResize)
      this.pixiApp.ticker.remove((delta) => {
        console.log('before destory delta', delta)
      }, this.pixiApp);
      delete this.pixiApp;
    },
    methods: {
      initPixiJS: function() {
        const options = {
          width: this.CANVAS_WIDTH,
          height: this.CANVAS_HEIGHT,
          transparent: true,
          antialias: true,
          resolution: window.devicePixelRatio || 1
        };

        this.pixiApp = new Application(options)

        this.pixiApp.renderer.backgroundColor = 0x061639;
        this.pixiApp.renderer.autoResize = true;
        this.pixiApp.renderer.view.style.position = "absolute";
        this.pixiApp.renderer.view.style.display = "block";

        this.pixiContainer = new Container();
        this.pixiContainer.x = this.pixiApp.screen.width / 2;
        this.pixiContainer.y = this.pixiApp.screen.height / 2;

        this.pixiContainer.pivot.x = this.pixiContainer.width / 2;
        this.pixiContainer.pivot.y = this.pixiContainer.height / 2;

        this.$el.appendChild(this.pixiApp.view)
      },
      ranInt: function(min, max) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
      },
      workLineInit: function() {
        Array(this.workLineScale).fill().map((_, i) => {
          this.workLineData.push(this.ranInt(0, 1));
        })
      },
      spriteInit: function (resourceDict) {
        Object.keys(resourceDict).forEach(key => {
          this.spriteDict[key] = new Sprite(resourceDict[key].texture)
          this.spriteDict[key].anchor.x = 0;
          this.spriteDict[key].anchor.y = 0;

          if (key === 'body.PNG' ||
            key === 'face-fuck.PNG' || key === 'face-happy.PNG' || key === 'face-normal.PNG') {

            this.spriteDict[key].anchor.x = 0.5;
            this.spriteDict[key].x = this.CANVAS_WIDTH / 2;
          }

          if (key === 'tile-set.png') {
            this.spriteDict['white'] = Sprite.from(new Texture(this.spriteDict[key].texture, new Rectangle(719, 1241, 512, 512)));
            this.spriteDict['white'].width = 128;
            this.spriteDict['white'].height = 128;
            this.spriteDict['white'].anchor.x = 0.5;
            this.spriteDict['white'].anchor.y = 0.5;
            this.spriteDict['white'].x = 64;
            this.spriteDict['white'].y = 64;
            this.spriteDict['black'] = Sprite.from(new Texture(this.spriteDict[key].texture, new Rectangle(719, 200, 512, 512)));
            this.spriteDict['black'].width = 128;
            this.spriteDict['black'].height = 128;
            this.spriteDict['black'].anchor.x = 0.5;
            this.spriteDict['black'].anchor.y = 0.5;
            this.spriteDict['black'].x = this.CANVAS_WIDTH - 64;
            this.spriteDict['black'].y = 64;

            return;
          }
          const ratio = this.spriteDict[key].height / this.spriteDict[key].width;
          this.spriteDict[key].width = this.CANVAS_WIDTH;
          this.spriteDict[key].height = this.CANVAS_WIDTH * ratio;

          this.spriteDict[key].y = this.CANVAS_HEIGHT - this.spriteDict[key].height;
        })

        Array(this.workLineScale).fill().forEach((_, i) => {
          this.workLine.push(
            Sprite.from(new Texture(this.spriteDict['tile-set.png'].texture, new Rectangle(719, 200, 512, 512)))
          )
          this.workLine[i].width = 128;
          this.workLine[i].height = 128;
          this.workLine[i].anchor.x = 0.5;
          this.workLine[i].anchor.y = 0.5;
          this.workLine[i].x = this.CANVAS_WIDTH / 2;
          this.workLine[i].y = 525 - (this.workLineScale - i) * this.workLineGap;
        });
      },
      keyInit: function() {
        window.addEventListener('keydown', (e) => {
          switch (e.key) {
            case 'ArrowRight':
              this.onTriggerRightDown();
              break;
            case 'ArrowLeft':
              this.onTriggerLeftDown();
              break;
          }
        });
        window.addEventListener('keyup', (e) => {
          switch (e.key) {
            case 'ArrowRight':
              this.onTriggerRightUp();
              break;
            case 'ArrowLeft':
              this.onTriggerLeftUp();
              break;
          }
        });
        console.log('stage', this.pixiApp.stage)
        this.pixiApp.renderer.plugins.interaction.on( 'pointerdown', ( event ) => {
          console.log( event.data.global.x, event.data.global.y, event.data.originalEvent )
          if (event.data.global.x > this.CANVAS_WIDTH / 2) {
            this.onTriggerRightDown();
          } else {
            this.onTriggerLeftDown();
          }
        });
        this.pixiApp.renderer.plugins.interaction.on( 'pointerup', ( event ) => {
          console.log( event.data.global.x, event.data.global.y, event.data.originalEvent )
          if (event.data.global.x > this.CANVAS_WIDTH / 2) {
            this.onTriggerRightUp();
          } else {
            this.onTriggerLeftUp();
          }
        } );
        // this.pixiApp.stage.mousedown = this.pixiApp.stage.touchstart = (e) => {
        //   console.log('down', e);
        // }
        // this.pixiApp.stage.mouseup = this.pixiApp.stage.touchend = (e) => {
        //   console.log('up', e);
        // }
        // window.addEventListener('keypress', (e) => {
        //   console.log('keypress', e);
        // });
      },
      onTriggerLeftDown: function() {
        this.player.left = true;
        this.player.right = false;
        this.checkWorkLine(0);
      },
      onTriggerLeftUp: function() {
        this.player.left = false;
      },
      onTriggerRightDown: function() {
        this.player.left = false;
        this.player.right = true;
        this.checkWorkLine(1);
      },
      onTriggerRightUp: function() {
        this.player.right = false;
      },
      checkWorkLine: function(input) {
        if (!this.isGameStarted) {
          return;
        }
        console.log(this.workLineData);
        if (this.workLineData[this.workLineScale - 1] === input) {
          this.combo ++;
          this.score += 100 + Math.floor(this.combo * 0.1);
          if (this.combo >= 10) {
            this.player.face = 'happy';
          } else {
            this.player.face = 'normal';
          }
        } else {
          this.combo = 0;
          this.score -= 90;
          if (this.score < 0) {
            this.score = 0;
          }
          this.player.face = 'fuck';
        }
        this.textDict['comboCnt'].text = `x${this.combo}`
        this.textDict['scoreCnt'].text = `${this.score}`;
        console.log(`combo ${this.combo} score: ${this.score}`)
        this.swapWorkLine();
      },
      swapWorkLine: function() {
        for(let i = this.workLineScale - 1; i >= 1; i --) {
          this.workLineData[i] = this.workLineData[i - 1];
        }
        this.workLineData[0] = this.ranInt(0, 1);
      },
      spriteLoad: function (path) {
        return new Promise((resolve, reject) => {
          const loader = new Loader();
          loader.add(`${this.assetsPath}${path}`);
          console.log('current load', `${this.assetsPath}${path}`)
          loader.onComplete.add((e, res) => {
            console.log('ok', `${this.assetsPath}${path}`, e, res)
            resolve({
              success: true,
              path,
              res: res[`${this.assetsPath}${path}`]
            })
          });
          loader.onError.add((e) => {
            console.log('fail', `${this.assetsPath}${path}`, e)
            reject({
              success: false,
              path,
              res: null
            })
          });
          loader.load();
        })
      },
      spriteLoader: function () {
        return Promise.allSettled(this.spriteList.map(i => this.spriteLoad(i)))
      },
      addSpritesToStage: function (spriteDict, workLine, order) {
        order.sort((a, b) => a.order - b.order).forEach(i => {
          this.pixiApp.stage.addChild(spriteDict[i.sprite]);
        })

        workLine.forEach(i => {
          this.pixiApp.stage.addChild(i);
        })
      },
      addDisplayToStage: function() {
        const _target = new DisplayObject();
        _target.setInteractive = true;
        this.pixiApp.stage.addChild(_target);
      },
      addTextToStage: function() {
        this.textDict['time'] = new Text('TIME');
        this.textDict['time'].x = this.CANVAS_WIDTH / 2;
        this.textDict['time'].y = 32;
        this.textDict['time'].anchor.set(0.5);
        this.textDict['time'].style.fontSize = 32;
        this.textDict['comboCnt'] = new Text('COMBO CNT');
        this.textDict['comboCnt'].x = this.CANVAS_WIDTH / 2;
        this.textDict['comboCnt'].y = 64;
        this.textDict['comboCnt'].anchor.set(0.5);
        this.textDict['comboCnt'].style.fontSize = 32;
        this.textDict['combo'] = new Text('COMBO');
        this.textDict['combo'].x = this.CANVAS_WIDTH / 2;
        this.textDict['combo'].y = 96;
        this.textDict['combo'].anchor.set(0.5);
        this.textDict['combo'].style.fontSize = 16;
        this.textDict['scoreCnt'] = new Text('SCORE CNT');
        this.textDict['scoreCnt'].x = this.CANVAS_WIDTH / 2;
        this.textDict['scoreCnt'].y = 128;
        this.textDict['scoreCnt'].anchor.set(0.5);
        this.textDict['scoreCnt'].style.fontSize = 32;
        this.textDict['score'] = new Text('SCORE');
        this.textDict['score'].x = this.CANVAS_WIDTH / 2;
        this.textDict['score'].y = 160;
        this.textDict['score'].anchor.set(0.5);
        this.textDict['score'].style.fontSize = 16;
        this.textDict['countDown'] = new Text('COUNTDOWN');
        this.textDict['countDown'].x = this.CANVAS_WIDTH / 2;
        this.textDict['countDown'].y = this.CANVAS_HEIGHT / 2;
        this.textDict['countDown'].anchor.set(0.5);
        this.textDict['countDown'].style.fontSize = 48;
        console.log('text', this.textDict['score'])
        // this.textDict['score'].fontSize = 16;

        this.pixiApp.stage.addChild(this.textDict['time']);
        this.pixiApp.stage.addChild(this.textDict['comboCnt']);
        this.pixiApp.stage.addChild(this.textDict['combo']);
        this.pixiApp.stage.addChild(this.textDict['scoreCnt']);
        this.pixiApp.stage.addChild(this.textDict['score']);
        this.pixiApp.stage.addChild(this.textDict['countDown']);
      },

      hideSpritesAll: function(spriteDict) {
        Object.keys(spriteDict).forEach(key => {
          spriteDict[key].visible = false;
        })
      },
      renderFlip: function(spriteDict, workLine) {
        if (Object.keys(spriteDict).length < this.spriteList.length) {
          return;
        }
        spriteDict['body.PNG'].scale.x *= -1;
        spriteDict['face-fuck.PNG'].scale.x *= -1;
        spriteDict['face-happy.PNG'].scale.x *= -1;
        spriteDict['face-normal.PNG'].scale.x *= -1;

        workLine.forEach(i => {
          i.scale.x *= -1;
        })
      },
      renderCurrentDoll: function(spriteDict, workLine, workLineData) {
        if (!this.spriteDict.hasOwnProperty('black')
          || !this.spriteDict.hasOwnProperty('white')) {
          return;
        }
        spriteDict['black'].visible = true;
        spriteDict['white'].visible = true;

        workLine.forEach((i, index) => {
          switch (workLineData[index]) {
            case 0:
              i.texture = spriteDict['black'].texture;
              break;
            case 1:
              i.texture = spriteDict['white'].texture;
              break;
          }
        });
      },
      renderCurrentPlayer: function (spriteDict, player) {

        if (Object.keys(spriteDict).length < this.spriteList.length) {
          return;
        }

        spriteDict['body.PNG'].visible = true;
        spriteDict['place-work.PNG'].visible = true;

        if (player.left) {
          spriteDict['left-rotate.PNG'].visible = true;
          spriteDict['left-piston-active.PNG'].visible = true;

          spriteDict['right-normal.PNG'].visible = true;
          spriteDict['right-piston-normal.PNG'].visible = true;
        } else if(player.right) {
          spriteDict['right-rotate.PNG'].visible = true;
          spriteDict['right-piston-active.PNG'].visible = true;

          spriteDict['left-normal.PNG'].visible = true;
          spriteDict['left-piston-normal.PNG'].visible = true;
        } else {
          spriteDict['left-normal.PNG'].visible = true;
          spriteDict['left-piston-normal.PNG'].visible = true;
          spriteDict['right-normal.PNG'].visible = true;
          spriteDict['right-piston-normal.PNG'].visible = true;
        }

        switch (player.face) {
          case 'normal':
            spriteDict['face-normal.PNG'].visible = true;
            break;
          case 'fuck':
            spriteDict['face-fuck.PNG'].visible = true;
            break;
          case 'happy':
            spriteDict['face-happy.PNG'].visible = true;
            break;
        }
      }
    }
  }
</script>

<style >
  div {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

  }
  div canvas {
    /*max-width: 800px;*/
    width: auto !important;
    height: 100% !important;
  }
</style>
