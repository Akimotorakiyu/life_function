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
    <button @click="initSandBox">重置sandBox</button>
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
      canvasData: [] as any[][],
      // sandBox 沙河数据
      sandBox: [] as any[],
      ctx: (undefined as any) as CanvasRenderingContext2D,
      sandSize: {
        width: 100,
        height: 100
      }
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
    initSandBox() {
      this.sandBox = Array.from(new Array(this.sandSize.width)).map(() =>
        Array.from(new Array(this.sandSize.height)).map(
          () => 255 * Math.random()
        )
      );
      this.canvasData = this.sandBox;
    },
    drawCall() {
      let ctx = this.ctx;
      console.log("draw call");
      if (ctx) {
        this.canvasData.forEach((elements, x) => {
          elements.forEach((element, y) => {
            ctx.fillStyle = `rgb(${element},${element},${element})`;
            ctx.fillRect(10 * x + 1, 10 * y + 1, 8, 8);
          });
        });
      } else {
        console.warn("no ctx to render");
      }
    },
    initCtx() {
      console.log(this.$refs.myCanvas);
      const canvas: HTMLCanvasElement = this.$refs.myCanvas as any;
      const ctx = canvas.getContext("2d", { alpha: false });
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
