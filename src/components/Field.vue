<template>
  <div class="fieldContainer">
    <div class="score">
      <div class="p1Score" :style="{ width: p1score * 300 + 'px' }">
        {{ p1score }}
      </div>
      <div class="p2Score" :style="{ width: p2score * 300 + 'px' }">
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
          <div :class="cellStatus(col)"></div>
        </div>
      </div>
    </div>
    <div class="control">
      <div class="button" @click="restart()">reset</div>
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
      player: 1,
      gameEnd: false,
      // actionMemory: [],
      fieldWidth: 8,
      fieldHeight: 8,
      ableCells: [],
      pass: false,
      p1score: 0,
      p2score: 0,
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
      this.gameEnd = false;
      this.ableCells = [];
      this.beforeAction();
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
    },
    end() {},
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
      for (let i = 1; i <= chain; i++) {
        this.fieldCells[row + move[0] * i][col + move[1] * i] = player;
      }
      for (let i = -1; i >= chain; i--) {
        this.fieldCells[row + move[0] * i][col + move[1] * i] = player;
      }
    },
    putCell(row, col, player) {
      let canPlacedCells;
      if (this.fieldCells[row][col] != 7 * player || this.gameEnd) return;
      for (let i = 0; i < this.ableCells.length; i++) {
        if (this.ableCells[i][0] == row && this.ableCells[i][1] == col) {
          canPlacedCells = this.ableCells[i][2];
        }
      }
      this.fieldCells[row].splice(col, 1, player);
      for (let i = 0; i < canPlacedCells.length; i++) {
        this.chainChange(
          row,
          col,
          canPlacedCells[i][0],
          canPlacedCells[i][1],
          player
        );
      }
      this.ableReset();
      this.ableCells = [];
      this.player *= -1;
      this.beforeAction();
      if (!this.ableCells[0]) {
        this.player *= -1;
        this.beforeAction();
        if (!this.ableCells[0]) this.end();
        else {
          this.ableReset();
          this.player *= -1;
          this.ableCells = [];
          this.beforeAction;
          this.pass = true;
        }
      }
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
    searchBestCell() {
      this.ableCells;
    },
  },
  mounted: function() {
    this.beforeAction();
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
  padding-top: 120px;
  background-color: rgb(217, 223, 224);
}
.p1Score {
  background-color: darksalmon;
  border-radius: 30px;
  text-align: center;
}
.p2Score {
  background-color: mediumaquamarine;
  border-radius: 30px;
  text-align: center;
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
  border: 2px solid rgb(235, 124, 91);
  border-radius: 50%;
}
.p2AbleCell {
  height: 100%;
  border: 2px solid rgb(84, 112, 103);
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
