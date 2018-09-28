<template>
<div 
  @mousedown="eMouseDown" 
  @mouseup="eMouseUp" 
  @mouseleave="eMouseLeave"
  >
  <i :class="'indicator-' + direction"></i>
</div>
</template>
<script>
export default {
  name: 'xIndicator',
  props: {
    direction: {
      validator: function(val) {
        return ['up', 'down'].indexOf(val) != -1
      },
      default: 'up'
    }
  },
  mounted() {
    this.intervalId = null
  },
  methods: {
    eMouseDown() {
      this.$emit('press')
      this.intervalId = setInterval((  )=> this.$emit('press'), 16)
    },
    eMouseUp() {
      this.stopEmit()
    },
    eMouseLeave() {
      this.stopEmit()
    },
    stopEmit() {
      clearInterval(this.intervalId)
    }
  }
}


</script>
<style lang="scss" scoped>

@mixin setWH($w, $h) {
  width: $w;
  height: $h;
}
$height: 7px;
$width: 4px;
$borderC: transparent;
@mixin specialBorder($direction) {
  border-#{$direction}: $height dashed $borderC;
}
%scrollbar-triangle {
  display: block;
  position: relative;
  cursor: pointer;
  @include setWH(2*$width, 2*$height - 2px);
  &:before, &:after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    @include setWH(0, 0);
    transition: .3s border-color;
    border-left: $width dashed transparent;
    border-right: $width dashed transparent;
    border-radius: 3px;
  }
  &:after {
    top: 5px;
  }
}
div {
  i.indicator-up {
    @extend %scrollbar-triangle;
    &:before, &:after {
      @include specialBorder(bottom);
    }
  }
  i.indicator-down {
    @extend %scrollbar-triangle;
    &:before, &:after {
       @include specialBorder(top);
    }
  }
}

</style>