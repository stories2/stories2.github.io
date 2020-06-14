<template>
  <div>
  </div>
</template>

<script>
import { utils, Application, Sprite, stage, Container } from 'pixi.js';
export default {
  name: 'app',
  data () {
    return {
      pixiApp: null,
      digChanSprite: null,
      gameSprite: null,
      pixiContainer: null,
      ratio: 1
    }
  },
  beforeDestroy: function () {
    window.removeEventListener('resize', this.handleResize)
  },
  beforeCreate: function() {
     let type = 'WebGL';
     if (!utils.isWebGLSupported()) {
       type = 'canvas'
     }
     utils.sayHello(type);
  },
  created: function() {
    console.log('event reg')
    window.addEventListener('resize', this.handleResize)
  },
  mounted: function() {
    const options = {
      width: 800,
      height: 600,
      transparent: false,
      antialias: true,
      resolution: window.devicePixelRatio || 1
    };

    this.ratio = options.width / options.height

    console.log('pixi options', options);
    this.pixiApp = new Application(options)
    console.log('pixi app', this.pixiApp)
    
    this.$el.appendChild(this.pixiApp.view)

    this.pixiApp.renderer.backgroundColor = 0x061639;
    this.pixiApp.renderer.autoResize = true;
    this.pixiApp.renderer.view.style.position = "absolute";
    this.pixiApp.renderer.view.style.display = "block";

    this.pixiContainer = new Container();
    this.pixiContainer.x = this.pixiApp.screen.width / 2;
    this.pixiContainer.y = this.pixiApp.screen.height / 2;

    this.pixiContainer.pivot.x = this.pixiContainer.width / 2;
    this.pixiContainer.pivot.y = this.pixiContainer.height / 2;
    // this.pixiApp.renderer.resize(options.width, options.height);
    // this.pixiApp.renderer.resize(512, 512);

    this.resourceLoader();
    this.gameSpriteLoader();

    this.pixiApp.ticker.add((delta) => {
      // console.log(`delta ${delta}`)
    })
  },
  methods: {
    handleResize: function($event) {
      console.log(`set pixi app size ${window.innerWidth} x ${window.innerHeight}`);
      // this.pixiApp.renderer.resize(window.innerWidth, window.innerHeight);

      // this.ratio = screen.innerWidth / screen.innerHeight
    },
    gameSpriteLoader: function() {
      this.gameSprite = Sprite.from('dist/assets/tile-set.png');
      this.pixiApp.stage.addChild(this.gameSprite);
      const ratio = this.gameSprite.width / this.gameSprite.height;

      this.gameSprite.width = 500;
      this.gameSprite.height = 500 * ratio;
    },
    resourceLoader: function() {
      // const texture = utils.TextureCache[require('assets/dig-chan.png')]
      this.digChanSprite = Sprite.from('dist/assets/dig-chan.png');

      console.log('sprite', this.digChanSprite)

      this.pixiApp.stage.addChild(this.digChanSprite);

      const digChanRatio = this.digChanSprite.width / this.digChanSprite.height;
      this.digChanSprite.width = 400;
      this.digChanSprite.height = 400 * digChanRatio;

      this.digChanSprite.anchor.x = 0.5
      this.digChanSprite.anchor.y = 0.5

      this.digChanSprite.x = 400;
      this.digChanSprite.y = 300

      this.digChanSprite.rotation = -0.25;

      console.log(`xy ${this.digChanSprite.x} ${this.digChanSprite.y}`)
    }
  },
  components: {

  }
}
</script>

<style>
  div {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

  }
  canvas {
    max-width: 800px;
    width: 100%;
    height: auto;
  }
</style>
