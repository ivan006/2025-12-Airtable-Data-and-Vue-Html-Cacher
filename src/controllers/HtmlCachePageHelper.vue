<template>
    <div style="max-height: 420px; overflow-y: auto">

        <!-- Select All -->
        <q-item clickable @click="toggleAll">
            <q-item-section>
                <strong>
                    {{ allSelected ? 'Unselect all' : 'Select all' }}
                </strong>
            </q-item-section>
        </q-item>

        <q-separator />

        <!-- Groups of 10 -->
        <div v-for="(group, gIdx) in groupedItems" :key="gIdx">
            <q-item clickable class="bg-grey-2" @click="toggleGroup(group)">
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
            items: []
        }
    },

    computed: {
        groupedItems() {
            const groups = []
            for (let i = 0; i < this.items.length; i += 10) {
                groups.push(this.items.slice(i, i + 10))
            }
            return groups
        },

        allSelected() {
            return this.items.length &&
                this.items.every(i => this.modelValue.includes(i.value))
        }
    },

    async mounted() {
        if (this.sitemapUrl) {
            await this.loadSitemap(this.sitemapUrl)
        } else {
            this.items = this.options
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

        toggleAll() {
            this.allSelected
                ? this.emit([])
                : this.emit(this.items.map(i => i.value))
        },

        toggleGroup(group) {
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

            if (sitemapNodes.length) {
                for (const n of sitemapNodes) {
                    await this.loadSitemap(n.textContent)
                }
                return
            }

            const urls = [...doc.querySelectorAll('url > loc')]

            const parsed = urls.map((n, i) => {
                const path = new URL(n.textContent).pathname.replace(/\/$/, '')
                return {
                    label: `${i + 1}. ${path}/`,
                    value: path.replace(/^\/+/, '')
                }
            })

            this.items.push(...parsed)
        }
    }
}
</script>
