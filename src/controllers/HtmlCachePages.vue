<template>
    <q-page class="q-pa-md">

        <!-- Title -->
        <div class="text-h5 q-mb-md">
            HTML Cache — Page List
        </div>

        <!-- Description -->
        <div class="text-body2 text-grey-7 q-mb-lg">
            Generate and manage static HTML snapshots for selected pages.
            Pages are rendered via same-origin iframe and saved as
            <code>/slug/index.html</code>.
        </div>

        <!-- Safety / Backup -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-sm">
                    Root Index Safety
                </div>

                <div class="text-body2 text-grey-7 q-mb-md">
                    Backup the root <code>/index.html</code> before caching pages.
                    This allows restoring the homepage later.
                </div>

                <q-btn label="Backup Root Index" color="secondary" outline disable />
            </q-card-section>
        </q-card>

        <!-- Page Selection -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Pages
                </div>

                <div class="q-mb-sm">
                    <q-checkbox v-model="dummy" label="Homepage (/)" disable />
                </div>

                <div v-for="(page, idx) in pages" :key="idx" class="q-mb-sm">
                    <q-checkbox v-model="dummy" :label="`/${page}/`" disable />
                </div>

                <div class="text-caption text-grey-6 q-mt-md">
                    Pages are loaded from <code>pages.json</code>.
                    Homepage is always available.
                </div>
            </q-card-section>
        </q-card>

        <!-- Actions -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Actions
                </div>

                <q-btn-group>
                    <q-btn label="Cache Selected" color="primary" unelevated disable />
                    <q-btn label="Delete Selected" color="negative" outline disable />
                </q-btn-group>

                <div class="text-caption text-grey-6 q-mt-sm">
                    Homepage restore uses the backup if available.
                    Other pages are written or removed as folders.
                </div>
            </q-card-section>
        </q-card>

        <!-- Current Target -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Current Processing Target
                </div>

                <div class="text-body2 q-mb-sm">
                    URL:
                </div>

                <div class="text-monospace bg-grey-2 q-pa-sm">
                    —
                </div>

                <div class="q-mt-md bg-grey-1" style="height: 300px;">
                    <div class="flex flex-center text-grey-6" style="height: 100%;">
                        Iframe preview will appear here
                    </div>
                </div>
            </q-card-section>
        </q-card>

        <!-- Activity Log -->
        <q-card flat bordered>
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Activity Log
                </div>

                <div class="text-body2 text-grey-6">
                    No actions yet.
                </div>
            </q-card-section>
        </q-card>

    </q-page>
</template>

<script>
export default {
    name: 'HtmlCachePages',

    data() {
        return {
            dummy: false,
            pages: [
                'services',
                'artists',
                'contact'
            ]
        }
    }
}
</script>
