<template>
  <div style="padding-top: 1em">
    <v-list-item v-for="(item, index) in settingsItems" :key="index">
      <v-btn text class="entry text-left text-capitalize" :to="item.to" :disabled="item.disabled">
        <v-icon size="30px" class="icon" v-text="item.icon" />
        {{ item.name }}
      </v-btn>
    </v-list-item>

    <!--   Dev Mode Open   -->
    <v-btn text class="entry" @click="dev()" v-if="!devmode" />

    <v-btn text class="entry text-left text-capitalize" style="margin: 0 0.75em 0 0.75em;" to="/mods/developer" v-if="devmode">
      <v-icon size="30px" class="icon" >mdi-database-edit</v-icon>
      {{ devmodebuttonname }}
    </v-btn>
    <!--   End Dev Mode   -->

  </div>
</template>

<style scoped>
  .entry {
    width: 100%;
    font-size: 1.2em;
    justify-content: left !important;
    padding: 1.5em 0.5em 1.5em 0.5em !important;
  }

  .icon {
    margin-right: 0.5em;
  }

</style>

<script>
  export default {
    data() {
      return {
        devClicks: 0,
        devmode: false,

        devmodebuttonname: "Developer Mode",

        settingsItems: [{
            name: "General",
            icon: "mdi-cog",
            to: "",
            disabled: true
          },
          {
            name: "Theme",
            icon: "mdi-brush-variant",
            to: "/mods/theme"
          },
          {
            name: "Player",
            icon: "mdi-motion-play-outline",
            to: "",
            disabled: true,
          },
          {
            name: "UI Tweaker",
            icon: "mdi-television-guide",
            to: "/mods/tweaks",
          },
          {
            name: "Startup Options",
            icon: "mdi-restart",
            to: "/mods/startup"
          },
          {
            name: "Plugins",
            icon: "mdi-puzzle",
            to: "",
            to: "/mods/plugins",
            disabled: true
          },
          {
            name: "Updates",
            icon: "mdi-cloud-download-outline",
            to: "/mods/updates",
          },
          {
            name: "Logs",
            icon: "mdi-text-box-outline",
            to: "/mods/logs"
          },
          {
            name: "About",
            icon: "mdi-information-outline",
            to: "/mods/about"
          },
        ],
      };
    },

    mounted() {
      this.settingsItems[0].name = this.$lang('settings').general;
      this.settingsItems[1].name = this.$lang('settings').theme;
      this.settingsItems[2].name = this.$lang('settings').player;
      this.settingsItems[3].name = this.$lang('settings').uitweaker;
      this.settingsItems[4].name = this.$lang('settings').startupoptions;
      this.settingsItems[5].name = this.$lang('settings').plugins;
      this.settingsItems[6].name = this.$lang('settings').updates;
      this.settingsItems[7].name = this.$lang('settings').logs;
      this.settingsItems[8].name = this.$lang('settings').about;
      this.devmodebuttonname = this.$lang('settings').devmode;

      this.devmode = localStorage.getItem('devmode');
    },

    methods: {
      dev() {
        this.devClicks++;
        if (this.devClicks >= 6) {
          localStorage.setItem('devmode', 'true');
          this.devmode = true;
        }
      },
    },
  };

</script>
