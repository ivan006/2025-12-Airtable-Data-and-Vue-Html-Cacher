<template>
    <q-page class="q-pa-md">

        <!-- Title -->
        <div class="text-h5 q-mb-md">
            Data Cache — Page Binder
        </div>

        <!-- Description -->
        <div class="text-body2 q-mb-lg text-grey-7">
            Compile a full Airtable dataset into a single cached JSON file by
            fetching all paginated pages and optionally caching attachments.
        </div>

        <!-- Configuration -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Configuration
                </div>

                <q-input label="Airtable API URL" outlined dense
                    placeholder="https://api.airtable.com/v0/appXXXX/TableName" class="q-mb-md" />

                <q-input label="Attachment Path (optional)" outlined dense
                    placeholder="Attachments[0].thumbnails.large.url" />
            </q-card-section>

            <q-card-actions align="right">
                <q-btn label="Start Compilation" color="primary" unelevated disable />
            </q-card-actions>
        </q-card>

        <!-- Progress -->
        <q-card flat bordered class="q-mb-md">
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Compilation Status
                </div>

                <div class="q-mb-sm">
                    Pages fetched: <strong>— / —</strong>
                </div>

                <div class="q-mb-sm">
                    Attachments cached: <strong>— / —</strong>
                </div>

                <div>
                    Elapsed time: <strong>—</strong>
                </div>
            </q-card-section>
        </q-card>

        <!-- Existing Caches -->
        <q-card flat bordered>
            <q-card-section>
                <div class="text-subtitle1 q-mb-md">
                    Existing Bound Caches
                </div>

                <q-table flat bordered dense :rows="[]" :columns="columns" row-key="file"
                    no-data-label="No bound caches yet" />
            </q-card-section>
        </q-card>

    </q-page>
</template>

<script>
export default {
    name: 'DataCacheBinder',

    data() {
        return {
            columns: [
                {
                    name: 'file',
                    label: 'File',
                    field: 'file',
                    align: 'left'
                },
                {
                    name: 'records',
                    label: 'Records',
                    field: 'records',
                    align: 'right'
                },
                {
                    name: 'size',
                    label: 'Size',
                    field: 'size',
                    align: 'right'
                },
                {
                    name: 'actions',
                    label: 'Actions',
                    field: 'actions',
                    align: 'right'
                }
            ]
        }
    }
}
</script>
