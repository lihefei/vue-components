<template>
    <div class="media" @click="click">
        <div class="media__thumb" :style="{ '--thumb-width': `${thumbWidth}`, '--thumb-height': `${thumbHeight}` }">
            <slot name="thumb">
                <div class="media__face"></div>
            </slot>
        </div>
        <div class="media__content">
            <slot name="content">
                <h3 class="media__title">{{ data.title }}</h3>
                <div v-for="item in data.list" class="media__item">{{ item }}</div>
            </slot>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from 'vue-property-decorator';
@Component({ name: 'Media' })
export default class extends Vue {
    @Prop({ default: '64px' }) thumbWidth!: string;
    @Prop({ default: 'auto' }) thumbHeight!: string;
    @Prop({
        default() {
            return {
                title: 'title',
                list: ['内容'],
            };
        },
    })
    data!: any;

    //点击事件
    private click() {
        this.$emit('click');
    }
}
</script>

<style lang="scss" scoped>
.media {
    display: flex;
    &__thumb {
        width: var(--thumb-width);
        height: var(--thumb-height);
        min-height: var(--thumb-width);
        margin-right: 5px;
    }
    &__face {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: rgba(#000, 0.2);
    }

    &__content {
        flex: 1;
        position: relative;
    }

    &__title {
        display: -webkit-box;
        -webkit-line-clamp: 1;
        -webkit-box-orient: vertical;
        overflow: hidden;
        text-overflow: ellipsis;
        font-size: 14px;
        line-height: 24px;
    }
    &__item {
        margin-top: 6px;
        color: #969799;
        font-size: 12px;
        line-height: 16px;
    }
}
</style>
