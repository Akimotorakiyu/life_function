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
        width="800"
        height="800"
        class="my-canvas"
        >不支持canvas</canvas
      >
    </div>
    <div>
      fps:{{ status.fps.value.toFixed(2) }} {{ status.fps.tag ? "❤️" : "🥰" }}
    </div>
    <button @click="initSandBox">重置sandBox</button>
    <button @click="run">运行</button>
    <button @click="animate">动画</button>
    <button @click="pause">暂停动画</button>
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
      // sandBox 沙盒数据
      sandBox: [] as any[][],
      ctx: (undefined as any) as CanvasRenderingContext2D,
      sandSize: {
        width: 60,
        height: 60
      },
      status: {
        animate: false,
        fps: {
          update: 0,
          value: 0,
          tag: true
        }
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
    run() {
      this.canvasData.forEach((elements, x) => {
        elements.forEach((element, y) => {
          const offset = [-1, 0, 1];
          let counterOutter = offset.reduce((counterOutter, offsetX) => {
            const counterInner = offset.reduce((counterInner, offsetY) => {
              if (offsetX === 0 && offsetY === 0) {
                return counterInner;
              } else {
                let posX: number = x + offsetX;
                let posY: number = y + offsetY;

                if (posX < 0) {
                  posX = this.canvasData.length + posX;
                } else if (posX > this.canvasData.length - 1) {
                  posX = posX - this.canvasData.length;
                }

                if (posY < 0) {
                  posY = elements.length + posY;
                } else if (posY > elements.length - 1) {
                  posY = posY - elements.length;
                }

                return counterInner + this.canvasData[posX][posY];
              }
            }, 0);
            return counterOutter + counterInner;
          }, 0);
          if (counterOutter === 2) {
            this.sandBox[x][y] = this.canvasData[x][y];
          } else if (counterOutter === 3) {
            this.sandBox[x][y] = 1;
          } else {
            this.sandBox[x][y] = 0;
          }
        });
      });

      this.canvasData = this.sandBox.map(elements =>
        elements.map(element => element)
      );
    },
    animate() {
      this.status.animate = true;
      this.animation(Date.now());
    },
    animation(starTime: number) {
      this.run();
      if (this.status.animate) {
        requestAnimationFrame(nextTime => {
          if (nextTime - this.status.fps.update > 100) {
            this.status.fps.update = nextTime;
            this.status.fps.tag = !this.status.fps.tag;
            this.status.fps.value = 1000 / (nextTime - starTime);
          }

          this.animation(nextTime);
        });
      }
      // this.status.animate = false;
    },
    pause() {
      this.status.animate = false;
    },
    stop() {
      this.status.animate = false;
    },
    initSandBox() {
      this.sandBox = Array.from(new Array(this.sandSize.width)).map(() =>
        Array.from(new Array(this.sandSize.height)).map(
          // () => 255 * Math.random()
          () => Math.round(Math.random())
        )
      );
      this.canvasData = this.sandBox.map(elements =>
        elements.map(element => element)
      );
    },
    drawCall() {
      // console.log("draw call");
      if (this.ctx) {
        let ctx = this.ctx;
        const offscreenCanvas = document.createElement("canvas");

        offscreenCanvas.width = this.sandSize.width * 10;
        offscreenCanvas.height = this.sandSize.height * 10;

        const offCtx = offscreenCanvas.getContext("2d");

        if (offCtx) {
          const offScreenCanvans = this.canvasData.forEach((elements, x) => {
            elements.forEach((element, y) => {
              offCtx.fillStyle = `rgb(${element * 255},${element *
                255},${element * 255})`;
              offCtx.fillRect(10 * x + 1, 10 * y + 1, 8, 8);
            });
          });
        }
        const offset = [-1, 0, 1];

        offset.forEach(offsetX => {
          offset.forEach(offsetY => {
            ctx.drawImage(
              offscreenCanvas,
              100 + offsetX * this.sandSize.width * 10,
              100 + offsetY * this.sandSize.height * 10
            );
          });
        });

        ctx.strokeStyle = `rgb(0,255,0)`;
        ctx.strokeRect(
          100,
          100,
          this.sandSize.width * 10,
          this.sandSize.height * 10
        );
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
