<template>
  <div></div>
</template>

<script>
  import { utils, Application, Loader, Sprite, stage, Container, RenderTexture, Texture, Rectangle } from 'pixi.js';
  export default {
    name: "Game",
    data () {
      return {
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
              this.spriteDict[i.value.path] = i.value.res;
            }
          })
          console.log('spdict', this.spriteDict)

          if (result.filter(i => i.status !== 'fulfilled').length > 0) {
            alert('Some sprites failed to load.');
          } else {
            console.log('all sprites are ready');
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
      delete this.pixiApp;
    },
    methods: {
      initPixiJS: function() {
        const options = {
          width: 375,
          height: 667,
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
