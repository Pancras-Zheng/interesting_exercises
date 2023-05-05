
<template>
    <div class="container">
        <Scale :max-value="options.maxValue"
               :min-value="options.minValue"
               :value="options.value"
               :scale-color="options.scaleColor"
               :slider-color="options.sliderColor"
               :value-color="options.valueColor"
               :background-color="options.backgroundColor"
        />
    </div>
</template>

<script setup>
import Scale from '@/components/Scale.vue'
import {reactive} from 'vue'
/*
Scale组件开放以下属性：
            :maxValue="maxValue"  刻度尺最大值
            :minValue="minValue"  刻度尺最小值
            :value="value"   当前值
            :scaleColor="scaleColor" 刻度颜色
            :sliderColor="sliderColor" 指示箭头颜色
            :valueColor="valueColor"  当前值的颜色
            :backgroundColor="backgroundColor"  刻度尺背景颜色
*/

let options=reactive({
    maxValue: 100,
    minValue: 0,
    value: 28,
    scaleColor: '#ffffff',
    sliderColor: '#c816d5',
    valueColor: '#123be7',
    backgroundColor: '#040404',
})

</script>

<style lang="scss" scoped>
/*设置可视窗口大小*/
.container{
    width: 100%;
    height: 80vh;
    border-radius: 5%;
    background-color: #f8f5f5;
}
</style>
