<template>
<div class="x-scrollbar" :class="{showThumb: isShowThumb}">
  <div class="xs-wrapper" ref="xs-wrapper" @scroll="scroll">
    <div class="xs-w-viewer" ref="xs-w-viewer">
      <slot></slot>
    </div>
  </div>
  <div class="xs-vertical-bar" :style="{
      width: scrollBarH === 0?'0px': `${barWidth}px`, 
      padding: addIndicator? `${indicatorH}px 0`: '0'
    }"
    @click="barClick"
    >
    <Indicator v-if="addIndicator" @click.native.stop direction="up" @press="toUp" class="indicator layoutTop"></Indicator>
    <div 
      class="xs-vb-thumb"
      :style="{height: `${scrollBarH}%`, transform: `translateY(${translateY}%)`}"
      @mousedown="thumbMouseDown"
      @click.stop
      >
    </div>
    <Indicator v-if="addIndicator" @click.native.stop direction="down" @press="toDown" class="indicator layoutBottom"></Indicator>
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
    }
  },
  props: {
    /**
     * 背景：外层容器高度变化，scrollbar无法检测，导致thumb位置和高度可能受到影响
     * 建议：
     * 1：外层容器高度不变： 设置autoUpdata为false；
     * 2. 外层容器高度变化
     *  2.1：设置autoUpdata为false，外层容器高度变化时候手动调用组件update方法
     *  2.2：设置autoUpdata为true，则会在滚动的时候低频更新thumb的位置和高度
     */
    autoUpdata: {
      type: Boolean,
      default: true
    },
    addIndicator: {
      type: Boolean,
      default: false
    },
    indicatorH: {
      type: Number,
      default: 12
    },
    speed: {  // click indicator move instance
      type: Number,
      default: 2
    },
    barWidth: {
      type: Number,
      default: 6
    }
  },
  methods: {
    update() {
      this.handleResize()
    },
    toUp() {
      this.moveScrollBar(-this.speed)
    },
    toDown() {
      this.moveScrollBar(this.speed)
    },
    scroll() {
      this.handleScrollBarPos()
      if (this.autoUpdata) {
        this.debounceHandleScrollBarH() //updata thumb's height
      }
    },
    barClick(e) {// jump to pos
      let clickPosY = this.addIndicator?e.layerY - this.indicatorH: e.layerY
      let thumbPosY = this.translateY * this.scrollBarPxH / 100
      let movePx = 0
      movePx = clickPosY - thumbPosY
      if (clickPosY < thumbPosY) {              // click pos above thumb
        movePx = clickPosY - thumbPosY - 2      // prevent multi call barClick
      } else {                                  // click pos under thumb
        movePx = clickPosY - thumbPosY - this.scrollBarPxH + 2 // prevent multi call barClick
      }
      this.moveScrollBar(movePx)
    },
    debounceHandleScrollBarH: debounce(function(){this.handleScrollBarH()}, 500, {leading: true, trailing: false }),//头和尾，低频调用
    thumbMouseDown(e) {
      document.addEventListener('mousemove', this.handleMouseMove)
      document.addEventListener('mouseup', this.handleMouseUp)
      this.startY = e.clientY
      this.isShowThumb = true
    },
    moveScrollBar(movePx) {// call this function will auto move content
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
      this.scrollBarH = `${percentage < 3? 3: percentage}`                  // % 高度
      // if add indicator, domWrapper's height is (domWrapperH - indicatorH)
      domWrapperH = this.addIndicator? domWrapperH - 24: domWrapperH
      this.scrollBarPxH = domWrapperH * this.scrollBarH / 100               // px 高度
      this.canScrollPxH = domWrapperH * (100 - this.scrollBarH) / 100       // 可滚动px高度

      this.maxTranslateY = (this.canScrollPxH / this.scrollBarPxH) * 100    // 滚动条可滑动的最大距离，以自身为基准 %
      this.maxScrollPxH = domViewerH - domWrapperH                          // 可滚动的最大高度px
    },
    handleScrollBarPos() {
      let domWrapper = this['xs-wrapper'], domViewer = this['xs-w-viewer']
      let offsetH = domWrapper.offsetHeight
      let percentage = domWrapper.scrollTop / this.maxScrollPxH

      // transform 3d turbo
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
@import "./../common/css/utils.scss";
@import "./../common/css/fiMixin.scss";
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
  @include setPos(absolute, right, 1px, top, 0);
  cursor: pointer;
  padding: 12px 0;
  box-sizing: border-box;
  overflow: hidden;
  height: 100%;
  width: 6px;
  transition: .3s background-color;
  &:hover {
    background-color: rgba(255, 255, 255, .1);
  }
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
  @include setPos(absolute, right, 1px);
  width: 100%;
  &.layoutTop {
    top: 0px;
  }
  &.layoutBottom {
    bottom: 0px;
  }
}

</style>