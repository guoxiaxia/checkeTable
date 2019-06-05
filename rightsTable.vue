<template>
    <el-table
            v-if="isTable"
            v-loading="loading"
            :data="tableData"
            :span-method="objectSpanMethod"
            border
            highlight-current-row
            header-align="center"
            style="width: 100%;">
        <el-table-column
                width="160"
                align="center"
                prop="ptitle"
                label="一级">
            <template slot-scope="scope">
                <el-checkbox
                        :indeterminate="isIndeterminateP[scope.$index]"
                        v-model="checkedParent[scope.$index]"
                        @change="handleCheckAllChange(scope.$index)">
                    {{scope.row.ptitle}}
                </el-checkbox>
            </template>
        </el-table-column>
        <el-table-column
                width="240"
                align="center"
                prop="stitle"
                label="二级">
            <template slot-scope="scope">
                <el-checkbox
                        :indeterminate="isIndeterminateS[scope.$index]"
                        v-model="checkedSecond[scope.$index]"
                        @change="handleCheckSecondChange(scope.$index)">
                    {{scope.row.stitle}}
                </el-checkbox>
            </template>
        </el-table-column>
        <el-table-column
                prop="rights"
                label="权限配置细则">
            <template slot-scope="scope">
                <el-checkbox-group v-model="checkedBtn[scope.$index]">
                    <el-checkbox
                            v-for="item in scope.row.rights"
                            :label="item.bid"
                            :key="item.bid"
                            @change="checkedBtnChange(scope.$index)">
                        {{item.btitle}}
                    </el-checkbox>
                </el-checkbox-group>
            </template>
        </el-table-column>
    </el-table>
</template>

<script>
    export default {
        name: "rightsTable",
        props: {
            loading: {
                type: Boolean,
                default: false
            },
            //权限树结构
            rightsData: {
                type: Array,
                default: []
            }
        },
        data() {
            return {
                isTable: true,
                tableData: [],
                //存放表格pid
                ids: [],
                //以下数组长度均为表格二级菜单条数
                //一级已选状态： true false
                checkedParent: [],
                //二级已选状态
                checkedSecond: [],
                //一级是否半选： true, false
                isIndeterminateP: [],
                //二级是否半选： true, false
                isIndeterminateS: [],
                //一级已选菜单id
                checkedP: [],
                //二级已选菜单id
                checkedS: [],
                //三级已选按钮id,二维数组
                checkedBtn: [],
            }
        },
        methods: {
            //将树结构数据处理为表单数据并初始化
            handlerTree(data) {
                let i = 0;
                this.tableData = [];
                data.forEach(item => {
                    item.children.forEach((ele, index) => {
                        this.tableData.push({
                            pid: item.id,
                            ptitle: item.title,
                            sid: ele.id,
                            stitle: ele.title,
                            rights: []
                        });
                        ele.children.forEach(e => {
                            this.tableData[i].rights.push({bid: e.id, btitle: e.title});
                        })
                        i++;
                        this.ids.push(item.id);
                        this.checkedBtn.push([]);
                        this.checkedSecond.push(false);
                        this.checkedParent.push(false);
                        this.isIndeterminateP.push(false);
                        this.isIndeterminateS.push(false);
                        this.checkedP.push("");
                        this.checkedS.push("");
                    })
                })
            },
            //处理当前角色的已有的权限树结构---获取权限树中的id
            setCurrentRights(data) {
                return new Promise(resolve => {
                    //解决在已有权限的情况下，部分复选框未显示高亮的问题
                    this.isTable = false;
                    data.forEach((item) => {
                        item.children.forEach(ele => {
                            let index = 0;
                            this.tableData.forEach((e, i) => {
                                if (e.sid === ele.id) {
                                    this.checkedS[i] = e.sid;
                                    index = i;
                                }
                            });
                            this.checkedBtn[index] = ele.children.map(e => e.id);
                            if (!ele.children.length) {
                                this.checkedSecond[index] = true;
                            }
                        });
                        this.tableData.forEach((e, i) => {
                            if (e.pid === item.id) {
                                this.checkedP[i] = item.id;
                            }
                        });
                    });
                    resolve();
                }).then(() => {
                    //多选框高亮已有权限
                    this.checkedBtn.forEach((item, index) => {
                        if (item && item.length) {
                            this.checkedBtnChange(index);
                            this.isTable = true;
                        }
                    })
                    this.checkedS.forEach((e, i) => {
                        if (e) this.handlerParent(i);
                    })
                })
            },
            //合并行
            objectSpanMethod({row, column, rowIndex, columnIndex}) {
                if (columnIndex === 0) {
                    //计算合并行的长度
                    //获取一级菜单第一次出现的下标和最后一次出现的下标
                    let first = this.ids.indexOf(row.pid);
                    let last = this.ids.lastIndexOf(row.pid);
                    if (rowIndex === first) {
                        return {
                            //合并行的高度，从1算起
                            rowspan: last - first + 1,
                            colspan: 1,
                        }
                    } else {
                        //属于合并行的中间部分---位于该一级菜单第一次出现和最后一次出现的中间部分，进行忽略
                        return {
                            rowspan: 0,
                            colspan: 0
                        }
                    }
                }
            },
            //触发一级菜单
            handleCheckAllChange(index) {
                let checkmap = [];
                let lastIndex = this.ids.lastIndexOf(this.tableData[index].pid);
                for (let i = index; i <= lastIndex; i++) {
                    if (this.checkedSecond[i]) checkmap.push(this.checkedSecond[i]);
                }
                this.tableData.forEach((ele, i) => {
                    //找到当前一级菜单下的二级菜单
                    if (ele.pid === this.tableData[index].pid) {
                        if (checkmap.length === lastIndex - index + 1) {
                            //二级菜单已全选
                            this.checkedP[i] = "";
                            this.checkedS[i] = "";
                            this.checkedBtn[i] = [];
                            this.checkedSecond[i] = false;
                            this.checkedParent[i] = false;
                        } else {
                            //二级菜单未全选
                            this.checkedP[i] = ele.pid;
                            this.checkedS[i] = ele.sid;
                            this.checkedBtn[i] = ele.rights.map(e => e.bid);
                            this.checkedSecond[i] = true;
                        }
                        //取消一级、二级菜单的不确定状态
                        this.isIndeterminateP[i] = false;
                        this.isIndeterminateS[i] = false;
                    }
                })
            },
            //触发二级菜单
            handleCheckSecondChange(index) {
                //一级菜单全选或全取消
                this.checkedBtn[index] = this.checkedSecond[index] ? this.tableData[index].rights.map(ele => ele.bid) : [];
                //取消二级菜单不确定状态
                this.isIndeterminateS[index] = false;
                this.handlerParent(index);

                //选择时
                if (this.checkedSecond[index]) {
                    this.checkedP[index] = this.tableData[index].pid;
                    this.checkedS[index] = this.tableData[index].sid;
                } else {
                    //取消选择时
                    this.checkedP[index] = "";
                    this.checkedS[index] = "";
                }
            },
            //触发三级菜单
            checkedBtnChange(index) {
                let checkedCount = this.checkedBtn[index].length;

                //二级菜单显示与否
                this.checkedSecond[index] = checkedCount === this.tableData[index].rights.length;
                this.isIndeterminateS[index] = checkedCount > 0 && checkedCount < this.tableData[index].rights.length;

                //选择时
                if (this.checkedBtn[index] && this.checkedBtn[index].length) {
                    this.checkedP[index] = this.tableData[index].pid;
                    this.checkedS[index] = this.tableData[index].sid;
                } else {
                    //不选择时
                    this.checkedP[index] = "";
                    this.checkedS[index] = "";
                }
                this.handlerParent(index);
            },
            //一级菜单显示与否
            handlerParent(index) {
                let arr = []
                this.ids.forEach((ele, i) => {
                    if (ele === this.tableData[index].pid) {
                        arr.push(i)
                    }
                });

                //是否全选该行二级菜单
                let flag = true;
                //该行一级菜单是否显示半角
                let isHave = false;
                for (let i in arr) {
                    if (this.checkedSecond[arr[i]] || this.isIndeterminateS[arr[i]]) {
                        isHave = true;
                    }
                    if (!this.checkedSecond[arr[i]]) {
                        flag = false;
                    }
                }

                if (flag) {
                    //对应所有二级菜单
                    this.checkedParent[arr[0]] = true;
                    this.isIndeterminateP[arr[0]] = false;
                } else {
                    //对应部分二级菜单
                    this.checkedParent[arr[0]] = false;

                    if (isHave) {
                        //'半角'
                        this.isIndeterminateP[arr[0]] = true;
                    } else {
                        //不显示半角
                        this.isIndeterminateP[arr[0]] = false;
                    }
                }
            },
            //获取当前的权限结果
            getRightsAll() {
                if (!this.checkedBtn.length) return;
                let rights = this.checkedBtn.filter(ele => ele.length);
                //rights为二维数组
                //数组的扁平化
                let result = rights.reduce((result, ele) => {
                    return result.concat([...ele])
                }, []);
                let rightsAll = [...new Set([...this.checkedP, ...this.checkedS, ...result])];
                //数组去重
                return rightsAll.join(',');
            },
        },
        watch:{
            rightsData(){
                if(this.rightsData && this.rightsData.length){
                    this.handlerTree(this.rightsData);
                }
            }
        }
    }
</script>

<style scoped>

</style>
