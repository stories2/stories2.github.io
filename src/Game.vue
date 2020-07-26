<template>
  <div></div>
</template>

<script>
  import { utils, Application, Loader, Sprite, stage, Container,
            RenderTexture, Texture, Rectangle,
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
          'right-rotate.PNG'
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
          {sprite: 'right-piston-normal.PNG', order: 5}, {sprite: 'right-piston-active.PNG', order: 5}
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
        }
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
            this.addSpritesToStage(this.spriteDict, this.spriteOrder);
          }
        })
        .catch(err => {
          console.log('err', err)
          alert('Error while load sprites.')
        });


      this.pixiApp.ticker.add((delta) => {
        this.renderCurrentPlayer(this.spriteDict, this.player)
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
          transparent: false,
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
      spriteInit: function (resourceDict) {
        Object.keys(resourceDict).forEach(key => {
          this.spriteDict[key] = new Sprite(resourceDict[key].texture)
          this.spriteDict[key].anchor.x = 0;
          this.spriteDict[key].anchor.y = 0;

          const ratio = this.spriteDict[key].height / this.spriteDict[key].width;
          this.spriteDict[key].width = this.CANVAS_WIDTH;
          this.spriteDict[key].height = this.CANVAS_WIDTH * ratio;

          this.spriteDict[key].y = this.CANVAS_HEIGHT - this.spriteDict[key].height;
        })
      },
      keyInit: function() {
        window.addEventListener('keydown', (e) => {
          switch (e.key) {
            case 'ArrowRight':
              this.player.left = false;
              this.player.right = true;
              break;
            case 'ArrowLeft':
              this.player.left = true;
              this.player.right = false;
              break;
          }
        });
        window.addEventListener('keyup', (e) => {
          switch (e.key) {
            case 'ArrowRight':
              this.player.right = false;
              break;
            case 'ArrowLeft':
              this.player.left = false;
              break;
          }
        });
        console.log('stage', this.pixiApp.stage)
        this.pixiApp.renderer.plugins.interaction.on( 'pointerdown', ( event ) => {
          console.log( event.data.global.x, event.data.global.y, event.data.originalEvent )
          if (event.data.global.x > this.CANVAS_WIDTH / 2) {
            this.player.left = false;
            this.player.right = true;
          } else {
            this.player.left = true;
            this.player.right = false;
          }
        });
        this.pixiApp.renderer.plugins.interaction.on( 'pointerup', ( event ) => {
          console.log( event.data.global.x, event.data.global.y, event.data.originalEvent )
          if (event.data.global.x > this.CANVAS_WIDTH / 2) {
            this.player.right = false;
          } else {
            this.player.left = false;
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
      addSpritesToStage: function (spriteDict, order) {
        order.sort((a, b) => a.order - b.order).forEach(i => {
          this.pixiApp.stage.addChild(spriteDict[i.sprite]);
        })
        const _target = new DisplayObject();
        _target.setInteractive = true;
        this.pixiApp.stage.addChild(_target);
      },

      hideSpritesAll: function(spriteDict) {
        Object.keys(spriteDict).forEach(key => {
          spriteDict[key].visible = false;
        })
      },
      renderCurrentPlayer: function (spriteDict, player) {
        this.hideSpritesAll(spriteDict);

        if (Object.keys(spriteDict).length !== this.spriteList.length) {
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
