<template>
    <v-col cols="12">
        <v-btn color="secondary" to="/admin/permohonan-surat/surat-keterangan/tambah">
            <v-icon>mdi-plus-thick</v-icon>Tambah Surat Keterangan
        </v-btn>
        <v-card>
            <v-toolbar flat>
                <v-toolbar-title class="text-h5">
                    Data Permohonan Surat Keterangan
                </v-toolbar-title>

                <v-spacer></v-spacer>
                <v-spacer></v-spacer>

                <v-text-field solo append-icon="mdi-magnify" v-model="search" label="Cari kata kunci" single-line
                    hide-details>
                </v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="primary" to="/admin/permohonan-surat/">Kembali</v-btn>
            </v-toolbar>
            <v-divider></v-divider>
            <v-card-text>
                <v-col cols="12">
                    <v-data-table :headers="headers" :items="surat_keterangans" disable-pagination
                        :options.sync="options" :server-items-length="totalSKs" :loading="loading"
                        class="elevation-1 mb-2" :hide-default-footer="true">
                        <template v-slot:[`item.actions`]="{ item }">
                            <router-link :to="'/admin/permohonan-surat/surat-keterangan/' + item.id"
                                class="primary--text" style="text-decoration: none;">
                                Selengkapnya</router-link>
                            |
                            <!-- <router-link :to="'/admin/permohonan-surat/surat-keterangan/' + item.id + '/edit'" class="primary--text"
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
    layout: 'admin',
    data() {
        return {
            totalSKs: 0,
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
                { text: 'Tanggal Pendaftaran', value: 'tanggal' },
                { text: 'NIK', value: 'nik' },
                { text: 'Nama Lengkap', value: 'nama' },
                { text: 'Status', value: 'status' },
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
                this.getSKData()
            },
            deep: true,
        },
        search(value) {
            this.search = value
            this.page = 1
            this.getSKData()
        }
    },
    methods: {
        async getSKData() {
            this.loading = true
            await this.$axios.$get('/surat-keterangan', {
                params: {
                    limit: this.pageSize,
                    page: this.page - 1,
                    search: this.search
                }
            }).then(res => {
                this.getDisplaySK(res)
                this.totalSKs = res.meta.total
                this.loading = false
                this.totalPages = res.meta.last_page
            })
        },
        getDisplaySK(data) {
            this.surat_keterangans = data.data.map((surat_keterangan, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = DateTime.fromISO(surat_keterangan.created_at).toFormat('yyyy-LL-dd')
                const status = (surat_keterangan.status == 1) ? 'Disetujui' : (surat_keterangan.status == 2) ? 'Surat Belum diambil' : (surat_keterangan.status == 3) ? 'Surat diambil' : 'Belum Diproses'
                return {
                    no: no,
                    id: surat_keterangan.id,
                    nama: surat_keterangan.nama,
                    nik: surat_keterangan.nik,
                    status: status,
                    tanggal: tgl
                };
            })
        },
        handlePageChange(value) {
            this.page = value;
            this.getSKData();
        },
        handlePageSizeChange(size) {
            this.pageSize = size;
            this.page = 1;
            this.getSKData();
        },
        hapus(val) {
            const surat_keterangan = val
            this.$swal.fire({
                title: 'Peringatan?',
                text: "Apakah anda yakin untuk hapus data " + surat_keterangan.nama,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#459EED',
                cancelButtonColor: '#d33',
                showLoaderOnConfirm: true,
                confirmButtonText: 'Yes, delete it!',
                preConfirm: (hapus) => {
                    return this.$axios.$delete(`/surat-keterangan/${surat_keterangan.id}`)
                        .then(res => {
                            console.log(res)
                        })
                        .catch(err => {
                            this.$swal.fire('Gagal!', 'Gagal hapus data' + surat_keterangan.nama, 'error')
                            this.$swal.hideLoading()
                        })
                },
            }).then((result) => {
                this.$swal.showLoading()
                if (result.isConfirmed) {
                    this.$swal.fire(
                        'Sukses!',
                        'Berhasil hapus data ' + surat_keterangan.nama,
                        'success'
                    )
                    this.getSKData()
                }
            })
        }
    },
    mounted() {
        this.getSKData()
    }
}
</script>