<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { fas } from '@fortawesome/free-solid-svg-icons';
import { fab } from '@fortawesome/free-brands-svg-icons';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';

library.add(fas, fab);

export default {
    name: 'IconList',
    components: {
        FontAwesomeIcon,
    },
    data() {
        const iconsSet = new Set([
            ...Object.keys(fas).map(key => fas[key]).filter(icon => icon.iconName),
            ...Object.keys(fab).map(key => fab[key]).filter(icon => icon.iconName)
        ]);
        const icons = Array.from(iconsSet);
        return {
            search: '',
            icons: icons,
            searchTimeout: null,
            searchText: '',
            copiedIconName: ''
        };
    },
    computed: {
        filteredIcons() {
            const searchText = this.searchText.replace(/\s/g, '-').toLowerCase();
            return this.icons.filter(icon => icon.iconName.includes(searchText));
        }
    },
    watch: {
        search(newSearch) {
            if (this.searchTimeout) {
                clearTimeout(this.searchTimeout);
            }
            this.searchTimeout = setTimeout(() => {
                this.searchText = newSearch;
            }, 1000);
        }
    },
    methods: {
        resetSearch() {
            this.search = '';
            this.searchText = '';
            if (this.searchTimeout) {
                clearTimeout(this.searchTimeout);
            }
        },
        copyToClipboard(icon) {
            navigator.clipboard.writeText(icon.iconName).then(() => {
                this.copiedIconName = icon.iconName;
                setTimeout(() => {
                    this.copiedIconName = '';
                }, 2000);
            }).catch(err => {
                console.error('Failed to copy: ', err);
            });
        }
    }
};
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
