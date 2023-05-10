<template>
  <div class="web-search-container">
    <div class="web-search-bar">
      <div
        class="web-search-icon"
        :style="searchLabelStyle"
        @click="toggleEngineSelector"
        ref="icon"
      ></div>
      <form
        v-if="selectedEngine != null"
        :action="selectedEngine.url"
        method="get"
      >
        <input
          :name="selectedEngine.key"
          type="search"
          ref="search"
          v-model="value"
          @input="search($event.target.value)"
          placeholder="Search the web"
        />
      </form>
      <transition name="fade">
        <div v-if="displayEngineSelect" class="engine-select">
          <ul v-for="engine in engines" v-bind:key="engine.name">
            <li @click="selectEngine(engine)" :key="engine.url">
              <img :src="engine.icon" />
            </li>
          </ul>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
export default {
  name: "WebSearchInput",
  props: {
    hotkey: {
      type: String,
      default: "?",
    },
    engines: {
      type: Array,
      required: true,
      validator: function (value) {
        return value.length > 0;
      },
    },
  },
  data: function () {
    return {
      value: "",
      displayEngineSelect: false,
      selectedEngine: null,
      searchLabelStyle: {
        backgroundImage: null,
      },
    };
  },
  mounted() {
    this.initEngine();
    this._keyListener = function (event) {
      if (event.key === this.hotkey) {
        event.preventDefault();
        this.focus();
      }
      if (event.key === "Escape") {
        this.cancel();
      }
    };
    document.addEventListener("keydown", this._keyListener.bind(this));

    // fill search from get parameter.
    const search = new URLSearchParams(window.location.search).get("search");
    if (search) {
      this.$refs.search.value = search;
      this.search(search);
      this.focus();
    }

    window.addEventListener("click", this.clickListener);
  },
  methods: {
    focus: function () {
      this.$emit("search-focus");
      this.$nextTick(() => {
        this.$refs.search.focus();
      });
    },
    cancel: function () {
      this.setSearchURL("");
      this.$refs.search.value = "";
      this.$refs.search.blur();
      this.$emit("search-cancel");
    },
    search: function (value) {
      this.$emit("input", value.toLowerCase());
    },
    toggleEngineSelector: function () {
      this.displayEngineSelect = !this.displayEngineSelect;
    },
    hideEngineSelector: function () {
      this.displayEngineSelect = false;
    },
    selectEngine: function (engine) {
      this.selectedEngine = engine;
      this.updateSelectedEngineStyle();
      localStorage.setItem("engine", engine.url);
    },
    updateSelectedEngineStyle: function () {
      if (this.selectedEngine == null) {
        return;
      }
      this.searchLabelStyle.backgroundImage = `url(${this.selectedEngine.icon})`;
    },
    initEngine: function () {
      let url = localStorage.getItem("engine");
      this.engines.forEach((el) => {
        if (el.url == url) {
          this.selectEngine(el);
        }
      });
      // nothing found
      if (this.selectedEngine == null) {
        this.selectEngine(this.engines[0]);
      }
    },
    clickListener: function (event) {
      const iconRef = this.$refs.icon;
      if (event.composedPath().includes(iconRef)) {
        return;
      }

      this.hideEngineSelector();
    },
  },
  beforeUnmount() {
    document.removeEventListener("keydown", this._keyListener);
    window.removeEventListener("click", this.clickListener);
  },
};
</script>

<style lang="css" scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>

<style lang="scss" scoped>
.web-search-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
}
.web-search-bar {
  display: flex;
  align-items: center;
  position: relative;

  input {
    border: none;
    background-color: var(--highlight-hover);
    // background-color: rgba(255, 255, 255, 0.1);
    border-radius: 15px 5px 5px 15px;
    padding: 2px 12px 2px 40px;
    transition: all 100ms linear;
    color: var(--text);
    height: 30px;
    font-size: 16px;
    -webkit-box-shadow: var(--box-shadow);
    box-shadow: var(--box-shadow);
    vertical-align: top;
    width: 200px;

    border: none;

    &::placeholder {
      color: var(--text-header);
      opacity: 0.5;
    }

    &:focus,
    &:focus-visible,
    &:active {
      border: none;
      outline: none;
    }
  }

  @media screen and (min-width: 375px) {
    input:focus {
      width: 180px;
    }
  }
  @media screen and (min-width: 768px) {
    input:focus {
      width: 240px;
    }
  }
  @media screen and (min-width: 1000px) {
    input:focus {
      width: 300px;
    }
  }
  .web-search-icon {
    position: absolute;
    background-size: contain;
    background-repeat: no-repeat;
    width: 30px;
    height: 30px;
  }

  .web-search-icon:hover {
    cursor: pointer;
  }

  &:focus-within .web-search-icon::before {
    color: #6e6e6e;
  }

  .engine-select {
    display: flex;
    z-index: 99;
    position: absolute;
    margin-top: 10px;
    margin-left: 0;
    li {
      display: inline-block;
      margin: 10px 10px 0 0;
      padding: 4px;
      &:hover {
        background-color: rgba(0, 0, 0, 0.1);
        cursor: pointer;
      }
      img {
        width: 42px;
        height: 42px;
      }
    }
  }
}
</style>
