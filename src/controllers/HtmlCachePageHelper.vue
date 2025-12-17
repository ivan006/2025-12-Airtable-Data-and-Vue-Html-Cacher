<template>
    <div style="max-height: 420px; overflow-y: auto">

        <!-- EACH SITEMAP SECTION -->
        <div v-for="section in sections" :key="section.key" class="q-mb-md">


            <!-- Section Header -->
            <q-item class="bg-grey-3">
                <q-item-section>
                    <strong>{{ section.label }}</strong>
                </q-item-section>
            </q-item>

            <!-- Select All (per sitemap) -->
            <q-item clickable @click="toggleAll(section)">
                <q-item-section>
                    <strong>
                        {{ allSelected(section) ? 'Unselect all' : 'Select all' }}
                    </strong>
                </q-item-section>
            </q-item>

            <q-separator />

            <!-- Groups of 10 -->
            <div v-for="(group, gIdx) in grouped(section.items)" :key="gIdx">
                <q-item clickable class="bg-grey-2" @click="toggleGroup(section, group)">
                    <q-item-section>
                        <strong>
                            Group {{ gIdx + 1 }}
                            ({{ groupSelectedCount(group) }}/{{ group.length }})
                        </strong>
                    </q-item-section>
                </q-item>

                <q-item v-for="item in group" :key="item.value" clickable @click="toggle(item.value)">
                    <q-item-section>
                        {{ item.label }}
                    </q-item-section>
                    <q-item-section side>
                        <q-checkbox :model-value="modelValue.includes(item.value)"
                            @update:model-value="toggle(item.value)" />
                    </q-item-section>
                </q-item>

                <q-separator />
            </div>

        </div>

    </div>
</template>

<script>
export default {
    name: 'HtmlCachePageHelper',

    props: {
        modelValue: {
            type: Array,
            required: true
        },

        options: {
            type: Array,
            default: () => []
        },

        sitemapUrl: {
            type: String,
            default: null
        }
    },

    emits: ['update:modelValue'],

    data() {
        return {
            sections: []
        }
    },

    async mounted() {
        if (this.sitemapUrl) {
            await this.loadSitemap(this.sitemapUrl)
        } else {
            this.sections = [{
                key: 'pages',
                label: 'Pages',
                items: this.options
            }]
        }
    },

    methods: {
        emit(vals) {
            this.$emit('update:modelValue', vals)
        },

        toggle(value) {
            const set = new Set(this.modelValue)
            set.has(value) ? set.delete(value) : set.add(value)
            this.emit([...set])
        },

        grouped(items) {
            const groups = []
            for (let i = 0; i < items.length; i += 10) {
                groups.push(items.slice(i, i + 10))
            }
            return groups
        },

        allSelected(section) {
            return section.items.length &&
                section.items.every(i => this.modelValue.includes(i.value))
        },

        toggleAll(section) {
            const set = new Set(this.modelValue)

            if (this.allSelected(section)) {
                section.items.forEach(i => set.delete(i.value))
            } else {
                section.items.forEach(i => set.add(i.value))
            }

            this.emit([...set])
        },

        toggleGroup(section, group) {
            const set = new Set(this.modelValue)
            const allInGroup = group.every(i => set.has(i.value))

            group.forEach(i =>
                allInGroup ? set.delete(i.value) : set.add(i.value)
            )

            this.emit([...set])
        },

        groupSelectedCount(group) {
            return group.filter(i => this.modelValue.includes(i.value)).length
        },

        async loadSitemap(url) {
            const xml = await fetch(url).then(r => r.text())
            const doc = new DOMParser().parseFromString(xml, 'text/xml')

            const sitemapNodes = [...doc.querySelectorAll('sitemap > loc')]

            // SITEMAP INDEX → MULTIPLE SECTIONS
            if (sitemapNodes.length) {
                for (const n of sitemapNodes) {
                    await this.loadSitemap(n.textContent)
                }
                return
            }

            // NORMAL URLSET
            const urls = [...doc.querySelectorAll('url > loc')]

            const items = urls.map((n, i) => {
                const path = new URL(n.textContent).pathname.replace(/\/$/, '')
                return {
                    label: `${i + 1}. ${path}/`,
                    value: path.replace(/^\/+/, '')
                }
            })

            this.sections.push({
                key: url,
                label: url.split('/').pop(),
                items
            })
        }
    }
}
</script>
