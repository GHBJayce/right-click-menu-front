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
                                v-if="!JW.isEmpty(drawer.rcm.nodeObj.data)"
                                :title="drawer.rcm.title"
                                :placement="drawer.rcm.placement"
                                :visible="drawer.rcm.visible"
                                :width="drawer.rcm.width"
                                @close="drawerClose">
                                <el-form v-if="!JW.isEmpty(drawer.rcm.nodeObj.data.children)" ref="form" :model="form" label-width="80px">
                                    <el-form-item v-for="(v, k) in drawer.rcm.rcmKeys" :key="k" :label="copywirting.fields[v].text">
                                        <el-input v-model="drawer.rcm.nodeObj.data[v]"></el-input>
                                    </el-form-item>
                                    <template v-if="drawer.rcm.nodeObj.level === 1">
                                        <el-form-item label="菜单位置">
                                            <el-select
                                                v-model="drawer.rcm.nodeObj.data.departments"
                                                multiple
                                                clearable
                                                filterable
                                                class="w-100"
                                                placeholder="请选择右键菜单的位置"
                                                @change="departmentSelectChange($event, drawer.rcm.nodeObj.data)">
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
                                            v-for="(v, k) in drawer.rcm.nodeObj.data.departments"
                                            :key="k"
                                            :label="'【' + copywirting.departments[v].text + '】按 Shift 键出现'"
                                            label-width="200">
                                            <el-radio-group v-model.number="drawer.rcm.nodeObj.data.extends[v]">
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
                                    <el-col>
                                        <el-form
                                            size="small"
                                            label-width="82px">
                                            <el-form-item
                                                v-for="(v, k) in drawer.rcm.menuSet.editable"
                                                :key="k"
                                                :label="copywirting.fields[v].text + '：'">
                                                <el-input v-model="drawer.rcm.nodeObj.data[v]"></el-input>
                                            </el-form-item>
                                        </el-form>
                                    </el-col>
                                    <el-col
                                        v-for="(v, k) in drawer.rcm.menuSet.notEditable"
                                        :key="k"
                                        :span="24"
                                        class="mb-4">
                                        <span class="mr-2">{{ copywirting.fields[v].text }}：</span>
                                        <span v-if="!JW.isEmpty(drawer.rcm.nodeObj.data.registry_name) && !JW.isEmpty(drawer.rcm.tmpData[drawer.rcm.nodeObj.data.registry_name])">{{ drawer.rcm.tmpData[drawer.rcm.nodeObj.data.registry_name][v] }}</span>
                                    </el-col>
                                    <el-col>
                                        <b-button
                                            block
                                            variant="primary"
                                            class="w-100 mt-2">更换菜单</b-button>
                                    </el-col>
                                </el-row>
                            </a-drawer>
                        </el-tab-pane>
                        <el-tab-pane label="菜单集" name="menuSets">
                            <el-row class="mt-2 mb-4">
                                <el-col :span="21">
                                    <el-tree
                                        :props="tree.props"
                                        :node-key="tree.id"
                                        :data="tree.menuSets.data"
                                        :default-expanded-keys="tree.menuSets.expandKeys"
                                        :allow-drag="menuSetsAllowDrag"
                                        :allow-drop="menuSetAllowDrop"
                                        @node-click="menuSetsClick"
                                        @node-drop="menuSetNodeDrop"
                                        draggable
                                        show-checkbox
                                        ref="menuSets">
                                        <span slot-scope="{ node, data }">
                                            <span>{{ node.label }}</span>
                                            <span v-if="!JW.isEmpty(data.labels)">
                                                <b-badge
                                                    v-for="(v, k) in data.labels"
                                                    :key="k"
                                                    variant="primary"
                                                    class="ml-1"
                                                    :title="title">{{ v.text }}</b-badge>
                                            </span>
                                        </span>
                                    </el-tree>
                                </el-col>
                                <el-col :span="3">
                                    <div class="text-right">
                                        <b-button @click="createMenuSet" variant="primary" size="sm" class="shadow-sm">
                                            新增菜单
                                        </b-button>
                                    </div>
                                </el-col>
                            </el-row>
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
                            :ref="refs.form.menuSet"
                            :label-position="drawer.menuSets.form.label.placement"
                            :label-width="drawer.menuSets.form.label.width"
                            :model="drawer.menuSets.form.data"
                            :rules="formRules">
                            <el-form-item
                                v-for="(v, k) in drawer.menuSets.form.items"
                                :key="k"
                                :label="copywirting.fields[v].text"
                                :prop="v">
                                <el-input
                                    v-model="drawer.menuSets.form.data[v]"
                                    :disabled="drawer.menuSets.node.disabled === true"
                                    @blur="menuSetBlur(drawer.menuSets.form.data, v)"></el-input>
                            </el-form-item>
                            <b-button
                                @click="menuSetSubmit(drawer.menuSets.node)"
                                variant="primary"
                                class="w-100">保存</b-button>
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
        let rules = {
            registryName: (rule, value, callback) => {
                let text = copywirting.fields.registry_name.text
                if (isEmpty(value)) {
                    callback(new Error(text + '不能为空！'))
                }

                let menuSetsTree = this.$refs.menuSets
                let existNode = menuSetsTree.getNode(value)
                // todolist 注册表名的唯一性需要另找解决方法
                if (!isEmpty(existNode)) {
                    callback(new Error('该' + text + '已存在！'))
                }

                callback()
            },
            menuName: (rule, value, callback) => {
                let text = copywirting.fields.menu_name.text
                if (isEmpty(value)) {
                    callback(new Error(text + '不能为空！'))
                }

                callback()
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
                    expandKeys: ['practical-function', 'power-supply-control', 'hashtab-Get-FileHash', 'hashtab-certutil']
                }
            },
            drawer: {
                rcm: {
                    title: '右键菜单设置',
                    placement: 'right',
                    visible: false,
                    width: 720,
                    nodeObj: {},
                    rcmKeys: ['registry_name', 'menu_name', 'icon'],
                    menuSet: {
                        editable: ['menu_name'],
                        notEditable: ['registry_name', 'path', 'icon']
                    },
                    departments: [],
                    extends: {},
                    tmpData: {}
                },
                menuSets: {
                    title: '菜单设置',
                    placement: 'right',
                    visible: false,
                    width: 720,
                    form: {
                        data: {},
                        label: {
                            placement: 'top',
                            width: '80px'
                        },
                        items: ['registry_name', 'menu_name', 'path', 'icon']
                    },
                    node: {}
                }
            },
            formRules: {
                registry_name: [{
                    validator: rules.registryName,
                    trigger: 'blur'
                }],
                menu_name: [{
                    validator: rules.menuName,
                    trigger: 'blur'
                }]
            },
            refs: {
                form: {
                    menuSet: 'drawerMenuSetForm'
                }
            },
            registrysName: []
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
            let rightClickMenu = localStorage.rightClickMenu

            if (!isEmpty(rightClickMenu)) {
                rightClickMenu = JSON.parse(rightClickMenu)
                this.tree.rcm.data = rightClickMenu.tree.rcm
                this.tree.menuSets.data = this.tree.menuSets.data.concat(rightClickMenu.tree.menuSets)
            }
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
                        registry_name: 'lock',
                        path: 'Rundll32 User32.dll,LockWorkStation',
                        icon: ''
                    }, {
                        menu_name: '注销',
                        registry_name: 'logout',
                        path: 'Shutdown -l',
                        icon: ''
                    }, {
                        menu_name: '切换用户',
                        registry_name: 'switch-user',
                        path: 'tsdiscon.exe',
                        icon: ''
                    }, {
                        menu_name: '睡眠',
                        registry_name: 'sleep',
                        path: 'rundll32.exe powrprof.dll,SetSuspendState Sleep',
                        icon: ''
                    }, {
                        menu_name: '休眠',
                        registry_name: 'dormancy',
                        path: 'Shutdown -h',
                        icon: ''
                    }, {
                        menu_name: '重启',
                        registry_name: 'reboot',
                        path: 'Shutdown -r -f -t 00',
                        icon: ''
                    }, {
                        menu_name: '关机',
                        registry_name: 'shutdown',
                        path: 'Shutdown -s -f -t 00',
                        icon: ''
                    }]
                }, {
                    labels: [{
                        text: 'PowerShell'
                    }, {
                        text: 'Get-FileHash'
                    }],
                    menu_name: '哈希校验',
                    registry_name: 'hashtab-Get-FileHash',
                    children: [{
                        menu_name: 'MD5',
                        registry_name: 'MD5',
                        path: 'PowerShell Get-FileHash -Algorithm MD5 "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA1',
                        registry_name: 'SHA1',
                        path: 'PowerShell Get-FileHash -Algorithm SHA1 "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA256',
                        registry_name: 'SHA256',
                        path: 'PowerShell Get-FileHash -Algorithm SHA256 "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA384',
                        registry_name: 'SHA384',
                        path: 'PowerShell Get-FileHash -Algorithm SHA384 "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA512',
                        registry_name: 'SHA512',
                        path: 'PowerShell Get-FileHash -Algorithm SHA512 "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'MACTripleDES',
                        registry_name: 'MACTripleDES',
                        path: 'PowerShell Get-FileHash -Algorithm MACTripleDES "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'RIPEMD160',
                        registry_name: 'RIPEMD160',
                        path: 'PowerShell Get-FileHash -Algorithm RIPEMD160 "%1" | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }]
                }, {
                    labels: [{
                        text: 'cmd'
                    }, {
                        text: 'certutil'
                    }],
                    menu_name: '哈希校验',
                    registry_name: 'hashtab-certutil',
                    children: [{
                        menu_name: 'MD5',
                        registry_name: 'certutil-MD5',
                        path: 'cmd certutil -hashfile "%1" MD5 | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA1',
                        registry_name: 'certutil-SHA1',
                        path: 'cmd certutil -hashfile "%1" SHA1 | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA256',
                        registry_name: 'certutil-SHA256',
                        path: 'cmd certutil -hashfile "%1" SHA256 | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA384',
                        registry_name: 'certutil-SHA384',
                        path: 'cmd certutil -hashfile "%1" SHA384 | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
                    }, {
                        menu_name: 'SHA512',
                        registry_name: 'certutil-SHA512',
                        path: 'cmd certutil -hashfile "%1" SHA512 | format-list;“按任意键退出...”;[Console]::Readkey() | Out-Null;exit',
                        icon: ''
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
                // eslint-disable-next-line
                this.drawer.menuSets.form.data = node
            }
        },
        drawerClose: function () {
            this.drawer.menuSets.visible = false
            this.drawer.rcm.visible = false
        },
        handleNodeClick: function (node, nodeObj, treeObj) {
            this.drawer.rcm.nodeObj = nodeObj
            if (!isEmpty(nodeObj.data.children)) {
                if (isEmpty(nodeObj.data.departments)) {
                    Vue.set(nodeObj.data, 'departments', [])
                }
                if (isEmpty(nodeObj.data.extends)) {
                    Vue.set(nodeObj.data, 'extends', {})
                }
            }
            this.drawer.rcm.visible = true
            nodeObj.expanded = true

            if (isEmpty(node.children) && !isEmpty(node.registry_name)) {
                let menuSetNodeObj = this.$refs.menuSets.getNode(node.registry_name)
                this.drawer.rcm.tmpData[node.registry_name] = menuSetNodeObj.data
            }
        },
        createMenu: function () {
            this.tree.rcm.data.push({
                registry_name: '',
                menu_name: '新的菜单'
            })
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
        },
        createMenuSet: function () {
            this.tree.menuSets.data.push({
                registry_name: '',
                menu_name: '新的菜单集'
            })
        },
        menuSetNodeDrop: function (before, after, inner, event) {
            let nodeObj = this.$refs.menuSets.getNode(before.data)
            if (isEmpty(nodeObj.data.children)) {
                Vue.set(nodeObj.data, 'path', '')
                Vue.set(nodeObj.data, 'icon', '')
                Vue.delete(nodeObj.data, 'children')
            } else {
                Vue.delete(nodeObj.data, 'path')
                Vue.delete(nodeObj.data, 'icon')
            }
            if (isEmpty(after.data.children)) {
                Vue.set(after.data, 'path', '')
                Vue.set(after.data, 'icon', '')
                Vue.delete(after.data, 'children')
            } else {
                Vue.delete(after.data, 'path')
                Vue.delete(after.data, 'icon')
            }
        },
        menuSetAllowDrop: function (node, dropNode, type) {
            if (dropNode.data.group === 1) {
                return false
            }

            return true
        },
        menuSetBlur: function (data, key) {
            if (key === 'registry_name') {
                data[key] = data[key].toLocaleLowerCase()
            }
        },
        // 需要解决：若即时存储，需要解决不依靠点击保存按钮，能够验证注册表名的唯一性；若依靠保存按钮，需要解决，在验证数据正确性后，将编辑的数据同步到当前节点数据上
        menuSetSubmit: function (node) {
            let _this = this
            if (!isEmpty(this.$refs[this.refs.form.menuSet])) {
                this.$refs[this.refs.form.menuSet].validate(function (valid) {
                    if (valid) {
                        // node = _this.drawer.menuSets.form.data
                        console.log(_this.drawer.menuSets.node, _this.tree.menuSets)
                    } else {
                        _this.$message.warning('必须填写正确才能下一步操作')
                        return false
                    }
                })
            }
        }
    }
}
</script>
