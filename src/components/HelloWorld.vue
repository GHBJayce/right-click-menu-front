<template>
    <div class="container">
        <div class="row">
            <div class="col-md-7">
                <b-card title="操作台" bg-variant="white" border-variant="light" class="shadow-sm">
                    <el-tabs v-model="activeName">
                        <el-tab-pane label="右键菜单" name="rcm">
                            <div class="text-left">
                                <b-button @click="createMenu" variant="primary" size="sm">新增菜单</b-button>
                            </div>
                            <el-tree
                                ref="rcm"
                                draggable
                                show-checkbox
                                node-key="id"
                                class="mt-2 mb-4"
                                :data="RCMTree"
                                :props="defaultProps"
                                :default-expanded-keys="RCMTreeExpandKeys"
                                @node-click="handleNodeClick">
                            </el-tree>
                            <b-button variant="primary" block class="w-100">生成</b-button>
                            <el-dialog title="新增菜单" :visible.sync="createMenuDialogVisible">
                                <el-form label-position="right" label-width="80px" :model="currentMenuDrawer.data">
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
                                    <b-button variant="primary" block class="w-100 mt-5">创建</b-button>
                                </el-form>
                                <p>选择</p>
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
                            </el-dialog>
                            <a-drawer
                                title="菜单设置"
                                placement="right"
                                :visible="rcmDrawer.show"
                                :width="rcmDrawer.width"
                                @close="onClose"
                            >
                                <el-form ref="form" :model="form" label-width="80px">
                                    <el-form-item label="注册表名">
                                        <el-input v-model="form.name"></el-input>
                                    </el-form-item>
                                    <el-form-item label="菜单名称">
                                        <el-input v-model="form.name"></el-input>
                                    </el-form-item>
                                    <el-form-item label="菜单图标">
                                        <el-input v-model="form.name"></el-input>
                                    </el-form-item>
                                    <el-form-item label="菜单位置">
                                        <el-select v-model="value5" multiple placeholder="请选择右键菜单的位置" class="w-100">
                                            <el-option
                                                v-for="item in options"
                                                :key="item.value"
                                                :label="item.label"
                                                :value="item.value">
                                            </el-option>
                                        </el-select>
                                    </el-form-item>
                                    <el-form-item label="特殊资源">
                                        <el-radio-group v-model="form.resource">
                                        <el-radio label="线上品牌商赞助"></el-radio>
                                        <el-radio label="线下场地免费"></el-radio>
                                        </el-radio-group>
                                    </el-form-item>
                                    <el-form-item>
                                        <el-button type="primary" @click="onSubmit">立即创建</el-button>
                                        <el-button>取消</el-button>
                                    </el-form-item>
                                </el-form>
                                <el-row>
                                    <el-col :span="24" class="mb-2">
                                        <strong>注册表名：</strong>
                                        <span></span>
                                    </el-col>
                                    <el-col :span="24" class="mb-2">
                                        <strong>菜单名称：</strong>
                                        <span></span>
                                    </el-col>
                                    <el-col :span="24" class="mb-2">
                                        <strong>程序路径：</strong>
                                        <span></span>
                                    </el-col>
                                    <el-col :span="24" class="mb-2">
                                        <strong>菜单图标：</strong>
                                        <span></span>
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
        return {
            activeName: 'rcm',
            tree: {
                rcm: {
                    expandKeys: [],
                    data: []
                },
                menuSets: {
                    expandKeys: [],
                    data: []
                }
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
            menuVisible: false,
            RCMTreeExpandKeys: ['my-right-click-menu'],
            RCMTree: [{
                label: '我的右键菜单',
                id: 'my-right-click-menu',
                children: [{
                    label: '右键菜单示例',
                    id: 'demo',
                    children: [{
                        label: '复制目标路径',
                        setid: 'copy-target-path'
                    }]
                }]
            }, {
                label: '我的右键菜单',
                id: 'my-right-click-menu',
                children: [{
                    label: '复制目标路径',
                    setid: 'copy-target-path'
                }]
            }],
            defaultProps: {
                children: 'children',
                label: 'label'
            },
            visible: false,
            createMenuDialogVisible: false,
            options: [{
                value: '选项1',
                label: '黄金糕'
            }, {
                value: '选项2',
                label: '双皮奶'
            }, {
                value: '选项3',
                label: '蚵仔煎'
            }, {
                value: '选项4',
                label: '龙须面'
            }, {
                value: '选项5',
                label: '北京烤鸭'
            }],
            value5: [],
            form: {
                name: '',
                region: '',
                date1: '',
                date2: '',
                delivery: false,
                type: [],
                resource: '',
                desc: ''
            }
        }
    },
    mounted () {
        this.init()
        /*
            eslint-disable-next-line
            eslint-disable no-undef
        */
        this.$nextTick(function () {
            console.log(this)
        })
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
        },
        createMenu: function () {
            this.createMenuDialogVisible = true
        }
    }
}
</script>
