<template>
  <div class="playerArea">
    <div class="player">
      <!-- thumbnail -->
      <img v-if="playList.length !== 0" :src="thumbnailURL" class="" />
      <div v-else class="notImg"></div>

      <!-- title -->
      <AppMarquee is-play="play">
        <template #default>
          <div v-html="title" class="title"></div>
        </template>
      </AppMarquee>

      <!-- Controller -->
      <PlayerController></PlayerController>

      <br />

      <!-- time -->
      <div class="timeLine">
        <AppProgressBar
          v-model="currentTimePercent"
          progress-type="time"
          @input="onInput"
        />
        <div class="timeLineText">
          <span>
            {{ startTime }}
          </span>
          <span>
            {{ lastTime }}
          </span>
        </div>
      </div>

      <!-- sound -->
      <div class="sound" @wheel.prevent>
        <PlayerSoundImg></PlayerSoundImg>

        <div>
          <AppProgressBar
            v-model="volume"
            progress-type="sound"
            @wheel="onMouseWheel"
          />
        </div>
      </div>
    </div>

    <!-- playlist -->
    <PlayerMusicList :music-list="playList"></PlayerMusicList>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";
import { useMusicControllerStore } from "@/stores/musicController";
import { useIndexedDBStore } from "@/stores/indexedDB";
import AppMarquee from "../common/AppMarquee.vue";
import { storeToRefs } from "pinia";
import PlayerMusicList from "./PlayerMusicList.vue";
import PlayerController from "./PlayerController.vue";
import AppProgressBar from "../common/AppProgressBar.vue";
import { useSoundControllerStore } from "@/stores/soundController";
import PlayerSoundImg from "./PlayerSoundImg.vue";
import timeFormat from "@/common/PlayTimeFormat";

const dbStore = useIndexedDBStore();
const soundStore = useSoundControllerStore();
const { volume, isMute } = storeToRefs(soundStore);
const musicStore = useMusicControllerStore();
const { currentIndex, playList, currentTime, duration, currentTimePercent } =
  storeToRefs(musicStore);
const title = ref("");

const thumbnailURL = computed(
  () =>
    `https://i.ytimg.com/vi/${playList.value[currentIndex.value]}/hqdefault.jpg`
);

const checkPlaying = () => {
  if (playList.value.length === 0) {
    musicStore.inputPlayList();
    return;
  }
};
// watch(playList.value, (newPlayList, oldPlayList) => {
//   console.log();

//   if (newPlayList.length != oldPlayList.length) {
//     musicStore.inputPlayList();
//   }
// });
// favorites가 update가 되면 갱신해야할듯

const getPlaylist = async (item: string) => {
  title.value = await dbStore.getPlaylistTitle(item);
};

const onMouseWheel = ({ deltaY }: WheelEvent) => {
  let value;
  if (isMute.value) return;
  if (deltaY > 0) {
    // progress.value = Math.max(min, progress.value - 5);
    value = Math.max(0, volume.value - 5);
    console.log(value);
  } else {
    value = Math.min(100, volume.value + 5);
    console.log(value);
  }
  soundStore.setVolume(value);
};

const onInput = (e: Event) => {
  if (playList.value.length === 0) {
    return;
  }
  const value = Number((e.target as HTMLInputElement).value);
  musicStore.setCurrentTimeClick(value);
};

const startTime = computed(() => {
  const current = currentTime.value;
  const [hours, minutes, seconds] = timeFormat(current);
  if (hours === 0) {
    return `${minutes}:${seconds}`;
  } else {
    return `${hours}:${minutes}:${seconds}`;
  }
});
const lastTime = computed(() => {
  const current = duration.value;
  const [hours, minutes, seconds] = timeFormat(current);
  if (hours === 0) {
    return `${minutes}:${seconds}`;
  } else {
    return `${hours}:${minutes}:${seconds}`;
  }
});

watch(thumbnailURL, async () => {
  title.value = await dbStore.getPlaylistTitle(
    playList.value[currentIndex.value]
  );
});
onMounted(() => {
  checkPlaying();
  getPlaylist(playList.value[currentIndex.value]);
});
</script>

<style scoped lang="scss">
.playerArea {
  gap: 1.8%;
  display: flex;
  flex-wrap: wrap;
  width: 70rem;
  margin: 0 auto;
  height: 40rem;

  .player {
    width: 40%;
    border: 0.3rem solid $playListBorder;
    border-radius: 2%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 0 2rem;
    box-sizing: border-box;

    .notImg {
      width: 375px;
      height: 280px;
      background-image: linear-gradient(
        45deg,
        $footerColor,
        $skeletonsColor,
        $skeletonsColor2
      );
      border-radius: 2rem;
    }

    .timeLine {
      width: 100%;
      .timeLineText {
        display: flex;
        width: 100%;
        justify-content: space-between;
      }
    }
    .sound {
      width: 25%;
      align-self: flex-end;
      display: flex;
      justify-content: space-between;
      gap: 0.3rem;
      margin-top: 0.5rem;
      div {
        display: flex;
        align-items: center;
      }
    }

    .title {
      display: flex;
      align-items: center;
      white-space: nowrap;
      font-size: 1.7rem;
      font-weight: 500;
      width: 100%;
    }

    img {
      width: 100%;
      margin-bottom: 2rem;
    }
  }
}
</style>
