<template>
    <div v-for="(mission, i) in data">
        <div v-if="mission.count > 0" style="padding: 4px; margin: 4px 0 4px 0; border: 1px solid #dcdee2;">
            <!-- 单个任务信息 -->
            <Icon type="md-star" /><span>{{ mission.star }}</span>&nbsp;
            <Tag :color="TYPE_COLOR_MAP[mission.type]">{{ TYPE_MAP[mission.type] }}</Tag>
            <Tag v-if="mission.place" :color="PLACE_COLOR_MAP[mission.place]">{{ PLACE_MAP[mission.place] }}</Tag>
            <Tag v-else>未记录地区</Tag>
            <!-- 剩余任务次数按钮，每次点击减少一次 -->
            <Button size="small" style="font-size: 12px; margin-left: 4px;" @click="clickCountButton(i)">
                {{ mission.count }}次
            </Button>
            <!-- 关闭任务按钮 -->
            <Button size="small" type="warning" ghost shape="circle" style="float: right;" @click="clickCloseButton(i)">
                <Icon type="md-close" />
            </Button>
        </div>
    </div>
</template>
<script>
import { TYPE_MAP, TYPE_COLOR_MAP, PLACE_MAP, PLACE_COLOR_MAP } from '../consts.js'
export default {
    props: {
        data: Array,
        index: Number,
    },
    emits: ['onUpdate'],
    setup(props) {
        return;
    },
    data() {
        return {};
    },
    computed: {
        TYPE_MAP() {
            return TYPE_MAP;
        },
        TYPE_COLOR_MAP() {
            return TYPE_COLOR_MAP;
        },
        PLACE_MAP() {
            return PLACE_MAP;
        },
        PLACE_COLOR_MAP() {
            return PLACE_COLOR_MAP;
        },
    },
    methods: {
        // 任务计数按钮响应
        clickCountButton(i) {
            this.data[i].count -= 1;
            if (this.data[i].count === 0) {
                this.data.splice(i, 1);
                this.$Message.info('任务次数已清零');
            }
            this.$emit('onUpdate', this.data, this.index);
        },
        // 关闭任务按钮
        clickCloseButton(i) {
            this.data.splice(i, 1);
            this.$Message.info('任务已关闭');
            this.$emit('onUpdate', this.data, this.index);
        },
    }
};
</script>