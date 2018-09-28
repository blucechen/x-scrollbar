<template>
<div class="x-scrollbar" :class="{showThumb: isShowThumb}">
  <div class="xs-wrapper" ref="xs-wrapper" @scroll="scroll">
    <div class="xs-w-viewer" ref="xs-w-viewer">
      <slot></slot>
    </div>
  </div>
  <div class="xs-vertical-bar" :style="{
      width: scrollBarH === 0?'0px': '6px', 
      padding: addIndicator? '12px 0': '0'
    }">
    <Indicator v-if="addIndicator" direction="up" @press="toUp" class="indicator layoutTop"></Indicator>
    <div 
      class="xs-vb-thumb"
      :style="{height: `${scrollBarH}%`, transform: `translateY(${translateY}%)`}"
      @mousedown="thumbMouseDown"
      >
    </div>
    <Indicator v-if="addIndicator" direction="down" @press="toDown" class="indicator layoutBottom"></Indicator>
  </div>
  <div class="xs-horizontal-bar"><div></div></div>
</div>
</template>
<script>
import { debounce } from "lodash"
import Indicator from './xIndicator.vue'
export default {
  name: 'xScrollbar',
  data() {
    return {
      scrollBarH: 0,      // 相对容器的%高度
      translateY: 0,      //滑动自身的%高度
      isShowThumb: false,
      speed: 2
    }
  },
  props: {
    autoUpdata: {// 是否关闭自动更新进度条，如果外层容器高度不变化，建议关闭
      type: Boolean,
      default: true
    },
    addIndicator: {// 是否添加指示器
      type: Boolean,
      default: false
    }
  },
  methods: {
    /**
     * 当外层容器高度发生变化时 建议：手动调用更新滚动条的高度(否则只能在滚动的时候才能修改滚动条位置和高度)
     */
    update() {
      this.handleResize()
    },
    toUp() {
      this.moveScrollBar(-speed)
    },
    toDown() {
      this.moveScrollBar(this.speed)
    },
    scroll() {
      this.handleScrollBarPos()
      if (this.autoUpdata) {
        this.debounceHandleScrollBarH() // 无法自动检测容器高度，只能在滚动的时候更新（或者调用update）
      }
    },
    debounceHandleScrollBarH: debounce(function(){this.handleScrollBarH()}, 500, {leading: true, trailing: false }),//头和尾，低频调用
    thumbMouseDown(e) {
      document.addEventListener('mousemove', this.handleMouseMove)
      document.addEventListener('mouseup', this.handleMouseUp)
      this.startY = e.clientY
      this.isShowThumb = true
    },
    moveScrollBar(movePx) {// 同时联动内容
      if (movePx > 0 && this.translateY === this.maxTranslateY)// prevent not use call (move down)
        return
      if (movePx < 0 && this.translateY === 0) // prevent not use call( move up)
        return

      let percentageH = (movePx / this.scrollBarPxH) * 100
      let tempTransY = this.translateY + percentageH
      this.translateY = tempTransY > this.maxTranslateY? this.maxTranslateY: tempTransY < 0? 0: tempTransY  //limit max
      
      this.handleMoveViewer()
    },
    handleMouseMove(e) {
      this.removeSelect() // TODO
      let movePx = e.clientY - this.startY
      this.moveScrollBar(movePx)
      this.startY = e.clientY
    },
    handleMoveViewer() {
      let percentage = this.translateY / this.maxTranslateY
      let scrollTop = this.maxScrollPxH * percentage
      this['xs-wrapper'].scrollTop = scrollTop
    },
    handleMouseUp() {
      document.removeEventListener('mousemove', this.handleMouseMove)
      document.removeEventListener('mouseup', this.handleMouseUp)
      this.isShowThumb = false
    },
    handleScrollBarH() {
      let domWrapper = this['xs-wrapper'], domViewer = this['xs-w-viewer']
      let domWrapperH = domWrapper.offsetHeight, domViewerH = domViewer.offsetHeight
      if (domWrapperH >= domViewerH) {
        this.scrollBarH = 0
        this.translateY = 0
        return
      }

      let percentage = ~~((domWrapperH / domViewerH)*100)
      this.scrollBarH = `${percentage < 3? 3: percentage}`                  // %高度
      this.scrollBarPxH = domWrapperH * this.scrollBarH / 100               // px高度
      this.canScrollPxH = domWrapperH * (100 - this.scrollBarH) / 100       // 可滚动px高度

      this.maxTranslateY = (this.canScrollPxH / this.scrollBarPxH) * 100    // 滚动条可滑动的最大距离，以自身为基准 %
      this.maxScrollPxH = domViewerH - domWrapperH                          // 可滚动的最大高度px
    },
    handleScrollBarPos() {
      let domWrapper = this['xs-wrapper'], domViewer = this['xs-w-viewer']
      let offsetH = domWrapper.offsetHeight
      let percentage = domWrapper.scrollTop / this.maxScrollPxH

      // 采用transform 调用3d加速
      let actScrollH = this.canScrollPxH * percentage
      let res = actScrollH / this.scrollBarPxH

      this.translateY = res * 100
    },
    handleResize() {
      this.handleScrollBarH()
      this.handleScrollBarPos()
    },
    removeListener() {
      this.observer.disconnect()
      window.removeEventListener('resize', this.handleResize)
    },
    autoResizeScrollbar() {
      const observer = new MutationObserver(this.handleResize)
      observer.observe(this['xs-w-viewer'], {
        childList: true,// watch children mutation
        subtree: true,  // watch all sub children mutation
        attributes: true // watch attributes mutation
      })
      this.observer = observer
      window.addEventListener('resize', this.handleResize)
    },
    init() {
      this['xs-w-viewer'] = this.$refs['xs-w-viewer']
      this['xs-wrapper'] = this.$refs['xs-wrapper']
      this.handleResize()
      let removeSelect = 'getSelection' in window? function(){window.getSelection().removeAllRanges()}: function(){document.selection.empty()}
      this.removeSelect = removeSelect
    }
  },
  mounted() {
    this.init()
    this.autoResizeScrollbar()
  },
  beforeDestroy() {
    this.removeListener()
  },
  components: {
    'Indicator': Indicator
  }
}

</script>
<style lang="scss">
@mixin setWH($w, $h) {
  width: $w;
  height: $h;
}
$bgColor: rgba(24,113,148,.7);
.x-scrollbar {
  @include setWH(100%, 100%);
  position: relative;
  &.showThumb, &:hover {
    .xs-vb-thumb {
      background-color: $bgColor
    }
    .indicator {
      i {
        &:before, &:after {
          border-top-color: $bgColor;
          border-bottom-color: $bgColor;
        }
      }
    }
  }
}
.xs-wrapper {
  @include setWH(100%, 100%);
  overflow: scroll;
 
}
.xs-vertical-bar {
  position: absolute;
  right: 1px;
  top: 0;
  padding: 12px 0;
  box-sizing: border-box;
  overflow: hidden;
  height: 100%;
  width: 6px;
  .xs-vb-thumb {
    width: 100%;
    background-color: transparent;
    // background-color: $bgColor;
    transition: .3s background-color;
    border-radius: 6px;
    cursor: pointer;
  }

}
.indicator {
  position: absolute;
  right: 1px;
  width: 100%;
  &.layoutTop {
    top: 0px;
  }
  &.layoutBottom {
    bottom: 0px;
  }
}

</style>