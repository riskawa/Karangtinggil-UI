<template>
    <v-col cols="12" class="d-flex flex-column align-center justify-space-between">
        <h1>Daftar Permohonan Surat Keterangan</h1>
        <v-col cols="6">
            <v-card class="mb-2">
                <v-card-text>
                    <v-simple-table>
                        <template v-slot:default>
                            <tbody>
                                <tr>
                                    <td>Pilih Berdasarkan</td>
                                    <td>:</td>
                                    <td>
                                        <v-select @change="gantiSort($event)" :items="items" item-text="text"
                                            item-value="value" v-model="pilih" label="Pilih Berdasarkan" solo>
                                        </v-select>
                                    </td>
                                </tr>
                                <tr v-if="pilih == 1">
                                    <td>Tanggal</td>
                                    <td>:</td>
                                    <td>
                                        <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
                                            transition="scale-transition" offset-y min-width="auto">
                                            <template v-slot:activator="{ on, attrs }">
                                                <v-text-field v-model="tanggal" label="Tanggal"
                                                    prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on" solo>
                                                </v-text-field>
                                            </template>
                                            <v-date-picker v-model="tanggal" @input="menu = false">
                                            </v-date-picker>
                                        </v-menu>
                                    </td>
                                </tr>
                                <tr v-else-if="pilih == 2">
                                    <td>Bulan</td>
                                    <td>:</td>
                                    <td>
                                        <v-select :items="months" item-text="text" item-value="value" v-model="bulan"
                                            solo label="Bulan"></v-select>
                                    </td>
                                </tr>
                                <tr v-else-if="pilih == 3">
                                    <td>Tahun</td>
                                    <td>:</td>
                                    <td class="">
                                        <v-select :items="years" v-model="tahun" label="Tahun" solo>
                                        </v-select>
                                    </td>
                                </tr>
                            </tbody>
                        </template>
                    </v-simple-table>
                </v-card-text>
            </v-card>
            <v-btn color="primary" @click="filterSurat">Tampilkan</v-btn>
            <v-btn color="secondary" @click="reset">Reset</v-btn>
        </v-col>
        <v-spacer></v-spacer>
        <v-col cols="10" class="d-flex justify-end">
            <v-btn color="primary" @click="cetak">
                <vueJsonExcelUmd class="btn btn-default" :data="json_data" :fields="json_fields"
                    worksheet="My Worksheet" :name="filename">CETAK</vueJsonExcelUmd>
            </v-btn>
        </v-col>
        <v-col cols="10">
            <v-card>
                <v-card-text>
                    <v-col cols="12">
                        <v-data-table :headers="headers" :items="surat_keterangans" disable-pagination
                            :options.sync="options" :server-items-length="totalSuratKeterangans" :loading="loading"
                            class="elevation-1 mb-2" :hide-default-footer="true">
                        </v-data-table>
                    </v-col>
                    <v-col cols="12">
                        <v-row class="d-flex justify-between align-center">
                            Tampilkan
                            <v-col cols="1" sm="1">
                                <v-select v-model="pageSize" :items="pageSizes" @change="handlePageSizeChange">
                                </v-select>
                            </v-col>
                            <v-col cols="1">baris</v-col>
                            <v-col cols="8" sm="8" class="d-flex justify-end">
                                <v-pagination v-model="page" :length="totalPages" total-visible="7"
                                    next-icon="mdi-menu-right" prev-icon="mdi-menu-left" @input="handlePageChange">
                                </v-pagination>
                            </v-col>
                        </v-row>
                    </v-col>
                </v-card-text>
            </v-card>
        </v-col>
    </v-col>
</template>

<script>
import { DateTime } from 'luxon'
import vueJsonExcelUmd from 'vue-json-excel';

export default {
    layout: 'admin',
    mounted() {
        this.getSuratKeteranganData()
        const date = new Date()
        const year = date.getFullYear()
        this.years = this.getYear(year - 50)
        this.generateFileName()
    },
    components: {
        vueJsonExcelUmd
    },
    data() {
        return {
            filename: '',
            items: [
                {
                    value: 1,
                    text: "Per Tanggal"
                },
                {
                    value: 2,
                    text: "Per Bulan"
                },
                {
                    value: 3,
                    text: "Per Tahun"
                },
            ],
            json_fields: {
                No: 'no',
                'Tanggal': 'tanggal',
                'NIK': 'nik',
                'Nama': 'nama',
                'Keterangan': 'keterangan',
                'Status': 'status'
            },
            json_data: [],
            json_meta: [
                [
                    {
                        key: 'charset',
                        value: 'utf-8',
                    },
                ],
            ],
            totalSuratKeterangans: 0,
            surat_keterangans: [],
            loading: true,
            options: {},
            search: '',
            headers: [
                {
                    text: 'No',
                    align: 'start',
                    sortable: false,
                    value: 'no',
                },
                { text: 'Tanggal', value: 'tanggal' },
                { text: 'NIK', value: 'nik' },
                { text: 'Nama Lengkap', value: 'nama' },
                { text: 'Keterangan', value: 'keterangan' },
                { text: 'Status', value: 'status' },
            ],
            pageSize: 5,
            pageSizes: [5, 10, 20, 50, 100],
            page: 1,
            totalPages: 0,
            tanggal: '',
            months: [
                { text: 'Januari', value: 1 },
                { text: 'Februari', value: 2 },
                { text: 'Maret', value: 3 },
                { text: 'April', value: 4 },
                { text: 'Mei', value: 5 },
                { text: 'Juni', value: 6 },
                { text: 'Juli', value: 7 },
                { text: 'Agustus', value: 8 },
                { text: 'September', value: 9 },
                { text: 'Oktober', value: 10 },
                { text: 'November', value: 11 },
                { text: 'Desember', value: 12 },
            ],
            years: [],
            bulan: 0,
            tahun: 0,
            jenis_kelamin: ['Laki-laki', 'Perempuan'],
            menu: false,
            pilih: 0,
            filterType: 0,
            filter: '',
        }
    },
    watch: {
        options: {
            handler() {
                this.getSuratKeteranganData()
            },
            deep: true,
        },
        search(value) {
            this.search = value
            this.page = 1
            this.getSuratKeteranganData()
        }
    },
    methods: {
        gantiSort(evt) {
            this.pilih = evt
        },
        generateFileName() {
            this.filename = `${DateTime.now().toISODate()}_SuratKeterangan.xls`
        },
        async fetchData() {
            const response = await this.$axios.$get(
                'https://jsonplaceholder.typicode.com/users'
            )
            console.log(response)
            return response
        },
        startDownload() {
            alert('show loading')
        },
        finishDownload() {
            alert('hide loading')
        },
        getYear(startYear) {
            var currentYear = new Date().getFullYear(), years = [];
            startYear = startYear || 1980;
            while (currentYear >= startYear) {
                years.push(currentYear--);
            }
            return years
        },
        async getSuratKeteranganData() {
            this.loading = true
            await this.$axios.$get('http://localhost:3333/surat-keterangan', {
                params: {
                    limit: this.pageSize,
                    page: this.page - 1,
                    search: this.search,
                    filter: this.filter,
                    filterType: this.filterType
                }
            }).then(res => {
                this.getDisplaySuratKeterangan(res)
                this.totalSuratKeterangans = res.meta.total
                this.loading = false
                this.totalPages = res.meta.last_page
            })
        },
        getDisplaySuratKeterangan(data) {
            this.json_data = data.data.map((surat_keterangan, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = DateTime.fromISO(surat_keterangan.created_at).toFormat('yyyy-LL-dd')
                const status = (surat_keterangan.status == 1) ? 'Disetujui' : (surat_keterangan.status == 2) ? 'Surat Belum diambil' : (surat_keterangan.status == 3) ? 'Surat diambil' : 'Belum Diproses'
                return {
                    no: no,
                    tanggal: tgl,
                    nik: surat_keterangan.nik,
                    nama: surat_keterangan.nama,
                    keterangan: surat_keterangan.keterangan,
                    status: status,
                };
            })
            this.surat_keterangans = data.data.map((surat_keterangan, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = DateTime.fromISO(surat_keterangan.created_at).toFormat('yyyy-LL-dd')
                const status = (surat_keterangan.status == 1) ? 'Disetujui' : (surat_keterangan.status == 2) ? 'Surat Belum diambil' : (surat_keterangan.status == 3) ? 'Surat diambil' : 'Belum Diproses'
                return {
                    no: no,
                    id: surat_keterangan.id,
                    nama: surat_keterangan.nama,
                    status: status,
                    keterangan: surat_keterangan.keterangan,
                    nik: surat_keterangan.nik,
                    tanggal: tgl
                };
            })
        },
        handlePageChange(value) {
            this.page = value;
            this.getSuratKeteranganData();
        },
        handlePageSizeChange(size) {
            this.pageSize = size;
            this.page = 1;
            this.getSuratKeteranganData();
        },
        cetak() {
            console.log(this.filename)
        },
        filterSurat() {
            switch (this.pilih) {
                case 1:
                    this.filter = this.tanggal
                    this.filterType = this.pilih
                    this.page = 1
                    this.getSuratKeteranganData()
                    break
                case 2:
                    this.filter = this.bulan
                    this.filterType = this.pilih
                    this.page = 1
                    this.getSuratKeteranganData()
                    break
                case 3:
                    this.filter = this.tahun
                    this.filterType = this.pilih
                    this.page = 1
                    this.getSuratKeteranganData()
                    break
                default:
                    break
            }
        },
        reset() {
            this.filter = ''
            this.pilih = 0
            this.filterType = 0
            this.tanggal = ''
            this.bulan = 0
            this.tahun = 0
            this.getSuratKeteranganData()
        },
    },

}
</script>