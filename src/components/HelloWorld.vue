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
                                :title="drawer.rcm.title"
                                :placement="drawer.rcm.placement"
                                :visible="drawer.rcm.visible"
                                :width="drawer.rcm.width"
                                @close="drawerClose">
                                <el-form v-if="!JW.isEmpty(drawer.rcm.node.data.children)" ref="form" :model="form" label-width="80px">
                                    <el-form-item v-for="(v, k) in drawer.rcm.rcmKeys" :key="k" :label="copywirting.fields[v].text">
                                        <el-input v-model="drawer.rcm.node.data[v]"></el-input>
                                    </el-form-item>
                                    <template v-if="drawer.rcm.node.level === 1">
                                        <el-form-item label="菜单位置">
                                            <el-select
                                                v-model="drawer.rcm.node.data.departments"
                                                multiple
                                                clearable
                                                filterable
                                                class="w-100"
                                                placeholder="请选择右键菜单的位置"
                                                @change="departmentSelectChange($event, drawer.rcm.node.data)">
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
                                            v-for="(v, k) in drawer.rcm.node.data.departments"
                                            :key="k"
                                            :label="'【' + copywirting.departments[v].text + '】按 Shift 键出现'"
                                            label-width="200">
                                            <el-radio-group v-model.number="drawer.rcm.node.data.extends[v]">
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
                                :props="tree.props"
                                :node-key="tree.id"
                                :data="tree.menuSets.data"
                                :default-expanded-keys="tree.menuSets.expandKeys"
                                :allow-drag="menuSetsAllowDrag"
                                @node-click="menuSetsClick"
                                draggable
                                show-checkbox
                                ref="menuSets"
                                class="mt-2 mb-4">
                            </el-tree>
                        </el-tab-pane>
                    </el-tabs>
                    <a-drawer
                        :title="drawer.menuSets.title"
                        :placement="drawer.menuSets.placement"
                        :visible="drawer.menuSets.visible"
                        :width="drawer.menuSets.width"
                        @close="drawerClose"
                    >
                        <el-form
                            :label-position="drawer.menuSets.form.label.placement"
                            :label-width="drawer.menuSets.form.label.width"
                            :model="drawer.menuSets.node">
                            <el-form-item label="注册表名">
                                <el-input v-model="drawer.menuSets.node.registry_name"></el-input>
                            </el-form-item>
                            <el-form-item label="菜单名称">
                                <el-input v-model="drawer.menuSets.node.menu_name"></el-input>
                            </el-form-item>
                            <el-form-item label="程序路径">
                                <el-input v-model="drawer.menuSets.node.path"></el-input>
                            </el-form-item>
                            <el-form-item label="菜单图标">
                                <el-input v-model="drawer.menuSets.node.icon"></el-input>
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
        let copywirting = {
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
                    title: '右键菜单设置',
                    placement: 'right',
                    visible: false,
                    width: 720,
                    node: {},
                    rcmKeys: ['registry_name', 'menu_name', 'icon'],
                    menuSetKeys: ['registry_name', 'menu_name', 'path', 'icon'],
                    departments: [],
                    extends: {}
                },
                menuSets: {
                    title: '菜单设置',
                    placement: 'right',
                    visible: false,
                    width: 720,
                    form: {
                        label: {
                            placement: 'top',
                            width: '80px'
                        }
                    },
                    node: {}
                }
            }
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
            let rightClickMenu = localStorage.rightClickMenu

            if (!isEmpty(rightClickMenu)) {
                rightClickMenu = JSON.parse(rightClickMenu)
                this.tree.rcm.data = rightClickMenu.tree.rcm
            }

            this.menuSets()
            this.beforeLeave()
        },
        officialMenu: function () {
            const data = [{
                menu_name: '官方菜单',
                registry_name: 'official',
                children: [{
                    menu_name: '实用功能',
                    registry_name: 'practical-function',
                    children: [{
                        menu_name: '复制目标路径',
                        registry_name: 'copy-target-path',
                        path: 'mshta vbscript:clipboarddata.setdata("text","%1")(close)',
                        icon: '%SYSTEMROOT%\\explorer.exe'
                    }]
                }, {
                    menu_name: '电源控制',
                    registry_name: 'power-supply-control',
                    children: [{
                        menu_name: '锁定',
                        registry_name: 'lock'
                    }, {
                        menu_name: '注销',
                        registry_name: 'logout'
                    }, {
                        menu_name: '切换用户',
                        registry_name: 'switch-user'
                    }, {
                        menu_name: '睡眠',
                        registry_name: 'sleep'
                    }, {
                        menu_name: '休眠',
                        registry_name: 'dormancy'
                    }, {
                        menu_name: '重启',
                        registry_name: 'reboot'
                    }, {
                        menu_name: '关机',
                        registry_name: 'shutdown'
                    }]
                }]
            }]

            function handle (data) {
                for (let i in data) {
                    data[i] = Object.assign({
                        disabled: true,
                        group: 1
                    }, data[i])
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
            let node = treeNode.data
            if (!isEmpty(node.group) && node.group === 1) {
                return false
            }

            return true
        },
        menuSetsClick: function (node) {
            if (isEmpty(node.children)) {
                this.drawer.menuSets.visible = true
                this.drawer.menuSets.node = node
            }
        },
        drawerClose: function () {
            this.drawer.menuSets.visible = false
            this.drawer.rcm.visible = false
            let keys = this.$refs.menuSets.getCurrentKey()
            console.log(keys)
        },
        handleNodeClick: function (e, node, sel) {
            console.log(e, node, sel)
            this.drawer.rcm.node = node
            if (isEmpty(node.data.departments)) {
                Vue.set(node.data, 'departments', [])
            }
            if (isEmpty(node.data.extends)) {
                Vue.set(node.data, 'extends', {})
            }
            this.drawer.rcm.visible = true
            node.expanded = true
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
        departmentSelectChange: function (v, node) {
            for (let i in v) {
                if (isEmpty(node.extends[v[i]])) {
                    Vue.set(node.extends, v[i], 0)
                }
            }
        },
        beforeLeave: function () {
            let _this = this

            $(window).on('beforeunload', function () {
                let rightClickMenu = localStorage.rightClickMenu

                if (isEmpty(rightClickMenu)) {
                    rightClickMenu = {
                        tree: {
                            rcm: [],
                            menuSets: []
                        }
                    }
                } else {
                    rightClickMenu = JSON.parse(rightClickMenu)
                }

                rightClickMenu.tree.rcm = _this.tree.rcm.data
                rightClickMenu.tree.menuSets = _this.tree.menuSets.data.filter(function (val, index) {
                    return val.registry_name !== 'official'
                })

                localStorage.rightClickMenu = JSON.stringify(rightClickMenu)
            })
        }
    }
}
</script>
