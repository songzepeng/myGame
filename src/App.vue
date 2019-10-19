<template>
  <div id="app">
    <h1>my first game 扫雷</h1>
    <div class="container">
      <div class="main">
        <table class="mine-table">
          <tr v-for="(rowItem, rIndex) of rows" :key="'row_' + rIndex">
            <!-- <td  把td格做成了一个组件
              :style="{ width: cellWidth + 'px', height: cellHeight + 'px' }"
              v-for="(colItem, cIndex) of cols"
              :key="'col_' + cIndex"></td> -->
            <!-- 单元格的数据为 行的索引乘上列，加上列的索引，为哪个单元格 -->
            <cell
              @clearboom="clearBoom"
              :cellData="cellArray[rIndex * cols + cIndex]"
              :style="{ width: cellWidth + 'px', height: cellHeight + 'px' }"
              v-for="(colItem, cIndex) of cols"
              :key="'col_' + cIndex"
            ></cell>
          </tr>
        </table>
      </div>
      <div class="aside">
        <el-radio-group @change="changeLevel" v-model="level" size="mini">
          <el-radio-button label="简单" class="xuanx"></el-radio-button>
          <el-radio-button label="中级" class="xuanx"></el-radio-button>
          <el-radio-button label="高级" class="xuanx"></el-radio-button>
          <el-radio-button label="困难" class="xuanx"></el-radio-button>
        </el-radio-group>
        <div class="block">
          <span class="demonstration">列数{{ cols }}</span>
          <el-slider v-model="cols" :min="5" :max="100"></el-slider>
        </div>
        <div class="block">
          <span class="demonstration">行数{{ rows }}</span>
          <el-slider v-model="rows" :min="5" :max="100"></el-slider>
        </div>
        <div class="block">
          <el-switch
            style="display: block"
            v-model="isCellWHSync"
            active-color="#13ce66"
            inactive-color="#ff4949"
            active-text="是否同步宽高"
          >
          </el-switch>
          <div class="block">
            <span class="demonstration">行高{{ cellWidth }}</span>
            <el-slider v-model="cellWidth" :min="5"></el-slider>
          </div>
          <div class="block">
            <span class="demonstration">列高{{ cellHeight }}</span>
            <el-slider v-model="cellHeight" :min="5"></el-slider>
          </div>
        </div>
        <div class="block">
          <el-button @click="initCellData" type="warning">重置</el-button>
        </div>
      </div>
    </div>
    <play-sound src-sound="/01.mp3"></play-sound>
    <Timer></Timer>
  </div>
</template>

<script>
import "element-ui/lib/theme-chalk/index.css";
import Cell from "./components/Cell";
import playSound from "./components/playSound";
import Timer from "./components/Timer";
import EventBus from "./eventBus.js";
export default {
  data: function() {
    return {
      level: "简单", // 如果是简单的返回1 中级的返回2 高级返回3 困难返回4  100个单元格 * 0.2 =20% 的是雷
      cols: 10,
      rows: 10,
      cellWidth: 40,
      cellHeight: 40,
      isCellWHSync: true,
      // 此数组表示每个单元格的各个数据 比如：isBoom（爆炸）:false， is Marked（是否被标记）， isClear（是否清空）
      cellArray: [],
      // 是否是重置状态
      isReset: true
    };
  },
  created() {
    // 把里边的代码封装成了一个函数，便于多次调用
    this.initCellData();
    document.oncontextmenu = () => {
      return false;
    };
    EventBus.$on("click-cell", () => {
      this.isReset && EventBus.$emit("start-timer");
      this.isReset = false;
    });
    EventBus.$on("boom-end", () => {
      this.boomEnd();
    });
  },
  methods: {
    boomEnd() {
      // 把所有雷显示，把所有非雷clear
      this.cellArray.forEach(item => {
        if (item.isBoom) {
          // 如果是炸弹
          this.$set(item, "isShowBoom", true);
        } else {
          this.$set(item, "isClear", true);
        }
      });
    },
    // 清理雷区的操作 点到空白
    clearBoom(index) {
      // 封装清理操作的方法 ，下面直接调用
      const innerClearBoom = cIndex => {
        if (cIndex >= 0 && cIndex < this.cellArray.length) {
          let cell = this.cellArray[cIndex];
          // 判断是不是雷
          if (cell.isMarked || cell.isBoom || cell.isClear) {
            return;
          }
          this.$set(this.cellArray[cIndex], "isClear", true);
          //如果自己也是0,继续清理
          this.clearBoom(cIndex);
        }
      };
      // 如果自己当前不是空白（0），那么不请空。 isClear： 为true
      let cell = this.cellArray[index];
      if (cell.data !== 0) {
        // 进行判断， 如果单元格内不为0，则不清空
        this.$set(cell, "isClear", "true");
        return;
      }
      // 如果是空白（0），那么周围8个都要clear清理掉
      for (let i = -1; i <= 1; i++) {
        let startIndex = index - this.cols * i - 1;
        if (index % this.cols > 0) {
          // 处理边界问题
          innerClearBoom(startIndex);
        }
        innerClearBoom(startIndex + 1);
        if (index % this.cols < this.cols - 1) {
          // 处理边界问题
          innerClearBoom(startIndex + 2);
        }
      }
      // 如果周围的元素也是0，那么此元素周围也要清理
    },
    // 初始化数组数据
    initCellData() {
      //只要调用此方法，启动时钟
      this.isReset = true;
      // 初始化每个单元格的数据,也就是cellArray数组里动态随机生成的地雷数据
      // 先拿到总的单元格数 然后计算需要随机生成多少个地雷   sum * 0.2
      let sum = this.cols * this.rows;
      // 拿到生成的地雷数
      let randomBooms = this.getLevelNum() * 0.2 * sum;
      // 拿到生成的地雷索引数
      let randomIndexSet = new Set();
      while (randomIndexSet.size < randomBooms) {
        // 拿到了一个随机生成地雷索引的整数 trunc方法  拿到20个不同的索引
        randomIndexSet.add(Math.trunc(Math.random() * sum));
      }
      // 先截断清空一下这个数组
      this.cellArray = [];
      // console.log(randomIndexSet);
      //循环遍历一下 i小于单元格的数量（100），
      for (let i = 0; i < sum; i++) {
        let isBoom = randomIndexSet.has(i);
        // 拿到炸弹的索引数
        let data = this.getBoomsNum(i, randomIndexSet);
        // 把一个对象压栈到此数组  is Marked（是否被标记） isClear（是否清空）
        this.cellArray.push({
          isBoom,
          data,
          isMarked: false,
          isShowBoom: false,
          isClear: false,
          cellIndex: i
        });
      }
      // this.$Message("重置成功");
      console.log(randomBooms); // 打印炸弹数量
    },
    // 根据按钮返回值切换难易模式
    changeLevel() {
      this.initCellData();
    },
    // 第一个参数为索引，第二个参数为炸弹的位置
    getBoomsNum(index, boomsSet) {
      let count = 0; // 周围的炸弹数
      // for循环遍历从第一行开始，第一行默认为-1
      for (let i = -1; i <= 1; i++) {
        // 这个拿到了中间点上下中行的索引
        let startIndex = index - i * this.cols - 1;
        if (index % this.cols > 0) {
          // 处理边界问题
        }
        count += boomsSet.has(startIndex);
        count += boomsSet.has(startIndex + 1);
        count += boomsSet.has(startIndex + 2);
        if (index % this.cols > this.cols - 1) {
          // 处理边界问题
        }
      }
      return count;
    },
    // 拿到层次
    getLevelNum() {
      if (this.level === "简单") {
        return 1;
      } else if (this.level === "中级") {
        return 2;
      } else if (this.level === "高级") {
        return 3;
      } else {
        return 4;
      }
    }
  },
  watch: {
    //  监听同步设置宽和高，如果isCellWHSync 为true 则设置
    cellWidth(newVal) {
      this.isCellWHSync && (this.cellHeight = newVal);
    },
    cellHeight(newVal) {
      this.isCellWHSync && (this.cellWidth = newVal);
    },
    // 监听行数列数，改变的时候要初始化一下，不然会报错
    rows() {
      this.initCellData();
    },
    cols() {
      this.initCellData();
    }
  },
  name: "app",
  components: {
    cell: Cell,
    "play-sound": playSound,
    Timer: Timer
  }
};
</script>

<style lang="scss">
h1 {
  text-align: center;
}
.container {
  display: flex;
  & > div {
    border: 1px solid #ccc;
  }
  .main {
    flex: 5 0 500px;
    min-height: 500px;
    .mine-table {
      border-collapse: collapse;
      td {
        width: 30px;
        height: 30px;
        border: 1px solid #4b5cc4;
        background-color: #c0c0c0;
      }
    }
  }
  .aside {
    flex: 2 0 200px;
    min-height: 500px;
    .xuanx {
      padding-top: 5px;
      margin-right: 2px;
    }
    .block {
      border-top: 1px solid #ccc;
      margin-top: 5px;
      .demonstration {
        color: #a1afc9;
      }
    }
  }
}
</style>
