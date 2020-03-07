<template>
  <div class="game-board">
    <template v-for="(v, x) in cells">
      <game-cell
        class="game-cell"
        v-for="(cell, y) in v"
        :key="`${x}_${y}`"
        :owner="cell.owner"
        :put="cell.put"
        @click="placeStone(x, y)"
      />
    </template>
  </div>
</template>

<script>
import GameCell from "./GameCell";
/* eslint-disable */
/** Player1を示す定数 */
const P1 = 1;
/** Player2を示す定数 */
const P2 = -1;
/** まだ石が置かれていないセル */
const BLANK = 0;
/** プレイヤー毎の置かれる石（文字） */
const STONES = { [P1]: "先手", [P2]: "後手" };
/** 盤面のサイズ(偶数のみ) */
const BOARD_SIZE = 6;
// /** 盤面の配列を作る？ */
// const CELL_RANGE = Array.from(Array(BOARD_SIZE), (_, i) => i);
/** 引っ繰り返す方向は8方向なので定義しておく */
const OFFSET = [
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, -1],
  [0, 1],
  [1, -1],
  [1, 0],
  [1, 1]
];
export default {
  components: { GameCell },
  data() {
    let cells = Array(BOARD_SIZE)
      .fill(0)
      .map(_ =>
        Array(BOARD_SIZE)
          .fill(0)
          .map(_ => ({ owner: BLANK, put: false }))
      );
    // 初期配置
    const ini1 = BOARD_SIZE / 2 - 1;
    const ini2 = BOARD_SIZE / 2;
    cells[ini1][ini1].owner = P2;
    cells[ini1][ini2].owner = P1;
    cells[ini2][ini1].owner = P1;
    cells[ini2][ini2].owner = P2;
    return {
      cells: cells,
      turn: P1
    };
  },
  computed: {
    /** 勝者(P1 or P2)。未決と引き分けの場合undefined */
    winner() {
      let winnerPName;
      // cellsの合計が+ならP1-ならP2の駒が多い
      const stoneCnt = this.cells.reduce(
        (a, b) => a + b.reduce((p, x) => p + x.owner, 0),
        0
      );
      if (stoneCnt > 0) {
        winnerPName = P1;
      } else if (stoneCnt < 0) {
        winnerPName = P2;
      }
      return winnerPName;
    },
    /** 盤面が埋まっていればtrue */
    isGameEnded() {
      return (
        this.cells.filter(v => v.filter(cell => cell.owner === BLANK).length)
          .length === 0
      );
    }
  },
  methods: {
    /** 指定のセルに現在のプレイヤーの石を置いて、ターンを交換。
     * すでにゲームが終了している場合や石が置けないセルの場合、何もしない。
     */
    placeStone(x, y) {
      if (this.isGameEnded) {
        return;
      }
      if (this.cells[x][y].owner !== BLANK) {
        return;
      }
      let flippables = this.checkStone(x, y);
      if (flippables.length == 0) {
        return;
      }
      flippables.map(v => {
        for (const [dx, dy] of v) {
          // 返せるところを返していく
          this.cells[dx][dy].owner = this.turn;
        }
      });
      this.cells[x][y].owner = this.turn;
      // ターン交代
      this.turn = this.turn === P1 ? P2 : P1;
      if (this.isGameEnded) {
        // どちらかの勝ち or 引き分け
        this.$emit("gameEnd", STONES[this.winner]);
      } else {
        if (this.checkAllStone() === 0) {
          // 置けない場合はターン交代
          this.turn = this.turn === P1 ? P2 : P1;
          if (this.checkAllStone() === 0) {
            // どちらも置けない場合終了
            this.$emit("gameEnd", STONES[this.winner]);
            return;
          }
        }
        this.$emit("nextTurn", STONES[this.turn]);
      }
    },
    /** 指定のセルに現在のプレイヤーの石を置いたとして、
     * ひっくり返せる石を返却する。
     */
    checkStone(x, y) {
      let flippables = [];
      if (this.cells[x][y].owner !== BLANK) {
        return flippables;
      }
      for (const [dx, dy] of OFFSET) {
        let tmp = [];
        let depth = 0;
        let rx, ry, request;
        for (;;) {
          depth++;
          rx = x + dx * depth;
          ry = y + dy * depth;
          if (0 <= rx && rx < BOARD_SIZE && 0 <= ry && ry < BOARD_SIZE) {
            request = this.cells[rx][ry].owner;
            if (request == BLANK) break;
            if (request == this.turn) {
              if (tmp.length != 0) {
                flippables.push(tmp);
              }
              break;
            } else {
              tmp.push([rx, ry]);
            }
          } else {
            break;
          }
        }
      }
      return flippables;
    },
    /** ひっくり返せるマスの枠色を変更し
     * 駒を置ける数を返却
     */
    checkAllStone() {
      // 置ける位置のカウント
      let putCnt = 0;
      this.cells.map((v, x) =>
        v.map((cell, y) => {
          cell.put = this.checkStone(x, y).length > 0;
          if (cell.put) {
            putCnt++;
          }
        })
      );
      return putCnt;
    }
  },
  mounted() {
    // 最初のターンを通知するためにイベントを発行
    this.checkAllStone();
    this.$emit("nextTurn", STONES[this.turn]);
  }
};
</script>

<style lang="scss" scoped>
.game-board {
  display: flex;
  flex-wrap: wrap;
  width: 100%;
  .game-cell {
    box-sizing: border-box;
    width: 16%;
    font-size: 100px;
    text-align: bottom;
    line-height: 100%;
    &::before {
      content: "";
      padding-top: 100%;
      display: block;
    }
  }
}
</style>
