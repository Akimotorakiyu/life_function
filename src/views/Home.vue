<style lang="stylus" scoped>
.my-canvas {
  border: solid rgba(3, 3, 3, 0.1) 1px;
}

.home {
  text-align: center;
}
</style>

<template>
  <div class="home">
    <h2>play with firefox or chrome browser !</h2>
    <div>
      <canvas
        ref="myCanvas"
        id="canvas"
        :width="sandSize.width * 10 * (1 + sandSize.outter)"
        :height="sandSize.height * 10 * (1 + sandSize.outter)"
        class="my-canvas"
        @click="set"
        >不支持canvas</canvas
      >
    </div>
    <div>
      <input
        type="range"
        v-model.number="setting.miniTime"
        min="0"
        max="88"
      />{{ Math.floor(Math.pow(1.1, setting.miniTime) - 1) }}ms
      <input
        type="range"
        v-model.number="sandSize.width"
        min="10"
        max="200"
        step="2"
      />{{ sandSize.width }}width
      <input
        type="range"
        v-model.number="sandSize.height"
        min="10"
        max="100"
        step="2"
      />{{ sandSize.height }}height
      <input
        type="range"
        v-model.number="sandSize.outter"
        min="0"
        max="1"
        step="0.1"
      />{{ sandSize.outter }}outter
      <input type="checkbox" v-model="setting.mutation" />mutation
    </div>
    <div>
      👪{{ status.generation }} ❤️{{ status.fps.value.toFixed(2) }}
      <button @click="initSandBox('chaos')">🌀⚽</button>
      <button @click="initSandBox('black')">🌀⚫</button>
      <button @click="run">⏯️</button>
      <button @click="animate" :disabled="status.animate">▶️</button>
      <button @click="pause" :disabled="!status.animate">⏸️</button>
    </div>
    <a href="https://github.com/Akimotorakiyu/life_function"
      >🤟check source code or make issue</a
    >
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
      canvasData: [[0]] as any[][],
      // sandBox 沙盒数据
      sandBox: [[0]] as any[][],
      ctx: (undefined as any) as CanvasRenderingContext2D,
      offscreenCanvas: (undefined as any) as HTMLCanvasElement,
      offCtx: (undefined as any) as CanvasRenderingContext2D,
      sandSize: {
        width: 50,
        height: 50,
        outter: 0.2
      },
      setting: {
        miniTime: 30,
        mutation: true
      },
      status: {
        generation: 0,
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
      if (this.setting.mutation) {
        let posX: number = Math.floor(this.canvasData.length * Math.random());
        let posY: number = Math.floor(
          this.canvasData[0].length * Math.random()
        );

        this.canvasData[posX][posY] = 1;
      }

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

                return counterInner + (this.canvasData[posX][posY] ? 1 : 0);
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

      // this.canvasData = this.sandBox.map(elements =>
      //   elements.map(element => element)
      // );

      const temp = this.canvasData;
      this.canvasData = [[]];
      this.canvasData = this.sandBox;
      this.sandBox = temp;
    },
    set(event: MouseEvent) {
      console.log(event);
      let rowOutter = (this.sandSize.width * 10 * this.sandSize.outter) / 2;
      let colOutter = (this.sandSize.height * 10 * this.sandSize.outter) / 2;

      let x: number, y: number;

      let offsetLeft: number = 0;
      let offsetTop: number = 0;

      if (
        event.srcElement &&
        (event.srcElement as HTMLElement).offsetLeft &&
        (event.srcElement as HTMLElement).offsetTop
      ) {
        offsetLeft = (event.srcElement as HTMLElement).offsetLeft;
        offsetTop = (event.srcElement as HTMLElement).offsetTop;
      }

      x = event.clientX - offsetLeft;
      y = event.clientY - offsetTop;

      if (x < rowOutter) {
        x = this.sandSize.width * 10 + x - rowOutter;
      } else if (x < rowOutter + this.sandSize.width * 10) {
        x = x - rowOutter;
      } else {
        x = x - rowOutter - this.sandSize.width * 10;
      }

      if (y < colOutter) {
        y = this.sandSize.width * 10 + y - colOutter;
      } else if (y < colOutter + this.sandSize.width * 10) {
        y = y - colOutter;
      } else {
        y = y - colOutter - this.sandSize.width * 10;
      }

      // console.log(x);
      // console.log(y);
      const posX = Math.floor(x / 10);
      const posY = Math.floor(y / 10);
      // console.log(posX);
      // console.log(posY);

      this.canvasData[posX][posY] = !this.canvasData[posX][posY];
      const temp = this.canvasData;
      this.canvasData = [[]];
      this.canvasData = temp;
    },
    animate() {
      this.status.animate = true;
      this.animation(Date.now());
    },
    animation(starTime: number) {
      this.run();
      this.status.generation++;
      if (this.status.animate) {
        // requestAnimationFrame(nextTime => {
        //   if (nextTime - this.status.fps.update > 100) {
        //     this.status.fps.update = nextTime;
        //     this.status.fps.tag = !this.status.fps.tag;
        //     this.status.fps.value = 1000 / (nextTime - starTime);
        //   }
        //   this.animation(nextTime);
        // });

        setTimeout(() => {
          const nextTime = Date.now();
          if (nextTime - this.status.fps.update > 100) {
            this.status.fps.update = nextTime;
            this.status.fps.tag = !this.status.fps.tag;
            this.status.fps.value = 1000 / (nextTime - starTime);
          }
          this.animation(nextTime);
        }, Math.floor(Math.pow(1.1, this.setting.miniTime) - 1));
      }
    },
    pause() {
      this.status.animate = false;
    },
    stop() {
      this.status.animate = false;
    },
    initSandBox(type: string) {
      this.status.generation = 0;
      this.sandBox = Array.from(new Array(this.sandSize.width)).map(() =>
        Array.from(new Array(this.sandSize.height)).map(
          // () => 255 * Math.random()
          // () => Math.round(Math.random())
          () => {
            switch (type) {
              case "black":
                return 0;

              case "chaos":
                return Math.round(Math.random());

              default:
                return 0;
            }
          }
        )
      );
      this.canvasData = this.sandBox.map(elements =>
        elements.map(element => element)
      );
    },
    drawCall() {
      // console.log("draw call");
      if (this.ctx && this.offscreenCanvas) {
        const { ctx, offscreenCanvas } = this;

        offscreenCanvas.width = this.sandSize.width * 10;
        offscreenCanvas.height = this.sandSize.height * 10;

        const offCtx = offscreenCanvas.getContext("2d");

        if (offCtx) {
          offCtx.fillRect(0, 0, offscreenCanvas.width, offscreenCanvas.height);
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
              (this.sandSize.width * 10 * this.sandSize.outter) / 2 +
                offsetX * this.sandSize.width * 10,
              (this.sandSize.height * 10 * this.sandSize.outter) / 2 +
                +offsetY * this.sandSize.height * 10
            );
          });
        });

        ctx.strokeStyle = `rgb(0,255,0)`;
        ctx.strokeRect(
          (this.sandSize.width * 10 * this.sandSize.outter) / 2,
          (this.sandSize.height * 10 * this.sandSize.outter) / 2,
          this.sandSize.width * 10,
          this.sandSize.height * 10
        );
      } else {
        console.warn("no ctx to render");
      }
    },
    initCtx() {
      // console.log(this.$refs.myCanvas);
      const canvas: HTMLCanvasElement = this.$refs.myCanvas as any;
      const ctx = canvas.getContext("2d", { alpha: false });
      this.offscreenCanvas = document.createElement("canvas");
      const offCtx = this.offscreenCanvas.getContext("2d");

      if (ctx && offCtx) {
        this.ctx = ctx;
        this.offCtx = offCtx;
      } else {
        console.warn("load ctx error");
      }
      this.initSandBox("chaos");
    }
  },
  mounted() {
    this.initCtx();
  }
});
</script>
