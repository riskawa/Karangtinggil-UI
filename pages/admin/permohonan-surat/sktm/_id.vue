<template>
    <v-col cols="9" class="mx-auto">
        <v-row class="d-flex justify-start pa-3">
            <v-btn :to="'/admin/permohonan-surat/sktm'" color="primary">KEMBALI</v-btn>
        </v-row>
        <v-row>
            <v-col cols="12">
                <v-card elevation="4">
                    <v-card-title>Data SKTM</v-card-title>
                    <v-divider></v-divider>
                    <v-card-text>
                        <v-col cols="12">
                            <div>
                                <v-simple-table>
                                    <template v-slot:default>
                                        <tbody>
                                            <tr v-for="(item, index) in title" :key="item.field">
                                                <td class="text-h6">{{ item.name }} </td>
                                                <td class="text-h6">:</td>
                                                <td v-if="index == title.length - 1" class="text-h6"><a :href="kk_link"
                                                        target="_blank">Lihat KK</a></td>
                                                <td v-else-if="item.field === 'tanggal_lahir'" class="text-h6">{{
                                                        tanggal_lahir
                                                }}</td>
                                                <td v-else class="text-h6">{{ sktm[0][item.field] }}</td>
                                            </tr>
                                        </tbody>
                                    </template>
                                </v-simple-table>
                            </div>
                        </v-col>
                    </v-card-text>
                </v-card>
            </v-col>
        </v-row>
    </v-col>
</template>
<script>
export default {
    layout: 'admin',
    data() {
        return {
            title: [
                { field: 'nik', name: 'NIK', },
                { field: 'nama', name: 'Nama' },
                { field: 'jenis_kelamin', name: 'Jenis Kelamin' },
                { field: 'tanggal_lahir', name: 'Tanggal Lahir' },
                { field: 'tempat_lahir', name: 'Tempat Lahir' },
                { field: 'agama', name: 'Agama' },
                { field: 'kewarganegaraan', name: 'Kewarganegaraan' },
                { field: 'alamat', name: 'Alamat' },
                { field: 'keperluan', name: 'Keperluan' },
                { field: 'kk', name: 'KK' },
            ]
        }
    },
    async asyncData({ $axios, params }) {
        const sktms = await $axios.$get(`/sktm/${params.id}`).then(res => {
            return res
        })
        return sktms
    },
}
</script>