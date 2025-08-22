<script setup lang="ts">
import { ref } from 'vue';
import CountUp from './animations/CountUp/CountUp.vue';
import { Select } from '@element-plus/icons-vue';
import Keyboard from './components/Keyboard.vue';
import { ElMessage } from 'element-plus';

const howToPlayVisible = ref(false);
const board = ref(Array.from({ length: 8 }, () => Array.from({ length: 8 }, () => -1)));
const cnt = ref(Array.from({ length: 8 }, () => 0));
const gocnt = ref(Array.from({ length: 8 }, () => false));
const cntbold = ref(Array.from({ length: 8 }, () => false));
const shake = ref(Array.from({ length: 8 }, () => Array.from({ length: 8 }, () => false)));
const winned = ref(false);
const overed = ref(false);
function rander(seed : number)
{
    let res = seed % 233280;
    return () => {
        res = (res * 9301 + 49297) % 233280;
        return res / 233280;
    }
}
const rnd = rander(Date.now());
let ans = Math.floor(rnd() * 0b100000000);
let nowx = 0, nowy = 0;

function init()
{
    overed.value = false;
    ans = Math.floor(rnd() * 0b100000000);
    nowx = nowy = 0;
    for (let i = 0; i < 8; ++ i)
    {
          cnt.value[i] = 0;
          gocnt.value[i] = false;
          cntbold.value[i] = false;
          for (let j = 0; j < 8; ++ j)
          {
              board.value[i][j] = -1;
              shake.value[i][j] = false;
          }
    }
}

function check()
{
    if (overed.value) return init();
    if (nowy !== 8) { ElMessage.error("请填完一行再按下回车！"); return; }
    let res = 0;
    for (let i = 0; i < 8; ++ i)
        res += +((board.value[nowx][i] | (nowx ? board.value[nowx - 1][i] : 0)) == (ans >> (7 - i) & 1));
    cnt.value[nowx] = res;
    gocnt.value[nowx] = true;
}

function gamewin()
{
    winned.value = true;
    overed.value = true;
}

function gameover()
{
    winned.value = false;
    overed.value = true;
}

function processaftercheck()
{
    cntbold.value[nowx] = true;
    ++ nowx; nowy = 0;
    if (cnt.value[nowx - 1] === 8) return gamewin();
    if (nowx === 8) return gameover();
}

function del()
{
    if (nowy === 0) { ElMessage.error("本行已清空！"); return; }
    -- nowy;
    board.value[nowx][nowy] = -1;
    shake.value[nowx][nowy] = false;
    shake.value[nowx][nowy] = true;
}

function typeNum(num : number)
{
    if (nowy === 8) { ElMessage.error("本行已满！"); return; }
    board.value[nowx][nowy] = num;
    shake.value[nowx][nowy] = false;
    shake.value[nowx][nowy] = true;
    ++ nowy;
}

function type(key : string)
{
    if (key === "Enter") check();
    if (key === "Backspace") del();
    if (key === "0" || key === "1") typeNum(+key);
}
window.addEventListener('keyup', (e) => type(e.key));
</script>

<template>
  <header style="text-align: center;">
    <h1 style="display: inline-block; padding-left: 120px; margin-bottom: 10px;">ORdle</h1>
    <a href="#" @click="howToPlayVisible = true;"
      style="float: right; margin-top: 30px; margin-right: 50px; font-size: 22px;">怎么玩？</a>
  </header>
  <hr>
  <div id="game">
    <div id="board">
      <div v-for="i in 8" class="row">
        <div v-for="j in 8">
          <div :class="['cell', board[i - 1][j - 1] === -1 ? 'empty' : 'filled', shake[i - 1][j - 1] && 'shake']"><p v-if="board[i - 1][j - 1] !== -1" class="cell-text">{{ board[i - 1][j - 1] }}</p></div>
          <div v-if="j === 4" class="spacer"></div>
        </div>
        <div class="spacer"></div>
        <div class="counter">
          <CountUp :to="cnt[i - 1]" :start-when="gocnt[i - 1]" :class="[cntbold[i - 1] && 'bold']" @end="processaftercheck"></CountUp>
          <Select style="width: 1em; height: 1em;" />
        </div>
      </div>
    </div>
    <Keyboard @handler="type" />
  </div>
  <footer style="text-align: center; margin-top: 30px;">
    <a href="https://github.com/ChenDennis2013/ordle" target="_blank">Source</a>
    欢迎open issue 或 PR
  </footer>
  <el-dialog v-model="howToPlayVisible" title="怎么玩？">
    <p>首先，系统会随机生成一个 8 位二进制数，接下来，你需要在 8 次猜测中猜出这个数字。</p>
    <p>每一次猜测，你需要给出一个 8 位二进制数，系统会告诉你你的本次猜测与上次猜测（一开始为 0000 0000）的按位或结果与答案有几位数字相同，但不会告诉你哪些数字是相同的。</p>
    <p>当系统告诉你有 8 位数字相同时，游戏胜利。</p>
    <p>当你 10 次都没猜对，游戏失败，系统会告诉你答案。</p>
    <p>在结束游戏后，你可以重新开始游戏<!--或者点击“分享”按钮分享你的成绩-->。</p>
    <p>祝你好运！</p>
  </el-dialog>
  <el-dialog v-model="overed" :title="winned ? '游戏胜利！' : '游戏结束！'">
    <p v-if="winned">恭喜你，你成功破解了答案！</p>
    <p v-else>很遗憾，你没有破解答案。</p>
    <p>答案是：{{ ans.toString(2).padStart(8, "0").substring(0, 4) + " " + ans.toString(2).padStart(8, "0").substring(4, 8) }}</p>
    <div style="text-align: right;">
      <el-button type="primary" @click="init();">再来一局</el-button>
      <!-- <el-button type="success" class="share">分享</el-button> -->
    </div>
  </el-dialog>
</template>

<style scoped>
.cell
{
  display: inline-block;
  width: 50px;
  height: 50px;
  line-height: 50px;
  text-align: center;
  font-size: 24px;
  color: #000;
  background-color: #fff;
  border-color: #ccc;
  border-style: solid;
  border-width: 3px;
  border-radius: 5px;
  margin-right: 10px;
  margin-bottom: 5px;
}

.cell-text
{
  margin: 0;
  font-size: 40px;
  font-weight: bold;
}

.counter
{
  display: inline-block;
  font-size: 30px;
}

.bold
{
  font-weight: bold;
}

.spacer
{
  display: inline-block;
  width: 10px;
}

.row
{
  display: flex;
  justify-content: center;
}

@keyframes shake {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

.shake
{
  animation: shake 0.2s linear 1;
}

.filled
{
  border-color: #000;
}

@media screen and (max-width: 480px)
{
    .cell
    {
        width: 30px;
        height: 30px;
        margin-right: 5px;
        margin-bottom: 2px;
    }
   .cell-text
    {
        font-size: 20px;
        line-height: 24px;
    }
   .counter
    {
        font-size: 20px;
    }
}
</style>
