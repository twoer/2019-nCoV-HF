<template>
  <div v-if="currentShow" class="modal" :class=" virtualShow && 'show' ">
    <div class="modal-body">
      <div class="content">
        <slot></slot>
      </div>
      <div class="footer">
        <a @click="close" class="btn-close">
          <img src="./btn-close.png" alt="">
          <span>收起</span>
        </a>
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
    },
    toggle () {
      this.currentShow = !this.currentShow
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
$footer-height: 32px;
.modal
{
  position: fixed;
  top: 58px;
  left: 8px;
  right: 8px;
  bottom: 15px;
  z-index: 999999;
  opacity: 0;
  transition: opacity 0.3s;
  background-color: #fff;
  &.show
  {
    opacity: 1;
  }
}
.modal-body
{
  position: absolute;
  top: 0px;
  left: 0px;
  right: 0px;
  bottom: 0px;
  padding-bottom: $footer-height;
  background-color: #fff;
  transition: transform 0.3s;
  overflow: hidden;
  box-shadow: 1px 1px 10px rgba(0, 0,0,.1), 1px 1px 2px rgba(0, 0,0,.1);
  .footer
  {
    flex: 0 0 $footer-height;
    height: $footer-height;
    .btn-close
    {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      height: $footer-height;
      color: #DC6450;
      font-size: 10px;
      img
      {
        max-width: 12px;
        max-height: 12px;
      }
    }
  }
  .content
  {
    flex: 1;
    display: flex;
    flex-direction: column;
    height: 100%;
    overflow: hidden;
  }
}
</style>
