<template>
    <div>
        <el-button type="danger" @click="batchDelete">批量删除</el-button>
        <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
            show-checkbox node-key="catId" :default-expanded-keys="defaultExpandKey" :draggable="true"
            :allow-drop="allowDrop" ref="menuTree">
            <span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
                        Append
                    </el-button>
                    <el-button type="text" size="mini" @click="edit(data)">
                        Edit
                    </el-button>
                    <el-button v-if="node.childNodes.length == 0" type="text" size="mini"
                        @click="() => remove(node, data)">
                        Delete
                    </el-button>
                </span>
            </span>

        </el-tree>
        <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="category.icon" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="计量单位">
                    <el-input v-model="category.productUnit" autocomplete="off"></el-input>
                </el-form-item>


            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="submitData">确 定</el-button>
            </span>
        </el-dialog>
    </div>

</template>

<script>
export default {
    data() {
        return {
            maxLevel: 0,
            dialogType: "",
            menus: [],
            title: "",
            defaultExpandKey: [],
            dialogVisible: false,
            category: {
                catId: "",
                name: "",
                parentCid: 0,
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                icon: "",
                productUnit: "",
            },
            defaultProps: {
                children: "children",
                label: "name",
            }
        }
    },
    methods: {
        batchDelete() {
            let catIds = []
            let checkedNodes = this.$refs.menuTree.getCheckedNodes()
            for (let i = 0; i < checkedNodes.length; i++) {
                catIds.push(checkedNodes[i].catId)
            }
            this.$confirm(`确定删除【${catIds}】菜单?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(catIds, false)
                }).then(({ data }) => {
                    this.$message({
                        message: "菜单删除成功",
                        type: "success"

                    })
                    this.getMenus()
                }).catch(() => {

                });
            })


        },
        handleNodeClick(data) {
            console.log(data)
        },
        allowDrop(draggingNode, dropNode, type) {
            this.maxLevel = 0
            this.countNodeLevel(draggingNode.data)
            let deep = this.maxLevel - draggingNode.data.catLevel + 1
            if (type == "inner") {
                return (deep + dropNode.level) <= 3
            } else {
                return (deep + dropNode.parent.level) <= 3
            }
        },
        countNodeLevel(node) {
            if (node.children != null && node.children.length > 0) {
                for (let i = 0; i < node.children.length; i++) {
                    if (node.children[i].catLevel > this.maxLevel) {
                        this.maxLevel = node.children[i].catLevel
                    }
                    this.countNodeLevel(node.children[i])
                }
            }
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
        submitData() {
            if (this.dialogType == "append") {
                this.addCategory()
            } else if (this.dialogType == "edit") {
                this.editCategory()
            }
        },

        append(data) {
            this.dialogType = "append"
            this.dialogVisible = true
            this.title = "增加分类"

            this.category.catId = null
            this.category.name = ""
            this.category.parentCid = data.catId
            this.category.catLevel = data.catLevel * 1 + 1
            this.category.icon = ""
            this.category.productUnit = ""
            this.category.sort = 0
            this.category.showStatus = 1
        },

        addCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                this.$message({
                    message: '添加成功',
                    type: 'success'
                });
                this.dialogVisible = false
                this.defaultExpandKey = [this.category.parentCid]
                this.getMenus()
            });
        },

        edit(data) {
            this.dialogType = "edit"
            this.dialogVisible = true
            this.title = "修改分类"


            this.$http({
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
                method: 'get',
            }).then(({ data }) => {

                this.category.catId = data.data.catId
                this.category.name = data.data.name
                this.category.parentCid = data.data.parentCid
                this.category.catLevel = data.data.catLevel
                this.category.icon = data.data.icon
                this.category.productUnit = data.data.productUnit

            })
        },

        editCategory() {
            var { catId, name, icon, productUnit } = this.category
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
                this.$message({
                    message: '修改成功',
                    type: 'success'
                });
                this.dialogVisible = false
                this.defaultExpandKey = [this.category.parentCid]
                this.getMenus()
            });
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