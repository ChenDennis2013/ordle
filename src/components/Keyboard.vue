<script setup lang="ts">
const emit = defineEmits<{
  handler: [key: string]
}>()
</script>

<template>
  <div id="keyboard">
    <button @click="emit('handler', 'Enter')"><svg class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" style="height: 30px; width: 30px;"><path d="M793.3 191.8h-185c-17.7 0-32 14.3-32 32s14.3 32 32 32h184.9c4.6 0 8.4 3.8 8.4 8.4v303.1c0 4.6-3.8 8.4-8.4 8.4h-507L458 403.9c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L213.2 558.3c-13.1 13.1-20.3 30.5-20.3 49s7.2 35.9 20.3 49l201.4 201.4c6.2 6.2 14.4 9.4 22.6 9.4 8.2 0 16.4-3.1 22.6-9.4 12.5-12.5 12.5-32.8 0-45.3L287.2 639.7h506.1c39.9 0 72.4-32.5 72.4-72.4V264.2c0-39.9-32.5-72.4-72.4-72.4z"></path></svg></button>
    <button @click="emit('handler', '0')"><p>0</p></button>
    <button @click="emit('handler', '1')"><p>1</p></button>
    <button @click="emit('handler', 'Backspace')"><svg class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" style="height: 30px; width: 30px;"><path d="M896 128H341.333333l-341.333333 384 341.333333 384h554.666667a128 128 0 0 0 128-128V256a128 128 0 0 0-128-128z m42.666667 640a42.666667 42.666667 0 0 1-42.666667 42.666667H379.733333l-265.472-298.666667L379.733333 213.333333H896a42.666667 42.666667 0 0 1 42.666667 42.666667v512z m-140.501334-158.165333a42.666667 42.666667 0 1 1-60.330666 60.330666L640 572.330667l-97.834667 97.834666a42.666667 42.666667 0 0 1-60.330666-60.330666L579.669333 512l-97.834666-97.834667a42.666667 42.666667 0 0 1 60.330666-60.330666L640 451.669333l97.834667-97.834666a42.666667 42.666667 0 0 1 60.330666 60.330666L700.330667 512z"></path></svg></button>
  </div>
</template>

<style scoped>
button
{
    background-color: #ddd;
    border-width: 0;
    border-radius: 3px;
    width: 60px;
    height: 80px;
    font-weight: bold;
    margin: 0 10px 0 0;
    font-size: 30px;
}

button > p
{
    margin: 0;
}

#keyboard
{
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 10px;
}
</style>
