<template>
  <div class="container">
    <div class="custom">
      <div class="color">
          <div>刻度尺背景颜色</div>
        <input
          type="color"
          v-model="options.backgroundColor"
          placeholder="Enter a color value here"
        />
      </div>
      <div class="color">
          <div>刻度尺刻度颜色</div>
        <input
          type="color"
          v-model="options.scaleColor"
          placeholder="Enter a color value here"
        /></div>
      <div class="color">
          <div>指示箭头颜色</div>
        <input
          type="color"
          v-model="options.sliderColor"
          placeholder="Enter a color value here"
        /></div>
      <div class="color">
          <div>当前值的颜色</div>
        <input
          type="color"
          v-model="options.valueColor"
          placeholder="Enter a color value here"
        /></div>
      <div class="num">
          <div>设置当前值</div>
        <input
          type="number"
          v-model="options.value"
          placeholder="Enter a value here"
        /></div>
      <div class="num">
          <div>最大值</div>
        <input
          type="number"
          v-model="options.maxValue"
          placeholder="Enter a value here"
        /></div>
      <div class="num">
          <div>最小值</div>
        <input
          type="number"
          v-model="options.minValue"
          placeholder="Enter a value here"
        /></div>
    </div>
      <Scale
            :max-value="options.maxValue"
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
import Scale from "@/components/Scale.vue";
import { reactive, ref} from "vue";

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
// const valueColor = ref(null);

const options = reactive({
  maxValue: 100,
  minValue: 0,
  value: 28,
  scaleColor:'#ffffff',
  sliderColor: "#06ee8e",
  valueColor: "#06ee8e",
  backgroundColor: "#040404"
});
</script>

<style lang="scss" scoped>
@media (max-width: 500px) {
  .color {
    font-size: 3em;
  }
}
/*设置可视窗口大小*/
.container {
  width: 100%;
  height: 80vh;
  border-radius: 5%;
  background-color: #f8f5f5;
  display: flex;
  flex-direction: row;
  .custom {
    width: 14vw;
    height: 100%;
    margin: 2vw 0 0 2vw;
    display: flex;
    flex-direction: column;
    .color {
      display: flex;
      align-items: center;
      justify-content: space-between;
      input{
        width: 3vw;
      }
    }
  }
}
</style>
