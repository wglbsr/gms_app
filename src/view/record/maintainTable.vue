<template>
    <div class="maintain-table-div">
        <scroller style="position: absolute;width: 100%;height: 90%" :on-infinite="infinite"
                  :on-refresh="refresh" noDataText="没有数据">
            <main style="width: 100%;height: 100%">
                <group>
                    <divider>总数:{{totalNum}},当前{{this.maintainLogData.length}}</divider>
                    <div v-for="(item, index) in maintainLogData">
                        <cell :title="item.generatorNo" :inline-desc="'操作人:'+item.username"
                              class="group-data-cell">
                            <!--基站编码:{{item.stationNo|translateText}}<br>-->
                            <!--基站名称:{{item.stationName|translateText}}<br>-->
                            {{item.createTime|translateTime}}
                        </cell>
                    </div>
                </group>
            </main>
        </scroller>
        <search-icon @doSearch="doSearch"></search-icon>
    </div>
</template>
<script>
    import SearchIcon from '../../components/FloatingSearchIcon'

    export default {
        name: "maintainTable",
        data() {
            return {
                onFetching: false,
                maintainLogData: [],
                customerNo: sessionStorage.getItem("usercus"),
                onlineLabel: "",
                totalNum: 0,
                pageNum: 0,
                pageSize: 20,
                condition: {},

            }
        },
        components: {
            SearchIcon,
        },
        mounted: function () {
            // this.getMaintainLogData();
        },
        filters: {
            translateText: function (value) {
                if (!value) {
                    return "暂无";
                } else {
                    return value;
                }
            },
            //截取时间
            translateTime: function (cellValue) {
                return (new Date(cellValue)).toLocaleString();
            },
            showStationName: function (cellValue) {

            }
        },
        methods: {
            infinite: function (done) {
                this.getMaintainLogData(false, done);
            },
            refresh: function (done) {
                this.getMaintainLogData(true, done);
            },
            doSearch: function (params) {
                this.condition = params;
                if (params.keyWord) {
                    this.condition.keyWord = params.keyWord;
                } else {
                    delete this.condition.keyWord;
                }
                this.getMaintainLogData(true, null);
            },
            getMaintainLogData(refresh, callback) {
                if (this.onFetching) {
                    return;
                }
                let params = {customerNo: this.customerNo, pageSize: this.pageSize, pageNum: this.pageNum};
                if (this.condition) {
                    params = Object.assign(params, this.condition);
                }
                if (refresh) {
                    this.pageNum = 1;
                } else {
                    this.pageNum++;
                }
                this.onFetching = true;
                this.$http.post(this.API_V2.maintainLog.select.default, params, {emulateJSON: true}).then(res => {
                    this.totalNum = res.body.totalNum;
                    if (!refresh) {
                        this.maintainLogData = this.maintainLogData.concat(res.body.data);
                    } else {
                        this.maintainLogData = res.body.data;
                    }
                    if (this.maintainLogData.length >= res.body.totalNum) {
                        this.$vux.toast.show({
                            text: '已全部加载!'
                        });
                        return;
                    }
                    this.onFetching = false;
                    if (callback instanceof Function) {
                        callback(2);
                    }
                }).catch(function (error) {
                    this.onFetching = false;
                    if (callback instanceof Function) {
                        callback(2);
                    }
                });
            }
        }
    }
</script>

<style scoped>
    .maintain-table-div {
        width: 100%;
        bottom: 50px;
    }
</style>
