<template>
  <div class="fieldContainer">
    <div class="message">
      <p class="p1Win" v-if="win == 1">P1 WIN ({{ p1score }})</p>
      <p class="p2Win" v-if="win == -1">P2 WIN ({{ p2score }})</p>
      <p class="draw" v-if="win == 0">DRAW ({{ p1score }})</p>
    </div>
    <div class="score">
      <div
        class="p1Score"
        :class="{ p1ScoreBorder: player == 1 }"
        :style="{ width: p1score * 300 + 20 + 'px' }"
      >
        {{ p1score }}
      </div>
      <div
        class="p2Score"
        :class="{ p2ScoreBorder: player == -1 }"
        :style="{ width: p2score * 300 + 20 + 'px' }"
      >
        {{ p2score }}
      </div>
    </div>
    <div class="field" :class="border">
      <div class="row" v-for="(row, rowIndex) in fieldCells" :key="rowIndex">
        <div
          class="cell"
          v-for="(col, colIndex) in row"
          :key="colIndex"
          @click="putCell(rowIndex, colIndex, player)"
        >
          <p v-if="rowIndex == finalPutCell[0] && colIndex == finalPutCell[1]">
            ・
          </p>
          <div :class="cellStatus(col)"></div>
        </div>
      </div>
    </div>
    <div class="control">
      <div class="button" v-if="!gameEnd" @click="restart()">reset</div>
      <div
        class="button"
        v-if="gameEnd"
        @click="
          restart();
          gameEnd = false;
          player = 1;
          beforeAction();
        "
      >
        start
      </div>
      <div
        class="button"
        v-if="gameEnd"
        @click="
          restart();
          gameEnd = false;
          player = 1;
          isCPU = 1;
          beforeAction();
          cpuPut();
        "
      >
        CPU(先手)
      </div>
      <div
        class="button"
        v-if="gameEnd"
        @click="
          restart();
          gameEnd = false;
          player = 1;
          isCPU = -1;
          beforeAction();
        "
      >
        CPU(後手)
      </div>
      <div class="button" v-if="pass" @click="doPass">pass</div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  data() {
    return {
      fieldCells: [
        [0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, -1, 1, 0, 0, 0],
        [0, 0, 0, 1, -1, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0],
      ],
      player: 0,
      gameEnd: true,
      gameMessage: "",
      // actionMemory: [],
      fieldWidth: 8,
      fieldHeight: 8,
      ableCells: [],
      pass: false,
      p1score: 2,
      p2score: 2,
      isCPU: 0,
      finalPutCell: [-1, -1],
      win: 10,
    };
  },
  computed: {
    border: function() {
      if (this.player < 0) return "borderP2";
      return "borderP1";
    },
  },
  methods: {
    restart() {
      for (let i = 0; i < this.fieldHeight; i++) {
        for (let j = 0; j < this.fieldWidth; j++) {
          if ((i == 3 && j == 3) || (i == 4 && j == 4)) {
            this.fieldCells[i].splice(j, 1, -1);
          } else if ((i == 3 && j == 4) || (i == 4 && j == 3)) {
            this.fieldCells[i].splice(j, 1, 1);
          } else {
            this.fieldCells[i].splice(j, 1, 0);
          }
        }
      }
      this.player = 1;
      this.gameMessage = "";
      this.gameEnd = true;
      this.ableCells = [];
      this.player = 0;
      this.isCPU = 0;
      this.win = 10;
      this.p1score = 2;
      this.p2score = 2;
      this.finalPutCell = [-1, -1];
      // this.beforeAction();
    },
    ableReset() {
      for (let i = 0; i < this.fieldHeight; i++) {
        for (let j = 0; j < this.fieldWidth; j++) {
          if (this.fieldCells[i][j] == 7 || this.fieldCells[i][j] == -7)
            this.fieldCells[i].splice(j, 1, 0);
        }
      }
    },
    cellStatus(col) {
      switch (col) {
        case 0:
          return "none";
        case 1:
          return "p1";
        case -1:
          return "p2";
        case 7:
          return "p1AbleCell";
        case -7:
          return "p2AbleCell";
      }
    },
    direction(direction) {
      let ret = [0, 0];
      switch (direction) {
        case 1:
          ret = [-1, 0];
          break;
        case 2:
          ret = [-1, 1];
          break;
        case 3:
          ret = [0, 1];
          break;
        case 4:
          ret = [1, 1];
          break;
      }
      return ret;
    },
    doPass() {
      this.pass = false;
      this.player *= -1;
      this.beforeAction();
      if (this.isCPU == this.player) this.cpuPut();
    },
    beforeAction() {
      this.p1score = 0;
      this.p2score = 0;
      for (let i = 0; i < this.fieldHeight; i++) {
        for (let j = 0; j < this.fieldWidth; j++) {
          if (this.fieldCells[i][j] == 0) {
            let canPlacedCells = this.judge(i, j, this.player);
            if (canPlacedCells[0]) {
              this.ableCells.push([i, j, canPlacedCells]);
              this.fieldCells[i].splice(j, 1, this.player * 7);
              this.fieldCells[i][j] = this.player * 7;
            }
          } else if (this.fieldCells[i][j] == 1) this.p1score++;
          else this.p2score++;
        }
      }
    },
    chainChange(row, col, direction, chain, player) {
      let move = this.direction(direction);
      let put = function(i, field) {
        field[row + move[0] * i].splice(col + move[1] * i, 1, player);
        if (i > 0) {
          i++;
          if (i <= chain) setTimeout(put, 100, i, field);
        } else {
          i--;
          if (i >= chain) setTimeout(put, 100, i, field);
        }
      };
      if (chain > 0) setTimeout(put, 100, 1, this.fieldCells);
      else setTimeout(put, 100, -1, this.fieldCells);
    },
    putCell(row, col, player) {
      this.finalPutCell[0] = row;
      this.finalPutCell[1] = col;
      let canPlacedCells;
      let maxChain = 0;
      if (this.fieldCells[row][col] != 7 * player || this.gameEnd) return;
      for (let i = 0; i < this.ableCells.length; i++) {
        if (this.ableCells[i][0] == row && this.ableCells[i][1] == col) {
          canPlacedCells = this.ableCells[i][2];
        }
      }
      this.fieldCells[row].splice(col, 1, player);
      for (let i = 0; i < canPlacedCells.length; i++) {
        if (maxChain < Math.abs(canPlacedCells[i][1]))
          maxChain = Math.abs(canPlacedCells[i][1]);
        this.chainChange(
          row,
          col,
          canPlacedCells[i][0],
          canPlacedCells[i][1],
          player
        );
      }
      let afterPutCell = () => {
        this.ableReset();
        this.ableCells = [];
        this.player *= -1;
        this.beforeAction();
        if (!this.ableCells[0]) {
          this.player *= -1;
          this.beforeAction();
          if (!this.ableCells[0]) {
            this.gameEnd = true;
            this.player = 0;
            if (this.p1score > this.p2score) {
              this.win = 1;
            } else if (this.p1score < this.p2score) {
              this.win = -1;
            } else {
              this.win = 0;
            }
          } else {
            this.ableReset();
            this.player *= -1;
            this.ableCells = [];
            this.beforeAction;
            this.pass = true;
          }
        }
        if (this.isCPU == this.player) this.cpuPut();
      };
      setTimeout(afterPutCell, 100 * (maxChain + 1));
    },
    judge(row, col, player) {
      let canPlacedCells = []; //[direction,count]
      let count;
      let moved;
      let directionAdd;
      for (let i = 1; i < 5; i++) {
        directionAdd = this.direction(i);
        count = 0;
        for (let j = 1; j < 10; j++) {
          moved = [row + directionAdd[0] * j, col + directionAdd[1] * j];
          if (
            !this.isAbleCell(moved[0], moved[1]) ||
            this.fieldCells[moved[0]][moved[1]] == 0 ||
            this.fieldCells[moved[0]][moved[1]] == 7 * this.player
          )
            break;
          if (this.fieldCells[moved[0]][moved[1]] == player * -1) count++;
          if (this.fieldCells[moved[0]][moved[1]] == player) {
            if (count > 0) canPlacedCells.push([i, count]);
            break;
          }
        }
        count = 0;
        for (let j = 1; j < 10; j++) {
          moved = [row + directionAdd[0] * -j, col + directionAdd[1] * -j];
          if (
            !this.isAbleCell(moved[0], moved[1]) ||
            this.fieldCells[moved[0]][moved[1]] == 0 ||
            this.fieldCells[moved[0]][moved[1]] == 7 * this.player
          )
            break;
          if (this.fieldCells[moved[0]][moved[1]] == player * -1) count--;
          if (this.fieldCells[moved[0]][moved[1]] == player) {
            if (count < 0) canPlacedCells.push([i, count]);
            break;
          }
        }
      }
      return canPlacedCells;
    },
    isAbleCell(row, col) {
      if (
        row >= 0 &&
        row < this.fieldHeight &&
        col >= 0 &&
        col < this.fieldWidth
      )
        return true;
      return false;
    },
    cpuPut() {
      if (this.ableCells[0]) {
        let putCell = this.searchBestCell(this.ableCells);
        this.putCell(putCell[0], putCell[1], this.player);
      } else {
        this.doPass();
      }
    },
    searchBestCell(cells) {
      const scoreBoard = [
        [20, -2, 8, 8, 8, 8, -2, 20],
        [-2, -8, -6, -6, -6, -6, -6, -2],
        [8, -6, 0, 0, 0, 0, -6, 8],
        [8, -6, 0, 0, 0, 0, -6, 8],
        [8, -6, 0, 0, 0, 0, -6, 8],
        [8, -6, 0, 0, 0, 0, -6, 8],
        [-2, -8, -6, -6, -6, -6, -8, -2],
        [20, -2, 8, 8, 8, 8, -2, 20],
      ];
      let maxScore = -1000;
      let score;
      let retArr = [-1, -1];
      for (let i = 0; i < cells.length; i++) {
        score = 0;
        score += Math.abs(cells[i][2][0][1]);
        score += scoreBoard[cells[i][0]][cells[i][1]];
        if (score > maxScore) {
          retArr[0] = cells[i][0];
          retArr[1] = cells[i][1];
          maxScore = score;
        }
      }
      return retArr;
    },
  },
  mounted: function() {
    // this.beforeAction();
  },
});
</script>

<style>
@media screen and (min-width: 600px) {
  body {
    width: 600px;
    margin: 0 auto;
  }
  html {
    background-color: rgb(34, 52, 70);
  }
}
</style>

<style scoped>
.fieldContainer {
  height: calc(100vh - 64px);
  padding-top: 40px;
  background-color: rgb(217, 223, 224);
}
.message {
  height: 68px;
}
.message p {
  text-align: center;
  line-height: 48px;
  font-size: 31px;
}
.p1Win {
  color: darksalmon;
}
.p2Win {
  color: mediumaquamarine;
}
.p1Score {
  background-color: darksalmon;
  border: 2px solid darksalmon;
  border-radius: 30px;
  text-align: center;
}
.p1ScoreBorder {
  border-color: rgb(153, 104, 88);
}
.p2Score {
  background-color: mediumaquamarine;
  border: 2px solid mediumaquamarine;
  border-radius: 30px;
  text-align: center;
}
.p2ScoreBorder {
  border-color: rgb(71, 126, 107);
}
.score {
  width: 90vw;
  margin: 0px auto 10px;
  padding: 5px 10px;
  background-color: rgb(156, 206, 238);
  border-radius: 30px;
  display: flex;
  justify-content: space-around;
}
.field {
  width: 90vw;
  height: 90vw;
  margin: 0 auto;
  border-bottom: solid 1px;
  border-right: solid 1px;
}
.borderP1 {
  border-color: darksalmon;
}
.borderP1 .cell {
  border-color: darksalmon;
}
.borderP2 {
  border-color: mediumaquamarine;
}
.borderP2 .cell {
  border-color: mediumaquamarine;
}
.row {
  display: flex;
  height: 12.5%;
}
.cell {
  width: 12.5%;
  border-top: solid 1px;
  border-left: solid 1px;
  position: relative;
}
.cell p {
  position: absolute;
  top: 50%;
  width: 100%;
  text-align: center;
  transform: translateY(-50%);
  font-size: 40px;
  color: deeppink;
}
.p1 {
  height: 100%;
  background-color: darksalmon;
  border-radius: 50%;
}
.p2 {
  height: 100%;
  background-color: mediumaquamarine;
  border-radius: 50%;
}
.p1AbleCell {
  height: 100%;
  border: 2px solid rgb(209, 102, 70);
  border-radius: 50%;
}
.p2AbleCell {
  height: 100%;
  border: 2px solid rgb(104, 143, 131);
  border-radius: 50%;
}
.control {
  width: 90vw;
  margin: 10px auto 0px;
  padding: 5px 0;
  background-color: rgb(156, 206, 238);
  border-radius: 30px;
  display: flex;
  justify-content: space-around;
}
.button {
  background-color: paleturquoise;
  padding: 0 7px;
  border-radius: 12px;
  cursor: pointer;
}
@media screen and (min-width: 445px) {
  .field {
    width: 400px;
    height: 400px;
  }
  .control {
    width: 400px;
  }
  .score {
    width: 400px;
  }
}
</style>
