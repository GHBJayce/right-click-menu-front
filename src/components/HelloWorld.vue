<template>
    <div class="container">
        <div class="row">
            <div class="col-md-7">
                <b-card title="操作台" bg-variant="white" border-variant="light" class="shadow-sm">
                    <el-tabs v-model="activeName">
                        <el-tab-pane label="右键菜单" name="rcm">
                            <el-row class="mt-2 mb-4">
                                <el-col :span="21">
                                    <el-tree
                                        :props="tree.props"
                                        :node-key="tree.id"
                                        :data="tree.rcm.data"
                                        :default-expanded-keys="tree.rcm.expandKeys"
                                        @node-click="handleNodeClick"
                                        draggable
                                        show-checkbox
                                        ref="rcm">
                                    </el-tree>
                                </el-col>
                                <el-col :span="3">
                                    <div class="text-right">
                                        <b-button @click="createMenu" variant="primary" size="sm" class="shadow-sm">
                                            新增菜单
                                        </b-button>
                                    </div>
                                </el-col>
                            </el-row>
                            <b-button variant="primary" block class="w-100">生成</b-button>
                            <a-drawer
                                v-if="!JW.isEmpty(drawer.rcm.node.data)"
                                title="菜单设置"
                                placement="right"
                                :visible="rcmDrawer.show"
                                :width="rcmDrawer.width"
                                @close="onClose">
                                <el-form v-if="!JW.isEmpty(drawer.rcm.node.data.children)" ref="form" :model="form" label-width="80px">
                                    <el-form-item v-for="(v, k) in drawer.rcm.rcmKeys" :key="k" :label="copywirting.fields[v].text">
                                        <el-input v-model="drawer.rcm.node.data[v]"></el-input>
                                    </el-form-item>
                                    <template v-if="drawer.rcm.node.level === 1">
                                        <el-form-item label="菜单位置">
                                            <el-select
                                                v-model="drawer.rcm.departments"
                                                multiple
                                                clearable
                                                filterable
                                                class="w-100"
                                                placeholder="请选择右键菜单的位置"
                                                @change="departmentSelectChange">
                                                <el-option-group
                                                    v-for="(v, k) in rcmDepartment"
                                                    :key="k"
                                                    :label="v.name">
                                                    <el-option
                                                        v-for="(vv, kk) in v.options"
                                                        :key="kk"
                                                        :label="vv.name"
                                                        :value="vv.val">
                                                    </el-option>
                                                </el-option-group>
                                            </el-select>
                                        </el-form-item>
                                        <el-form-item
                                            v-for="(v, k) in drawer.rcm.departments"
                                            :key="k"
                                            :label="'【' + copywirting.departments[v].text + '】按 Shift 键出现'"
                                            label-width="200">
                                            <el-radio-group v-model.number="drawer.rcm.extends[v]">
                                                <el-radio :label="0">否</el-radio>
                                                <el-radio :label="1">是</el-radio>
                                            </el-radio-group>
                                        </el-form-item>
                                    </template>
                                    <el-form-item>
                                        <el-button type="primary" @click="onSubmit">立即创建</el-button>
                                    </el-form-item>
                                </el-form>
                                <el-row v-else>
                                    <el-col v-for="(v, k) in drawer.rcm.menuSetKeys" :key="k" :span="24" class="mb-2">
                                        <strong>{{ copywirting.fields[v].text }}：</strong>
                                        <span>{{ drawer.rcm.node.data[v] }}</span>
                                    </el-col>
                                </el-row>
                            </a-drawer>
                        </el-tab-pane>
                        <el-tab-pane label="菜单集" name="menuSets">
                            <el-tree
                                ref="menuSets"
                                draggable
                                show-checkbox
                                node-key="id"
                                class="mt-2 mb-4"
                                :data="tree.menuSets.data"
                                :default-expanded-keys="tree.menuSets.expandKeys"
                                :allow-drag="menuSetsAllowDrag"
                                @node-click="menuSetsClick">
                            </el-tree>
                        </el-tab-pane>
                    </el-tabs>
                    <a-drawer
                        title="菜单设置"
                        placement="right"
                        :visible="currentMenuDrawer.show"
                        :width="currentMenuDrawer.width"
                        @close="onClose"
                    >
                        <el-form :label-position="currentMenuDrawer.formLabelPosition" label-width="80px" :model="currentMenuDrawer.data">
                            <el-form-item label="注册表名">
                                <el-input v-model="currentMenuDrawer.data.id"></el-input>
                            </el-form-item>
                            <el-form-item label="菜单名称">
                                <el-input v-model="currentMenuDrawer.data.label"></el-input>
                            </el-form-item>
                            <el-form-item label="程序路径">
                                <el-input v-model="currentMenuDrawer.data.path"></el-input>
                            </el-form-item>
                            <el-form-item label="菜单图标">
                                <el-input v-model="currentMenuDrawer.data.icon"></el-input>
                            </el-form-item>
                            <b-button variant="primary" block class="w-100 mt-5">保存</b-button>
                        </el-form>
                    </a-drawer>
                </b-card>
            </div>
            <div class="col-md-5">
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'HelloWorld',
    data () {
        var copywirting = {
            fields: {
                registry_name: {
                    text: '注册表名'
                },
                menu_name: {
                    text: '菜单名称'
                },
                icon: {
                    text: '菜单图标'
                },
                path: {
                    text: '程序路径'
                }
            },
            departments: {
                1: {
                    text: '所有文件'
                },
                2: {
                    text: '文件夹'
                },
                3: {
                    text: '驱动器'
                },
                4: {
                    text: '文件夹和驱动器'
                },
                5: {
                    text: '文件夹背景'
                },
                6: {
                    text: '桌面背景'
                }
            }
        }
        return {
            activeName: 'rcm',
            copywirting: copywirting,
            rcmDepartment: [{
                name: '选中',
                options: [{
                    name: copywirting.departments[1].text,
                    val: 1
                }, {
                    name: copywirting.departments[2].text,
                    val: 2
                }, {
                    name: copywirting.departments[3].text,
                    val: 3
                }, {
                    name: copywirting.departments[4].text,
                    val: 4
                }]
            }, {
                name: '背景',
                options: [{
                    name: copywirting.departments[5].text,
                    val: 5
                }, {
                    name: copywirting.departments[6].text,
                    val: 6
                }]
            }],
            tree: {
                id: 'registry_name',
                props: {
                    children: 'children',
                    label: 'menu_name'
                },
                rcm: {
                    data: [{
                        registry_name: 'my-right-click-menu',
                        menu_name: '我的右键菜单',
                        icon: 'i am icon',
                        children: [{
                            menu_name: '复制目标路径',
                            registry_name: 'copy-target-path'
                        }]
                    }],
                    expandKeys: ['my-right-click-menu']
                },
                menuSets: {
                    data: [],
                    expandKeys: []
                }
            },
            drawer: {
                rcm: {
                    node: {},
                    rcmKeys: ['registry_name', 'menu_name', 'icon'],
                    menuSetKeys: ['registry_name', 'menu_name', 'path', 'icon'],
                    departments: [],
                    extends: {}
                },
                menuSets: {}
            },
            currentMenuDrawer: {
                width: '720',
                show: false,
                formLabelPosition: 'top',
                data: {}
            },
            rcmDrawer: {
                width: '720',
                show: false,
                data: {}
            },
            menuVisible: false
        }
    },
    mounted () {
        this.init()
        /*
            eslint-disable-next-line
            eslint-disable no-undef
        */
    },
    methods: {
        init: function () {
            this.menuSets()
        },
        officialMenu: function () {
            const data = [{
                label: '官方菜单',
                id: 'official',
                children: [{
                    label: '实用功能',
                    id: 'practical-function',
                    children: [{
                        label: '复制目标路径',
                        id: 'copy-target-path',
                        path: 'mshta vbscript:clipboarddata.setdata("text","%1")(close)',
                        icon: '%SYSTEMROOT%\\explorer.exe'
                    }]
                }, {
                    label: '电源控制',
                    id: 'power-supply-control',
                    children: [{
                        label: '锁定',
                        id: 'lock'
                    }, {
                        label: '注销',
                        id: 'logout'
                    }, {
                        label: '切换用户',
                        id: 'switch-user'
                    }, {
                        label: '睡眠',
                        id: 'sleep'
                    }, {
                        label: '休眠',
                        id: 'dormancy'
                    }, {
                        label: '重启',
                        id: 'reboot'
                    }, {
                        label: '关机',
                        id: 'shutdown'
                    }]
                }]
            }]

            function handle (data) {
                for (var i in data) {
                    data[i] = Object.assign({
                        disabled: true,
                        group: 1
                    }, data[i])
                    // eslint-disable-next-line
                    if (!isEmpty(data[i].children)) {
                        data[i].children = handle(data[i].children)
                    }
                }

                return data
            }

            return handle(data)
        },
        getMenuSets: function () {
            return this.officialMenu()
        },
        menuSets: function () {
            this.tree.menuSets.expandKeys = ['practical-function', 'power-supply-control']
            this.tree.menuSets.data = this.getMenuSets()
        },
        menuSetsAllowDrag: function (treeNode) {
            var node = treeNode.data
            // eslint-disable-next-line
            if (!isEmpty(node.group) && node.group === 1) {
                return false
            }

            return true
        },
        menuSetsClick: function (node) {
            // eslint-disable-next-line
            if (isEmpty(node.children)) {
                this.currentMenuDrawer.show = true
                this.currentMenuDrawer.data = node
            }
        },
        showDrawer: function () {
            this.visible = true
        },
        onClose: function () {
            this.currentMenuDrawer.show = false
            this.rcmDrawer.show = false
            let keys = this.$refs.menuSets.getCurrentKey()
            console.log(keys)
        },
        handleNodeClick: function (e, node, sel) {
            console.log(e, node, sel)
            console.log(this.$refs.menuSets.getNode(e.setid))
            this.rcmDrawer.show = true
            node.expanded = true
            this.drawer.rcm.node = node
            console.log(this.drawer.rcm.node)
        },
        createMenu: function () {
            this.tree.rcm.data.push({
                registry_name: '',
                menu_name: '新的菜单'
            })
            console.log(this.tree.rcm.data)
        },
        onSubmit: function () {
            console.log(this.drawer.rcm.node)
            console.log(this.RCMTree)
        },
        departmentSelectChange: function (v) {
            var data = this.drawer.rcm.extends
            for (var i in v) {
                // eslint-disable-next-line
                if (isEmpty(data[v[i]])) {
                    // eslint-disable-next-line
                    Vue.set(this.drawer.rcm.extends, v[i], 0)
                }
            }
        }
    }
}
</script>
