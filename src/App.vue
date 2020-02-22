<template>
  <div>
  </div>
</template>

<script>
import { utils, Application, Sprite, stage} from 'pixi.js';
export default {
  name: 'app',
  data () {
    return {
      pixiApp: null,
      digChanSprite: null,
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
      width: screen.availWidth,
      height: screen.availHeight,
      transparent: false,
      antialias: true,
      resolution: 1
    };

    this.ratio = screen.availWidth / screen.availHeight

    console.log('pixi options', options);
    this.pixiApp = new Application(options)
    console.log('pixi app', this.pixiApp)
    
    this.$el.appendChild(this.pixiApp.view)

    this.pixiApp.renderer.backgroundColor = 0x061639;
    this.pixiApp.renderer.autoResize = true;
    this.pixiApp.renderer.view.style.position = "absolute";
    this.pixiApp.renderer.view.style.display = "block";
    this.pixiApp.renderer.resize(window.innerWidth, window.innerHeight);
    // this.pixiApp.renderer.resize(512, 512);

    this.resourceLoader();
  },
  methods: {
    handleResize: function($event) {
      console.log(`set pixi app size ${window.innerWidth} x ${window.innerHeight}`);
      this.pixiApp.renderer.resize(window.innerWidth, window.innerHeight);

      this.ratio = screen.innerWidth / screen.innerHeight
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

      this.digChanSprite.x = screen.width / 2;
      this.digChanSprite.y = screen.height / 2

      this.digChanSprite.rotation = -0.25;
    }
  },
  components: {

  }
}
</script>

<style>
  div {
    height: 100%;
  }
</style>
