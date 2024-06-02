<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const lotteryData = ref(null)
const lotteryStatus = ref({})
const showOverlay = ref(false)
const showOverlay2 = ref(false)
const winnerData = ref(null)
let apiData = null  // 新增這行來儲存 API 的回應

onMounted(async () => {
  const response = await axios.get('https://script.google.com/macros/s/AKfycbwykrN6fMBUeNSVVA6AAzicd21nk3_WWRwLIyBQn8o3IwdUp3ZaojG4vvIeKyfODn2g2Q/exec?act=api')
  lotteryData.value = response.data
  apiData = response.data  // 將 API 的回應儲存到 apiData 變數中
  console.log(apiData)
  for (let prize in response.data) {
    lotteryStatus.value[prize] = '未抽獎'
  }
})

const startLottery = async (prize) => {
  lotteryStatus.value[prize] = '抽獎中'
  const winner = apiData[prize]  // 使用 apiData 變數，而不是再次呼叫 API

  // 使用 setTimeout 函數來等待五秒
  setTimeout(() => {
    console.log(`恭喜 ${winner}，你贏得了 ${prize}！`)
    lotteryStatus.value[prize] = '已抽獎'
    winnerData.value = { winner, prize }
    showOverlay.value = true
  }, 5000)
}

const closeOverlay = () => {
  showOverlay.value = false
}

const showWinners = () => {
  showOverlay2.value = true
}

const closeOverlay2 = () => {
  showOverlay2.value = false
}
</script>

<template>
  <h1>國立臺北大學第 112 學年度畢業典禮抽獎</h1>

  <div v-if="lotteryData">
    <div v-for="(values, prize) in lotteryData" :key="prize" style="margin-bottom: 20px;">
      <button
        :class="{ 'not-started': lotteryStatus[prize] === '未抽獎', 'in-progress': lotteryStatus[prize] === '抽獎中', 'finished': lotteryStatus[prize] === '已抽獎' }"
        :style="{ 'background-color': lotteryStatus[prize] === '未抽獎' ? 'red' : 'green' }"
        :disabled="lotteryStatus[prize] !== '未抽獎'" @click="startLottery(prize)">
        {{ prize }} 抽獎
      </button>
      <span class="status" v-if="lotteryStatus[prize] === '抽獎中'"> 抽獎中...</span>
    </div>
    <button @click="showWinners">顯示中獎名單</button>
  </div>

  <div class="overlay" v-if="showOverlay">
    <div class="confetti"></div>
    <div class="winner-info">
      <h2>恭喜以下的同學獲得「{{ winnerData.prize }}」</h2>
      <div class="winner-grid">
        <div v-for="(winner, index) in winnerData.winner" :key="index" style="font-size: large;">
          {{ winner }}
        </div>
      </div>
      <button @click="closeOverlay" style="margin-top: 20px;">關閉</button>
    </div>
  </div>

  <div class="overlay2" v-if="showOverlay2">
    <div class="confetti"></div>
    <div class="winner-info">
      <h2>恭喜以下的同學獲得獎項</h2>
      <div>
        <div v-for="(winners, prize) in apiData" :key="prize">
          <h3>{{ prize }}</h3>
          <div class="winner-grid2">
            <div v-for="(winner, index) in winners" :key="index" style="font-size: large;">
              {{ winner }}
            </div>
          </div>
        </div>
      </div>
      <button @click="closeOverlay2" style="margin-top: 20px;">關閉</button>
    </div>
    <div class="side-image">
      <h2>查看中獎名單</h2>
      <img src="../../public/qr-code.png" alt="Description" style="width: 100%;">
    </div>
  </div>
</template>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.confetti {
  /* 在這裡添加你的彩花動畫樣式 */
}

.winner-info {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  text-align: center;
  color: black;
}

.winner-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10px;
  overflow-y: auto;
  max-height: 500px;
}

.winner-grid2 {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10px;
  overflow-y: auto;
  max-height: 230px;
}

.winner-grid-group {
  max-height: 500px;
}

.winner-list {
  display: flex;
  flex-wrap: wrap;
}

.overlay2 {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.scrollable {
  overflow-y: auto;
  height: calc(100% - 60px);
}

.side-image {
  position: absolute;
  right: 30px;
  top: 50%;
  transform: translateY(-50%);
  width: 15%;
  height: auto;
  background-color: rgba(255, 255, 255, 0.5);
  padding: 20px;
  box-sizing: border-box;
  border-radius: 10px;
}
</style>