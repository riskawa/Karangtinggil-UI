<template>
    <v-card>
        <v-toolbar flat>
            <v-toolbar-title class="text-h5">
                Data Permohonan SKTM
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
                <v-data-table :headers="headers" :items="sktms" disable-pagination :options.sync="options"
                    :server-items-length="totalSKTMs" :loading="loading" class="elevation-1 mb-2"
                    :hide-default-footer="true">
                    <template v-slot:[`item.actions`]="{ item }">
                        <v-row>
                            <div v-if="item.statusCode == 0">
                                <a href="javascript:void(0)" class="primary--text" @click="setuju(item)"
                                    style="text-decoration: none;">Disetujui</a>
                            </div>
                            <div v-else-if="item.statusCode == 1">
                                <a href="javascript:void(0)" class="primary--text" @click="ambil(item)"
                                    style="text-decoration: none;">Ambil Surat</a>
                            </div>
                            |&nbsp;
                            <!-- <a :href="'/admin/permohonan-surat/sktm/print/' + item.id" target="_blank" class="primary--text"
                            style="text-decoration: none;">Cetak Surat <v-icon color="primary">
                                mdi-printer</v-icon></a> -->
                            <a href="javascript:void(0)" @click="cetak(item.id)" class="primary--text"
                                style="text-decoration: none;">Cetak Surat
                                <v-icon color="primary">
                                    mdi-printer</v-icon>
                            </a>
                        </v-row>
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
                        <v-pagination v-model="page" :length="totalPages" total-visible="7" next-icon="mdi-menu-right"
                            prev-icon="mdi-menu-left" @input="handlePageChange">
                        </v-pagination>
                    </v-col>
                </v-row>
            </v-col>
        </v-card-text>
    </v-card>
</template>


<script>
import { DateTime } from 'luxon'
import { jsPDF } from 'jspdf'
export default {
    name: 'SktmTable',
    data() {
        return {
            totalSKTMs: 0,
            sktms: [],
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
                this.getSKTMData()
            },
            deep: true,
        },
        search(value) {
            this.search = value
            this.page = 1
            this.getSKTMData()
        }
    },
    methods: {
        async getSKTMData() {
            this.loading = true
            await this.$axios.$get('/sktm', {
                params: {
                    limit: this.pageSize,
                    page: this.page - 1,
                    search: this.search
                }
            }).then(res => {
                this.getDisplaySKTM(res)
                this.totalSKTMs = res.meta.total
                this.loading = false
                this.totalPages = res.meta.last_page
            })
        },
        getDisplaySKTM(data) {
            this.sktms = data.data.map((sktm, i) => {
                let no = (data.meta.current_page - 1) * data.meta.per_page + 1 + i
                const tgl = DateTime.fromISO(sktm.created_at).toFormat('yyyy-LL-dd')
                const status = (sktm.status == 1) ? 'Disetujui' : (sktm.status == 2) ? 'Surat Sudah diambil' : 'Belum Diproses'
                return {
                    no: no,
                    id: sktm.id,
                    nama: sktm.nama,
                    statusCode: sktm.status,
                    status: status,
                    nik: sktm.nik,
                    tanggal: tgl
                };
            })
        },
        handlePageChange(value) {
            this.page = value;
            this.getSKTMData();
        },
        handlePageSizeChange(size) {
            this.pageSize = size;
            this.page = 1;
            this.getSKTMData();
        },
        setuju(val) {
            const sktm = val
            this.$swal.fire({
                title: 'Peringatan?',
                text: "Apakah anda yakin menyetujui permohonan SKTM " + sktm.nama,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#459EED',
                cancelButtonColor: '#d33',
                showLoaderOnConfirm: true,
                confirmButtonText: 'Yes, delete it!',
                preConfirm: (hapus) => {
                    const fd = new FormData()
                    fd.append('status', '1')
                    return this.$axios.$put(`/sktm/status/${sktm.id}`, fd)
                        .then(res => {
                            console.log(res)
                        })
                        .catch(err => {
                            this.$swal.fire('Gagal!', 'Gagal hapus data' + sktm.nama, 'error')
                            this.$swal.hideLoading()
                        })
                },
            }).then((result) => {
                this.$swal.showLoading()
                if (result.isConfirmed) {
                    const Toast = this.$swal.mixin({
                        toast: true,
                        position: 'top-end',
                        showConfirmButton: false,
                        showCloseButton: true,
                        background: '#7C9885',
                        color: 'white',
                        timer: 3000,
                        timerProgressBar: true,
                        didOpen: (toast) => {
                            toast.addEventListener('mouseenter', this.$swal.stopTimer)
                            toast.addEventListener('mouseleave', this.$swal.resumeTimer)
                        }
                    })
                    Toast.fire({
                        icon: 'success',
                        title: 'Sukses menyetujui permohonan'
                    })
                    this.getSKTMData()
                }
            })
        },
        ambil(val) {
            const sktm = val
            this.$swal.fire({
                title: 'Peringatan?',
                text: "Apakah anda yakin mengubah status permohonan SKTM " + sktm.nama + "menjadi sudah diambil?",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#459EED',
                cancelButtonColor: '#d33',
                showLoaderOnConfirm: true,
                confirmButtonText: 'Yes, delete it!',
                preConfirm: (hapus) => {
                    const fd = new FormData()
                    fd.append('status', '2')
                    return this.$axios.$put(`/sktm/status/${sktm.id}`, fd)
                        .then(res => {
                            console.log(res)
                        })
                        .catch(err => {
                            this.$swal.fire('Gagal!', 'Gagal hapus data' + sktm.nama, 'error')
                            this.$swal.hideLoading()
                        })
                },
            }).then((result) => {
                this.$swal.showLoading()
                if (result.isConfirmed) {
                    const Toast = this.$swal.mixin({
                        toast: true,
                        position: 'top-end',
                        showConfirmButton: false,
                        showCloseButton: true,
                        background: '#7C9885',
                        color: 'white',
                        timer: 3000,
                        timerProgressBar: true,
                        didOpen: (toast) => {
                            toast.addEventListener('mouseenter', this.$swal.stopTimer)
                            toast.addEventListener('mouseleave', this.$swal.resumeTimer)
                        }
                    })
                    Toast.fire({
                        icon: 'success',
                        title: 'Sukses menyetujui permohonan'
                    })
                    this.getSKTMData()
                }
            })
        },
        async cetak(id) {
            const data = await this.$axios.$get(`/sktm/${id}`)
            const doc = new jsPDF('p', 'mm', [330, 210])
            const tanggal = DateTime.now().toFormat('yyyy-LL-dd')
            doc.addImage("/logo.png", 'PNG', 10, 10, 35, 40)
            doc.setFont('times', '', 700)
            doc.setFontSize(14)
            doc.text('PEMERINTAH KABUPATEN LAMONGAN', 60, 15)
            doc.text('KECAMATAN PUCUK', 87, 23)
            doc.text('DESA KARANGTINGGIL', 83, 31)
            doc.text('Jalan Karya No 37 Karangtinggil 62257', 67, 39)
            doc.text('Email : pemdeskarangtinggil@gmail.com', 65, 47)
            doc.line(10, 52, 200, 52)
            doc.line(10, 53, 200, 53)
            doc.setFont('times', '', 700)
            doc.setFontSize(13)
            doc.text('SURAT KETERANGAN TIDAK MAMPU', 69, 62)
            doc.line(69, 62.5, 147, 62.5)
            doc.text(`Nomor : 470/${data.sktm[0].id}/413.309.06/${DateTime.now().year}`, 80, 67)
            doc.setFont('times', '', 400)
            doc.text('Yang Bertanda tangan di bawah ini :', 15, 78)
            doc.text('Nama', 20, 85)
            doc.text(':', 60, 85)
            doc.text('H. GATOT SUKOCO', 62, 85)
            doc.text('Alamat', 20, 91)
            doc.text(':', 60, 91)
            doc.text('Desa Karangtinggil, Kecamatan Pucuk, Kabupaten Lamongan', 62, 91)
            doc.text('Jabatan', 20, 97)
            doc.text(':', 60, 97)
            doc.text('Kepala Desa Karangtinggil, Kecamatan Pucuk, Kabupaten Lamongan', 62, 97)
            doc.text('Dengan ini menyatakan bahwa Surat Keterangan Miskin (SKM) atau Surat Keterangan Tidak', 20, 105)
            doc.text('Mampu (SKTM) yang saya buat untuk : ', 15, 111)
            doc.text('NIK', 20, 119)
            doc.text(':', 60, 119)
            doc.text(data.sktm[0].nik, 62, 119)
            doc.text('Nama', 20, 126)
            doc.text(':', 60, 126)
            doc.text(data.sktm[0].nama, 62, 126)
            doc.text('Tempat Lahir', 20, 133)
            doc.text(':', 60, 133)
            doc.text(data.sktm[0].tempat_lahir, 62, 133)
            doc.text('Tanggal Lahir', 20, 140)
            doc.text(':', 60, 140)
            doc.text(DateTime.fromISO(data.sktm[0].tanggal_lahir).toFormat('dd LLLL yyyy'), 62, 140)
            doc.text('Jenis Kelamin', 20, 147)
            doc.text(':', 60, 147)
            doc.text(data.sktm[0].jenis_kelamin, 62, 147)
            doc.text('Kewarganegaraan', 20, 154)
            doc.text(':', 60, 154)
            doc.text(data.sktm[0].kewarganegaraan, 62, 154)
            doc.text('Agama', 20, 161)
            doc.text(':', 60, 161)
            doc.text(data.sktm[0].agama, 62, 161)
            doc.text('Pekerjaan', 20, 168)
            doc.text(':', 60, 168)
            doc.text(data.sktm[0].pekerjaan, 62, 168)
            doc.text('Alamat', 20, 175)
            doc.text(':', 60, 175)
            doc.text(data.sktm[0].alamat, 62, 175)
            doc.text('Keterangan', 20, 182)
            doc.text(':', 60, 182)
            doc.text('1. ', 62, 182)
            doc.text("Orang tersebut adalah penduduk desa Karangtinggil, Kecamatan", 67, 182)
            doc.text("Pucuk, Kabupaten Lamongan", 67, 189)
            doc.text('2. ', 62, 196)
            doc.text("Bahwa yang bernama tersebut benar-benar dari keluarga", 67, 196)
            doc.text("Tidak Mampu", 67, 203)
            doc.text('Keperluan', 20, 210)
            doc.text(':', 60, 210)
            doc.text(data.sktm[0].keperluan, 62, 210)
            doc.text('Demikian Surat Keterangan ini kami buat dengan sebenarnya dan untuk dipergunakan', 20, 217)
            doc.text('sebagaimana mestinya.', 15, 223)
            doc.text(`Karangtinggil, ${tanggal}`, 135, 230)
            doc.text('Pemohon', 25, 237)
            doc.text('Kepala Desa Karangtinggil', 135, 237)
            doc.text(data.sktm[0].nama, 25, 267)
            doc.text('H. GATOT SUKOCO', 135, 267)
            const filename = `${tanggal} - SKTM - ${data.sktm[0].nama}`
            doc.setDocumentProperties({ title: filename, subject: filename })
            window.open(doc.output('bloburl'), '_blank')
            // doc.save()
        },
    },
    mounted() {
        this.getSKTMData()
    }
}
</script>