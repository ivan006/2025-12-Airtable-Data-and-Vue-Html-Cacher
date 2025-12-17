<template>
    <q-page class="q-pa-md">

        <!-- Title -->
        <div class="text-h5 q-mb-sm">
            HTML Cache — Static Page Freezer
        </div>

        <div class="text-body2 text-grey-7 q-mb-lg">
            Freeze Vue-rendered pages into static HTML files for SEO.
            Pages are rendered locally and saved as <code>/slug/index.html</code>.
        </div>

        <!-- Backup -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-sm">Backup</div>
                <q-btn label="Backup Root Index" color="secondary" unelevated :loading="loading" @click="backupRoot" />
            </q-card-section>
        </q-card>

        <!-- Tabs -->
        <q-card flat bordered class="q-mb-md">
            <q-tabs v-model="activeTab" dense>
                <q-tab name="pages" label="Pages" />
                <q-tab v-for="(s, i) in sitemaps" :key="i" :name="'sitemap-' + i" :label="s.split('/').pop()" />
            </q-tabs>

            <q-separator />

            <q-tab-panels v-model="activeTab" animated>

                <!-- Pages.json -->
                <q-tab-panel name="pages">
                    <HtmlCachePageHelper v-model="selected" :options="pageOptions" />
                </q-tab-panel>

                <!-- Sitemaps -->
                <q-tab-panel v-for="(s, i) in sitemaps" :key="i" :name="'sitemap-' + i">
                    <HtmlCachePageHelper v-model="selected" :sitemap-url="s" />
                </q-tab-panel>

            </q-tab-panels>
        </q-card>

        <!-- Actions -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="row q-gutter-sm">
                    <q-btn label="Cache Selected" color="positive" unelevated :loading="loading" @click="cachePages" />
                    <q-btn label="Delete Selected" color="negative" flat :loading="loading" @click="deletePages" />
                </div>
            </q-card-section>
        </q-card>

        <!-- Status -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-sm">Status</div>

                <div class="text-body2">
                    Target URL:
                    <strong>{{ currentUrl || '—' }}</strong>
                </div>

                <div class="text-body2 q-mt-sm" v-if="status">
                    {{ status }}
                </div>
            </q-card-section>
        </q-card>

        <!-- Preview -->
        <q-card flat bordered>
            <q-card-section>
                <iframe ref="iframe" class="full-width" style="height:500px;border:1px solid #ccc" />
            </q-card-section>
        </q-card>

    </q-page>
</template>

<script>
import HtmlCachePageHelper from './HtmlCachePageHelper.vue'

export default {
    name: 'HtmlCachePages',

    components: {
        HtmlCachePageHelper
    },

    data() {
        return {
            activeTab: 'pages',
            pageOptions: [{ label: 'Homepage', value: '' }],
            sitemaps: [],
            selected: [],
            loading: false,
            status: '',
            currentUrl: ''
        }
    },

    mounted() {
        this.loadPages()
        this.loadSitemaps()
    },

    methods: {
        cacheBase() {
            return import.meta.env.VITE_CACHE_BASE || ''
        },

        async loadPages() {
            try {
                const res = await fetch(`${this.cacheBase()}/html-cache/pages.json`)
                const pages = await res.json()

                pages.forEach(slug => {
                    const clean = slug.replace(/^\/+|\/+$/g, '')
                    this.pageOptions.push({
                        label: `/${clean}/`,
                        value: clean
                    })
                })
            } catch {
                this.status = '❌ Failed to load pages.json'
            }
        },

        async loadSitemaps() {
            try {
                const res = await fetch(`${this.cacheBase()}/html-cache/sitemaps.json`)
                const json = await res.json()
                this.sitemaps = json.sitemaps || []
            } catch {
                this.sitemaps = []
            }
        },

        async post(payload) {
            const body = new URLSearchParams(payload)
            return fetch(`${this.cacheBase()}/html-cache/index.php`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                body
            }).then(r => r.text())
        },

        async backupRoot() {
            this.loading = true
            this.status = await this.post({ action: 'backup', slug: '' })
            this.loading = false
        },

        async deletePages() {
            if (!this.selected.length) return
            this.loading = true

            for (const slug of this.selected) {
                this.status = await this.post({
                    action: 'delete',
                    slug
                })
            }

            this.loading = false
        },

        async cachePages() {
            if (!this.selected.length) return

            this.loading = true
            const iframe = this.$refs.iframe
            const base = window.location.origin

            for (const slug of this.selected) {
                const url = slug ? `${base}/${slug}/` : `${base}/`
                this.currentUrl = url
                iframe.src = url

                await new Promise(resolve => {
                    iframe.onload = async () => {
                        setTimeout(async () => {
                            try {
                                const html = iframe.contentDocument.documentElement.outerHTML
                                const encoded = btoa(unescape(encodeURIComponent(html)))
                                this.status = await this.post({
                                    action: 'save',
                                    slug,
                                    html: encoded
                                })
                            } catch {
                                this.status = '❌ Iframe access blocked (same-origin required)'
                            }
                            resolve()
                        }, 1000)
                    }
                })
            }

            this.loading = false
        }
    }
}
</script>
