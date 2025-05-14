<script setup>
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue';
import { ElMessage } from 'element-plus';

const difficulty = ref('easy');
const level = ref(1);
const score = ref(0);
const isRunning = ref(false);
const showDialog = ref(false);
const basketX = ref(200);
const basketWidth = 80;
const gameAreaWidth = 500;
const gameAreaHeight = 500;
const fruits = reactive([]);
const fruitIcons = [
  'el-icon-apple',
  'el-icon-orange',
  'el-icon-watermelon',
  'el-icon-grape',
  'el-icon-cherry',
  'el-icon-pear',
  'el-icon-banana',
];
const fruitColors = [
  '#ff4d4f', '#ffa940', '#36cfc9', '#9254de', '#ff85c0', '#bae637', '#ffd666'
];
let fruitId = 0;
let gameTimer = null;
let fruitTimer = null;
let speed = 2;
let fruitInterval = 1200;

const gameArea = ref(null);

const difficultyMap = {
  easy: { speed: 2, interval: 1200 },
  medium: { speed: 4, interval: 900 },
  hard: { speed: 7, interval: 600 },
};

function startGame() {
  if (isRunning.value) return;
  isRunning.value = true;
  showDialog.value = false;
  score.value = 0;
  fruits.length = 0;
  fruitId = 0;
  runGameLoop();
  fruitTimer = setInterval(spawnFruit, fruitInterval);
}

function resetGame() {
  isRunning.value = false;
  showDialog.value = false;
  score.value = 0;
  level.value = 1;
  fruits.length = 0;
  fruitId = 0;
  setDifficulty();
  clearInterval(gameTimer);
  clearInterval(fruitTimer);
}

function setDifficulty() {
  const conf = difficultyMap[difficulty.value];
  speed = conf.speed;
  fruitInterval = conf.interval;
}

function runGameLoop() {
  gameTimer = setInterval(() => {
    for (let i = fruits.length - 1; i >= 0; i--) {
      fruits[i].y += speed;
      if (fruits[i].y > gameAreaHeight - 60) {
        // 判断是否接到
        if (
          fruits[i].x + 40 > basketX.value &&
          fruits[i].x < basketX.value + basketWidth
        ) {
          score.value += 10 * level.value;
          fruits.splice(i, 1);
        } else {
          // 没接到，游戏失败
          endGame(false);
          return;
        }
      }
    }
    // 关卡晋级
    if (score.value >= level.value * 100) {
      nextLevel();
    }
  }, 16);
}

function spawnFruit() {
  if (!isRunning.value) return;
  const x = Math.floor(Math.random() * (gameAreaWidth - 40));
  const icon = fruitIcons[Math.floor(Math.random() * fruitIcons.length)];
  const color = fruitColors[Math.floor(Math.random() * fruitColors.length)];
  fruits.push({ id: ++fruitId, x, y: 0, icon, color });
}

function nextLevel() {
  level.value++;
  if (level.value > 10) {
    endGame(true);
    return;
  }
  setDifficulty();
  ElMessage.success(`进入第${level.value}关，难度提升！`);
  fruits.length = 0;
  clearInterval(fruitTimer);
  fruitTimer = setInterval(spawnFruit, fruitInterval);
}

function endGame(success) {
  isRunning.value = false;
  showDialog.value = true;
  clearInterval(gameTimer);
  clearInterval(fruitTimer);
}

function handleKeydown(e) {
  if (!isRunning.value) return;
  if (e.key === 'ArrowLeft') {
    basketX.value = Math.max(0, basketX.value - 30);
  } else if (e.key === 'ArrowRight') {
    basketX.value = Math.min(gameAreaWidth - basketWidth, basketX.value + 30);
  }
}

onMounted(() => {
  setDifficulty();
  window.addEventListener('keydown', handleKeydown);
});

onBeforeUnmount(() => {
  clearInterval(gameTimer);
  clearInterval(fruitTimer);
  window.removeEventListener('keydown', handleKeydown);
});
</script>

<template>
  <el-container class="fruit-catch-game">
    <el-header height="80px" class="game-header">
      <div class="game-title">
        <el-icon><i class="el-icon-apple"></i></el-icon>
        <span>高端大气接水果游戏</span>
      </div>
      <el-select v-model="difficulty" placeholder="选择难度" class="difficulty-select" @change="resetGame">
        <el-option label="初级" value="easy" />
        <el-option label="中级" value="medium" />
        <el-option label="高级" value="hard" />
      </el-select>
      <el-tag class="level-tag" type="success">第 {{ level }} / 10 关</el-tag>
      <el-tag class="score-tag" type="warning">得分：{{ score }}</el-tag>
    </el-header>
    <el-main class="game-main">
      <div class="game-area" ref="gameArea">
        <transition-group name="fruit" tag="div">
          <div
            v-for="fruit in fruits"
            :key="fruit.id"
            class="fruit"
            :style="{ left: fruit.x + 'px', top: fruit.y + 'px', background: fruit.color }"
          >
            <el-icon><i :class="fruit.icon"></i></el-icon>
          </div>
        </transition-group>
        <div class="basket" :style="{ left: basketX + 'px' }">
          <el-icon><i class="el-icon-shopping-cart-full"></i></el-icon>
        </div>
      </div>
      <el-row class="game-controls" justify="center">
        <el-button type="primary" @click="startGame" :disabled="isRunning">开始游戏</el-button>
        <el-button type="danger" @click="resetGame">重置</el-button>
      </el-row>
      <el-dialog v-model="showDialog" title="游戏结束" width="400px" center>
        <div v-if="level > 10">
          <el-result icon="success" title="恭喜通关！" sub-title="您的总得分：{{ score }}" />
        </div>
        <div v-else>
          <el-result icon="error" title="挑战失败" sub-title="再接再厉！" />
        </div>
        <template #footer>
          <el-button type="primary" @click="resetGame">重新开始</el-button>
        </template>
      </el-dialog>
    </el-main>
    <el-footer class="game-footer">
      <span>© 2024 高端大气接水果游戏 | Powered by Vue3 + Element Plus</span>
    </el-footer>
  </el-container>
</template>

<style scoped>
.fruit-catch-game {
  min-height: 100vh;
  background: linear-gradient(135deg, #e0e7ff 0%, #fffbe6 100%);
  font-family: 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', 'Arial', sans-serif;
}
.game-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(255,255,255,0.95);
  box-shadow: 0 2px 8px #e6f7ff;
  padding: 0 40px;
  font-size: 22px;
}
.game-title {
  display: flex;
  align-items: center;
  font-weight: bold;
  font-size: 28px;
  color: #1677ff;
}
.difficulty-select {
  width: 120px;
}
.level-tag, .score-tag {
  margin-left: 20px;
  font-size: 18px;
}
.game-main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-height: 600px;
  padding: 30px 0 0 0;
}
.game-area {
  position: relative;
  width: 500px;
  height: 500px;
  background: linear-gradient(180deg, #fffbe6 60%, #e0e7ff 100%);
  border-radius: 24px;
  box-shadow: 0 8px 32px #b7eb8f44;
  overflow: hidden;
  margin-bottom: 30px;
}
.fruit {
  position: absolute;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  box-shadow: 0 2px 8px #ffd66699;
  transition: top 0.1s linear;
}
.basket {
  position: absolute;
  bottom: 0;
  width: 80px;
  height: 40px;
  background: #1677ff;
  border-radius: 0 0 40px 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  font-size: 36px;
  box-shadow: 0 2px 12px #1677ff44;
  left: 200px;
  transition: left 0.1s linear;
}
.game-controls {
  margin-top: 10px;
}
.game-footer {
  text-align: center;
  color: #888;
  font-size: 16px;
  padding: 16px 0;
  background: transparent;
}
</style>
