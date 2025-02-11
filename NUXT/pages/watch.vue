<template>
  <div class="background" id="watch-body">
    <div id="player-container">
      <v-btn text style="position: fixed; z-index: 69420" to="home">
        <v-icon>mdi-chevron-down</v-icon>
      </v-btn>
      <!--   VueTube Player V1   -->
      <vuetubePlayer
        :sources="sources"
        v-if="useBetaPlayer === 'true' && sources.length > 0"
      />

      <!--   Stock Player   -->
      <legacyPlayer
        id="player"
        ref="player"
        v-touch="{ down: () => $router.push('/home') }"
        class="background"
        :vid-src="vidSrc"
        v-if="useBetaPlayer !== 'true'"
      />
    </div>

    <div
      v-bind:class="{
        'overflow-y-auto': !showComments,
        'overflow-y-hidden': showComments,
      }"
      id="content-container"
    >
      <v-card v-if="loaded" class="ml-2 mr-2 background rounded-0" flat>
        <div
          v-ripple
          class="d-flex justify-space-between align-start px-3 pt-3"
          @click="showMore = !showMore"
        >
          <div class="d-flex flex-column">
            <v-card-title
              class="pa-0"
              style="font-size: 0.95rem; line-height: 1.15rem"
              v-text="video.title"
              v-emoji
            />
            <v-card-text
              style="font-size: 0.75rem"
              class="background--text pa-0"
              :class="
                $vuetify.theme.dark ? 'text--lighten-4' : 'text--darken-4'
              "
            >
              <div style="margin-bottom: 1rem">
                <template
                  v-for="text in video.metadata.contents.find(
                    (content) => content.slimVideoInformationRenderer
                  ).slimVideoInformationRenderer.collapsedSubtitle.runs"
                  >{{ text.text }}
                </template>
              </div>
            </v-card-text>
          </div>
          <v-icon class="ml-4" v-if="showMore">mdi-chevron-up</v-icon>
          <v-icon class="ml-4" v-else>mdi-chevron-down</v-icon>
        </div>
        <div class="d-flex">
          <v-btn
            v-for="(item, index) in interactions"
            :key="index"
            text
            fab
            class="vertical-button ma-1"
            elevation="0"
            style="width: 4.2rem !important; height: 4.2rem !important"
            :disabled="item.disabled"
            @click="callMethodByName(item.actionName)"
          >
            <v-icon v-text="item.icon" />
            <div
              class="mt-2"
              style="font-size: 0.66rem"
              v-text="item.value || item.name"
            />
          </v-btn>
          <!--   End Scrolling Div For Interactions   --->
          <!-- <hr /> -->
        </div>
        <!-- <v-bottom-sheet
          v-model="showMore"
          color="background"
          style="z-index: 9999999"
        >
          <v-sheet style="padding: 12px">
            <v-btn block @click="showMore = !showMore"
              ><v-icon>mdi-chevron-down</v-icon></v-btn
            ><br />

            <slim-video-description-renderer
              class="scroll-y"
              :render="video.renderedData.description"
            />
          </v-sheet>
        </v-bottom-sheet> -->

        <!-- <v-bottom-sheet v-model="share" color="background" style="z-index: 9999999">
          <v-sheet style="padding: 1em">
            <div class="scroll-y">
              {{ response.renderedData.description }}
            </div>
          </v-sheet>
        </v-bottom-sheet> -->
      </v-card>
      <v-divider />

      <!--   Channel Bar   -->
      <div class="channel-container" v-if="loaded">
        <v-card
          class="channel-section background px-3 rounded-0"
          :to="video.channelUrl"
        >
          <div id="details">
            <div class="avatar-link mr-3">
              <v-img class="avatar-thumbnail" :src="video.channelImg" />
            </div>
            <div class="channel-byline" v-emoji>
              <div class="channel-name" v-text="video.channelName" />
              <div
                class="caption background--text"
                :class="
                  $vuetify.theme.dark ? 'text--lighten-4' : 'text--darken-4'
                "
                v-text="video.channelSubs"
              />
            </div>
          </div>
          <div
            class="channel-buttons"
            style="color: rgb(204, 0, 0); text-transform: uppercase"
          >
            subscribe
          </div>
        </v-card>
        <v-divider />
      </div>

      <!-- Description -->
      <div v-if="showMore">
        <div class="scroll-y ma-4">
          <slim-video-description-renderer
            :render="video.renderedData.description"
          />
        </div>
        <v-divider />
      </div>

      <!-- Comments -->
      <div
        v-if="loaded && video.commentData"
        @click="showComments = !showComments"
      >
        <v-card flat class="background comment-renderer">
          <v-text class="comment-count keep-spaces">
            <template v-for="text in video.commentData.headerText.runs">
              <template v-if="text.bold">
                <strong :key="text.text">{{ text.text }}</strong>
              </template>
              <template v-else>{{ text.text }}</template>
            </template>
          </v-text>
          <v-icon v-if="showComments">mdi-unfold-less-horizontal</v-icon>
          <v-icon v-else>mdi-unfold-more-horizontal</v-icon>
        </v-card>
        <v-divider />
      </div>

      <swipeable-bottom-sheet
        v-model="showComments"
        hide-overlay
        persistent
        no-click-animation
        attach="#content-container"
        v-if="loaded && video.commentData"
      >
        <mainCommentRenderer
          :defaultContinuation="video.commentContinuation"
          :commentData="video.commentData"
          v-model="showComments"
        ></mainCommentRenderer>
      </swipeable-bottom-sheet>

      <!-- <swipeable-bottom-sheet
      :v-model="showComments"
      style="z-index: 9999999"
    ></swipeable-bottom-sheet> -->

      <!-- Related Videos -->
      <div class="loaders" v-if="!loaded">
        <v-skeleton-loader
          type="list-item-two-line, actions, divider, list-item-avatar, divider, list-item-three-line"
        />
        <vid-load-renderer :count="5" />
      </div>
      <item-section-renderer v-else :render="recommends" />
    </div>
  </div>
</template>

<script>
import { Share } from "@capacitor/share";
import VidLoadRenderer from "~/components/vidLoadRenderer.vue";
import { getCpn } from "~/plugins/utils";
import SlimVideoDescriptionRenderer from "~/components/UtilRenderers/slimVideoDescriptionRenderer.vue";
import ItemSectionRenderer from "~/components/SectionRenderers/itemSectionRenderer.vue";
import legacyPlayer from "~/components/Player/legacy.vue"
import vuetubePlayer from "~/components/Player/index.vue";
import ShelfRenderer from "~/components/SectionRenderers/shelfRenderer.vue";
import mainCommentRenderer from "~/components/Comments/mainCommentRenderer.vue";
import SwipeableBottomSheet from "~/components/ExtendedComponents/swipeableBottomSheet";

import { App as CapacitorApp } from "@capacitor/app";

export default {
  components: {
    ShelfRenderer,
    VidLoadRenderer,
    SlimVideoDescriptionRenderer,
    vuetubePlayer,
    legacyPlayer,
    ItemSectionRenderer,
    SwipeableBottomSheet,
    mainCommentRenderer,
  },
  layout: "empty",
  // transition(to) { // TODO: fix layout switching
  //   return to.name == "watch"
  //     ? { name: "slide-up", mode: "" }
  //     : { name: "slide-down", mode: "" };
  // },
  data: function () {
    return this.initializeState();
  },
  watch: {
    // Watch for change in the route query string (in this case, ?v=xxxxxxxx to ?v=yyyyyyyy)
    $route: {
      deep: true,
      handler(newRt, oldRt) {
        if (newRt.query.v && newRt.query.v != oldRt.query.v) {
          // Exit fullscreen if currently in fullscreen
          // if (this.$refs.player) this.$refs.player.webkitExitFullscreen();
          // Reset player and run getVideo function again
          // this.vidSrc = "";
          // this.startTime = Math.floor(Date.now() / 1000);
          // this.getVideo();
          clearInterval(this.interval);
          Object.assign(this.$data, this.initializeState());
          this.mountedInit();
        }
      },
    },
  },

  mounted() {
    this.mountedInit();

    this.backHandler = CapacitorApp.addListener(
      "backButton",
      ({ canGoBack }) => {
        //---   Back Closes Search   ---//
        if (this.showComments) {
          this.showComments = false;

          //---   Back Goes Back   ---//
        } else if (!canGoBack) {
          this.$router.replace(
            `/${localStorage.getItem("startPage") || "home"}`
          );
        } else {
          window.history.back();
        }
      }
    );
  },

  beforeDestroy() {
    clearInterval(this.interval);
    if (this.backHandler) this.backHandler.remove();
  },

  methods: {
    getVideo() {
      this.loaded = false;

      this.$youtube.getVid(this.$route.query.v).then((result) => {
        this.video = result;
        console.log("Video info data", result);
        console.log(result.availableResolutions);

        //---   VueTube Player v1   ---//
        this.sources = result.availableResolutions;

        //---   Legacy Player   ---//
        this.vidSrc =
          result.availableResolutions[
            result.availableResolutions.length - 1
          ].url; // Takes the highest available resolution with both video and Audio. Note this will be lower than the actual highest resolution

        //---   Content Stuff   ---//
        this.likes = result.metadata.likes.toLocaleString();
        this.interactions[0].value = result.metadata.likes.toLocaleString();
        this.loaded = true;
        this.recommends = result.renderedData.recommendations;
        // .catch((error) => this.$logger("Watch", error, true));
        console.log("recommendations:", this.recommends);

        //---   API WatchTime call   ---//
        this.playbackTracking = result.playbackTracking;
        this.st = 0;
        this.cpn = getCpn();
        this.initWatchTime().then(() => {
          this.sendWatchTime();
          this.interval = setInterval(this.sendWatchTime, 60000);
        });
      });

      this.$youtube.getReturnYoutubeDislike(this.$route.query.v, (data) => {
        this.dislikes = data.dislikes.toLocaleString();
        this.interactions[1].value = data.dislikes.toLocaleString();
      });
    },
    callMethodByName(name) {
      // Helper function needed because of issues when directly calling method
      // using item.action in the v-for loop
      this[name]();
    },
    dislike() {},
    async share() {
      // this.share = !this.share;
      await Share.share({
        title: this.video.title,
        text: this.video.title,
        url: "https://youtu.be/" + this.$route.query.v,
        dialogTitle: "Share video",
      });
    },
    sendWatchTime() {
      const player = this.$refs.player.getPlayer();
      const rt = Math.floor(Date.now() / 1000) - this.startTime;
      const params = {
        cpn: this.cpn,
        rt: rt,
        rti: rt,
        rtn: rt,
        cmt: player.currentTime,
        et: player.currentTime,
        st: this.st,
        state: player.paused ? "paused" : "playing",
        volume: 100,
        muted: 0,
        fmt: 396,
      };
      this.st = player.currentTime;
      this.$youtube.saveApiStats(
        params,
        this.playbackTracking.videostatsWatchtimeUrl.baseUrl
      );
    },

    async initWatchTime() {
      await this.$youtube.saveApiStats(
        {
          cpn: this.cpn,
          fmt: 243,
          rtn: Math.floor(Date.now() / 1000) - this.startTime,
          rt: Math.floor(Date.now() / 1000) - this.startTime,
          muted: 0,
        },
        this.playbackTracking.videostatsPlaybackUrl.baseUrl
      );
    },

    initializeState() {
      return {
        interactions: [
          {
            name: "Likes",
            icon: "mdi-thumb-up-outline",
            // action: null,
            value: this.likes,
            disabled: true,
          },
          {
            name: "Dislikes",
            icon: "mdi-thumb-down-outline",
            // action: this.dislike(),
            actionName: "dislike",
            value: this.dislikes,
            disabled: true,
          },
          {
            name: "Share",
            icon: "mdi-share-outline",
            // action: this.share(),
            actionName: "share",
            disabled: false,
          },
        ],
        showMore: false,
        showComments: false,
        // share: false,
        vidSrc: null,
        sources: [],
        recommends: null,
        loaded: false,
        interval: null,
        video: null,
        useBetaPlayer: false,
      };
    },

    mountedInit() {
      this.startTime = Math.floor(Date.now() / 1000);
      this.getVideo();
      this.useBetaPlayer = localStorage.getItem("debug.BetaPlayer");

      //  Reset vertical scrolling
      const scrollableList = document.querySelectorAll(".overflow-y-auto");
      scrollableList.forEach((scrollable) => {
        scrollable.scrollTo(0, 0);
      });
    },
  },
};
</script>

<style>
#watch-body {
  height: 100%;
  max-height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

#content-container {
  height: 100%;
  position: relative;
}

.vertical-button span.v-btn__content {
  flex-direction: column;
  justify-content: space-around;
}

.channel-section,
.comment-renderer {
  display: flex;
  align-items: center;
  padding: 12px;
}

.channel-section #details,
.comment-renderer .comment-count {
  flex-grow: 1;
  display: flex;
  align-items: center;
  min-width: 0;
}

.channel-section .channel-byline {
  min-width: 0;
}

.channel-section .avatar-thumbnail {
  border-radius: 50%;
  width: 35px;
  height: 35px;
}

.channel-section .channel-name {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.keep-spaces {
  white-space: pre-wrap;
}
</style>
