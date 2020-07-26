<template>
  <div></div>
</template>

<script>
  import { utils, Application, Loader, Sprite, stage, Container, RenderTexture, Texture, Rectangle } from 'pixi.js';
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
        spriteDict: {}
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
            this.addSpritesToStage(this.spriteDict);
            this.spriteDict['full-sample.PNG'].visible = false;
            this.spriteDict['face-fuck.PNG'].visible = false;
            this.spriteDict['face-happy.PNG'].visible = false;
            this.spriteDict['left-piston-active.PNG'].visible = false;
            this.spriteDict['left-rotate.PNG'].visible = false;
            this.spriteDict['right-piston-active.PNG'].visible = false;
            this.spriteDict['right-rotate.PNG'].visible = false;
          }
        })
        .catch(err => {
          console.log('err', err)
          alert('Error while load sprites.')
        });


      this.pixiApp.ticker.add((delta) => {
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
      addSpritesToStage: function (spriteDict) {
        Object.keys(spriteDict).forEach(key => {
          this.pixiApp.stage.addChild(spriteDict[key]);
        })
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
