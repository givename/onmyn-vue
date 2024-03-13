<template>
  <div>
    <h1>PlayerId: {{ playerId }}</h1>
    <h1>GameId: {{ gameId }}</h1>
    <GameGrid v-if="gameMapCells" :gameId="gameId" :playerId="playerId" :width="gameMapCells.width"
      :height="gameMapCells.height" :cellwords="gameMapCells.cellwords" />
  </div>
</template>

<script>
/// TODO: добавить отображеине статуса игры (создание / ожидание второго игрока / информация что игра завершена)
/// TODO: добавить отображения текущего игрока, хода и очков
/// TODO: блокировать действия click, если ход не на нашей стороне или игра завершена
/// TODO: подсвечивать пустые ячейки, а то не ясно, что они прокликаны

import io from 'socket.io-client';
import { v4 as uuidv4 } from 'uuid';

import GameGrid from './components/GameGrid.vue';

export default {
  components: {
    GameGrid
  },
  data() {
    return {
      socket: null,
      gameMapCells: null,
      playerId: uuidv4(),
      gameId: '',
    }
  },
  created() {
    this.fetchGameId().then(() => this.connectSocketByGameId())
  },
  methods: {
    async fetchGameId() {
      try {
        /// TODO: вынести в отдельный файл
        const response = await fetch('http://localhost:3000/game/enter', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ playerId: this.playerId })
        });
        const data = await response.json();
        this.gameId = data.gameId;
        console.log('GameId fetched successfully');
      } catch (error) {
        console.error('Error fetching GameId:', error);
      }
    },
    async connectSocketByGameId() {
      /// TODO: вынести в отдельный файл

      this.socket = io('http://localhost:3000');
      this.socket.on('connect', () => {
        this.socket.emit('enter', {
          gameId: this.gameId,
        })
      });

      this.socket.on('game', ({ gameSessionMemory }) => {
        console.log('Game data:', gameSessionMemory);

        if (gameSessionMemory.status !== 'loop') {
          return;
        }

        /// TODO: refactoring

        const cellwords = {};

        for (const [positionKey, cellState] of Object.entries(gameSessionMemory.map.cells)) {
          const value = cellState.entity;

          /// TODO: этого не должно быть, это фильтрует бэк
          if (cellState.status === 'hidden') {
            continue;
          }

          if (typeof value !== 'number' && value !== 'diamond') {
            continue;
          }

          if (value === 'diamond') {
            cellwords[positionKey] = '💎';
          } else {
            cellwords[positionKey] = value + '';
          }
        }

        this.gameMapCells = {
          width: gameSessionMemory.map.width,
          height: gameSessionMemory.map.height,
          cellwords,
        }

      });
    }
  }
}
</script>