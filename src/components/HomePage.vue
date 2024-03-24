<template>
  <div class="container">
    <div class="box">
      <div class="list" v-for="item in urlList" :key="item" @click="redirectPage(item)">
        <div class="imgBox">
          <img class="imgItem" :src="item+'/favicon.ico'" alt="">
        </div>
        <div class="content">
          <h4>{{ item }}</h4>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import {ref, onMounted} from "vue";
const queryParams = new URLSearchParams(window.location.search);
const urlList = ref([])
onMounted(() => {
  dnsResolve(queryParams.get('name'),queryParams.get('type'))
})
const dnsResolve = (name,type) => {
  axios({
    method: 'GET',
    url: 'https://dns.alidns.com/resolve' + '?name=' + name + '&type=' + type,
  }).then(response => {
    switch (type) {
      case '16':
        txtDecode(response)
        break;
      case '28':
        ip4pDecode(response)
        break;
      case '33':
        srvDecode(response)
        break;
    }
  })
}
const txtDecode = (response) => {
  for (let ans of response.data.Answer) {
    const name = ans.name.replace(/\.$/, '')
    const port = ans.data.replace(/[^0-9]/ig, '')
    urlList.value.push('https://' + name + ':' + port)
  }
}
const ip4pDecode = (response) => {
  for (let ans of response.data.Answer) {
    const parts = ans.data.split(':');
    const ipHi = parseInt(parts[3], 16);
    const ipLo = parseInt(parts[4], 16);
    const ipv4 = `${(ipHi >> 8)}.${ipHi & 0xFF}.${(ipLo >> 8)}.${ipLo & 0xFF}`;
    const port = parseInt(parts[2], 16);
    urlList.value.push('https://' + ipv4 + ':' + port)
  }
}
const srvDecode = (response) => {
  for (let ans of response.data.Answer) {
    const parts = ans.data.split(' ');
    const server = parts[3].replace(/\.$/, '')
    const port = parts[2]
    urlList.value.push('https://' + server + ':' + port)
  }
}
const redirectPage = (url) => {
  window.location.href = url
}
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #F7F7F7;
}

.box {
  position: relative;
  min-width: 350px;
  width: 50vw;
}

.list {
  position: relative;
  display: flex;
  padding: 15px;
  border-radius: 10px;
  margin: 10px 0;
  cursor: pointer;
  transition: 0.5s;
  overflow: hidden;
  background-color: rgba(255, 255, 255, 0.5);
}

.content {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.box:hover .list {
  filter: blur(5px);
}

.box .list:hover {
  transform: scale(1.1);
  filter: blur(0px);
}

.imgBox {
  position: relative;
  width: 25px;
  height: 25px;
  border-radius: 5px;
  overflow: hidden;
  margin-right: 10px;
}

.imgItem {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
