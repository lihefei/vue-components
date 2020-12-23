<template>
    <div class="langue-switch">
        <el-dropdown @command="handleCommand" trigger="click">
            <span class="el-dropdown-link" v-if="btnType === 'btn'">
                {{ langueName }}
                <i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <span class="el-dropdown-link langue-icon" v-else>
                <i class="iconfont icon-zhongyingwen"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
                <el-dropdown-item
                    v-for="item in list"
                    :command="item.val"
                    :key="item.val"
                    :disabled="item.val === langue"
                >{{item.label}}</el-dropdown-item>
            </el-dropdown-menu>
        </el-dropdown>
    </div>
</template>
<script>
import { mapGetters } from 'vuex';
export default {
    props: {
        /**
         * @param {String} btnType
         * {
         *      btn // 显示按钮
         *      icon // 显示图标
         * }
         */
        btnType: {
            type: String,
            default: 'btn'
        }
    },
    data: () => ({
        list: [
            { val: 'zh_CN', label: '简体中文' },
            { val: 'en_US', label: 'English' }
        ]
    }),
    computed: {
        ...mapGetters(['langue']),
        langueName() {
            return this.list.filter(item => {
                return item.val === this.langue;
            })[0].label;
        }
    },
    methods: {
        /**
         * 语言切换
         */
        handleCommand(val) {
            this.$store.dispatch('alterLangue', val);
            sessionStorage.setItem('langue', val);
            if (this.$route.name !== 'login') {
                this.$store.dispatch('changeMenusLang');
            }
            // console.log(this.$lodash);
            this.$i18n.locale = val.substring(0, 2);
            this.$message.success('Switch Language Success');
        }
    },
    mounted() {}
};
</script>
<style lang="scss" scoped>
.langue-switch {
    display: inline-block;
    cursor: pointer;
    .el-dropdown-link {
        color: #409eff;
    }
    .langue-icon {
        color: #000;
        & > i {
            font-size: 20px;
        }
        padding: 0 10px;
    }
}
</style>