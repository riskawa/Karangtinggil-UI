<template>
    <v-col cols="12">
        <v-card>
            <v-toolbar flat>
                <v-toolbar-title class="text-h5">
                    Laporan
                </v-toolbar-title>

                <v-spacer></v-spacer>
            </v-toolbar>
            <v-divider></v-divider>
            <v-card-text>
                <v-row class="d-flex justify-space-around">
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/pemohon" color="primary" dark :elevation="hover ? 8 : 4"
                                height="140" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h6 text-center">
                                    Pemohon
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/sktm" color="primary" dark :elevation="hover ? 8 : 4"
                                height="140" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h6 text-center">
                                    SKTM
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/skck" color="primary" dark :elevation="hover ? 8 : 4"
                                height="140" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h6 text-center">
                                    SKCK
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                </v-row>
                <v-row class="d-flex justify-space-around">
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/domisili" color="primary" dark :elevation="hover ? 8 : 4"
                                height="140" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h6 text-center">
                                    Surat Domisili
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/sku" color="primary" dark :elevation="hover ? 8 : 4" 
                                height="140" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h6 text-center">
                                    SKU
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/surat-keterangan" color="primary" dark :elevation="hover ? 8 : 4"
                               height="140" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h6 text-center">
                                    Surat Keterangan
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                </v-row>
                <v-row class="ml-13">
                    <v-col cols="3">
                        <v-hover v-slot="{ hover }">
                            <v-card to="/kades/laporan/kehilangan-kk" color="primary" dark :elevation="hover ? 8 : 4"
                                height="160" class="d-flex flex-column pa-4" shaped>
                                <v-icon size=70>mdi-email</v-icon>
                                <!-- <v-img src="/surat.png" width="50"></v-img> -->
                                <v-card-text class="d-flex justify-center white--text text-h7 text-center">
                                    Surat Kehilangan KK
                                </v-card-text>
                            </v-card>
                        </v-hover>
                    </v-col>
                </v-row>
            </v-card-text>
        </v-card>
    </v-col>
</template>

<script>
export default {
    layout: 'kades',
    name: 'LaporanView'
}
</script>