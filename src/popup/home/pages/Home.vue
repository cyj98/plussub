<template>
  <PageLayout :content-transition-name="contentTransitionName">
    <template #toolbar>
      <div ref="draggableAreaRef" class="home-toolbar--container--content">
        <img :src="logo" alt="logo" style="grid-area: logo; height: 100%; width: 100%; object-fit: contain" />
        <div style="grid-area: buttons; display: flex; justify-content: flex-end">
          <a v-if="appState.state !== 'NONE'" class="knopf flat pill buttonOnPrimary" @click="toTranscript()">
            <!-- This icon comes from material design icons which is under Apache license -->
            <img :src="subtitleIcon" style="filter: invert(1)" />
          </a>
          <a class="knopf flat pill buttonOnPrimary" @click="close"><i class="fa fa-times fa-lg"></i></a>
        </div>
      </div>
    </template>
    <template #content>
      <div :class="{ 'home-content--container': appState.state !== 'NONE' }">
        <ResultFromSearch
          v-if="appState.state !== 'NONE' && appState.src === 'SEARCH'"
          style="grid-area: current-sub; margin-top: 20px"
          :state="appState.state"
          :search-state="subtitleSearchState"
          @remove="remove"
        >
          <template #settings>
            <Settings :parsed="subtitleState.parsed" :offset-time="subtitleState.offsetTime" @offset-time="setOffsetTime" />
          </template>
        </ResultFromSearch>
        <ResultFromFile
          v-else-if="appState.state !== 'NONE' && appState.src === 'FILE'"
          style="grid-area: current-sub; margin-top: 20px"
          :state="appState.state"
          :file-state="fileState"
          @remove="remove"
        >
          <template #settings>
            <Settings :parsed="subtitleState.parsed" :offset-time="subtitleState.offsetTime" @offset-time="setOffsetTime" />
          </template>
        </ResultFromFile>
        <PageVideos v-show="appState.state === 'NONE'" style="grid-area: videos" @selected-src="selectedSrc" />
      </div>
    </template>
  </PageLayout>
</template>

<script setup="props, {emit}" lang="ts">
import { ref } from 'vue';
import { useDraggableArea } from '@/composables';
import { reset } from '@/app/state';
import { setCurrentSelectedSrc, toSearch } from '@/navigation/state';

export { close } from '@/util/close';

declare const props: {
  contentTransitionName?: string;
};

export { default as logo } from '@/res/plussub128.png';
export { default as subtitleIcon } from '@/res/subtitles-24px.svg';
export { default as PageLayout } from '@/components/PageLayout';
export { default as ResultFromSearch } from './components/ResultFromSearch.vue';
export { default as ResultFromFile } from './components/ResultFromFile.vue';
export { default as PageVideos } from './components/PageVideos';
export { default as Settings } from './components/Settings.vue';
export { subtitleState, setOffsetTime } from '@/subtitle/state';
export { toTranscript } from '@/navigation/state';
export { appState } from '@/app/state';
export { subtitleSearchState } from '@/search/state';
export { fileState } from '@/file/state';

export const remove = (): void => {
  reset();
  setCurrentSelectedSrc(null);
};

export const selectedSrc = (src: string): void => {
  setCurrentSelectedSrc(src);
  toSearch();
};

export const draggableAreaRef = ref(null);
useDraggableArea({ draggableAreaRef });
</script>

<style scoped>
/* plussub header */
.home-toolbar--container--content {
  box-shadow: var(--toolbar-shadow);
  display: grid;
  grid-template-areas: 'logo buttons .';
  grid-template-rows: 40px;
  grid-template-columns: 64px 1fr 16px;
}

.home-content--container {
  overflow-y: scroll;
  width: 100%;
  height: 100%;
  display: grid;
  justify-content: center;
  grid-template-areas:
    '. current-sub .'
    '. videos .'
    '. debug .';
  grid-template-rows: auto auto auto;
  grid-template-columns: var(--content-lr-space) 1fr var(--content-lr-space);
  row-gap: 16px;
}
</style>
