<template>
  <TheViewLayout>
    <template #title> Home </template>
    <template #default>
      <div class="contents">
        <PlayerArea></PlayerArea>
        <AppMarquee></AppMarquee>
      </div>
    </template>
  </TheViewLayout>
</template>

<script setup lang="ts">
import TheViewLayout from "@/components/layouts/TheViewLayout.vue";
import PlayerArea from "@/components/player/PlayerArea.vue";
import AppMarquee from "@/components/common/AppMarquee.vue";
</script>
<style scoped lang="scss">
.contents {
  width: 100%;
}
</style>
