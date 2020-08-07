<template>
<v-app>
  <v-app>

    <v-navigation-drawer
      v-model="drawer"
      app
      clipped
    >
      <v-list dense> 
        <v-list-item link v-for="song in songs" :key="song.src" class="song" @click="play(song)">
          <v-list-item-avatar>
            <img class="cover" :src="song.cover" />
           </v-list-item-avatar>

           <v-list-item-content>
            <v-list-item-title v-text="song.title"></v-list-item-title>
            <v-list-item-subtitle v-text="song.artist"></v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>


    <v-app-bar app clipped-left>
      <v-toolbar-title>pez potify</v-toolbar-title>
      <v-spacer></v-spacer>
            <v-switch
              v-model="$vuetify.theme.dark"
              hide-details
              inset
              dense>
            </v-switch>
            <v-icon size="40" left dark >mdi-lightbulb-on-outline</v-icon>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <br>
        <br>
        <br>
        <div class="cover-wrapper">
          <img v-bind:class="coverObject" :src="current.cover" width="30%" height="30%"/>
        </div>
      </v-container>
    </v-main>
      <v-footer app padless="true" height="100">
        <v-container fluid>
          <v-row no-gutters>
            <v-col md="auto">
              <p class="start">{{ currentlyTimer }}</p>
            </v-col><v-col>
              <KProgress
                :show-text="false"
                class="progress-bar-wrapper"
                v-bind:percent="current.percent"
                :color="['#33ACFF']"
                line-height="7"
              />
            </v-col>
          <v-col md="auto">
            <p class="end">{{ current.totalTimer }}</p>
          </v-col>
        </v-row>

          <v-row no-gutters>
            <v-col>
              <h2 class="song-title">
                {{ current.title }}
              </h2>
              <p class="artist">{{ current.artist }}</p>
            </v-col>
            <v-col>
              <div class="controls">
                <v-btn class="mx-2 prev" fab color="secondary" @click="prev" v-if="songs.length > 1">
                  <v-icon size="40">mdi-skip-backward</v-icon>
                </v-btn>
                <v-btn class="mx-2 play" fab color="primary" v-if="!isPlaying" @click="play">
                  <v-icon size="40">mdi-play-circle</v-icon>
                </v-btn>
                <v-btn class="mx-2" fab color="primary"  v-else @click="pause">
                  <v-icon size="40">mdi-pause-circle</v-icon>
                </v-btn>
                <v-btn class="mx-2" fab color="secondary" @click="next" v-if="songs.length > 1">
                  <v-icon size="40">mdi-skip-forward</v-icon>
                </v-btn>
              </div>
            </v-col><v-col>
            </v-col>
          </v-row>
        </v-container>
      </v-footer>
    </v-app>
  </v-app>
</template>

<script>
import KProgress from "k-progress";

import { formatTimer } from "@/helpers/timer";
import {  threatSongs } from "@/helpers/utils";
import songs from "@/mocks/songs";

export default {
  components: { KProgress },
  name: "App",
  data: () => ({

      drawers: ['Default (no property)', 'Permanent', 'Temporary'],
      current: {},
      coverObject: { cover: true, animated: false },
      index: 0,
      isPlaying: false,
      currentlyTimer: "00:00",
      songs: songs,
      player: new Audio()

  }),
  methods: {
    listenersWhenPlay() {
      this.player.addEventListener("timeupdate", () => {
        var playerTimer = this.player.currentTime;

        this.currentlyTimer = formatTimer(playerTimer);
        this.current.percent = (playerTimer * 100) / this.current.seconds;
        this.current.isPlaying = true;
      });
      this.player.addEventListener(
        "ended",
        function() {
          this.next();
        }.bind(this)
      );
    },
    setCover() {
      this.coverObject.animated = true;

      setTimeout(() => {
        this.coverObject.animated = false;
      }, 1000);
    },
    setCurrentSong() {
      this.current = this.songs[this.index];
      this.play(this.current);
      this.setCover();
    },
    play(song) {
      if (typeof song.src !== "undefined") {
        this.current.isPlaying = false;
        this.index = this.songs.indexOf(this.current);
        this.current = song;
        this.player.src = this.current.src;
      }

      this.player.play();
      this.isPlaying = true;

      this.setCover();
      this.listenersWhenPlay();
    },
    pause() {
      this.player.pause();
      this.isPlaying = false;
    },
    next() {
      this.current.isPlaying = false;
      this.index = this.songs.indexOf(this.current);
      this.index++;
      if (this.index > this.songs.length - 1) {
        this.index = 0;
      }
      this.setCurrentSong();
    },
    prev() {
      this.current.isPlaying = false;
      this.index = this.songs.indexOf(this.current);
      this.index--;
      if (this.index < 0) {
        this.index = this.songs.length - 1;
      }
      this.setCurrentSong();
    }
  },
  mounted() {
    this.songs = threatSongs(this.songs);
    this.current = this.songs[this.index];
    this.player.src = this.current.src;
  },
  created () {
  this.$vuetify.theme.dark = true
  },
};
</script>

<style lang="css">
  .controls {
    display: flex;
    justify-content: center;
  }
  .progress-bar-wrapper{
    display: flex;
    justify-content: center;
  }

.start{
  position: fixed;
  bottom: 6%;
  }
.end{
  position: fixed;
  bottom: 6%;
  right: 1%;
  }
.song-title{
  position: fixed;
  bottom: 4%;
}
.artist{
  position: fixed;
  bottom: -1%;
}
.cover-wrapper{
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
