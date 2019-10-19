<template>
  <!-- 绑定属性：mine-clear，把cellDate数据的isClear方法赋值给她 -->
  <td
    :class="{
      boom: cellData.isShowBoom,
      'mine-clear': cellData.isClear,
      marked: cellData.isMarked
    }"
    class="cell"
    @mousedown="cellClick"
  >
    <!-- 模板显示判断 如果单元格是isboom 显示giao -->
    <!-- <template v-if="cellData.isBoom">
      giao
    </template> -->
    <!-- 否则显示单元格的数据 -->
    <span
      :style="{ color: getNumColor(cellData.data) }"
      v-if="cellData.data != 0 && cellData.isClear"
    >
      <!-- 再次判断，单元格内容不等于0的时候不显示 -->
      {{ cellData.data }}
    </span>
  </td>
</template>

<script>
import EventBus from "../eventBus.js";
export default {
  name: "cell",
  data() {
    return {};
  },
  methods: {
    // 美化字体
    getNumColor(num) {
      switch (num) {
        case 1:
          return "#44cef6";
        case 2:
          return "#f0c239";
        case 3:
          return "#f20c00";
        case 4:
          return "#ff0097";
        case 5:
          return "#f9906f";
        case 6:
          return "#bddd22";
        default:
          return "#ccc";
      }
    },
    cellClick(e) {
      // console.log(e.button);
      if (this.cellData.isClear) {
        return; // 误操作直接结束
      }

      EventBus.$emit("click-cell");

      // 如果点击了鼠标右键 标记为marked状态
      if (e.button == 2) {
        EventBus.$emit("click-cell");
        this.$set(this.cellData, "isMarked", !this.cellData.isMarked);
        return;
      }
      if (e.button === 0) {
        // 点击了鼠标左键的时候，处理操作  0代表点击了鼠标左键
        if (this.cellData.isBoom) {
          // 判断他的单元格内容是不是炸弹，如果是发送 游戏结束的事件
          EventBus.$emit("boom-end");
          // console.log("炸弹");
        } else {
          // 如果是标记了小旗子，那么让鼠标左键无效
          if (this.cellData.isMarked) {
            return;
          }
          EventBus.$emit("click-cell");
          // 否则：不是炸弹， 让里边的数字正常显示
          // this.$set(this.cellData, "isClear", true); // 此行代码调用set方法，修改他的cellData数据（单元格数据），cellData（单元格数据）里的isClear，值改为true
          // console.log("clear");

          //清理操作 不应该子组件做，发送清理的坐标index给父组件
          this.$emit("clearboom", this.cellData.cellIndex); // 给父组件发送一个名为clearboom 的方法
        }
      }
    }
  },
  props: {
    // 单元格的数据 类型是对象
    cellData: {
      type: Object,
      required: false
    }
  }
};
</script>
<style lang="scss" scoped>
td {
  text-align: center;
  // 表格内的方法，单元格内的内容垂直居中
  vertical-align: middle;
}
.mine-clear {
  background-color: #333 !important;
}
.marked {
  background-image: url(/qz.png) !important;
  background-size: cover;
}
.boom {
  position: relative;
  &::before {
    content: "";
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: url(/爆炸.png) !important;
    background-size: contain;
  }
}
</style>
