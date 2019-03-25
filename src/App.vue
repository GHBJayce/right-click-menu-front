<template>
    <div id="app">
        <b-navbar toggleable="lg" type="dark" variant="dark">
            <div class="container">
                <b-navbar-brand :href="navbar.logo.url">{{ navbar.logo.name }}</b-navbar-brand>

                <b-navbar-toggle target="nav_collapse" />

                <b-collapse is-nav id="nav_collapse" class="ml-5">
                    <b-navbar-nav>
                        <template v-for="(v, k) in navbar.menu.left">
                            <b-nav-item v-if="v.show" :key="k" :href="v.url" target="_blank">{{ v.name }}</b-nav-item>
                        </template>
                    </b-navbar-nav>

                    <b-navbar-nav class="ml-auto">
                        <template v-for="(v, k) in navbar.menu.right">
                            <b-nav-item v-if="v.show && JW.isEmpty(v.children)" :key="k" :href="v.url" target="_blank">{{ v.name }}</b-nav-item>
                            <b-nav-item-dropdown v-else :key="k" :text="v.name" right>
                                <b-dropdown-item v-for="(cv, k) in v.children" :key="k" :href="cv.url" target="_blank">{{ cv.name }}</b-dropdown-item>
                            </b-nav-item-dropdown>
                        </template>
                    </b-navbar-nav>
                </b-collapse>
            </div>
        </b-navbar>
        <div class="bg-light pt-4 pb-4">
            <router-view/>
        </div>
    </div>
</template>

<script>
export default {
    name: 'App',
    data () {
        return {
            navbar: {
                logo: {
                    name: 'RightClickMenu',
                    url: '#/'
                },
                menu: {
                    left: [{
                        name: '首页',
                        url: '#/index',
                        show: false
                    }],
                    right: [{
                        name: 'For Github',
                        url: 'https://github.com/GHBJayce/RightClickMenu',
                        show: true
                    }]
                }
            }
        }
    },
    mounted () {

    }
}
</script>
