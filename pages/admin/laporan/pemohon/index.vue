<template>
    <v-col cols="12" class="d-flex flex-column align-center justify-space-between">
        <h1>Daftar Pemohon Surat</h1>
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
                                    <td>Tanggal Lahir</td>
                                    <td>:</td>
                                    <td>
                                        <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
                                            transition="scale-transition" offset-y min-width="auto">
                                            <template v-slot:activator="{ on, attrs }">
                                                <v-text-field v-model="tanggal_lahir" label="Tanggal Lahir"
                                                    prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on" solo>
                                                </v-text-field>
                                            </template>
                                            <v-date-picker v-model="tanggal_lahir" @input="menu = false">
                                            </v-date-picker>
                                        </v-menu>
                                    </td>
                                </tr>
                                <tr v-if="pilih == 4">
                                    <td>Tanggal Pendaftaran</td>
                                    <td>:</td>
                                    <td>
                                        <v-menu v-model="menu_daftar" :close-on-content-click="false" :nudge-right="40"
                                            transition="scale-transition" offset-y min-width="auto">
                                            <template v-slot:activator="{ on, attrs }">
                                                <v-text-field v-model="tanggal_daftar" label="Tanggal Daftar"
                                                    prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on" solo>
                                                </v-text-field>
                                            </template>
                                            <v-date-picker v-model="tanggal_daftar" @input="menu_daftar = false">
                                            </v-date-picker>
                                        </v-menu>
                                    </td>
                                </tr>
                                <tr v-else-if="pilih == 2">
                                    <td>Bulan Lahir</td>
                                    <td>:</td>
                                    <td>
                                        <v-select :items="months" item-text="text" item-value="value" v-model="bulan"
                                            solo label="Bulan"></v-select>
                                    </td>
                                </tr>
                                <tr v-else-if="pilih == 3">
                                    <td>Tahun Lahir</td>
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
                        <v-data-table :headers="headers" :items="pemohons" disable-pagination :options.sync="options"
                            :server-items-length="totalPemohons" :loading="loading" class="elevation-1 mb-2"
                            :hide-default-footer="true">
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
        this.getPemohonData()
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
                    text: "Per Tanggal Lahir"
                },
                {
                    value: 2,
                    text: "Per Bulan Lahir"
                },
                {
                    value: 3,
                    text: "Per Tahun Lahir"
                },
                {
                    value: 4,
                    text: "Per Tanggal Pendaftaran"
                },
            ],
            json_fields: {
                No: 'no',
                'Tanggal Lahir': 'tanggal_lahir',
                'Tanggal Daftar': 'tanggal_daftar',
                'NIK': 'nik',
                'Nama': 'nama',
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
            totalPemohons: 0,
            pemohons: [],
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
                {
                    text: 'Tanggal Pendaftaran', align: 'start', value: 'tanggal_daftar',
                },
                { text: 'NIK', value: 'nik' },
                { text: 'Nama Lengkap', value: 'nama' },
                { text: 'Tanggal Lahir', value: 'tanggal_lahir' },
            ],
            pageSize: 5,
            pageSizes: [5, 10, 20, 50, 100],
            page: 1,
            totalPages: 0,
            tanggal_lahir: '',
            tanggal_daftar: '',
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
            menu_daftar: false,
            pilih: 0,
            filterType: 0,
            filter: '',
        }
    },
    watch: {
        options: {
            handler() {
                this.getPemohonData()
            },
            deep: true,
        },
        search(value) {
            this.search = value
            this.page = 1
            this.getPemohonData()
        }
    },
    methods: {
        gantiSort(evt) {
            this.pilih = evt
        },
        generateFileName() {
            this.filename = `${DateTime.now().toISODate()}_Pemohon.xls`
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
        async getPemohonData() {
            this.loading = true
            await this.$axios.$get('/pemohon', {
                params: {
                    limit: this.pageSize,
                    page: this.page - 1,
                    search: this.search,
                    filter: this.filter,
                    filterType: this.filterType
                }
            }).then(res => {
                this.getDisplayPemohon(res)
                this.totalPemohons = res.meta.total
                this.loading = false
                this.totalPages = res.meta.last_page
            })
        },
        getDisplayPemohon(data) {
            this.json_data = data.data.map((pemohon, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = DateTime.fromISO(pemohon.tanggal_lahir).toFormat('yyyy-LL-dd')
                const tgl_daftar = DateTime.fromISO(pemohon.created_at).toFormat('yyyy-LL-dd')
                return {
                    no: no,
                    tanggal_lahir: tgl,
                    tanggal_daftar: tgl_daftar,
                    nik: pemohon.nik,
                    nama: pemohon.nama,
                };
            })
            this.pemohons = data.data.map((pemohon, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = (pemohon.tanggal_lahir == null) ? '' : DateTime.fromISO(pemohon.tanggal_lahir).toFormat('yyyy-LL-dd')
                const tgl_daftar = (pemohon.created_at == null) ? '' : DateTime.fromISO(pemohon.created_at).toFormat('yyyy-LL-dd')
                const status = (pemohon.status == 1) ? 'Disetujui' : (pemohon.status == 2) ? 'Surat Belum diambil' : (pemohon.status == 3) ? 'Surat diambil' : 'Belum Diproses'
                console.log(pemohon)
                return {
                    no: no,
                    id: pemohon.id,
                    nama: pemohon.nama,
                    nik: pemohon.nik,
                    tanggal_lahir: tgl,
                    tanggal_daftar: tgl_daftar,
                };
            })
        },
        handlePageChange(value) {
            this.page = value;
            this.getPemohonData();
        },
        handlePageSizeChange(size) {
            this.pageSize = size;
            this.page = 1;
            this.getPemohonData();
        },
        cetak() {
            console.log(this.filename)
        },
        filterSurat() {
            switch (this.pilih) {
                case 1:
                    this.filter = this.tanggal_lahir
                    this.filterType = this.pilih
                    this.page = 1
                    this.getPemohonData()
                    break
                case 2:
                    this.filter = this.bulan
                    this.filterType = this.pilih
                    this.page = 1
                    this.getPemohonData()
                    break
                case 3:
                    this.filter = this.tahun
                    this.filterType = this.pilih
                    this.page = 1
                    this.getPemohonData()
                    break
                case 4:
                    this.filter = this.tanggal_daftar
                    this.filterType = this.pilih
                    this.page = 1
                    this.getPemohonData()
                    break
                default:
                    break
            }
        },
        reset() {
            this.filter = ''
            this.pilih = 0
            this.filterType = 0
            this.tanggal_lahir = ''
            this.bulan = 0
            this.tahun = 0
            this.getPemohonData()
        },
    },
}
</script>