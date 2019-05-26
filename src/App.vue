<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link>|
      <router-link to="/about">About</router-link>
    </div>
    <router-view/>
    <stylizer/>
    <identity/>
    <menus/>
    <version/>
  </div>
</template>

<script>
import identity from "./components/main/identity.vue";
import stylizer from "./components/main/stylizer.vue";
import menus from "./components/main/menus.vue";
import version from "./components/main/version.vue";

export default {
  name: "App",
  components: {
    identity,
    stylizer,
    menus,
    version
  },
  computed: {
    storage() {
      return window.localStorage;
    }
  },
  data: () => ({
    csInterface: null,
    identity: null,
    stylizer: null,
    menus: null,
    isMounted: false
  }),
  mounted() {
    console.clear();
    this.csInterface = new CSInterface();
    this.csInterface.addEventListener("console", this.consoler);

    // Initialize utility components
    this.identity.init();
    this.stylizer.init();
    this.menus.init();
    console.log(
      `${this.identity.extName} ${this.identity.extVersion} : ${
        this.identity.isDev ? "DEV" : "BUILD"
      }`
    );
    this.isMounted = true;

    this.loadUniversalScripts();

    // Vue Router must be manually initialized in CEP:
    this.$router.push({ name: "home" });
  },
  methods: {
    dispatchEvent(name, data) {
      var event = new CSEvent(name, "APPLICATION");
      event.data = data;
      this.csInterface.dispatchEvent(event);
    },
    loadScript(path) {
      this.csInterface.evalScript(`$.evalFile('${path}')`);
    },
    loadUniversalScripts() {
      // Preloads universal scripts and main host script file
      this.loadScript(`${this.identity.root}/src/host/universal/json2.jsx`);
      this.loadScript(`${this.identity.root}/src/host/universal/Console.jsx`);
      this.loadScript(
        `${this.identity.root}/src/host/${this.identity.appName}/host.jsx`
      );
    },
    consoler(msg) {
      // Catches all console.log() usage in .jsx files via CSEvent
      console.log(`${this.identity.appName}: ${msg.data}`);
    },
    getCSS(prop) {
      // Returns current value of CSS variable
      // prop == typeof String as name of variable, with or without leading dashes:
      // this.getCSS('color-bg') || this.getCSS('--scrollbar-width')
      return window
        .getComputedStyle(document.documentElement)
        .getPropertyValue(`${/^\-\-/.test(prop) ? prop : "--" + prop}`);
    },
    setCSS(prop, data) {
      // Sets value of CSS variable
      // prop == typeof String as name of variable, with or without leading dashes:
      // this.setCSS('color-bg', 'rgba(25,25,25,1)') || this.setCSS('--scrollbar-width', '20px')
      document.documentElement.style.setProperty(
        `${/^\-\-/.test(prop) ? prop : "--" + prop}`,
        data
      );
    }
  }
};
</script>

<style>
#nav {
  padding: 10px;
  display: flex;
  justify-content: center;
  color: var(--color-text-default);
}

#nav a {
  padding: 0px 5px;
  font-weight: bold;
  color: var(--color-text-default);
}

#nav a.router-link-exact-active {
  color: white;
}

:root {
  /* 
      Below are CSS variables controlled by stylizer
      By default hard-coded to Illustrator's dark theme but will ultimately be overridden on launch
   */
  --color-bg: #323232;
  --color-dark: #1f1f1f;
  --color-selection: #46a0f5;
  --color-selection-dark: #3273af;
  --color-disabled: #525252;
  --color-hover: rgba(255, 255, 225, 0.2);

  --color-icon: #a1a1a1;
  --color-border: #3e3e3e;
  --color-text-active: #1b1b1b;
  --color-text-default: #a1a1a1;
  --color-text-disabled: #525252;
  --color-input-focus: #fcfcfc;
  --color-input-idle: #262626;
  --color-input-text: rgba(255, 255, 255, 0.7);
  --color-input-label: rgba(255, 255, 255, 0.6);

  --color-scrollbar: #2a2a2a;
  --color-scrollbar-thumb: #3e3e3e;
  --color-scrollbar-thumb-hover: #525252;
  --scrollbar-width: 14px;
  --scrollbar-thumb-width: 14px;
  --scrollbar-thumb-radius: 20px;

  --toolbar-height: 40;

  --quad: cubic-bezier(0.48, 0.04, 0.52, 0.96);
  --quart: cubic-bezier(0.76, 0, 0.24, 1);
  --quint: cubic-bezier(0.84, 0, 0.16, 1);

  background-color: var(--color-bg);
  font-family: "Roboto", "Helvetica", sans-serif;
}

/* Override certain default features to be more inline with Adobe's host app style */
body::-webkit-scrollbar {
  width: 0px;
}
#app::-webkit-scrollbar {
  display: block;
}
::-webkit-scrollbar {
  background-color: var(--color-scrollbar);
  width: var(--scrollbar-width);
}
::-webkit-scrollbar-thumb {
  width: var(--scrollbar-width);
  background: var(--color-scrollbar-thumb);
  border-radius: var(--scrollbar-thumb-radius);
}
::-webkit-scrollbar-thumb:hover {
  background: var(--color-scrollbar-thumb-hover);
}
::-webkit-scrollbar-resizer {
  display: none;
}
::-webkit-scrollbar-button {
  height: 0px;
}

/* Minor changes to vuetify's default component style */
.theme--dark.v-list {
  background-color: var(--color-input-idle);
}
.theme--dark.v-text-field--solo > .v-input__control > .v-input__slot {
  background-color: var(--color-scrollbar);
}
</style>
