# checkeTable
表格形式的三级复选框结构

支持三级结构的复选框，以表格形式展现,
数据结构为：
```js
 [
   {pid:1, title :"一级", children: [
     {id:2, title: "二级", children:[
       {id:"3", title: "三级"}
     ]} 
   ]}
 ]
```
可请求getRightsAll()方法获取当前所选复选框的id集合，字符串形式（'，'隔开）



使用时： 

```js
<rightsTable ref="rightsTable" :loading="table.loading" :rightsData = table.rightsData></rightsTable>
```

```js
data(){
    return{
        table:{
            loading: false,
            rightsData: []
        }
    }
},
methods:{
    //获取整个权限树
    rightsTree(){
        this.table.loading = true;
        return new Promise((resolve, reject) => {
            roleRightsAll().then(res => {
                if (res) {
                    this.table.rightsData = res;
                } else {
                    this.$message.warning("数据请求失败！");
                }
                this.table.loading = false;
                resolve();
            }).catch(err => {
                this.$message.error(err);
                reject();
            })
        })
    }
}，
mounted(){
    //已有权限赋值
    this.loading = true;
    this.$nextTick(() => {
        this.rightsTree().then(() => {
            getRightSTree(this.$route.params.id).then(res => {
                if (res && res.data) {
                    //设置当前已有权限
                this.$refs.rightsTable.setCurrentRights(res.data.tree);
                } else {
                    this.$message.warning("数据请求失败！");
                }
                this.loading = false;
            })
        }).catch(err => {
            this.$message.error(err);
        })
    })
}
```





效果图：

![rightsTable](https://github.com/guoxiaxia/checkeTable/blob/master/rightsTable.png)
