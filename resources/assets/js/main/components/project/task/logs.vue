<template>
    <drawer-tabs-container>
        <div class="project-task-logs">
            <ul class="logs-activity">
                <li v-for="(items, key) in lists">
                    <div class="logs-date">{{key}}</div>
                    <div class="logs-section">
                        <Timeline>
                            <TimelineItem v-for="(item, index) in items" :key="index">
                                <div slot="dot" class="logs-dot">
                                    <img :src="item.userimg"/>
                                </div>
                                <div class="log-summary">
                                    <span class="log-creator">{{item.username}}</span>
                                    <span class="log-text-secondary">{{item.detail}}</span>
                                    <span v-if="item.other.type=='task'" class="log-text-link">{{item.other.title}}</span>
                                    <a v-if="item.other.type=='file'" class="log-text-link" target="_blank" :href="fileDownUrl(item.other.id)">{{item.other.name}}</a>
                                    <span class="log-text-info">{{item.timeData.ymd}} {{item.timeData.segment}} {{item.timeData.hi}}</span></div>
                            </TimelineItem>
                        </Timeline>
                    </div>
                </li>
                <li v-if="hasMorePages" class="logs-more" @click="getMore">加载更多</li>
            </ul>
        </div>
    </drawer-tabs-container>
</template>
<style lang="scss" scoped>
    .project-task-logs {
        margin: 0 12px;
        .logs-activity {
            position: relative;
            word-break: break-all;
            padding: 12px 12px;
            > li {
                padding-top: 22px;
                &.logs-more {
                    cursor: pointer;
                    &:hover {
                        color: #048be0;
                    }
                }
                &:first-child {
                    padding-top: 0;
                }
                .logs-date {
                    color: rgba(0, 0, 0, .36);
                    padding-bottom: 14px;
                }
                .logs-dot {
                    width: 18px;
                    height: 18px;
                    margin-left: 10px;
                    img {
                        width: 100%;
                        height: 100%;
                        border-radius: 50%;
                        overflow: hidden;
                    }
                }
                .log-summary {
                    > span,
                    > a {
                        padding-right: 6px;
                        word-wrap: break-word;
                        word-break: break-word;
                    }
                    .log-creator {
                        color:rgba(0, 0, 0, 0.85)
                    }
                    .log-text-secondary {
                        color: rgba(0,0,0,.54);
                    }
                    .log-text-link {
                        color: #048be0;
                    }
                    .log-text-info {
                        color: rgba(0,0,0,.36);
                    }
                }
            }
        }
    }
</style>

<script>

    import DrawerTabsContainer from "../../DrawerTabsContainer";
    export default {
        name: 'ProjectTaskLogs',
        components: {DrawerTabsContainer},
        props: {
            projectid: {
                default: 0
            },
            taskid: {
                default: 0
            },
            canload: {
                type: Boolean,
                default: true
            },
        },
        data() {
            return {
                loadYet: false,

                loadIng: 0,

                lists: {},
                listPage: 1,
                hasMorePages: false,
            }
        },

        created() {

        },

        mounted() {
            if (this.canload) {
                this.loadYet = true;
                this.getLists(true);
            }
        },

        watch: {
            projectid() {
                if (this.loadYet) {
                    this.getLists(true);
                }
            },
            taskid() {
                if (this.loadYet) {
                    this.getLists(true);
                }
            },
            canload(val) {
                if (val && !this.loadYet) {
                    this.loadYet = true;
                    this.getLists(true);
                }
            }
        },

        methods: {
            getLists(resetLoad) {
                if (resetLoad === true) {
                    this.listPage = 1;
                }
                if (this.projectid == 0) {
                    this.lists = {};
                    this.hasMorePages = false;
                    return;
                }
                this.loadIng++;
                $A.aAjax({
                    url: 'project/log/lists',
                    data: {
                        projectid: this.projectid,
                        taskid: this.taskid,
                        page: Math.max(this.listPage, 1),
                        pagesize: 100,
                    },
                    complete: () => {
                        this.loadIng--;
                    },
                    success: (res) => {
                        if (res.ret === 1) {
                            let timeData,
                                key;
                            res.data.lists.forEach((item) => {
                                timeData = item.timeData;
                                key = timeData.ymd + " " + timeData.week;
                                if (typeof this.lists[key] !== "object") {
                                    this.$set(this.lists, key, []);
                                }
                                this.lists[key].push(item);
                            });
                            console.log(this.lists);
                            this.hasMorePages = res.data.hasMorePages;
                        } else {
                            this.lists = [];
                            this.hasMorePages = false;
                        }
                    }
                });
            },

            getMore() {
                if (!this.hasMorePages) {
                    return;
                }
                this.hasMorePages = false;
                this.listPage++;
                this.getLists();
            },

            fileDownUrl(id) {
                return $A.aUrl('project/files/download?fileid=' + id);
            }
        }
    }
</script>