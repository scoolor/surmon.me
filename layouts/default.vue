<template>
  <div id="app" :class="theme" v-cloak>
    <div id="app-tools">
      <input type="text" v-model="clipboardText" class="clipboard-input" ref="clipboard">
    </div>
    <div id="app-aside" v-if="mobileLayout" :class="{ open: mobileSidebar }">
      <mobile-aside :class="{ open: mobileSidebar }"></mobile-aside>
    </div>
    <div id="app-main" :class="{ open: mobileSidebar }" @click="closeMobileSidebar">
      <wall-flower-box v-if="!mobileLayout && !powerSavingMode"></wall-flower-box>
      <emojo-rain v-if="!powerSavingMode"></emojo-rain>
      <background v-if="!mobileLayout"></background>
      <barrage v-if="!mobileLayout && barrageMounted" v-cloak></barrage>
      <transition name="fade">
        <webrtc v-if="!mobileLayout && !powerSavingMode && openWebrtc" v-cloak></webrtc>
      </transition>
      <header-view v-if="!mobileLayout"></header-view>
      <mobile-header v-if="mobileLayout"></mobile-header>
      <theme-view v-if="!mobileLayout && !powerSavingMode" @theme="setTheme"></theme-view>
      <main id="main" :class="{ 'mobile': mobileLayout, [$route.name]: true }">
        <transition name="module">
          <nav-view v-if="!errorColumn && !mobileLayout" keep-alive></nav-view>
        </transition>
        <div id="main-content" 
             class="main-content" 
             :class="{ 
               'full-column': fullColumn, 
               'error-column': errorColumn,
               'mobile-layout': mobileLayout,
               [$route.name]: true
              }">
          <nuxt></nuxt>
        </div>
        <transition name="aside">
          <aside-view v-if="!fullColumn && !errorColumn && !mobileLayout" keep-alive></aside-view>
        </transition>
      </main>
      <share-view class="sidebar-share" v-if="!mobileLayout && !['service'].includes($route.name)"></share-view>
      <tool-left v-if="!mobileLayout && !['service'].includes($route.name)"></tool-left>
      <tool-right v-if="!mobileLayout && !['app', 'music', 'service'].includes($route.name)"></tool-right>
      <footer-view v-if="!mobileLayout"></footer-view>
      <mobile-footer v-else></mobile-footer>
    </div>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import eventBus from '~/utils/event-bus'
  import { MobileHeader, MobileFooter, MobileAside } from '~/components/mobile'
  import { Background, EmojoRain, ToolLeft, ToolRight, Barrage, Webrtc, Header, Footer, Aside, Share, Theme, Nav } from '~/components/layout'
  export default {
    name: 'app',
    head() {
      return !this.mobileLayout ? {} : {
        bodyAttrs: {
          class: 'mobile'
        }
      }
    },
    data() {
      return {
        theme: 'default',
        clipboardText: ''
      }
    },
    mounted() {
      // this.watchFullScreen()
      this.watchTabActive()
      if (!this.mobileLayout) {
        this.$store.dispatch('loadMuiscPlayerList')
      }
      this.$root.$eventBus = eventBus
      this.$root.$copyToClipboard = this.copyToClipboard
    },
    components: {
      Webrtc,
      Barrage,
      EmojoRain,
      ToolLeft,
      ToolRight,
      Background,
      HeaderView: Header,
      FooterView: Footer,
      AsideView: Aside,
      ShareView: Share,
      ThemeView: Theme,
      NavView: Nav,
      MobileHeader,
      MobileFooter,
      MobileAside
    },
    computed: {
      ...mapState('option', ['openWebrtc', 'powerSavingMode', 'barrageMounted', 'fullColumn', 'errorColumn', 'mobileLayout', 'mobileSidebar'])
    },
    methods: {
      setTheme(theme) {
        this.theme = theme
      },
      copyToClipboard(text) {
        this.clipboardText = text
        // 维护中间量用于给拦截器做标识
        window.clickCopy = true
        setTimeout(() => {
          this.$refs.clipboard.select()
          document.execCommand('Copy')
          window.clickCopy = false
        })
      },
      closeMobileSidebar() {
        if (this.mobileLayout) {
          this.$store.commit('option/SET_MOBILE_SIDEBAR', false)
        }
      },
      watchTabActive() {
        let reallyDocumentTitle
        document.addEventListener('visibilitychange', event => {
          if (event.target.hidden || event.target.webkitHidden) {
            reallyDocumentTitle = document.title
            document.title = '皮皮虾，快回来！'
          } else {
            document.title = reallyDocumentTitle
          }
        }, false)
      },
      watchFullScreen() {
        const fullscreenchange = event => {
          // console.log('fullscreenchange', event)
        }
        document.addEventListener("fullscreenchange", fullscreenchange, false)
        document.addEventListener("mozfullscreenchange", fullscreenchange, false)
        document.addEventListener("webkitfullscreenchange", fullscreenchange, false)
        document.addEventListener("msfullscreenchange", fullscreenchange, false)
      }
    }
  }
</script>

<style lang="scss" scoped>
  #app {
    color: $text;

    &[v-cloak] {
      color: transparent;
      -webkit-text-fill-color: transparent;
    }

    #app-tools {
      height: 0px;
      opacity: 0;
      // overflow: hidden;
    }

    #app-aside {
      width: 68%;
      position: fixed;
      top: 0;
      left: 0;
      height: 100%;
      z-index: 9999;
      transform: translateX(-100%);
      transition: $mobile-aisde-transition;
      background-color: $mobile-aside-bg;

      &.open {
        overflow: hidden;
        transform: translateX(0);
        transition: $mobile-aisde-transition;
        -webkit-overflow-scrolling: touch;
      }
    }

    #app-main {
      transition: $mobile-aisde-transition;

      &.open {
        transition: $mobile-aisde-transition;
        transform: translateX(68%);
      }

      main {
        position: relative;

        &.service {
          width: 100%;
        }

        .main-content {
          float: left;
          width: 42.5em;
          margin: 0 0 0 12.5em;
          position: relative;
          overflow: hidden;
          @include css3-prefix(transition, width .35s);

          &:-moz-full-screen {
            overflow-y: auto;
          }
           
          &:-webkit-full-screen {
            overflow-y: auto;
          }
            
          &:fullscreen {
            overflow-y: auto;
          }

          &.full-column {
            width: 62.5em;
            @include css3-prefix(transition, width .35s);
          }

          &.error-column {
            width: 100%;
            margin: 0;
            @include css3-prefix(transition, width .35s);
          }

          &.mobile-layout {
            width: 100%;
            margin: 0;
            padding: 1rem;
            padding-top: $navbar-height + 1;
          }

          &.service {
            width: 100%;
            margin: -1em 0;

            &.mobile-layout {
              margin: 0;
            }
          }
        }
      }
    }
  }
</style>
