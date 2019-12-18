<template>
  <div class="home">
    <div>
      <canvas id="canvas"></canvas>
    </div>
  </div>
</template>

<script>
export default {
  name: "home",
  components: {},
  data() {
    return {
      // canvas 绘制数据
      canvasData: [],
      // sandBox 沙河数据
      sandBox: []
    };
  },
  watch: {
    canvasData: {
      deep: true,
      handler() {
        console.log("draw call");
      }
    }
  },
  methods: {
    run() {},
    pause() {},
    stop() {}
  },
  mounted() {}
};
</script>
