<template>
    <div class="wrapper" :style="{ '--background': background, padding, color }">
        <div class="wrapper__container">
            <h3 v-if="title" class="wrapper__title">{{ title }}</h3>
            <slot></slot>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from 'vue-property-decorator';
@Component({ name: 'Wrapper' })
export default class extends Vue {
    @Prop({ default: '' }) title!: string; //标题
    @Prop({ default: '50px 0' }) padding!: string; //内边距
    @Prop({ default: '#333' }) color!: string; //文字颜色
    @Prop({ default: '#fff' }) background!: string; //背景颜色
}
</script>

<style lang="scss" scoped>
.wrapper {
    background: var(--background);
    &__container {
        width: 1200px;
        margin: 0 auto;
    }

    &__title {
        margin-bottom: 30px;
        line-height: 48px;
        font-size: 28px;
        text-align: center;
    }
}
</style>
