<template>
  <div id="app">
      <button @click="change()">高度不固定</button>
      <!-- 只需要显示可视区 -->
      <!-- 1. size 列表每一项的高度，需要算出一个滚动条  -->
      <!-- 2. remain 一屏幕展示几个  -->
      <!-- 3. variable 表示每个item高度不确定-->
      <VitualList :size="size" :remain="5" :items="items" :variable="variable" v-if="show">
        <Item slot-scope="{item}" :item="item" :height="height"></Item>
      </VitualList>
  </div>
</template>

<script>
import Mock from 'mockjs'
import VitualList from './components/vitual-list';
import Item from './components/item'

let items = [];
for (let i = 0; i < 10000; i++) {
  items.push({ id: i, value: Mock.Random.sentence(5) })
}
export default {
    components: {
      VitualList,
      Item
    },
    data() {
      return {
        items,
        size: 40,
        variable: false,
        show: true,
        height: '40px'
      }
    },
    methods: {
      change() {
        let items = [];
        for (let i = 0; i < 10000; i++) {
          items.push({ id: i, value: Mock.Random.sentence(10, 50) })
        }
        this.items = items
        this.size = 100
        this.variable = true
        this.show = false;
        this.height = ''
        setTimeout(() => {
          this.show = true;
        })
      }
    }
}
</script>

<style lang="less">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
