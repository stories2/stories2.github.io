<template>
  <div></div>
</template>

<script>
  import { utils, Application, Sprite, stage, Container, RenderTexture, Texture, Rectangle } from 'pixi.js';
  export default {
    name: "Game",
    data () {
      return {
        pixiApp: null,
        pixiContainer: null
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
