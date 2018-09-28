<template>
<div>
  <div class="x-box" :style="{top: top + 'px'}">
    <x-scrollbar ref="x">
      <ul>
        <li v-for="item in count" :key="item" :style="{height: height + 'px'}">{{item}}</li>
      </ul>
    </x-scrollbar>
  </div>
</div>
</template>
<script>
import xScrollbar from './xScrollbar.vue'
export default {
  data() {
    return {
      count: 40,
      top: 100,
      height: 20
    }
  },
  mounted() {
    let tid = setInterval((  )=>{
      // this.count -= 5
      this.top += 10
    }, 1000)
    setTimeout(() => {
      clearInterval(tid)
      // this.$refs.x.handleResize()
    }, 5000)
    setInterval((  )=>{
      // this.height += 2
    }, 1000)
  },
  methods: {
    xScroll() {
      console.log( 111 )
    }
  },
  components: {
    'x-scrollbar': xScrollbar
  }
}
</script>
<style lang="scss">
.x-box {
  border: 1px solid #f00;
  position: absolute;
  top: 100px;
  bottom: 200px;
  left: 40px;
  width: 300px;
}
</style>