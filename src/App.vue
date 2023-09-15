<script lang="ts">
import { defineComponent } from 'vue';

interface Cell {
  opened: boolean;
  mine: boolean;
  exploded: boolean;
  content: string | number;
  row: number,
  col: number,
}

export default defineComponent({
  data() {
    return {
      board: [] as Cell[][],
      rows: 10,
      columns: 10,
      mines: 10,
      gameOver: false,
      sweepedCnt: 0
    };
  },
  mounted() {
    this.resetBoard();
  },
  methods: {
    resetBoard() {
      this.board = [];
      this.gameOver = false;

      // 初始化游戏板
      for (let i = 0; i < this.rows; i++) {
        const row: Cell[] = [];
        for (let j = 0; j < this.columns; j++) {
          row.push({ opened: false, mine: false, exploded: false, content: '', row: i, col: j });
        }
        this.board.push(row);
      }

      // 埋雷
      let minesCount = 0;
      while (minesCount < this.mines) {
        const randomRow = Math.floor(Math.random() * this.rows);
        const randomColumn = Math.floor(Math.random() * this.columns);
        if (!this.board[randomRow][randomColumn].mine) {
          this.board[randomRow][randomColumn].mine = true;
          minesCount++;
        }
      }
    },
    openCell(row: number, column: number) {
      if (this.gameOver) return;

      const cell = this.board[row][column];

      if (cell.opened || cell.content === '☥') return;
      cell.opened = true;

      if (cell.mine) {
        cell.exploded = true;
        this.gameOver = true;
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

      if (cell.content === '') {
        // 计算相邻格子的雷数
        const neighbors = this.getNeighbors(row, column);
        const minesCount = neighbors.filter((neighbor) => neighbor.mine).length;

        if (minesCount === 0) {
          // 递归打开相邻的空格子
          neighbors.forEach((neighbor) => {
            this.openCell(neighbor.row, neighbor.col);
          });
        } else {
          cell.content = minesCount;
        }
      }
    },
    flagCell(row: number, column: number) {
      if (this.gameOver) return;

      const cell = this.board[row][column];

      if (cell.opened) return;

      cell.content = cell.content === '☥' ? '' : '☥';
    },
    getNeighbors(row: number, column: number) {
      const neighbors: Cell[] = [];

      for (let i = Math.max(0, row - 1); i <= Math.min(row + 1, this.rows - 1); i++) {
        for (let j = Math.max(0, column - 1); j <= Math.min(column + 1, this.columns - 1); j++) {
          if (i === row && j === column) continue;
          neighbors.push({ ...this.board[i][j] });
        }
      }

      return neighbors;
    },
  },
});
</script>

<template>
  <div class="minesweeper">
    <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
      <div v-for="(cell, columnIndex) in row" :key="columnIndex" class="cell"
        :class="{ opened: cell.opened, mine: cell.exploded }" @click="openCell(rowIndex, columnIndex)"
        @contextmenu.prevent="flagCell(rowIndex, columnIndex)">
        {{ cell.content }}
      </div>
    </div>
    <button class="reset" @click="resetBoard">重新开始</button>
  </div>
</template>



<style scoped>
@import "@/assets/button.css"
</style>