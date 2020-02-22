<template>
  <div>
  </div>
</template>

<script>
import { utils, Application } from 'pixi.js';
export default {
  name: 'app',
  data () {
    return {
      pixiApp: null,
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
  },
  methods: {
    handleResize: function($event) {
      console.log(`set pixi app size ${window.innerWidth} x ${window.innerHeight}`);
      this.pixiApp.renderer.resize(window.innerWidth, window.innerHeight);
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
