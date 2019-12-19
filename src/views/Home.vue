<style lang="stylus" scoped>
.my-canvas {
  border: solid rgba(3, 3, 3, 0.1) 1px;
}
</style>

<template>
  <div class="home">
    <div>
      <canvas
        ref="myCanvas"
        id="canvas"
        width="1000"
        height="800"
        class="my-canvas"
        >不支持canvas</canvas
      >
    </div>
    <button @click="drawCall">绘制</button>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  name: "home",
  components: {},
  data() {
    return {
      // canvas 绘制数据
      canvasData: [],
      // sandBox 沙河数据
      sandBox: [],
      ctx: (undefined as any) as CanvasRenderingContext2D
    };
  },
  watch: {
    canvasData: {
      deep: true,
      handler() {
        this.drawCall();
      }
    }
  },
  methods: {
    run() {},
    pause() {},
    stop() {},
    drawCall() {
      let ctx = this.ctx;
      console.log("draw call");
      if (ctx) {
        ctx.fillStyle = "rgb(200,0,0)";
        ctx.fillRect(10, 10, 55, 50);

        ctx.fillStyle = "rgba(0, 0, 200, 0.5)";
        ctx.fillRect(30, 30, 55, 50);
      } else {
        console.warn("no ctx to render");
      }
    },
    initCtx() {
      console.log(this.$refs.myCanvas);
      const canvas: HTMLCanvasElement = this.$refs.myCanvas as any;
      const ctx = canvas.getContext("2d");
      if (ctx) {
        this.ctx = ctx;
      }
    }
  },
  mounted() {
    this.initCtx();
  }
});
</script>
