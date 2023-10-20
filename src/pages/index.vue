<script setup lang="ts">
import { isDev, toggleDev } from '~/composables'
import { GamePlay } from '~/composables/logic'

const play = new GamePlay(5, 5, 3)

const now = $(useNow())
const timerMS = $computed(() => Math.round(((+play.state.value.endMS || +now) - +play.state.value.startMS) / 1000))

useStorage('vuesweeper-state', play.state)
const state = computed(() => play.board)

const mineRest = $computed(() => {
  if (!play.state.value.mineGenerated)
    return play.mines
  return play.blocks.reduce((a, b) => a + (b.mine ? 1 : 0) - (b.flagged ? 1 : 0), 0)
})
function newGame(difficulty: 'easy' | 'medium' | 'hard') {
  switch (difficulty) {
    case 'easy':
      play.reset(9, 9, 10)
      break
    case 'medium':
      play.reset(16, 16, 40)
      break
    case 'hard':
      play.reset(16, 30, 90)
      break
  }
}

watchEffect(() => {
  play.checkGameState()
})
</script>

<template>
  <div>
    MineWeeper
    <div
      flex="~ gap1"
      justify-center p4
    >
      <button btn @click="play.reset()">
        New Game
      </button>
      <button btn @click="newGame('easy')">
        Easy
      </button>
      <button btn @click="newGame('medium')">
        Medium
      </button>
      <button btn @click="newGame('hard')">
        Hard
      </button>
    </div>
    <div flex="~ gap-10" justify-center>
      <div flex="~ gap-1" text-2x1 items-center font-mono>
        <div i-mdi-clock />
        {{ timerMS }}
      </div>
      <div flex="~ gap-1" text-2x1 items-center font-mono>
        <div i-mdi-mine />
        {{ mineRest }}
      </div>
    </div>
    <div w-full overflow-hidden p5>
      <div
        v-for="(row, y) in state"
        :key="y"
        flex="~"

        ma w-max items-center justify-center
      >
        <MineBlock
          v-for="(block, x) in row"
          :key="x"
          :block="block"
          @click="play.onClick(block)"
          @dblclick="play.autoExpand(block)"
          @contextmenu.prevent="play.onRightClick(block)"
        />
      </div>
    </div>

    <div flex="~ gap-1 justify-center">
      <button btn @click="toggleDev()">
        {{ isDev ? 'DEV' : 'NORMAL' }}
      </button>
    </div>
    <Confetti :passed="play.state.value.status === 'won'" />
  </div>
</template>
