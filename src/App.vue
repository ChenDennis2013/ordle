<script setup lang="ts">
import { ref } from 'vue';
import CountUp from './animations/CountUp/CountUp.vue';
import { Select } from '@element-plus/icons-vue';
import Keyboard from './components/Keyboard.vue';
import { ElMessage } from 'element-plus';
import ClipboardJS from 'clipboard';

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
const dict = "1qQaAzZ2wWsSxX3+eEdDcC4rRfFvV5tTgGbB6yYhHnN7/uUjJmM8iIkK9oOlL0pP=".split("");
const idx = "1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ+/=".split("");
function encrypt(n : number)
{
    let t = btoa((n * 998244353 + 123456789).toString(36)), res = "";
    t.split("").forEach((c) => {res += dict[idx.indexOf(c)]});
    return res;
}
function decrypt(s : string)
{
    let t = "";
    s.split("").forEach((c) => {t += idx[dict.indexOf(c)]});
    try
    {
        let n = (parseInt(atob(t), 36) - 123456789) / 998244353;
        if (n % 1 !== 0 || n < 0 || n >= 0b100000000) throw "Invalid level";
        return n;
    }
    catch (e)
    {
        ElMessage.error("自定义关卡错误，已更改为随机关卡！");
        return Math.floor(rnd() * 0b100000000);
    }
}
const rnd = rander(Date.now());
const params = new URLSearchParams(window.location.search);
let ans = params.get("level") == null ? Math.floor(rnd() * 0b100000000) : decrypt(params.get("level") as string);
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
    shake.value[nowx][nowy] = true;
    shake.value[nowx][nowy] = false;
}

function typeNum(num : number)
{
    if (nowy === 8) { ElMessage.error("本行已满！"); return; }
    board.value[nowx][nowy] = num;
    shake.value[nowx][nowy] = true;
    shake.value[nowx][nowy] = false;
    ++ nowy;
}

function type(key : string)
{
    if (key === "Enter") check();
    if (key === "Backspace") del();
    if (key === "0" || key === "1") typeNum(+key);
}
window.addEventListener('keyup', (e) => type(e.key));

function getshareinfo()
{
    let res = "";
    if (winned.value) res += `我在 ORdle 里用了 ${nowx} 次猜测，成功破解了答案！\n你能比我更快吗？\n`;
    else res += `我在 ORdle 里没有破解答案。\n你也来试试吧！\n`;
    res += window.location.href.split("?")[0].split("#")[0] + "?level=" + encrypt(ans);
    return res;
}

let clipboard = new ClipboardJS('#share', {text: getshareinfo});
clipboard.on('success', function(e) {
    ElMessage.success('已将分享信息复制到剪贴板，您可以粘贴到任何地方！');
});
clipboard.on('error', function(e) {
    ElMessage.error('复制失败！');
});
</script>

<template>
  <header style="text-align: center;">
    <h1 class="title">ORdle</h1>
    <a href="#" @click="howToPlayVisible = true;" class="how-to-play">怎么玩？</a>
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
          <CountUp :to="cnt[i - 1]" :duration="1" :start-when="gocnt[i - 1]" :class="[cntbold[i - 1] && 'bold']" @end="processaftercheck"></CountUp>
          <Select style="width: 1em; height: 1em;" />
        </div>
      </div>
    </div>
    <Keyboard @handler="type" />
  </div>
  <footer style="text-align: center; margin-top: 30px;">
    <a href="https://github.com/ChenDennis2013/ordle" target="_blank">Source</a>
    欢迎open issue 或 PR
    好玩的话给个 star 吧
  </footer>
  <el-dialog v-model="howToPlayVisible" title="怎么玩？">
    <p>首先，系统会随机生成一个 8 位二进制数，接下来，你需要在 8 次猜测中猜出这个数字。</p>
    <p>每一次猜测，你需要给出一个 8 位二进制数，系统会告诉你你的本次猜测与上次猜测（一开始为 0000 0000）的按位或结果与答案有几位数字相同，但不会告诉你哪些数字是相同的。</p>
    <p>当系统告诉你有 8 位数字相同时，游戏胜利。</p>
    <p>当你 8 次都没猜对，游戏失败，系统会告诉你答案。</p>
    <p>在结束游戏后，你可以重新开始游戏<!--或者点击“分享”按钮分享你的成绩-->。</p>
    <p>祝你好运！</p>
  </el-dialog>
  <el-dialog v-model="overed" :title="winned ? '游戏胜利！' : '游戏结束！'">
    <p v-if="winned">恭喜你，你使用了{{ nowx }}次猜测，成功破解了答案！</p>
    <p v-else>很遗憾，你没有破解答案。</p>
    <p>答案是：{{ ans.toString(2).padStart(8, "0").substring(0, 4) + " " + ans.toString(2).padStart(8, "0").substring(4, 8) }}</p>
    <div style="text-align: right;">
      <el-button type="primary" @click="init();">再来一局</el-button>
      <el-button type="success" id="share">分享</el-button>
    </div>
  </el-dialog>
</template>

<style scoped>
.title
{
  display: inline-block;
  padding-left: 120px;
  margin-bottom: 10px;
}

.how-to-play
{
  float: right;
  margin-top: 30px;
  margin-right: 50px;
  font-size: 22px;
}

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
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

/* .shake
{
  animation: shake 0.2s linear;
} */

.filled
{
  border-color: #000;
  animation: shake 0.2s linear;
}

@media screen and (max-width: 480px)
{
    .how-to-play
    {
        font-size: 20px;
        margin-right: 20px;
    }
    .title
    {
        padding-left: 80px;
    }
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
