<template>
    <div class="container">
        <div class="row">
            <div class="col-md-7 mb-3">
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
                                        @check-change="rcmCheckChange"
                                        draggable
                                        show-checkbox
                                        ref="rcm">
                                    </el-tree>
                                </el-col>
                                <el-col :span="3">
                                    <div class="text-right">
                                        <b-button
                                            @click="createMenu"
                                            variant="primary"
                                            size="sm"
                                            class="shadow-sm mb-3">
                                            新增菜单
                                        </b-button>
                                        <b-button
                                            v-if="visible.removeRcmBtn"
                                            @click="removeRcmNode"
                                            variant="danger"
                                            size="sm"
                                            class="shadow-sm">
                                            删除菜单
                                        </b-button>
                                    </div>
                                </el-col>
                            </el-row>
                            <b-button
                                @click="generateRcm"
                                variant="primary"
                                block
                                class="w-100">生成</b-button>
                            <a-drawer
                                v-if="!JW.isEmpty(drawer.rcm.nodeObj.data)"
                                :title="drawer.rcm.title"
                                :placement="drawer.rcm.placement"
                                :visible="drawer.rcm.visible"
                                :width="drawer.rcm.width"
                                @close="drawerClose">
                                <el-form
                                    v-if="!JW.isEmpty(drawer.rcm.nodeObj.data.children)"
                                    :ref="refs.form.rcm"
                                    :model="drawer.rcm.nodeObj.data"
                                    :rules="formRules.rcm"
                                    label-width="80px">
                                    <el-form-item
                                        v-for="(v, k) in drawer.rcm.rcmKeys"
                                        :label="copywirting.fields[v].text"
                                        :prop="v"
                                        :key="k">
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
                                </el-form>
                                <el-row v-else>
                                    <el-col>
                                        <el-form
                                            size="small"
                                            label-width="82px">
                                            <el-form-item
                                                v-for="(v, k) in drawer.rcm.menuSet.editable"
                                                :key="k"
                                                :label="copywirting.fields[v].text + '：'"
                                                class="mb-4">
                                                <el-input v-model="drawer.rcm.nodeObj.data[v]"></el-input>
                                            </el-form-item>
                                        </el-form>
                                    </el-col>
                                    <el-col
                                        v-for="(v, k) in drawer.rcm.menuSet.notEditable"
                                        :key="k"
                                        :span="24"
                                        class="mb-4">
                                        <div class="mr-2 mb-2 float-left">{{ copywirting.fields[v].text }}：</div>
                                        <div v-if="!JW.isEmpty(drawer.rcm.nodeObj.data.origin_registry_name) && !JW.isEmpty(drawer.rcm.tmpData[drawer.rcm.nodeObj.data.origin_registry_name])">{{ drawer.rcm.tmpData[drawer.rcm.nodeObj.data.origin_registry_name][v] }}</div>
                                    </el-col>
                                    <el-col>
                                        <b-button
                                            @click="toggleMenuSet"
                                            block
                                            variant="primary"
                                            class="w-100 mt-2">更换菜单</b-button>
                                    </el-col>
                                    <a-drawer
                                        :title="drawer.toggleMenuSet.title"
                                        :placement="drawer.toggleMenuSet.placement"
                                        :visible="drawer.toggleMenuSet.visible"
                                        :width="drawer.toggleMenuSet.width"
                                        @close="toggleMenuSetClose">
                                        <el-tree
                                            v-if="drawer.toggleMenuSet.visible"
                                            :props="tree.props"
                                            :node-key="tree.id"
                                            :data="tree.menuSets.data"
                                            :default-expanded-keys="tree.menuSets.expandKeys"
                                            @node-click="toggleMenuSetDetailClick"
                                            :ref="refs.tree.toggleMenuSet">
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
                                        <a-drawer
                                            :title="drawer.toggleMenuSetDetail.title"
                                            :placement="drawer.toggleMenuSetDetail.placement"
                                            :visible="drawer.toggleMenuSetDetail.visible"
                                            :width="drawer.toggleMenuSetDetail.width"
                                            @close="toggleMenuSetDetailClose">
                                            <el-row>
                                                <el-col
                                                    v-for="(v, k) in drawer.menuSets.form.items"
                                                    :key="k"
                                                    :span="24"
                                                    class="mb-4">
                                                    <div class="mb-2 float-left">{{ copywirting.fields[v].text }}：</div>
                                                    <div>{{ drawer.toggleMenuSetDetail.node[v] }}</div>
                                                </el-col>
                                                <el-col>
                                                    <b-button
                                                        @click="applyMenuSet"
                                                        variant="primary"
                                                        class="w-100"
                                                        block>选用</b-button>
                                                </el-col>
                                            </el-row>
                                        </a-drawer>
                                    </a-drawer>
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
                                        :ref="refs.tree.menuSets"
                                        @node-click="menuSetsClick"
                                        @node-drop="menuSetNodeDrop"
                                        @check-change="menuSetCheckChange"
                                        draggable
                                        show-checkbox>
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
                                        <b-button
                                            @click="createMenuSet"
                                            variant="primary"
                                            size="sm"
                                            class="shadow-sm mb-3">
                                            新增菜单
                                        </b-button>
                                        <b-button
                                            v-if="visible.removeMenuSetBtn"
                                            @click="removeMenuSetNode"
                                            variant="danger"
                                            size="sm"
                                            class="shadow-sm">
                                            删除菜单
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
                            :model="drawer.menuSets.node"
                            :rules="formRules.menuSet">
                            <el-form-item
                                v-for="(v, k) in drawer.menuSets.form.items"
                                :key="k"
                                :label="copywirting.fields[v].text"
                                :prop="v">
                                <el-input
                                    v-model="drawer.menuSets.node[v]"
                                    :disabled="drawer.menuSets.node.disabled === true"
                                    @blur="menuSetBlur(drawer.menuSets.node, v)"></el-input>
                            </el-form-item>
                        </el-form>
                    </a-drawer>
                </b-card>
            </div>
            <div class="col-md-5">
                <el-alert
                    title="请在本地环境中使用该应用"
                    type="warning"
                    class="mb-3 border-bottom"
                    show-icon>
                </el-alert>
                <b-card title="描述" bg-variant="white" border-variant="light" class="shadow-sm">
                    <div class="mb-3 pt-2">
                        <a href="https://github.com/GHBJayce/RightClickMenu" target="_blank" class="mr-2 d-inline-block align-top">
                            <i class="fab fa-github"></i> GHBJayce/RightClickMenu
                        </a>
                        <iframe src="https://ghbtns.com/github-btn.html?user=GHBJayce&repo=RightClickMenu&type=star" frameborder="0" scrolling="0" width="55px" height="20px"></iframe>
                        <iframe src="https://ghbtns.com/github-btn.html?user=GHBJayce&repo=RightClickMenu&type=fork" frameborder="0" scrolling="0" width="55px" height="20px"></iframe>
                    </div>
                    <!-- <div class="mb-3">
                        <img class="mr-2" src="https://img.shields.io/github/stars/GHBJayce/RightClickMenu.svg" alt="stars">
                        <img class="mr-2" src="https://img.shields.io/github/forks/GHBJayce/RightClickMenu.svg" alt="forks">
                        <img src="https://img.shields.io/github/license/GHBJayce/RightClickMenu.svg" alt="license">
                    </div> -->
                    <p>应用工具，可以将<b>windows</b>程序/命令设置在鼠标的右键菜单上。</p>
                    <p>能够生成右键菜单的注册表文件（含<b>创建</b>、<b>移除</b>两个文件）。</p>
                    <p class="mb-0 tips" title="Tips">
                        <i class="far fa-lightbulb text-warning mr-2 h6 mb-0"></i>{{ nowDescription }}
                    </p>
                </b-card>
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

                if (this.drawer.menuSets.originNode.registry_name !== value) {
                    let menuSetsTree = this.$refs.menuSets
                    let existNode = menuSetsTree.getNode(value)
                    if (!isEmpty(existNode)) {
                        callback(new Error('该' + text + '已存在！'))
                    }
                }
                callback()
            },
            menuName: (rule, value, callback) => {
                let text = copywirting.fields.menu_name.text
                if (isEmpty(value)) {
                    callback(new Error(text + '不能为空！'))
                }

                callback()
            },
            path: (rule, value, callback) => {
                let text = copywirting.fields.path.text
                if (isEmpty(value)) {
                    callback(new Error(text + '不能为空！'))
                }

                callback()
            }
        }
        let rulesRcm = {
            registryName: (rule, value, callback) => {
                let text = copywirting.fields.registry_name.text
                if (isEmpty(value)) {
                    callback(new Error(text + '不能为空！'))
                }

                if (this.drawer.rcm.originNode.registry_name !== value) {
                    let rcmTree = this.$refs.rcm
                    let existNode = rcmTree.getNode(value)
                    if (!isEmpty(existNode)) {
                        callback(new Error('该' + text + '已存在！'))
                    }
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
                            registry_name: 'copy-target-path',
                            origin_registry_name: 'copy-target-path'
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
                    tmpData: {},
                    originNode: {}
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
                        },
                        items: ['registry_name', 'menu_name', 'path', 'icon']
                    },
                    node: {},
                    originNode: {}
                },
                toggleMenuSet: {
                    title: '更换菜单',
                    placement: 'right',
                    visible: false,
                    width: 500,
                    node: {}
                },
                toggleMenuSetDetail: {
                    title: '菜单信息',
                    placement: 'right',
                    visible: false,
                    width: 500,
                    node: {}
                }
            },
            formRules: {
                rcm: {
                    registry_name: [{
                        validator: rulesRcm.registryName,
                        trigger: 'blur'
                    }],
                    menu_name: [{
                        validator: rules.menuName,
                        trigger: 'blur'
                    }]
                },
                menuSet: {
                    registry_name: [{
                        validator: rules.registryName,
                        trigger: 'blur'
                    }],
                    menu_name: [{
                        validator: rules.menuName,
                        trigger: 'blur'
                    }],
                    path: [{
                        validator: rules.path,
                        trigger: 'blur'
                    }]
                }
            },
            refs: {
                tree: {
                    rcm: 'rcm',
                    menuSets: 'menuSets',
                    toggleMenuSet: 'toggleMenuSet'
                },
                form: {
                    rcm: 'drawerRcmForm',
                    menuSet: 'drawerMenuSetForm'
                }
            },
            visible: {
                removeRcmBtn: false,
                removeMenuSetBtn: false
            },
            descriptions: ['支持多层右键菜单，但是windows会限制右键菜单的数量', '在你离开页面之前，它会自动保存你在操作台中的数据', '实际右键菜单的显示顺序跟注册表名有关', '应用更好的实现方式应该是直接和系统进行交互，不需要任何环境的依赖，减少用户的麻烦', '这个应用是拿来练习PHP设计模式的实例，为了更好地理解设计模式'],
            nowDescription: ''
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
            this.getNowDescription()
            this.menuSets()
            let rightClickMenu = localStorage.rightClickMenu

            if (!isEmpty(rightClickMenu)) {
                rightClickMenu = JSON.parse(rightClickMenu)
                if (!isEmpty(rightClickMenu.tree.rcm)) {
                    this.tree.rcm.data = rightClickMenu.tree.rcm
                }
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
                this.drawer.menuSets.originNode = deepCopy(node)
            }
        },
        drawerClose: function () {
            let _this = this
            if (!isEmpty(this.$refs[this.refs.form.menuSet])) {
                this.$refs[this.refs.form.menuSet].validate(function (valid) {
                    if (valid) {
                        _this.drawer.menuSets.visible = false
                    } else {
                        _this.$message.warning('必须填写正确才能下一步操作')
                        return false
                    }
                })
            }
            if (!isEmpty(this.$refs[this.refs.form.rcm])) {
                this.$refs[this.refs.form.rcm].validate(function (valid) {
                    if (valid) {
                        rcmDrawerClose()
                    } else {
                        _this.$message.warning('必须填写正确才能下一步操作')
                        return false
                    }
                })
            } else if (!isEmpty(this.drawer.rcm.nodeObj) && isEmpty(this.drawer.rcm.nodeObj.data.children)) {
                rcmDrawerClose()
            }
            function rcmDrawerClose () {
                _this.drawer.rcm.visible = false
            }
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
            // eslint-disable-next-line
            this.drawer.rcm.originNode = deepCopy(node)
            this.drawer.rcm.visible = true
            nodeObj.expanded = true

            this.getMenuSetsDetail(node)
        },
        getMenuSetsDetail: function (node) {
            // 用于获取、展示菜单集中的数据
            if (isEmpty(node.children) && !isEmpty(node.origin_registry_name)) {
                let menuSetNodeObj = this.$refs[this.refs.tree.menuSets].getNode(node.origin_registry_name)
                if (!isEmpty(menuSetNodeObj)) {
                    this.drawer.rcm.tmpData[node.origin_registry_name] = menuSetNodeObj.data
                }
            }
        },
        createMenu: function () {
            this.tree.rcm.data.push({
                registry_name: this.generateUniqueRegistryName(),
                menu_name: '新的菜单'
            })
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
                _this.saveLocalStore()
            })
        },
        createMenuSet: function () {
            this.tree.menuSets.data.push({
                registry_name: this.generateUniqueRegistryName(),
                menu_name: '新的菜单集'
            })
        },
        menuSetNodeDrop: function (before, after, inner, event) {
            let nodeObj = this.$refs.menuSets.getNode(before.data)
            if (isEmpty(nodeObj.data.children)) {
                Vue.set(nodeObj.data, 'path', nodeObj.data.path)
                Vue.set(nodeObj.data, 'icon', nodeObj.data.icon)
                Vue.delete(nodeObj.data, 'children')
            } else {
                Vue.delete(nodeObj.data, 'path')
                Vue.delete(nodeObj.data, 'icon')
            }
            if (isEmpty(after.data.children)) {
                Vue.set(after.data, 'path', after.data.path)
                Vue.set(after.data, 'icon', after.data.icon)
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
        menuSetCheckChange: function (nodeObj, isCheck, childIsCheck) {
            let nodes = this.$refs.menuSets.getCheckedNodes()
            this.visible.removeMenuSetBtn = !isEmpty(nodes)
        },
        generateUniqueRegistryName: function () {
            return md5(new Date().getTime())
        },
        removeMenuSetNode: function () {
            this.$confirm({
                title: '确认删除选中的节点？',
                okText: '确认',
                cancelText: '再想想',
                onOk: () => {
                    let menuSetsTree = this.$refs.menuSets
                    let nodes = menuSetsTree.getCheckedNodes()

                    for (let i in nodes) {
                        menuSetsTree.remove(nodes[i])
                    }
                    this.menuSetCheckChange('', '', '')
                }
            })
        },
        rcmCheckChange: function (nodeObj, isCheck, childIsCheck) {
            let nodes = this.$refs.rcm.getCheckedNodes()
            this.visible.removeRcmBtn = !isEmpty(nodes)
        },
        removeRcmNode: function () {
            this.$confirm({
                title: '确认删除选中的节点？',
                okText: '确认',
                cancelText: '再想想',
                onOk: () => {
                    let rcmTree = this.$refs.rcm
                    let nodes = rcmTree.getCheckedNodes()

                    for (let i in nodes) {
                        rcmTree.remove(nodes[i])
                    }
                    this.rcmCheckChange('', '', '')
                }
            })
        },
        toggleMenuSet: function () {
            this.drawer.toggleMenuSet.visible = true
        },
        toggleMenuSetClose: function () {
            this.drawer.toggleMenuSet.visible = false
        },
        toggleMenuSetDetailClick: function (node) {
            if (isEmpty(node.children)) {
                this.drawer.toggleMenuSetDetail.visible = true
                this.drawer.toggleMenuSetDetail.node = node
            }
        },
        toggleMenuSetDetailClose: function () {
            this.drawer.toggleMenuSetDetail.visible = false
        },
        applyMenuSet: function () {
            let rcmMenuSetNode = this.drawer.rcm.nodeObj.data
            let togglemenuSetNode = this.drawer.toggleMenuSetDetail.node

            rcmMenuSetNode.origin_registry_name = togglemenuSetNode.registry_name
            for (let i in this.drawer.menuSets.form.items) {
                let key = this.drawer.menuSets.form.items[i]
                if (key !== 'registry_name') {
                    rcmMenuSetNode[key] = togglemenuSetNode[key]
                }
            }
            this.getMenuSetsDetail(rcmMenuSetNode)
            this.drawer.toggleMenuSetDetail.visible = false
            this.drawer.toggleMenuSet.visible = false
        },
        saveLocalStore: function () {
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

            rightClickMenu.tree.rcm = this.tree.rcm.data
            rightClickMenu.tree.menuSets = this.tree.menuSets.data.filter(function (val, index) {
                return val.registry_name !== 'official'
            })

            localStorage.rightClickMenu = JSON.stringify(rightClickMenu)
        },
        getNowDescription: function () {
            this.nowDescription = this.takeTurnsDescriptions()

            setTimeout(() => {
                this.getNowDescription()
            }, 7000)
        },
        takeTurnsDescriptions: function () {
            let descriptions = this.descriptions
            let key = generateKey()
            let lastKey = window.takeTurnsDescriptionKey

            if (lastKey === undefined || lastKey !== key) {
                window.takeTurnsDescriptionKey = key

                return descriptions[key]
            }

            function generateKey () {
                return Math.floor(Math.random() * descriptions.length)
            }

            return this.takeTurnsDescriptions()
        },
        generateRcm: function () {
            let rcmRef = this.$refs[this.refs.tree.rcm]
            let checkedList = rcmRef.getCheckedNodes()
            checkedList = checkedList.concat(rcmRef.getHalfCheckedNodes())
            console.log(checkedList, rcmRef.getHalfCheckedNodes())
            let tree = {}
            for (let i in checkedList) {
                let id = checkedList[i].$treeNodeId
                let node = tree[id] = checkedList[i]
                let checkedNode = rcmRef.getNode(checkedList[i])
                tree[id].pid = checkedNode.parent.level ? checkedNode.parent.id : 0

                // 验证数据
                if (isEmpty(node.children)) {
                    if (isEmpty(node.path)) {
                        // this.$message.error('必须选用一个菜单：' + node.menu_name)
                    }
                }
            }

            let data = test(tree)
            // 堆栈溢出，让后台处理
            function test (list, pid, res) {
                pid = pid === undefined ? 0 : pid
                res = res || {}

                for (let i in list) {
                    let node = list[i]
                    if (node.pid === pid) {
                        res[node.pid] = node
                        let children = test(list, node.id)
                        if (!isEmpty(children)) {
                            delete res[node.pid].children
                            res[node.pid].children = children
                        }
                    }
                }

                return res
            }
            console.log(data)
        }
    }
}
</script>

<style>
.tips {
    color: #e2af50
}
</style>
