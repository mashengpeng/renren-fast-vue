<template>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
        show-checkbox node-key="catId">
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
            console.log(node, data)
        },
    },
    created() {
        this.getMenus();
    },
}
</script>

<style>

</style>