<template>
    <el-popover placement="bottom" width="229" trigger="click" :disabled="disabled">
        <ul class="icon-list">
            <li v-for="(item, index) in icons" :key="index" :class="{ active: item === activeIndex }" @click="setValue(item)">
                <span :class="`icon-square icon-${item}`" :style="{ 'background-image': `url(${getValue(item)})` }"></span>
            </li>
        </ul>
        <div slot="reference" class="icon-picker" :class="{ 'is-disabled': disabled }">
            <span :class="`icon-square icon-${activeIndex}`" :style="{ 'background-image': `url(${getValue(activeIndex)})` }"></span>
            <span class="icon-picker-suffix">
                <i class="el-icon-arrow-up"></i>
            </span>
        </div>
    </el-popover>
</template>

<script>
import { getDeviceIcon } from '@/utils/device-icons'; //设备图标
export default {
    name: 'IconPicker',
    components: {},
    props: {
        disabled: {
            type: Boolean,
            default: false,
        },
        defaultValue: {
            type: String,
            default: 'default',
        },
    },
    data() {
        return {
            activeIndex: '',
            icons: ['default', 'person', 'vehicle', 'ship', 'aircraft'],
        };
    },
    watch: {
        activeIndex() {
            this.$emit('change', this.activeIndex);
        },
    },
    computed: {},
    mounted() {
        this.activeIndex = this.defaultValue;
    },
    methods: {
        /**
         * 设置当前图标选项值
         * @param {String} val 图标标识
         */
        setValue(val = 'default') {
            this.activeIndex = val;
        },
        /**
         * 获取图标
         * @param {String} icon 图标标识
         */
        getValue(icon) {
            return getDeviceIcon(icon);
        },
    },
};
</script>

<style lang="scss" scoped>
.icon-list {
    float: left;
    max-height: 185px;
    margin: 5px;
    border: solid #eaeefb;
    border-width: 1px 0 0 1px;
    overflow: hidden;
    overflow-y: auto;

    li {
        float: left;
        width: 48px;
        height: 48px;
        padding: 8px;
        border: 1px solid;
        border-color: transparent #eee #eee transparent;
        color: #666;
        text-align: center;
        cursor: pointer;

        &:hover {
            background-color: rgba(0, 0, 0, 0.03);
        }

        &.active {
            $blue: #0099ff;
            position: relative;
            z-index: 1;
            border: 1px solid $blue;

            &::before,
            &::after {
                content: '';
                position: absolute;
            }
            &::before {
                right: 0;
                bottom: 0;
                width: 0;
                height: 0;
                border: 8px solid;
                border-color: transparent $blue $blue transparent;
            }
            &::after {
                right: 2px;
                bottom: 2px;
                width: 4px;
                height: 7px;
                border-right: 1px solid #fff;
                border-bottom: 1px solid #fff;
                transform: rotate(45deg);
            }
        }
    }
}

.icon-square {
    display: inline-block;
    width: 30px;
    height: 30px;
    background-size: 120% 120%;
    background-position: center 0;
    background-repeat: no-repeat;
}

.icon-default {
    background-size: 130% 130%;
    background-position: center -40%;
}

.icon-picker {
    display: inline-block;
    position: relative;
    width: 100%;
    height: 32px;
    padding: 0 15px;
    line-height: 14px;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
    font-size: 13px;
    cursor: pointer;
    &:hover {
        border-color: #c0c4cc;
    }

    &.is-disabled {
        border-color: #e4e7ed;
        color: #c0c4cc;
        background-color: #f5f7fa;
        cursor: not-allowed;

        .icon-square {
            opacity: 0.5;
        }
    }

    .icon-picker-suffix {
        position: absolute;
        top: 0;
        right: 5px;
        height: 100%;
        color: #c0c4cc;
        text-align: center;
        transition: all 0.3s;
        .el-icon-arrow-up {
            height: 100%;
            width: 25px;
            color: #c0c4cc;
            font-size: 14px;
            line-height: 32px;
            transition: transform 0.3s;
            transform: rotateZ(180deg);

            &.is-reverse {
                transform: rotateZ(0deg);
            }
        }
    }
}
</style>
