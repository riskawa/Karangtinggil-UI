<template>
    <v-col cols="12">
        <v-card>
            <v-toolbar flat>
                <v-toolbar-title class="text-h5">
                    Data Permohonan Domisili
                </v-toolbar-title>

                <v-spacer></v-spacer>
                <v-spacer></v-spacer>

                <v-text-field solo append-icon="mdi-magnify" v-model="search" label="Cari kata kunci" single-line
                    hide-details>
                </v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="primary" to="/sekdes/permohonan-surat/">Kembali</v-btn>
            </v-toolbar>
            <v-divider></v-divider>
            <v-card-text>
                <v-col cols="12">
                    <v-data-table :headers="headers" :items="domisilis" disable-pagination :options.sync="options"
                        :server-items-length="totalDomisilis" :loading="loading" class="elevation-1 mb-2"
                        :hide-default-footer="true">
                        <template v-slot:[`item.actions`]="{ item }">
                            <router-link :to="'/sekdes/permohonan-surat/domisili/' + item.id" class="primary--text"
                                style="text-decoration: none;">
                                Selengkapnya</router-link>
                            |
                            <!-- <router-link :to="'/admin/permohonan-surat/domisili/' + item.id + '/edit'" class="primary--text"
                                style="text-decoration: none;">
                                Edit</router-link>
                            | -->
                            <a href="javascript:void(0)" class="primary--text" @click="hapus(item)"
                                style="text-decoration: none;">Hapus</a>
                        </template>
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
</template>

<script>
import { DateTime } from 'luxon'

export default {
    layout: 'sekdes',
    data() {
        return {
            totalDomisilis: 0,
            domisilis: [],
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
                { text: 'Aksi', value: 'actions' },
            ],
            pageSize: 5,
            pageSizes: [5, 10, 20, 50, 100],
            page: 1,
            totalPages: 0,
        }
    },
    watch: {
        options: {
            handler() {
                // this.getDataFromApi()
                this.getDomisiliData()
            },
            deep: true,
        },
        search(value) {
            this.search = value
            this.page = 1
            this.getDomisiliData()
        }
    },
    methods: {
        async getDomisiliData() {
            this.loading = true
            await this.$axios.$get('http://localhost:3333/domisili', {
                params: {
                    limit: this.pageSize,
                    page: this.page - 1,
                    search: this.search
                }
            }).then(res => {
                this.getDisplayDomisili(res)
                this.totalDomisilis = res.meta.total
                this.loading = false
                this.totalPages = res.meta.last_page
            })
        },
        getDisplayDomisili(data) {
            this.domisilis = data.data.map((domisili, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = DateTime.fromISO(domisili.created_at).toFormat('yyyy-LL-dd')
                return {
                    no: no,
                    id: domisili.id,
                    nama: domisili.nama,
                    nik: domisili.nik,
                    tanggal: tgl
                };
            })
        },
        handlePageChange(value) {
            this.page = value;
            this.getDomisiliData();
        },
        handlePageSizeChange(size) {
            this.pageSize = size;
            this.page = 1;
            this.getDomisiliData();
        },
        hapus(val) {
            const domisili = val
            this.$swal.fire({
                title: 'Peringatan?',
                text: "Apakah anda yakin untuk hapus data " + domisili.nama,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#459EED',
                cancelButtonColor: '#d33',
                showLoaderOnConfirm: true,
                confirmButtonText: 'Yes, delete it!',
                preConfirm: (hapus) => {
                    return this.$axios.$delete(`http://localhost:3333/domisili/${domisili.id}`)
                        .then(res => {
                            console.log(res)
                        })
                        .catch(err => {
                            this.$swal.fire('Gagal!', 'Gagal hapus data' + domisili.nama, 'error')
                            this.$swal.hideLoading()
                        })
                },
            }).then((result) => {
                this.$swal.showLoading()
                if (result.isConfirmed) {
                    this.$swal.fire(
                        'Sukses!',
                        'Berhasil hapus data ' + domisili.nama,
                        'success'
                    )
                    this.getDomisiliData()
                }
            })
        }
    },
    mounted() {
        this.getDomisiliData()
    }
}
</script>