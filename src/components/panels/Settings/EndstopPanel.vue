<template>
    <v-card>
        <v-toolbar flat dense >
            <v-toolbar-title>
                <span class="subheading"><v-icon left>mdi-arrow-expand-vertical</v-icon>Endstops</span>
            </v-toolbar-title>
        </v-toolbar>
        <v-card-text class="pb-0">
            <v-container px-0 py-0>
                <v-row v-for="(status, index) of sortEndstops" v-bind:key="index">
                    <v-col>
                        <label class="mt-1 d-inline-block">Endstop <b>{{ index.toUpperCase() }}</b></label>
                        <v-chip class="float-right" :color="status === 'open' ? 'green' : 'red' " text-color="white">{{ status }}</v-chip>
                    </v-col>
                </v-row>
                <v-row v-if="(Object.keys(endstops).length === 0 && endstops.constructor === Object)" >
                    <v-col>
                        <p>Press the sync-button on the right-bottom to load the current endstop status.</p>
                    </v-col>
                </v-row>
            </v-container>
        </v-card-text>
        <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn icon @click="syncEndstops" :loading="loadings.includes('queryEndstops')">
                <v-icon>mdi-sync</v-icon>
            </v-btn>
    </v-card-actions>
    </v-card>
</template>

<script>
    import { mapState } from 'vuex'

    export default {
        components: {

        },
        data: function() {
            return {
                sortEndstops: {},
            }
        },
        computed: {
            ...mapState({
                loadings: state => state.socket.loadings,
                endstops: state => state.printer.endstops,
            })
        },
        created() {
            this.getEndstops();
        },
        methods: {
            syncEndstops() {
                this.$store.commit('socket/addLoading', { name: 'queryEndstops' });
                this.$socket.sendObj('printer.query_endstops.status', { }, "printer/getEndstopStatus");
            },
            getEndstops() {
                this.sortEndstops = {};

                let keys = Object.keys(this.endstops);
                keys.sort();

                for (let i = 0; i < keys.length; i++) {
                    let k = keys[i];
                    this.sortEndstops[k] = this.endstops[k];
                }
            }
        },
        watch: {
            endstops: function() {
                this.getEndstops();
            }
        }
    }
</script>