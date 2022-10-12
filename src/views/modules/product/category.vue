<template>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
        show-checkbox node-key="catId" :default-expanded-keys="defaultExpandKey">
        <span class="custom-tree-node" slot-scope="{ node, data }">
            <span>{{ node.label }}</span>
            <span>
                <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
                    Append
                </el-button>
                <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
                    Delete
                </el-button>
            </span>
        </span>
    </el-tree>
</template>

<script>
export default {
    data() {
        return {
            menus: [],
            defaultExpandKey: [],
            defaultProps: {
                children: "children",
                label: "name",
            }
        }
    },
    methods: {
        handleNodeClick(data) {
        },
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
                params: this.$http.adornParams({})
            }).then(({ data }) => {
                this.menus = data.data
            })
        },
        append(data) {
            console.log(data)
        },

        remove(node, data) {
            var ids = [data.catId]

            this.$confirm(`确定删除【${data.name}】菜单?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl("/product/category/delete"),
                    method: 'post',
                    data: this.$http.adornData(ids, false)
                }).then(({ data }) => {
                    this.$message({
                        message: '删除成功', 
                        type: 'success'
                    });
                    this.defaultExpandKey = [node.parent.data.catId]
                    this.getMenus()
                });
            }).catch(() => {

            })


        }
    },
    created() {
        this.getMenus();
    },
}
</script>

<style>

</style>