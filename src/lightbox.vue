<template>
  <div class="vue-images">
    <gallery :images="images" @changeIndex="changeImg($event)"></gallery>
    <div ref="lightbox" class="lightbox" v-show="isShow" @click="isShow=!modalclose">
      <fancybox ref="fancybox" :images="images" :index="index" :reset="!isShow" @play="playImg" @pause="pauseImg" @close="closeImg" @addIndex="nextImg" @decIndex="prevImg" :showplaybutton="showplaybutton" :showfullbutton="showfullbutton" :showclosebutton="showclosebutton" :showcaption="showcaption" :imagecountseparator="imagecountseparator" :showimagecount="showimagecount"></fancybox>
      <paginator :images="images" :activeIndex="index" @changeIndex="changeImg($event)" v-show="showthumbnails"></paginator>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import gallery from 'components/gallery'
  import fancybox from 'components/fancybox'
  import paginator from 'components/paginator'

  export default {
    name: 'lightbox',
    props: {
      imgs: Array,
      modalclose: Boolean,
      keyinput: Boolean,
      mousescroll: Boolean,
      showplaybutton: Boolean,
      showfullbutton: Boolean,
      showclosebutton: Boolean,
      showcaption: Boolean,
      imagecountseparator: String,
      showimagecount: Boolean,
      showthumbnails: Boolean
    },
    computed: {
      images () {
        let retArr = []
        let len = this.imgs.length
        this.imgs.forEach((item, index) => {
          item['index'] = index + 1
          item['total'] = len
          item['isActive'] = false
          retArr.push(item)
        })
        return retArr
      }
    },
    data () {
      return {
        isShow: false,
        index: 1,
        playTimer: null,
        touchPoint: {
          prev: 0,
          now: 0
        }
      }
    },
    created () {
      if (this.isShow) {
        window.addEventListener('keydown', this.keyFun)
        window.addEventListener('mousewheel', this.wheelFun)
        this.$refs.lightbox.addEventListener('touchstart', this.touchFun)
      }
    },
    methods: {
      openImg () {
        this.isShow = true
      },
      playImg () {
        var that = this
        this.playTimer = window.setInterval(that.nextImg, 2000)
      },
      pauseImg () {
        window.clearInterval(this.playTimer)
      },
      closeImg () {
        this.isShow = false
      },
      nextImg () {
        if (this.index < this.images.length - 1) {
          this.index++
        } else {
          this.index = 0
        }
      },
      prevImg () {
        if (this.index > 0) {
          this.index--
        }
      },
      changeImg (event) {
        this.isShow = true
        this.index = event
      },
      keyFun (event) {
        event.preventDefault()
        if (this.keyinput) {
          var that = this
          switch (event.keyCode) {
            case 27:
              this.closeImg()
              break
            case 37:
              if (this.index > 0) {
                this.$refs.fancybox.$refs.images[this.index].classList.add('slideOutRight')
                window.setTimeout(() => {
                  that.$refs.fancybox._data.next = true
                  that.$refs.fancybox._data.animation = true
                  that.prevImg()
                }, 375)
              }
              break
            case 39:
              if (this.index < this.images[this.index].total - 1) {
                this.$refs.fancybox.$refs.images[this.index].classList.add('slideOutLeft')
                window.setTimeout(() => {
                  that.$refs.fancybox._data.next = false
                  that.$refs.fancybox._data.animation = true
                  that.nextImg()
                }, 375)
              }
              break
            default:
              return
          }
        } else {
          return
        }
      },
      wheelFun (event) {
        if (this.mousescroll) {
          event.stopPropagation()
          var that = this
          if (event.deltaY > 0) {
            if (this.index < this.images[this.index].total - 1) {
              this.$refs.fancybox.$refs.images[this.index].classList.add('slideOutLeft')
              window.setTimeout(() => {
                that.$refs.fancybox._data.next = false
                that.$refs.fancybox._data.animation = true
                that.nextImg()
              }, 375)
            }
          } else {
            if (this.index > 0) {
              this.$refs.fancybox.$refs.images[this.index].classList.add('slideOutRight')
              window.setTimeout(() => {
                that.$refs.fancybox._data.next = true
                that.$refs.fancybox._data.animation = true
                that.prevImg()
              }, 375)
            }
          }
        } else {
          return
        }
      },
      touchFun (event) {
        event.stopPropagation()
        this.touchPoint.prev = event.touches[0].clientX
      },
      endFun (event) {
        event.stopPropagation()
        this.touchPoint.now = event.changedTouches[0].clientX
        var that = this
        if (this.touchPoint.prev > this.touchPoint.now + 50) {
          if (this.index < this.images[this.index].total - 1) {
            this.$refs.fancybox.$refs.images[this.index].classList.add('slideOutLeft')
            window.setTimeout(() => {
              that.$refs.fancybox._data.next = false
              that.$refs.fancybox._data.animation = true
              that.nextImg()
            }, 375)
          }
        } else if (this.touchPoint.now > this.touchPoint.prev + 50) {
          if (this.index > 0) {
            this.$refs.fancybox.$refs.images[this.index].classList.add('slideOutRight')
            window.setTimeout(() => {
              that.$refs.fancybox._data.next = true
              that.$refs.fancybox._data.animation = true
              that.prevImg()
            }, 375)
          }
        }
      }
    },
    watch: {
      isShow () {
        if (this.isShow) {
          document.body.style.position = 'fixed'
          window.addEventListener('keydown', this.keyFun)
          this.$refs.lightbox.addEventListener('mousewheel', this.wheelFun)
          this.$refs.lightbox.addEventListener('touchstart', this.touchFun)
          this.$refs.lightbox.addEventListener('touchend', this.endFun)
        } else {
          this.pauseImg()
          document.body.style.position = 'static'
          window.removeEventListener('keydown', this.keyFun)
          this.$refs.lightbox.removeEventListener('mousewheel', this.wheelFun)
          this.$refs.lightbox.removeEventListener('touchstart', this.touchFun)
          this.$refs.lightbox.removeEventListener('touchend', this.endFun)
        }
      }
    },
    components: {
      gallery,
      fancybox,
      paginator
    }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .lightbox
    position: fixed
    top: 0
    left: 0
    z-index: 9999
    width: 100%
    height: 100%
    padding: 2px
    background: rgba(0, 0, 0, 0.8)
    box-sizing: border-box
    font-size: 0
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif
</style>
