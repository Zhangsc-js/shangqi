<template>
    <div class="margin20 mr15">
            <el-form :inline="true" class="demo-form-inline"  ref="inMeterDtlForm"  >
                    <el-form-item label="货物名称" prop="goodsName">
                        <el-input
                                id="goodsName"
                                v-model="inMeterDtlForm.goodsName"
                                :maxlength="20"
                                plain="true"
                                placeholder="请输入货物名称"
                                style="width: 200px"
                        />
                    </el-form-item>
                    <el-form-item label="车号" prop="truckNo">
                        <el-select v-model="inMeterDtlForm.truckNo" filterable placeholder="请选择" id="truckNo">
                            <el-option
                                    v-for="item in carsList"
                                    :key="item.value"
                                    :label="item.truckNo"
                                    :value="item.truckNo">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="供应商" prop="supplier">
                        <el-input
                                id="supplier"
                                v-model="inMeterDtlForm.supplier"
                                :maxlength="20"
                                plain="true"
                                placeholder="供应商名称"
                                style="width: 200px"
                        />
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" icon="el-icon-search" @click="submitSearch(1)">查询</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button href="javascript:void(0)" class="btn-w"  @click="clearSearchBox()">清空</el-button>
                    </el-form-item>
            </el-form>
                <el-table :data="inMetersData"
                          :summary-method="getSummaries"
                          show-summary
                          style="width: 100%">
                    <el-table-column prop="weighingNo" align="center" label="检斤序号" width="180"></el-table-column>
                    <el-table-column prop="truckNo" align="center" label="车号" ></el-table-column>
                    <el-table-column prop="supplier" align="center" label="供应商" width="180"></el-table-column>
                    <el-table-column prop="goodsName" align="center" label="货物名称"></el-table-column>
                    <el-table-column prop="gross" align="center" label="毛重(KG)" ></el-table-column>
                    <el-table-column prop="tare" align="center" label="皮重(KG)" ></el-table-column>
                    <el-table-column prop="net" align="center" label="净重(KG)" ></el-table-column>
                    <el-table-column prop="createdBy" align="center" label="司磅员" ></el-table-column>
                    <el-table-column prop="createdOn" align="center" label="称重时间" width="180"></el-table-column>

                </el-table>
                <div style="height: 60px;">
                    <pagination :total="total" :page.sync="page.pageNum" :limit.sync="page.pageSize"
                                @pagination="getData"/>
                </div>
        </div>
</template>

<script>
    import {createNamespacedHelpers} from 'vuex'
    import Pagination from '@/components/Pagination/index'
    const {mapState, mapActions} = createNamespacedHelpers('inMeter')
    export default {
        name: "InMeterDetails",
        components: {Pagination},
        data() {
            return {
                page: {
                    pageNum: 0,
                    pageSize: 10
                },
                inMeterDtlForm: {
                    goodsName: '',
                    truckNo: '',
                    supplier: ''
                },
            };
        },
        computed: {
            ...mapState(['inMetersData', 'total']),
            carsList() {
                return this.$store.state.weiCars.weiCarData
            }
        },
        mounted() {
            this.getData()
            this.selectCars()
        },
        methods: {
            ...mapActions(['getAllInMeters']),
            selectCars() {
                this.$store.dispatch('weiCars/getAllWeiCars')
            },
            getData(pageNum) {
              if(pageNum === 1){
                this.page.pageNum =pageNum;
              }
                this.getAllInMeters({
                    ...this.page,
                    ...this.inMeterDtlForm
                })
            },
            submitSearch(pageNum) {
                this.getData(pageNum)
                    // this.getSummaries()
            },
            clearSearchBox() {
                this.inMeterDtlForm = {
                    goodsName: '',
                    truckNo: '',
                    supplier: ''
                }
            },
            getSummaries(param) {
                const {columns, data} = param;
                const sums = [];
                // const total= this.total
                columns.forEach((column, index) => {
                    if (index === 5) {
                        sums[index] = '总净重';
                        return;
                    }
                    const values = data.map(item => Number(item[column.property]));
                    if (column.property == 'net') {
                        sums[index] = values.reduce((prev, curr) => {
                            const value = Number(curr);
                            if (!isNaN(value)) {
                                return prev + curr;
                            } else {
                                return prev;
                            }
                        }, 0);
                        sums[index] += '   KG';
                    }

                    if (index === 1) {
                        sums[index] = '' + this.total + '车';
                    }
                    if (index === 0) {
                        sums[index] = '总车数'
                    }
                    /*if(column.property=='truckNo'){
                        sums[index] = values.reduce((prev,this.total)=>{
                            const value = Number(curr)
                        })
                    }*/

                });
                return sums;
            }
        }
    };
</script>

<style scoped>

</style>
