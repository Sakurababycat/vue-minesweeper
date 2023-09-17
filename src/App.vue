<script lang="ts" >

interface Cell {
  opened: boolean;
  mine: boolean;
  exploded: boolean;
  content: string | number;
  row: number;
  col: number;
  shin: boolean;
}

export default {
  data() {
    return {
      board: [] as Cell[][],
      rows: 9,
      columns: 9,
      mines: 10,
      gameOver: false,
      sweepedCnt: 0,
      win: false,
      custom: false,
      fisrtClick: true,
      time: 0,
      timer: 0,
      minecnt: 0,
    };
  },
  watch: {
    sweepedCnt(newV: number) {
      if (this.rows * this.columns - this.mines === newV) {
        this.gameOver = true;
        this.win = true;
      }
    },
  },
  computed: {
    rowsC: {
      get(): number {
        return this.rows;
      },
      set(newValue: number) {
        this.rows = newValue < 1 ? 1 : newValue > 40 ? 40 : newValue;
        this.minesC = this.mines
      }
    },
    columnsC: {
      get(): number {
        return this.columns;
      },
      set(newValue: number) {
        this.columns = newValue < 1 ? 1 : newValue > 40 ? 40 : newValue;
        this.minesC = this.mines
      }
    },
    minesC: {
      get(): number {
        return this.mines;
      },
      set(newValue: number) {
        this.mines = newValue > this.rowsC * this.columnsC - 9 || newValue < 0 ? 0 : newValue;
      }
    }
  },
  mounted() {
    this.resetBoard();
  },
  methods: {
    resetBoard() {
      if (this.rows < 5 || this.rows > 40 || this.columns < 5 || this.columns > 40 || this.mines > this.columns * this.rows - 9) return;
      this.board = [];
      this.gameOver = false;

      // 初始化游戏板
      for (let i = 0; i < this.rows; i++) {
        const row: Cell[] = [];
        for (let j = 0; j < this.columns; j++) {
          row.push({ opened: false, mine: false, exploded: false, content: '', row: i, col: j, shin: false });
        }
        this.board.push(row);
      }
      this.fisrtClick = true;
      this.sweepedCnt = 0;
      this.minecnt = this.mines;
      this.time = 0;
      clearInterval(this.timer);
    },
    setMines(crow: number, ccol: number) {

      // 埋雷
      let minesCount = 0;
      while (minesCount < this.mines) {
        const randomRow = Math.floor(Math.random() * this.rows);
        const randomColumn = Math.floor(Math.random() * this.columns);
        if (!this.board[randomRow][randomColumn].mine && !(Math.abs(randomRow - crow) <= 1 && Math.abs(randomColumn - ccol) <= 1)) {
          this.board[randomRow][randomColumn].mine = true;
          minesCount++;
        }
      }
    },
    openCell(row: number, column: number, from_click: boolean = true) {
      if (this.gameOver) return;
      if (this.fisrtClick) {
        this.fisrtClick = false;
        this.setMines(row, column);
        this.timer = setInterval(() => {
          if (this.gameOver) {
            clearInterval(this.timer);
          }
          else {
            this.time++;
          }
        }, 1000);
      }

      const cell = this.board[row][column];

      if (cell.content === '☥') return;
      if (cell.opened) {
        if (from_click) {
          this.flagCell(row, column);
        }
        return;
      }
      cell.opened = true;

      if (cell.mine) {
        cell.exploded = true;
        this.gameOver = true;
        this.win = false
        this.board.forEach((rowCells) => {
          rowCells.forEach((cell) => {
            if (cell.mine && cell.content != '☥') cell.content = '✿';
            if (!cell.mine && cell.content === '☥') { cell.exploded = true; }
          });
        });

        // this.$nextTick().then(() => {
        //   alert('游戏结束！你踩中了地雷。');
        //   this.resetBoard();
        // });
        // 处理游戏结束逻辑
        return;
      }
      this.sweepedCnt++;

      if (cell.content === '') {
        // 计算相邻格子的雷数
        const neighbors = this.getNeighbors(row, column);
        const minesCount = neighbors.filter((neighbor) => neighbor.mine).length;

        if (minesCount === 0) {
          // 递归打开相邻的空格子
          neighbors.forEach((neighbor) => {
            this.openCell(neighbor.row, neighbor.col, false);
          });
        } else {
          cell.content = minesCount;
        }
      }
    },
    flagCell(row: number, column: number) {
      if (this.gameOver || this.fisrtClick) return;

      const cell = this.board[row][column];

      if (!cell.opened) {
        cell.content === '☥' ? this.minecnt++ : this.minecnt--;
        cell.content = cell.content === '☥' ? '' : '☥';
      }
      else {
        const neighbors = this.getNeighbors(row, column);
        const neighborMines = neighbors.filter((neighbor) => neighbor.mine);
        const neighborFlags = neighbors.filter((neighbor) => neighbor.content === '☥');
        const neighborCanOpen = neighbors.filter((neighbor) => !neighbor.opened);

        if (neighborFlags.length === neighborMines.length) {
          // 递归打开相邻的空格子
          neighbors.forEach((neighbor) => {
            this.openCell(neighbor.row, neighbor.col, false);
          });
        } else if (neighborCanOpen.length === neighborMines.length) {
          neighborCanOpen.filter((neighbor) => neighbor.content === '').forEach((neighbor) => {
            this.flagCell(neighbor.row, neighbor.col);
          });
        }
        else {
          neighbors.filter((neighbor) => !neighbor.opened).forEach((ncell) => {
            let blinkCount = 0;
            const blinkInterval = setInterval(() => {
              ncell.shin = !ncell.shin;
              blinkCount++;
              if (blinkCount === 6) { // 闪烁3次，6个状态切换
                clearInterval(blinkInterval);
              }
            }, 100);
          })
        }
      }
    },
    getNeighbors(row: number, column: number) {
      const neighbors: Cell[] = [];

      for (let i = Math.max(0, row - 1); i <= Math.min(row + 1, this.rows - 1); i++) {
        for (let j = Math.max(0, column - 1); j <= Math.min(column + 1, this.columns - 1); j++) {
          if (i === row && j === column) continue;
          neighbors.push(this.board[i][j]);
        }
      }

      return neighbors;
    },
    onEasy() {
      this.rows = 9;
      this.columns = 9;
      this.mines = 10;
      this.custom = false;
    },
    onMedium() {
      this.rows = 16;
      this.columns = 16;
      this.mines = 40;
      this.custom = false;
    },
    onHard() {
      this.rows = 16;
      this.columns = 30;
      this.mines = 99;
      this.custom = false;
    },
    onCustom() {
      this.custom = true;
    }
  },
}

</script>

<template>
  <h1 class="header">Tomoyo-MineSweeper</h1>
  <div class="input-container" @submit.prevent="resetBoard" @input="resetBoard">
    <input type="radio" id="low" name="level" value="low" @input="onEasy" checked>
    <label for="low">低级</label>
    <input type="radio" id="medium" name="level" value="medium" @input="onMedium">
    <label for="medium">中级</label>
    <input type="radio" id="high" name="level" value="high" @input="onHard">
    <label for="high" class="space">高级</label>

    <input type="radio" id="custom" name="level" value="custom" @input="onCustom">
    <label for="custom">自定义</label>
    <label for="length">长:</label><input type="number" min="5" max="99" v-model="rowsC" :disabled="!custom">
    <label for="length">宽:</label><input type="number" min="5" max="99" v-model="columnsC" :disabled="!custom">
    <label for="length">雷数:</label><input type="number" min="5" max="99" v-model="minesC" :disabled="!custom">
  </div>
  <div class="show-box">
    <div class="showl">
      <p>{{ time }}</p>
    </div>
    <div class="showr">
      <p>{{ minecnt }}</p>
    </div>
  </div>
  <div class="minesweeper">
    <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
      <div v-for="(cell, columnIndex) in row" :key="columnIndex" class="cell" :class="{
        opened: cell.opened || cell.shin, mine: cell.exploded, ['number-' + cell.content]: typeof cell.content === 'number'
      }" @click="openCell(rowIndex, columnIndex)" @contextmenu.prevent="flagCell(rowIndex, columnIndex)">
        {{ cell.content }}
      </div>
    </div>
    <h2 v-if="gameOver" class="minesweeper">{{ !win ? "游戏结束！你踩中了地雷。" : "游戏胜利" }}</h2>
    <button class="reset" @click="resetBoard">重新开始</button>
  </div>
</template>



<style scoped>
@import "@/assets/button.css"
</style>