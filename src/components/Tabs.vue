<template>
    <div>
        <div class=" tab-pane border-bottom create-tabs">
            <ul role="tablist" class=" nav nav-tabs border-0  d-flex mx-auto row ">
                <li
                    v-for="(tab, i) in tabs"
                    :key="i"
                    :class="{ 'is-active': tab.isActive, 'is-disabled': tab.isDisabled }"
                    class=" nav-item"
                    role="presentation"
                    v-show="tab.isVisible"
                >
                    <a v-html="tab.header"
                       :aria-controls="tab.hash"
                       :aria-selected="tab.isActive"
                       @click="selectTab(tab.hash, $event)"
                       :href="tab.hash"
                       class=" nav-link small py-md-3 py-sm-2 py-1 border-0 mx-2"
                       :class="{'show active': tab.isActive}"
                       role="tab"
                    ></a>
                </li>
            </ul>

        </div>
        <div class="tab-content">
            <slot/>
        </div>

    </div>

</template>

<script>
    import expiringStorage from '../expiringStorage';

    export default {
        props: {
            cacheLifetime: {
                default: 5,
            },
            options: {
                type: Object,
                required: false,
                default: () => ({
                    useUrlFragment: true,
                }),
            },
        },

        data: () => ({
            tabs: [],
            activeTabHash: '',
        }),

        computed: {
            storageKey() {
                return `vue-tabs-component.cache.${window.location.host}${window.location.pathname}`;
            },
        },

        created() {
            this.tabs = this.$children;
        },

        mounted() {
            window.addEventListener('hashchange', () => this.selectTab(window.location.hash));

            if (this.findTab(window.location.hash)) {
                this.selectTab(window.location.hash);
                return;
            }

            const previousSelectedTabHash = expiringStorage.get(this.storageKey);

            if (this.findTab(previousSelectedTabHash)) {
                this.selectTab(previousSelectedTabHash);
                return;
            }

            if (this.tabs.length) {
                this.selectTab(this.tabs[0].hash);
            }
        },

        methods: {
            findTab(hash) {
                return this.tabs.find(tab => tab.hash === hash);
            },

            selectTab(selectedTabHash, event) {
                // See if we should store the hash in the url fragment.
                if (event && !this.options.useUrlFragment) {
                    event.preventDefault();
                }

                const selectedTab = this.findTab(selectedTabHash);

                if (! selectedTab) {
                    return;
                }

                if (selectedTab.isDisabled) {
                    return;
                }

                this.tabs.forEach(tab => {
                    tab.isActive = (tab.hash === selectedTab.hash);
                });

                this.$emit('changed', { tab: selectedTab });

                this.activeTabHash = selectedTab.hash;

                expiringStorage.set(this.storageKey, selectedTab.hash, this.cacheLifetime);
            },

            setTabVisible(hash, visible) {
                const tab = this.findTab(hash);

                if (! tab) {
                    return;
                }

                tab.isVisible = visible;

                if (tab.isActive) {
                    // If tab is active, set a different one as active.
                    tab.isActive = visible;

                    this.tabs.every((tab, index, array) => {
                        if (tab.isVisible) {
                            tab.isActive = true;

                            return false;
                        }

                        return true;
                    });
                }
            },
        },
    };
</script>
