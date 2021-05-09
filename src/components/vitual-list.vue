<template>
    <!-- 可视区 -->
    <div class="viewport" ref="viewport" @scroll="scrollFn">
        <!-- 自己做一个滚动条 -->
        <div class="scroll-bar" ref="scrollBar"></div>
        <!-- 真实渲染的内容 -->
        <div class="content" :style="{transform: `translate3d(0,${offset}px,0)`}">
            <div v-for="item in visibleData" :vid="item.id" :key="item.id" ref="item">
                <slot :item="item"></slot>
            </div>
        </div>
    </div>
</template>

<script>
import throttle from 'lodash/throttle'
export default {
    props: {
        size: Number, // 当前每一项的高度
        remain: Number, // 可见多少个
        items: Array, // 所有数据项
        variable: Boolean // 高度是不确定的
    },
    data() {
        return {
            start: 0,
            end: this.remain, // 默认显示个数
            offset: 0, // 可视区内容位置偏移
        }
    },
    computed: {
        preCount() { // 前面预留几个
            return Math.min(this.start, this.remain);
        },
        nextCount() { // 后面预留几个
            return Math.min(this.remain, this.items.length - this.end);
        },
        // 渲染三屏幕数据
        visibleData() {
            let startIndex = this.start - this.preCount;
            let endIndex = this.end + this.nextCount;
            
            return this.items.slice(startIndex, endIndex)
        },
    },
    created() {
        this.scrollFn = throttle(this.handleScroll, 200, { leading: false })
    },
    mounted() {
        this.$refs.viewport.style.height = this.size * this.remain + 'px';
        this.$refs.scrollBar.style.height = this.items.length * this.size + 'px';

        // 如果加载完毕 需要缓存每一项高度
        // 滚动时，获取页面真实dom的高度来更新缓存的内容
        if (this.variable) {
            this.cacheList()
        }
    },
    updated() {
        if (this.variable) {
            // 页面渲染完成后 需要根据当前展示的数据 更新缓存区的内容
            this.$nextTick(() => {
                // 根据当前显示的 更新缓存中数据，最后更新滚动条的高度
                let nodes = this.$refs.item;
                if (nodes && nodes.length > 0) {
                    nodes.forEach(node => {
                        let { height } = node.getBoundingClientRect(); // 真实高度
                        let id = node.getAttribute('vid') - 0;
                        let oldHeight = this.positions[id].height;
                        let val = oldHeight - height;
                        // 判断高度是否有变化
                        if (val) {
                            this.positions[id].height = height;
                            this.positions[id].bottom = this.positions[id].bottom - val;
                            // 当前高度变了，后面的项top都变了
                            for (let i = id + 1; i < this.positions.length; i++) {
                                this.positions[i].top = this.positions[i-1].bottom;
                                this.positions[i].bottom = this.positions[i].bottom - val;
                            }
                        }
                    })
                    // 重新计算滚动条的高度
                    this.$refs.scrollBar.style.height = this.positions[this.positions.length - 1].bottom + 'px';
                }
            })
        }
    },
    methods: {
        // 缓存当前项的高度和top值bottom值
        cacheList() {
            this.positions = this.items.map((item, index) => {
                return {
                    top: index * this.size,
                    bottom: (index + 1) * this.size,
                    height: this.size
                }
            })
        },
        handleScroll() {
            // 1、先计算出当前滚动的距离
            let scrollTop = this.$refs.viewport.scrollTop;

            if (this.variable) {
                // 高度不确定, 使用二分查找
                this.start = this.getStartIndex(scrollTop);
                this.end = this.start + this.remain;
                console.log('startIndex', this.start)
                console.log('endIndex', this.end )
                this.offset = this.positions[this.start - this.preCount] ?this.positions[this.start - this.preCount].top : 0
            } else {
                // 1、获取当前应该从第几个开始渲染
                this.start = Math.floor(scrollTop / this.size);
                // 2、最后一个渲染位置
                this.end = this.start + this.remain;
                console.log('startIndex', this.start)
                console.log('endIndex', this.end)
                // 3、定位当前可视区
                this.offset = this.start * this.size - this.preCount * this.size;
            }
        },
        /**
         * value 查找当前滚动的需要找到的值
         */
        getStartIndex(value) {
            let start = 0; // 开始
            let end = this.positions.length - 1; // 结束
            let temp = null; // 有可能没找到，这个是最接近的那一项
            while(start <= end) {
                let middleIndex = parseInt((start + end)/2);
                let middleValue = this.positions[middleIndex].bottom;
                if (middleValue == value) {
                    return middleIndex + 1;
                } else if (middleValue < value) { // 当前要查找的项在 右边
                    start = middleIndex + 1;
                } else if (middleValue > value) { // 当前要查找的项在 左边
                    if (temp == null || temp > middleIndex) {
                        temp = middleIndex;
                    }
                    end = middleIndex - 1;
                }
            }
            return temp;
        }
    }
}
</script>
<style lang="less">
    .viewport {
        overflow-y: scroll;
        position: relative;
        margin-top: 20px;
    }
    .content {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
    }
</style>

