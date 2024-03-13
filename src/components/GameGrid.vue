<template>
  <div>
    <div v-for="y in height" :key="y" class="row">
      <div v-for="x in width" :key="x" class="cell" @click="sendRequest(x - 1, y - 1)">
        {{ getCellContent(x - 1, y - 1) }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
    cellwords: {
      type: Object,
      required: true,
    },
    gameId: {
      type: String,
      required: false,
    },
    playerId: {
      type: String,
      required: false,
    }
  },
  methods: {
    getCellContent(x, y) {
      const key = `${x}.${y}`;
      return this.cellwords[key] || "";
    },

    async sendRequest(x, y) {
      /// TODO: вынести в api файл
      await fetch('http://localhost:3000/game/click', {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          gameId: this.gameId,
          playerId: this.playerId,
          x,
          y
        })
      });

    },
  },
};
</script>

<style scoped>
.row {
  display: flex;
}

.cell {
  width: 20px;
  height: 20px;
  border: 1px solid #ccc;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
