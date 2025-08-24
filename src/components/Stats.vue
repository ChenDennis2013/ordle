<script setup lang="ts">
import { ElProgress } from 'element-plus';
import { useCookies } from 'vue3-cookies';

const { cookies } = useCookies();
const tot = +cookies.get("tot") || 0;
const won = Array.from({ length: 9 }, (_, i) => +cookies.get(`won${i}`) || 0);
const wonsum = won.reduce((a, b) => a + b, 0);
const streak = +cookies.get("streak") || 0;
const maxstreak = +cookies.get("maxstreak") || 0;

function go(x : number)
{
    return isNaN(x) ? "--" : x + "%";
}

function gop(x : number)
{
    return isNaN(x) ? 0 : x;
}
</script>

<template>
<div class="total">
    <div class="total-text">一共局数<br>{{ tot }}</div>
    <div class="total-text">获胜局数<br>{{ wonsum }}</div>
    <div class="total-text">胜率<br>{{ go(Math.round(wonsum / tot * 100)) }}</div>
    <div class="total-text">当前连胜<br>{{ streak }}</div>
    <div class="total-text">最高连胜<br>{{ maxstreak }}</div>
</div>
<div class="progress">
    尝试次数分布
    <div class="progress-text" v-for="i in 8">
        <span style="display: inline-block;">{{ i }}次</span>
        <el-progress class="progressbar" :percentage="gop(Math.round(won[i] / wonsum * 100))" :stroke-width="20" :text-inside="true" status="success" />
        <span style="display: inline-block;">{{ won[i] }}</span>
    </div>
</div>
</template>

<style scoped>
.total
{
    display: flex;
    justify-content: space-between;
}

.total-text
{
    display: inline-block;
    text-align: center;
}

.progress
{
    text-align: center;
    width: 100%;
}

.progressbar
{
    width: 93%;
    display: inline-block;
}

.progress-text
{
    display: flex;
    justify-content: space-between;
    width: 100%;
    margin-top: 10px;
}
</style>