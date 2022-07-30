<template>
  <div>
    <nav :class="{ 'has-custom-titlebar': hasCustomTitlebar }">
      <Win32Titlebar v-if="enableWin32Titlebar" />
      <LinuxTitlebar v-if="enableLinuxTitlebar" />
      <div class="navigation-buttons">
        <button-icon v-focusable="true" @click.native="go('back')"
          ><svg-icon icon-class="arrow-left"
        /></button-icon>
        <button-icon v-focusable="true" @click.native="go('forward')"
          ><svg-icon icon-class="arrow-right"
        /></button-icon>
      </div>
      <div class="navigation-links">
        <router-link
          v-focusable="true"
          to="/"
          :class="{ active: $route.name === 'home' }"
          >{{ $t('nav.home') }}</router-link
        >
        <router-link
          v-focusable="true"
          to="/explore"
          :class="{ active: $route.name === 'explore' }"
          >{{ $t('nav.explore') }}</router-link
        >
        <router-link
          v-focusable="true"
          to="/library"
          :class="{ active: $route.name === 'library' }"
          >{{ $t('nav.library') }}</router-link
        >
      </div>
      <div class="right-part">
        <div class="search-box">
          <div
            class="container"
            @onFocus="inputFocus = true"
            @onBlur="inputFocus = false"
            v-focusable="true"
            @click="
              inputFocus = true;
              tv_search();
            "
            :class="{ active: inputFocus }"
          >
            <svg-icon icon-class="search" />
            <div class="input">
              <input
                ref="searchInput"
                v-model="keywords"
                type="search"
                :placeholder="inputFocus ? '' : $t('nav.search')"
                @keydown.enter="doSearch"
                @focus="inputFocus = true"
                @blur="inputFocus = false"
              />
            </div>
          </div>
        </div>
        <img
          class="avatar"
          :src="avatarUrl"
          @click="showUserProfileMenu()"
          loading="lazy"
          v-focusable="true"
        />
      </div>
    </nav>

    <ContextMenu title="个人中心" class="tv-profile-menu" ref="userProfileMenu">
      <!-- 设置 -->
      <div
        ref="tv_settings"
        id="tv_settings"
        class="item"
        @click="toSettings"
        v-focusable="true"
      >
        <svg-icon icon-class="settings" />
        {{ $t('library.userProfileMenu.settings') }}
      </div>
      <!-- 登出 -->
      <div
        v-focusable="true"
        v-if="!isLooseLoggedIn"
        class="item"
        ref="tv_toLogin"
        @click="toLogin"
      >
        <svg-icon icon-class="login" />
        {{ $t('login.login') }}
      </div>
      <!-- Github仓库 -->
      <div
        v-focusable="true"
        v-if="isLooseLoggedIn"
        class="item"
        ref="tv_logout"
        @click="logout"
        @up="tv_up('github')"
        @down="tv_down('logout')"
      >
        <svg-icon icon-class="logout" />
        {{ $t('library.userProfileMenu.logout') }}
      </div>
      <hr />
      <div
        ref="tv_github"
        @up="tv_up('github')"
        v-focusable="true"
        class="item"
        @click="toGitHub"
      >
        <svg-icon icon-class="github" />
        {{ $t('nav.github') }}
      </div>
      <div ref="tv" @up="tv_up('tv')" v-focusable="true" class="item">
        <svg-icon icon-class="github" />
        TV版作者 github@groupguanfang
      </div>
      <hr />
      <div
        @up="tv_up('close')"
        @down="tv_down('close')"
        v-focusable="true"
        class="item"
      >
        <svg-icon icon-class="logout" />
        关闭菜单
      </div>
    </ContextMenu>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import { isLooseLoggedIn, doLogout } from '@/utils/auth';

// import icons for win32 title bar
// icons by https://github.com/microsoft/vscode-codicons
import 'vscode-codicons/dist/codicon.css';

import Win32Titlebar from '@/components/Win32Titlebar.vue';
import LinuxTitlebar from '@/components/LinuxTitlebar.vue';
import ContextMenu from '@/components/ContextMenu.vue';
import ButtonIcon from '@/components/ButtonIcon.vue';

export default {
  name: 'Navbar',
  components: {
    Win32Titlebar,
    LinuxTitlebar,
    ButtonIcon,
    ContextMenu,
  },
  data() {
    return {
      inputFocus: false,
      langs: ['zh-CN', 'zh-TW', 'en', 'tr'],
      keywords: '',
      enableWin32Titlebar: false,
      enableLinuxTitlebar: false,
    };
  },
  computed: {
    ...mapState(['settings', 'data']),
    isLooseLoggedIn() {
      return isLooseLoggedIn();
    },
    avatarUrl() {
      return this.data?.user?.avatarUrl && this.isLooseLoggedIn
        ? `${this.data?.user?.avatarUrl}?param=512y512`
        : 'http://s4.music.126.net/style/web2/img/default/default_avatar.jpg?param=60y60';
    },
    hasCustomTitlebar() {
      return this.enableWin32Titlebar || this.enableLinuxTitlebar;
    },
  },
  created() {
    if (process.platform === 'win32') {
      this.enableWin32Titlebar = true;
    } else if (
      process.platform === 'linux' &&
      this.settings.linuxEnableCustomTitlebar
    ) {
      this.enableLinuxTitlebar = true;
    }
  },
  methods: {
    tv_up(key) {
      if (key == 'logout') {
        this.$tv.requestFocus(this.$refs.tv_settings);
      } else if (key == 'github') {
        this.$tv.requestFocus(this.$refs.tv_logout);
      } else if (key == 'tv') {
        this.$tv.requestFocus(this.$refs.tv_github);
      } else if (key == 'close') {
        this.$tv.requestFocus(this.$refs.tv);
      }
    },
    tv_down(key) {
      if (key == 'logout') {
        this.$tv.requestFocus(this.$refs.tv_github);
      } else if (key == 'close') {
        this.$tv.requestFocus(this.$refs.tv_settings);
      }
    },
    tv_search() {
      this.$refs.searchInput.focus();
    },
    go(where) {
      if (where === 'back') this.$router.go(-1);
      else this.$router.go(1);
    },
    doSearch() {
      if (!this.keywords) return;
      if (
        this.$route.name === 'search' &&
        this.$route.params.keywords === this.keywords
      ) {
        return;
      }
      this.$router.push({
        name: 'search',
        params: { keywords: this.keywords },
      });
    },
    showUserProfileMenu(e) {
      this.$refs.userProfileMenu.openMenu(e);
    },
    logout() {
      if (!confirm('确定要退出登录吗？')) return;
      doLogout();
      this.$router.push({ name: 'home' });
    },
    toSettings() {
      this.$router.push({ name: 'settings' });
    },
    toGitHub() {
      window.open('https://github.com/qier222/YesPlayMusic');
    },
    toLogin() {
      if (process.env.IS_ELECTRON === true) {
        this.$router.push({ name: 'loginAccount' });
      } else {
        this.$router.push({ name: 'login' });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
nav {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 64px;
  padding: {
    right: 10vw;
    left: 10vw;
  }
  backdrop-filter: saturate(180%) blur(20px);

  background-color: var(--color-navbar-bg);
  z-index: 100;
  -webkit-app-region: drag;
}

@media (max-width: 1336px) {
  nav {
    padding: 0 5vw;
  }
}

@supports (-moz-appearance: none) {
  nav {
    background-color: var(--color-body-bg);
  }
}

nav.has-custom-titlebar {
  padding-top: 20px;
  -webkit-app-region: no-drag;
}

.navigation-buttons {
  flex: 1;
  display: flex;
  align-items: center;
  .svg-icon {
    height: 24px;
    width: 24px;
  }
  button {
    -webkit-app-region: no-drag;
  }
}
@media (max-width: 970px) {
  .navigation-buttons {
    flex: unset;
  }
}

.navigation-links {
  flex: 1;
  display: flex;
  justify-content: center;
  text-transform: uppercase;
  user-select: none;
  a {
    -webkit-app-region: no-drag;
    font-size: 18px;
    font-weight: 700;
    text-decoration: none;
    border-radius: 6px;
    padding: 6px 10px;
    color: var(--color-text);
    transition: 0.2s;
    -webkit-user-drag: none;
    margin: {
      right: 12px;
      left: 12px;
    }
    &:hover,
    &.focus {
      background: var(--color-secondary-bg-for-transparent);
    }
    &:active {
      transform: scale(0.92);
      transition: 0.2s;
    }
  }
  a.active {
    color: var(--color-primary);
  }
}

.search {
  .svg-icon {
    height: 18px;
    width: 18px;
  }
}

.search-box {
  display: flex;
  justify-content: flex-end;
  -webkit-app-region: no-drag;

  .container {
    display: flex;
    align-items: center;
    height: 32px;
    background: var(--color-secondary-bg-for-transparent);
    border-radius: 8px;
    width: 200px;
  }

  .svg-icon {
    height: 15px;
    width: 15px;
    color: var(--color-text);
    opacity: 0.28;
    margin: {
      left: 8px;
      right: 4px;
    }
  }

  input {
    font-size: 16px;
    border: none;
    background: transparent;
    width: 96%;
    font-weight: 600;
    margin-top: -1px;
    color: var(--color-text);
  }

  .active {
    background: var(--color-primary-bg-for-transparent);
    input,
    .svg-icon {
      opacity: 1;
      color: var(--color-primary);
    }
  }
}

[data-theme='dark'] {
  .search-box {
    .active {
      input,
      .svg-icon {
        color: var(--color-text);
      }
    }
  }
}

.right-part {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  .avatar {
    user-select: none;
    height: 30px;
    margin-left: 12px;
    vertical-align: -7px;
    border-radius: 50%;
    cursor: pointer;
    -webkit-app-region: no-drag;
    -webkit-user-drag: none;
    &:hover,
    &.focus {
      filter: brightness(80%);
    }
  }
  .search-button {
    display: none;
    -webkit-app-region: no-drag;
  }
}
.navigation-links .focus {
  background: var(--color-secondary-bg-for-transparent);
}
.tv-profile-menu .item.focus {
  background: var(--color-primary);
}
.button-icon.focus {
  background: var(--color-secondary-bg-for-transparent);
}
</style>
