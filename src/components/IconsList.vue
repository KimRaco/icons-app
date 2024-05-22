<script lang="ts">
import { defineComponent, ref, computed, watch } from 'vue';
import { library } from '@fortawesome/fontawesome-svg-core';
import { fas } from '@fortawesome/free-solid-svg-icons';
import { fab } from '@fortawesome/free-brands-svg-icons';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';

library.add(fas, fab);

interface Icon {
  prefix: string;
  iconName: string;
}

export default defineComponent({
  name: 'IconList',
  components: {
    FontAwesomeIcon,
  },
  setup() {
    const search = ref<string>('');
    const searchTimeout = ref<number | null>(null);
    const searchText = ref<string>('');
    const copiedIconName= ref<string>('');

    const iconsSet = new Set<Icon>([
      ...Object.keys(fas).map(key => fas[key]).filter((icon: Icon) => icon.iconName),
      ...Object.keys(fab).map(key => fab[key]).filter((icon: Icon) => icon.iconName)
    ]);
    const icons = Array.from(iconsSet);

    const filteredIcons = computed(() => {
      const searchTextValue = searchText.value.replace(/\s/g, '-').toLowerCase();
      return icons.filter((icon: Icon) => icon.iconName.includes(searchTextValue));
    });

    watch(search, (newSearch) => {
      if (searchTimeout.value) {
        clearTimeout(searchTimeout.value);
      }
      searchTimeout.value = window.setTimeout(() => {
        searchText.value = newSearch;
      }, 800);
    });

    const resetSearch = () => {
      search.value = '';
      searchText.value = '';
      if (searchTimeout.value) {
        clearTimeout(searchTimeout.value);
      }
    };

    const copyToClipboard = (icon: Icon) => {
      navigator.clipboard.writeText(icon.iconName).then(() => {
        copiedIconName.value = icon.iconName;
        setTimeout(() => {
          copiedIconName.value = '';
        }, 2000);
      }).catch(err => {
        console.error('Failed to copy: ', err);
      });
    };

    return {
      search,
      filteredIcons,
      resetSearch,
      copyToClipboard,
      copiedIconName
    };
  }
});
</script>

<template>
  <div class="text-center p-20 bg-gray-100 h-screen">
    <div class="sticky top-0 z-10 bg-gray-100">
      <div class="relative mb-5 inline-block">
        <input type="text" v-model="search" placeholder="Search for an icon..."
          class="flex h-10 w-full rounded-md border border-input text-sm file:border-0 file:bg-transparent file:text-sm file:font-medium focus-visible:ring-cyan-500 placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 p-3 text-gray-600 border-cyan-500 bg-gray-50 px-10" />
        <font-awesome-icon :icon="['fas', 'search']"
          class="absolute left-3 top-1/2 transform -translate-y-1/2 text-cyan-900" />
        <button @click="resetSearch" v-if="search !== ''"
          class="absolute right-3 top-1/2 transform -translate-y-1/2 text-cyan-900">
          <font-awesome-icon :icon="['fas', 'times']" />
        </button>
      </div>
    </div>
    <div class="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-6 gap-10 mt-5 overflow-y-auto"
      style="max-height: calc(100vh - 160px);">
      <div v-for="(icon, index) in filteredIcons" :key="icon.iconName + '-' + index"
        class="bg-white hover:bg-cyan-50 rounded-lg p-4 flex flex-col items-center justify-center my-2 cursor-pointer"
        @click="copyToClipboard(icon)">
        <font-awesome-icon :icon="[icon.prefix, icon.iconName]" size="2x" class="text-cyan-900" />
        <p class="mt-2 text-sm text-cyan-500">{{ icon.iconName }}</p>
        <p v-if="copiedIconName === icon.iconName" class="text-xs text-green-500 mt-1">Copied!</p>
      </div>
    </div>
  </div>
</template>
