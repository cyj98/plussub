<template>
  <div class="knopf-group" style="display: flex; position: relative">
    <a class="knopf even active flat subtitle-dropdown-label sharp start" style="flex-grow: 1; margin-left: -7px" @click="showLanguageSelection = !showLanguageSelection"
      >Subtitle language: {{ prettySelected }}</a
    >
    <transition :name="showLanguageSelection ? 'menu-more' : 'menu-less'">
      <span v-if="showLanguageSelection"
        ><a class="knopf even active pale sharp subtitle-dropdown-chevron" style="width: 40px" @click="showLanguageSelection = !showLanguageSelection"><i class="fa fa-chevron-up fa-lg"></i></a
      ></span>
      <span v-else
        ><a class="knopf even active pale sharp subtitle-dropdown-chevron" style="width: 40px" @click="showLanguageSelection = !showLanguageSelection"><i class="fa fa-chevron-down fa-lg"></i></a
      ></span>
    </transition>
    <transition name="slide-down">
      <div v-show="showLanguageSelection" class="search-toolbar--container--language--accordion" style="position: absolute; top: 27px; margin-left: -40px">
        <input ref="inputRef" v-model="query" style="grid-area: search-bar" placeholder="Search language" type="text" @keydown.stop @keypress.stop />
        <div style="grid-area: content; overflow-y: auto">
          <a v-for="lang in languageList" :key="lang.iso639_2" class="knopf flat block" style="width: 100%" @click="select(lang)">{{ lang.iso639Name }} ({{ lang.iso639_2 }})</a>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup="props, { emit }" lang="ts">
import iso639List from '@/res/iso639List.json';
import { computed, ref } from 'vue';
import { capitalizeFirst } from '@/util/string';

declare const props: {
  selected: string;
};

export default {
  emits: ['update:selected']
};

export const query = ref('');
export const inputRef = ref<HTMLElement | null>(null);
export const showLanguageSelectionInternal = ref(false);
export const showLanguageSelection = computed<boolean>({
  get: () => showLanguageSelectionInternal.value,
  set: (val) => {
    showLanguageSelectionInternal.value = val;
    if (showLanguageSelectionInternal.value && inputRef.value) {
      inputRef.value.focus();
    }
  }
});

export const prettySelected = computed(() => capitalizeFirst(props.selected));
export const languageList = computed(() => {
  if (query.value === '') {
    return iso639List;
  }
  const lowerCaseQuery = query.value.toLowerCase();
  return iso639List.filter(({ iso639Name, iso639_2 }) => iso639Name.toLowerCase().startsWith(lowerCaseQuery) || iso639_2.toLowerCase().startsWith(lowerCaseQuery));
});

export const select = ({ iso639_2 }): void => {
  showLanguageSelection.value = false;
  emit('update:selected', iso639_2);
};
</script>

<style scoped>
/* plussub header */
.search-toolbar--container--language--accordion {
  --search-bar-size: 25px;
  --accordion-size: 250px;
  width: 400px;
  height: var(--accordion-size);
  max-height: 250px;
  background-color: var(--surface-color);
  color: var(--on-surface);
  box-shadow: var(--card-shadow);
  display: grid;
  grid-template-areas:
    'search-bar'
    'content';
  grid-template-rows: var(--search-bar-size) max(calc(var(--accordion-size) - var(--search-bar-size)));
}

.active.subtitle-dropdown-label:hover,
.active.subtitle-dropdown-label {
  --knopf-font-size: 1em;
}

.active.subtitle-dropdown-chevron,
.active.subtitle-dropdown-label {
  --knopf-text-color: var(--onPrimary);
}

.slide-down-enter-active {
  transition-duration: 0.3s;
  transition-timing-function: ease-in;
}

.slide-down-leave-active {
  transition-duration: 0.3s;
  transition-timing-function: cubic-bezier(0, 1, 0.5, 1);
}

.slide-down-enter-to,
.slide-down-leave-from {
  max-height: 100px;
  overflow: hidden;
}

.slide-down-enter-from,
.slide-down-leave-to {
  overflow: hidden;
  max-height: 0;
}
</style>
