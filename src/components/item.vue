<template>
    <!-- <div style="height: 40px;border: 1px solid blue;">
        {{item.value}}
    </div> -->
    <div :style="{height: height}" class="item">
        {{item.id}} {{item.value}}
    </div>
</template>

<script>
export default {
    props: {
        item: Object,
        height: String
    }
}
</script>
<style scoped>
.item {
    border: 1px solid blue;box-sizing:border-box;
}
</style>
