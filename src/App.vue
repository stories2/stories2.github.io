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
  beforeCreate: function() {
     let type = 'WebGL';
     if (!utils.isWebGLSupported()) {
       type = 'canvas'
     }
     utils.sayHello(type);
  },
  created: function() {
  },
  mounted: function() {
    const options = {
      width: screen.availWidth,
      height: screen.availHeight,
      transparent: false,
      antialias: true
    };
    console.log('pixi options', options);
    this.pixiApp = new Application(options)
    console.log('pixi app', this.pixiApp)
    
    this.$el.appendChild(this.pixiApp.view)
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
