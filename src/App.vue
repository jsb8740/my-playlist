<template>
  <!-- <div v-if="initialized" id="container"> -->
  <div id="container">
    <TheHeader class="head"></TheHeader>
    <!-- :class="{ topZero: !isTopZero }" -->
    <TheView></TheView>
    <TheFooter class="foot"></TheFooter>

    <!-- display none -->
    <AppYoutube
      v-show="false"
      :key="reload"
      :video-id="videoID"
      @unstarted="unstarted"
      @ended="ended"
      @playing="playing"
      @paused="paused"
      @buffering="buffering"
      @stateChange="stateChange"
      @ready="ready"
    ></AppYoutube>
  </div>
</template>
<script setup lang="ts">
import AppYoutube from "@/components/common/AppYoutube.vue";
import TheHeader from "./components/layouts/TheHeader.vue";
import TheView from "./components/layouts/TheView.vue";
import TheFooter from "./components/layouts/TheFooter.vue";
import { onMounted, ref, watch } from "vue";
import { useIndexedDBStore } from "@/stores/indexedDB";
import { useMusicControllerStore } from "./stores/musicController";
import { useSoundControllerStore } from "./stores/soundController";
import { storeToRefs } from "pinia";
import usePressSpaceBar from "./composables/usePressSpaceBar";

// const isTopZero = ref(true);

// // const isScrolledIntoView = (el: Element) => {
// //   let rect = el.getBoundingClientRect();
// //   let elemTop = rect.top;
// //   let elemBottom = rect.bottom;

// //   console.log("elemTop", elemTop);
// //   console.log("elemBottom", elemBottom);

// //   // let isVisible = elemTop < window.innerHeight && elemBottom >= 0;
// //   let isVisible;
// //   if (elemTop === 0) {
// //     isVisible = false;
// //   } else {
// //     isVisible = true;
// //   }

// //   return isVisible;
// // };

// // const scroll = () => {
// //   window.onscroll = () => {
// //     let scrolledTo = document.querySelector("#container");

// //     if (scrolledTo && isScrolledIntoView(scrolledTo)) {
// //       console.log("scrolled");
// //       isTopZero.value = false;
// //     } else {
// //       isTopZero.value = true;
// //     }
// //   };
// // };

// const changePauseState = (e: KeyboardEvent) => {
//   musicStore.changePauseState();
// };

// usePressSpaceBar(changePauseState);
onMounted(() => {
  store.updateFavList();
  // scroll();
});

// const tables = [
//   { name: "favorites" },
//   { name: "playlists" },
//   { name: "sams" },
// ] as TableProperties[];
// const initializeIndexedDB = async () => {
//   await Connector.create({
//     database: "favorite_music",
//     tables,
//     version: 1,
//     sync: true,
//   });

//   let database = await Connector.instance.getDatabase();
// };

// youtube iframe
const currentTimeInterval = ref(0);
const store = useIndexedDBStore();
const musicStore = useMusicControllerStore();
const { playList, currentIndex, isPaused, currentTimePercent } =
  storeToRefs(musicStore);

const soundStore = useSoundControllerStore();
const { isMute, volume } = storeToRefs(soundStore);

const unstarted = (player: YT.Player) => {
  musicStore.setCurrentTime(0);
  console.log("unstarted");
};

const ended = (player: YT.Player) => {
  clearInterval(currentTimeInterval.value);

  // update video
  musicStore.nextIndex();
  const videoId = playList.value[currentIndex.value];
  player.loadVideoById(videoId);
  musicStore.setCurrentTime(0);
};
const playing = (player: YT.Player) => {
  if (!isPaused.value) {
    clearInterval(currentTimeInterval.value);
    currentTimeInterval.value = window.setInterval(() => {
      const currentTime = Math.ceil(player.getCurrentTime() as number);
      musicStore.setCurrentTime(currentTime);
      console.log(currentTime);
    }, 1000);
  }
};
const paused = (player: YT.Player) => {
  if (isPaused.value) {
    clearInterval(currentTimeInterval.value);
  }
};

const buffering = (player: YT.Player) => {};

const stateChange = (player: YT.Player) => {
  const fullTime = player.getDuration();
  musicStore.setDuration(fullTime);
  if (isPaused.value) {
    clearInterval(currentTimeInterval.value);
  }
};

const ready = (player: YT.Player) => {
  soundStore.volumeInit();
  player.setVolume(volume.value);
  if (isMute.value === true) {
    player.mute();
  } else {
    player.unMute();
  }
};

const videoID = ref(playList.value[currentIndex.value]);
const reload = ref(0);
watch(
  () => playList.value.length,
  (newLength) => {
    if (newLength === 1) {
      reload.value++;
    }
    videoID.value = playList.value[currentIndex.value];
  }
);
</script>

<style scoped lang="scss">
#container {
  width: 100%;
  height: 100%;
  background-color: black;
  display: grid;
  grid-template-rows: $headerHeight auto $footerHeight;
  .topZero {
    border-bottom: 2px solid $progressBackground;
  }
  .head {
    position: sticky;
    top: 0;
    z-index: 1;
    background-color: black;
  }
  .foot {
    position: sticky;
    bottom: 0;
    z-index: 1;
    background-color: $footerColor;
    // align-content: end;
    //
  }
}
</style>
