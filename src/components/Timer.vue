<template>
  <div class="timer">{{ hh }}小时 {{ mm }}分钟 {{ ss }}秒</div>
</template>

<script>
import EventBus from "../eventBus.js";
export default {
  name: "timer",
  data() {
    return {
      hh: "00",
      mm: "00",
      ss: "00",
      startTimer: 0,
      endTimer: 0,
      timerId: 0
    };
  },
  created() {
    EventBus.$on("boom-end", () => {
      this.endTimer = Date.now();
      this.showTimer(this.endTimer - this.startTimer);
      clearInterval(this.timerId);
      // alert("game over");
    });
    EventBus.$on("start-timer", () => {
      this.startTimer = Date.now();
      this.timerId = setInterval(() => {
        //  拿到毫秒数
        let s = Date.now() - this.startTimer;
        this.showTimer(s);
      }, 500);
    });
  },
  methods: {
    showTimer(s) {
      this.ss = String(Math.trunc(s / 1000) % 60).padStart(2, "0");
      this.mm = String(Math.trunc(s / (1000 * 60)) % 60).padStart(2, "0");
      this.hh = String(Math.trunc(s / (1000 * 60 * 60)) % 60).padStart(2, "0");
    }
  }
};
</script>
<style lang="scss" scoped></style>
