<template>
    <div id="status-template" class="vgt-table-styling">
        <backstretch :slug="general.randomShowSlug" />
        <div class="row">
            <div class="col-lg-12">
                <h2 class="header">Scheduler</h2>
                <vue-good-table
                    :columns="columnsScheduler"
                    :rows="system.schedulers"
                    :sort-options="{
                        enabled: true,
                        initialSortBy: { field: 'name', type: 'asc' }
                    }"
                    styleClass="vgt-table condensed"
                    class="vgt-table-styling"
                >
                    <template slot="table-row" slot-scope="props">
                        <span v-if="props.column.label == 'Cycle Time'" class="align-center">
                            {{props.row.cycleTime ? prettyTimeDelta(props.row.cycleTime) : ''}}
                        </span>

                        <span v-else-if="props.column.label == 'Next Run'" class="align-center">
                            {{props.row.nextRun ? prettyTimeDelta(props.row.nextRun) : ''}}
                        </span>

                        <span v-else-if="props.column.label == 'Last Run'" class="align-center">
                            {{props.row.lastRun ? fuzzyParseDateTime(props.row.lastRun) : ''}}
                        </span>

                        <span v-else>
                            {{props.formattedRow[props.column.field]}}
                        </span>
                    </template>
                </vue-good-table>
            </div>
        </div>

        <div class="row">
            <div class="col-lg-12">
                <h2 class="header">Show Queue</h2>
                <vue-good-table
                    :columns="columnsShowQueue"
                    :rows="system.showQueue"
                    styleClass="vgt-table condensed"
                >
                    <template slot="table-row" slot-scope="props">
                        <span v-if="props.column.label == 'Added'" class="align-center">
                            {{fuzzyParseDateTime(props.row.added)}}
                        </span>

                        <span v-else>
                            {{props.formattedRow[props.column.field]}}
                        </span>
                    </template>
                    <div slot="emptystate">
                        Nothing in queue
                    </div>
                </vue-good-table>
            </div>
        </div>

        <div class="row">
            <div class="col-lg-12">
                <h2 class="header">Disk Space</h2>
                <vue-good-table
                    :columns="columnsDiskSpace"
                    :rows="collectDiskSpace"
                    styleClass="vgt-table condensed"
                >
                    <template slot="table-row" slot-scope="props">
                        <span v-if="props.column.label == 'Location'" class="align-center">
                            {{props.row.location ? props.row.location : 'N/A'}}
                        </span>
                        <span v-else-if="props.column.label == 'Free Space'" class="align-center">
                            {{props.row.freeSpace ? props.row.freeSpace : 'N/A'}}
                        </span>
                        <span v-else>
                            {{props.formattedRow[props.column.field]}}
                        </span>
                    </template>
                </vue-good-table>
            </div>
        </div>
    </div>
</template>
<script>
import { mapActions, mapGetters, mapState } from 'vuex';
import { VueGoodTable } from 'vue-good-table';
import Backstretch from './backstretch.vue';

export default {
    name: 'status',
    components: {
        Backstretch,
        VueGoodTable
    },
    data() {
        return {
            columnsScheduler: [{
                label: 'Scheduler',
                field: 'name'
            }, {
                label: 'Alive',
                field: 'isAlive'
            }, {
                label: 'Enabled',
                field: 'isEnabled',
                type: 'boolean'
            }, {
                label: 'Active',
                field: 'isActive',
                type: 'boolean'
            }, {
                label: 'Start Time',
                field: 'startTime'
            }, {
                label: 'Cycle Time',
                field: 'cycleTime',
                type: 'number'
            }, {
                label: 'Next Run',
                field: 'nextRun',
                type: 'number'
            }, {
                label: 'Last Run',
                field: 'lastRun',
                type: 'date',
                dateInputFormat: 'yyyy-MM-dd\'T\'HH:mm:ssXXX',
                dateOutputFormat: 'yyyy-MM-dd\'T\'HH:mm:ssXXX'
            }, {
                label: 'Silent',
                field: 'isSilent',
                type: 'boolean'
            }],
            columnsShowQueue: [{
                label: 'Show Slug',
                field: 'showSlug'
            }, {
                label: 'In Progress',
                field: 'inProgress',
                type: 'boolean'
            }, {
                label: 'Priority',
                field: 'priority'
            }, {
                label: 'Added',
                field: 'added',
                type: 'date',
                dateInputFormat: 'yyyy-MM-dd\'T\'HH:mm:ssXXX',
                dateOutputFormat: 'yyyy-MM-dd\'T\'HH:mm:ssXXX'
            }, {
                label: 'Queue Type',
                field: 'queueType'
            }],
            columnsDiskSpace: [{
                label: 'type',
                field: 'type'
            }, {
                label: 'Location',
                field: 'location'
            }, {
                label: 'Free Space',
                field: 'freeSpace'
            }]
        };
    },
    computed: {
        ...mapState({
            general: state => state.config.general,
            system: state => state.config.system
        }),
        ...mapGetters({
            fuzzyParseDateTime: 'fuzzyParseDateTime',
            prettyTimeDelta: 'prettyTimeDelta'
        }),
        collectDiskSpace() {
            const { system } = this;
            const { diskSpace } = system;
            if (diskSpace.tvDownloadDir === undefined) {
                return [];
            }

            return [diskSpace.tvDownloadDir, ...diskSpace.rootDir];
        }
    },
    methods: {
        ...mapActions({
            getConfig: 'getConfig'
        })
    },
    watch: {
        searchQueueItems() {
            const { getConfig } = this;
            getConfig('system');
        }
    }
};
</script>

<style scoped src="../style/vgt-table.css">
</style>
