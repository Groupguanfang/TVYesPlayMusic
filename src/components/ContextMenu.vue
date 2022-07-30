<template>
  <div ref="contextMenu" class="context-menu">
    <div
      v-if="showMenu"
      id="menu"
      ref="menu"
      class="menu"
      tabindex="-1"
      @click="closeMenu"
      @blur="closeMenu"
    >
      <div class="content">
        <h1>{{ title }}</h1>
        <div class="main-content">
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex';

export default {
  name: 'ContextMenu',
  data() {
    return {
      showMenu: false,
      top: '0px',
      left: '0px',
    };
  },
  props: ['title'],
  computed: {
    ...mapState(['player']),
  },
  mounted() {
    let self = this;
    this.$nextTick(function () {
      document.addEventListener('keyup', function (e) {
        //此处填写你的业务逻辑即可
        if (e.keyCode == 27) {
          self.closeMenu();
          self.closeScreenFull();
        }
      });
    });
  },
  methods: {
    setMenu(top, left) {
      let heightOffset = this.player.enabled ? 64 : 0;
      let largestHeight =
        window.innerHeight - this.$refs.menu.offsetHeight - heightOffset;
      let largestWidth = window.innerWidth - this.$refs.menu.offsetWidth - 25;
      if (top > largestHeight) top = largestHeight;
      if (left > largestWidth) left = largestWidth;
      this.top = top + 'px';
      this.left = left + 'px';
    },

    closeMenu() {
      this.showMenu = false;
      if (this.$parent.closeMenu !== undefined) {
        this.$parent.closeMenu();
      }
      this.$store.commit('enableScrolling', true);
    },

    openMenu() {
      this.showMenu = true;
      this.$tv.limitingEl = document.querySelector('#menu');
      // this.$nextTick(
      //   function () {
      //     this.$refs.menu.focus();
      //     this.setMenu(e.y, e.x);
      //   }.bind(this)
      // );
      // e.preventDefault();
      // this.$store.commit('enableScrolling', false);
    },
  },
};
</script>

<style lang="scss" scoped>
@keyframes fade {
  from {
    transform: scale(0) perspective(2em) rotateY(3deg);
  }
  to {
    transform: scale(1);
  }
}
.context-menu {
  width: 100%;
  height: 100%;
  user-select: none;
}

.menu {
  transition: ease 0.2s;
  position: fixed;
  min-width: 136px;
  max-width: 240px;
  list-style: none;
  background: rgba(255, 255, 255, 0.88);
  box-shadow: 0 6px 12px -4px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(0, 0, 0, 0.06);
  backdrop-filter: blur(12px);
  border-radius: 12px;
  box-sizing: border-box;
  padding: 6px;
  z-index: 1000;
  -webkit-app-region: no-drag;
  transition: background 125ms ease-out, opacity 125ms ease-out,
    transform 125ms ease-out;

  &:focus {
    outline: none;
  }
}

[data-theme='dark'] {
  .menu {
    background: #fff0;
    backdrop-filter: blur(100px) contrast(120%) brightness(100%);
    border: 1px solid rgba(255, 255, 255, 0.08);
    box-shadow: 0 0 6px rgba(255, 255, 255, 0.08);
  }
  .menu .item:hover {
    color: var(--color-text);
  }
}

@supports (-moz-appearance: none) {
  .menu {
    background-color: var(--color-body-bg) !important;
  }
}

.menu .item {
  font-weight: 600;
  font-size: 26px;
  padding: 10px 14px;
  border-radius: 8px;
  cursor: default;
  color: var(--color-text);
  display: flex;
  align-items: center;
  &:hover {
    color: var(--color-primary);
    background: var(--color-primary-bg-for-transparent);
    transition: opacity 125ms ease-out, transform 125ms ease-out;
  }
  &:active {
    opacity: 0.75;
    transform: scale(0.95);
  }

  .svg-icon {
    height: 35px;
    width: 25px;
    margin-right: 10px;
  }
}

hr {
  margin: 4px 10px;
  background: rgba(128, 128, 128, 0.18);
  height: 1px;
  box-shadow: none;
  border: none;
}

.item-info {
  padding: 10px 10px;
  display: flex;
  align-items: center;
  color: var(--color-text);
  cursor: default;
  img {
    height: 38px;
    width: 38px;
    border-radius: 4px;
  }
  .info {
    margin-left: 10px;
  }
  .title {
    font-size: 16px;
    font-weight: 600;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
    word-break: break-all;
  }
  .subtitle {
    font-size: 12px;
    opacity: 0.68;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
    word-break: break-all;
  }
}
.menu {
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100%;
  border-radius: 0;
  max-width: 100%;
}
.content {
  animation: fade infinite 1s;
  animation-iteration-count: 1;
  max-width: 500px;
  margin-top: 5%;
  margin-left: 5%;
}
h1 {
  font-size: 48px;
}
.main-content {
  padding: 8px;
  border: 1px solid rgba(128, 128, 128, 0.18);
  border-radius: 8px;
}
</style>
