<template>
    <div>
        <v-col cols="12" class="mb-4">
            <SktmTables />
        </v-col>
        <v-col cols="12" class="mg-4">
            <SkckTable />
        </v-col>
        <v-col cols="12" class="mg-4">
            <DomisiliTable />
        </v-col>
        <v-col cols="12" class="mg-4">
            <SkuTable />
        </v-col>
        <v-col cols="12" class="mg-4">
            <KehilanganKKVue />
        </v-col>
        <v-col cols="12" class="mg-4">
            <SuratKeteranganTable />
        </v-col>

    </div>
</template>
<script>
import SktmTablesVue from '~/components/sekdes/SktmTables.vue'
import SkckTableVue from '~/components/SkckTable.vue'
import DomisiliTable from '~/components/DomisiliTable.vue'
import SkuTable from '../../../components/SkuTable.vue'
import KehilanganKKVue from '~/components/KehilanganKK.vue'
import SuratKeteranganTable from '../../../components/SuratKeteranganTable.vue'

export default {
    layout: 'sekdes',
    components: {
        SktmTablesVue,
        SkckTableVue,
        DomisiliTable,
        SkuTable,
        KehilanganKKVue,
        SuratKeteranganTable
    },
    watch: {
    },
    methods: {
    },

}
</script>