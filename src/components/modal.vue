<template>
  <div v-if="currentShow" class="modal" :class=" virtualShow && 'show' ">
    <div class="modal-body">
      <div class="header">
        <h3 class="title">{{title}}</h3>
        <a @click="close" class="btn-close">
          <img src="./btn-close.png" alt="">
        </a>
      </div>
      <div class="content">
        <div class="slot-wrapper">
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'modal',
  data () {
    return {
      currentShow: false,
      virtualShow: false
    }
  },
  props: {
    title: {
      type: String,
      default: ''
    }
  },
  methods: {
    show () {
      this.currentShow = true
    },
    close () {
      this.currentShow = false
    }
  },
  watch: {
    currentShow (val) {
      setTimeout(() => {
        this.virtualShow = val
      }, 20)
    }
  }
}
</script>

<style lang="scss" scoped>
$header-height: 42px;
.modal
{
  position: fixed;
  top: 0%;
  left: 0%;
  z-index: 99909;
  width: 100%;
  height: 100%;
  background-color: rgba(#25262d, 0.4);
  opacity: 0;
  transition: opacity 0.3s;
  &.show
  {
    opacity: 1;
    .modal-body
    {
      transform: translate(-50%, -50%) scale(1);
    }

  }
}
.modal-body
{
  position: absolute;
  top: 49%;
  left: 50%;
  width: 94%;
  border-radius: 5px;
  background-color: #fff;
  transform: translate(-50%, -50%) scale(0.8);
  transition: transform 0.3s;
  overflow: hidden;
  .header
  {
    position: relative;
    background-color: #dc6450;
    // background-color: #FAFAFA;
    .title
    {
      margin: 0px;
      height: $header-height;
      line-height: $header-height;
      text-align: center;
      font-size: 16px;
      font-weight: normal;
      color: #fff;
      letter-spacing: 1px;
    }
    .btn-close
    {
      position: absolute;
      top: 0px;
      right: 0px;
      width: $header-height;
      height: $header-height;
      img
      {
        position: absolute;
        top: 50%;
        left: 50%;
        max-width: $header-height / 2.5;
        max-height: $header-height / 2.5;
        transform: translate(-50%, -50%);
      }
    }
  }
  .content
  {
    flex: 1;
    margin: 8px 0px 8px 6px;
    height: 100%;
    overflow: hidden;
    .slot-wrapper
    {
      flex: 1;
      // position: relative;
      max-height: calc(85vh - #{$header-height});
      overflow: hidden;
      overflow-y: auto;
      -webkit-overflow-scrolling: touch;
    }
  }
}
</style>
