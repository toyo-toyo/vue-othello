<template>
  <div id="app">
    <div class="msg">{{ gameMsg }}</div>
    <game-board @nextTurn="nextTurn" @gameEnd="gameEnd" />
  </div>
</template>

<script>
import GameBoard from "@/components/GameBoard";
export default {
  name: "app",
  components: {
    GameBoard
  },
  data() {
    return {
      gameMsg: ""
    };
  },
  methods: {
    nextTurn(player) {
      this.gameMsg = `${player}の番です`;
    },
    gameEnd(winner) {
      this.gameMsg = winner ? `${winner}の勝ちです` : "引き分けです";
    }
  }
};
</script>

<style lang="scss">
// 略
</style>
