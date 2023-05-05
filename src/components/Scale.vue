<template>
    <div class="scale-container" id="scale-container">
        <!--指示器-->
        <div class="indicator">
<!--            <div class="value">{{props.value}}</div>-->
            <div id="indicator-val" class="value">{{ruleNumber }}</div>
            <div id="indicator-arr" class="arrow"></div>
        </div>
        <!--刻度尺-->
<!--        <div class="inner-container">
            <div class="scale" @drag="onScaleDrag">
                <div v-for="tick in ticks" :key="tick" class="tick">
                </div>
                <div class="measure">
                    dd
                </div>
            </div>
        </div>-->
        <canvas ref="canvas"
                id="test-canvas"
                width="50px"
                @touchmove="canvasTouchMove"
                @touchend="canvasTouchEnd"
                @touchstart="canvasTouchStart"
                @mousemove="canvasTouchMove"
                @mouseup="canvasTouchEnd"
                @mousedown="canvasTouchStart"
        ></canvas>
    </div>
</template>

<!--<script setup>
import {computed, onMounted, ref, toRefs} from "vue";

const props = defineProps({
    minValue: {
        type: Number,
        default: 0,
    },
    maxValue: {
        type: Number,
        default: 100,
    },
    value: {
        type: Number,
        default: 50,
    },
    scaleColor: {
        type: String,
        default: '#ffffff',
    },
    sliderColor: {
        type: String,
        default:'#06ee8e',
    },
    valueColor: {
        type: String,
        default:'#06ee8e',
    },
    backgroundColor: {
        type: String,
        default:'#040404',
    },
})
let sliderPosition = ref('50%')
// Calculate the number of ticks needed for the scale
const ticks = computed(() => {
    const { minValue, maxValue } = toRefs(props);
    const step = 1;
    const numTicks = (maxValue - minValue) / step;
    return Array.from({ length: numTicks }, (_, i) => minValue + i * step);
});

// Update the slider position based on the current value
const updateSliderPosition = (value) => {
    const { minValue, maxValue } = props;
    const range = maxValue.value - minValue.value;
    const percent = (value - minValue.value) / range * 100;
    sliderPosition.value = `${percent}%`;
    console.log(`sliderPosition.value: ${sliderPosition.value}`)
};

// Handle slider drag events
const onScaleDrag = (event) => {
    const { minValue, maxValue } = props;
    const scaleRect = event.target.getBoundingClientRect();
    const y = event.clientY - scaleRect.top;
    const range = maxValue.value - minValue.value;
    const percent = y / scaleRect.height;
    const value = minValue.value + Math.round(range * percent / 1) * 1;
    console.log(`value: ${value}`)
    updateSliderPosition(value);
};

onMounted(()=>{
    console.log(`the component is now mounted.`, props)
    updateSliderPosition(props.value)
})

</script>-->

<script lang="ts">
import { defineComponent, reactive, toRefs, onMounted, nextTick } from 'vue';
export default defineComponent({
    name: 'Home',
    props :{
        minValue: {
            type: Number,
            default: 0,
        },
        maxValue: {
            type: Number,
            default: 100,
        },
        value: {
            type: Number,
            default: 0,
        },
        scaleColor: {
            type: String,
            default: '#ffffff',
        },
        sliderColor: {
            type: String,
            default:'#06ee8e',
        },
        valueColor: {
            type: String,
            default:'#06ee8e',
        },
        backgroundColor: {
            type: String,
            default:'#040404',
        },
    },
    setup(props) {
        const state = reactive({
            ruleNumber:  props.value
        })
        /**  每10个格子就写一次刻度数字*/
        const divider = 10;
        /**  隔10个像素就画一个刻度线*/
        const itemWidth = 10;
        /**  画刻度线的起始x坐标*/
        const startX = 0;
        /*接收父组件传入的自定义样式*/
        const { minValue, maxValue, backgroundColor,
            scaleColor, sliderColor, valueColor } = props;

        // 修改指示器样式
        const root = document.documentElement;
        root.style.setProperty('--scale-container', backgroundColor);
        root.style.setProperty('--indicator-value', valueColor);
        root.style.setProperty('--indicator-arrow', sliderColor);

        let isDown= false;
        let topMin;
        let topMax;
        /**  惯性滑动用到的计时器 */
        let enableInertiaMove = true;
        // 手指按下时的时间
        // let startTime = 0;
        /**  手指按下时的y坐标，用来比较本次滑动的方向和距离，加在currentCanvasLocation上，就能让尺子移动了*/
        let touchStartY = 0;
        /* 手指按下时，当前 currentCanvasLocation 的值  */
        let startValue = 0;
        /**
         * 尺子最核心的值，单位为像素，这个值记录了当前手指总共滑动了多少像素距离和方向
         * 如果把尺子最低刻度理解为1厘米的话，那么这个就是1毫米。
         */
        let currentCanvasLocation = 10;
        // let timer = 0;
        /**  画布元素 canvas = document.getElementById('test-canvas');*/
        let canvas;
        /** 画布的宽 */
        let canvasWidth;
        /* 画布的高 */
        let canvasHeight;
        /* 画布context，通过操作ctx来画内容 */
        let ctx;
        /* 画布上侧到画布中间格子的数量，加上这个偏移值就能符合视觉的尺子当前值 */
        let numberOffset = 0;
        /** 手指抬起之前的滑动距离，用来发起惯性滑动*/
        let lastScrollDistacne = 0;
        /** 手指最后抬起之前接触的y坐标*/
        let lastTouchY = 0;
        /* 初始化 Canvas */
        const initCanvas = () => {
            const ruleContainer = document.getElementById('scale-container');
            canvas = document.getElementById('test-canvas');
            canvas.width = ruleContainer.clientWidth;
            canvas.height = ruleContainer.clientHeight;
            ctx = canvas.getContext('2d', { alpha: false });

            // 计算容器能放下的尺子格数
            const screenCount = parseInt((canvas.clientHeight / itemWidth).toFixed(0))
            // 计算尺子读数需要的偏移刻度数
            numberOffset = parseInt((screenCount / 2).toFixed(0)) ;
            topMin = minValue - numberOffset;
            topMax = maxValue - numberOffset;
            // 设置宽高
            canvasWidth = canvas.clientWidth;
            canvasHeight = canvas.clientHeight;
            // 设置字体
            ctx.font = "14px Arial";
            // 初始化完成后渲染一下
            window.requestAnimationFrame(draw);
        }
        // 画尺子
        const draw = () => {
            // 设置笔触颜色为黑色，每次绘制之前，先把画布用黑色清空
            ctx.fillStyle = backgroundColor;
            ctx.beginPath();
            ctx.fillRect(0, 0, canvasWidth, canvasHeight);
            ctx.closePath();
            // 清空完画布，再把笔触设置成白色, 绘制数值
            ctx.fillStyle = scaleColor;
            // 尺子的最小刻度为10个像素，for循环渲染尺度时会以 start 为准，
            // 所以每次会出现每次滑动时就一格一格的跳动，不够顺滑
            // 把 currentCanvasLocation 末尾的像素值取回来，让滑动更加顺滑
            let offset: number;
            // 取当前的位移量的最后一位
            const str = currentCanvasLocation.toString();
            const lastNumber = Number(str.charAt(str.length - 1));
            // currentCanvasLocation 大于和小于0时有不同的取值方式
            if (currentCanvasLocation > 0) {
                offset = itemWidth - lastNumber;
            } else if (currentCanvasLocation < 0) {
                offset = lastNumber;
                if (currentCanvasLocation === 0) {
                    offset = itemWidth;
                }
            }else{
                offset = 0;
            }
            // console.log('offset',offset)
            // for循环绘制尺子刻度,横向绘制
            for (let i = offset; i < canvasHeight; i+=itemWidth) {
                ctx.moveTo(startX,i);
                // 开头偏移的像素
                const scaleNumber = i+currentCanvasLocation;
                // 只绘制在尺子数值范围内的
                if(canDraw(scaleNumber)===0){
                    if (scaleNumber % (divider*itemWidth) === 0) {
                        // 绘制整刻度值
                        // const metrics = ctx.measureText(String(i)); // TextMetrics 对象,表示文本的尺寸
                        const textY = i.toFixed(2);
                        ctx.fillText(String(scaleNumber / itemWidth), 40, Number(textY));
                        ctx.lineTo(30, i);
                    } else {
                        // 绘制小刻度
                        ctx.lineTo(10, i);
                    }
                }
            }
            // 绘制刻度线
            ctx.strokeStyle = scaleColor;
            // 绘制在尺子数值范围内的竖线
            ctx.moveTo(startX,0);
            ctx.lineTo(startX, canvasHeight);
            ctx.stroke();
            // 绘制完成，加上数量
            nextTick(() => {
                state.ruleNumber =  Math.floor(currentCanvasLocation / itemWidth) + numberOffset;
            })
        }
        // 判断是否可以绘制
        const canDraw = (y:number): number => {
            const currentNumber = Math.floor(y / itemWidth);
            if (currentNumber >= minValue && currentNumber <= maxValue) {
                return 0;
            }
            return -1;
        }
        /**
         * 这里使用 1、-1、0 来标志当前尺子的状态
         * 当为0时表示可以滑动，1和-1则不行
         * 判断是否可以滑动时，使用如下代码：
         * if(value * canScroll() >0){
         *     return;
         * }
         * 如果一直向上划，划到最小值时，再划，value为负数，负负的正，此时被return则不能继续滑动
         * 如果一直向下划，划到最大值，再划，value为正数，canScroll()为1，此时也会相乘大于0，被return
         *
         */
        const canScroll = (y:number): number => {
            const currentNumber = Math.floor(y / itemWidth);
            if (currentNumber <= topMin) {
                return -1;
            }else if (currentNumber >= topMax) {
                return 1;
            }else{
                return 0;
            }
        }
        /* 按下事件 */
        const canvasTouchStart = (e) => {
            isDown = true;
            touchStartY = e.pageY;
            console.log('touchStartY',touchStartY)
            // touchStartY = e.changedTouches[0].clientY;
            startValue = currentCanvasLocation;
            // 清除之前的惯性滑动
            enableInertiaMove = false;
        }
        // 移动事件
        const canvasTouchMove = (e): void => {
            if(!isDown){
                return;
            }
            console.log('isDown',isDown)
            const touchClientY = e.pageY;
            // const touchClientY = e.targetTouches[0].clientY;
            const moveY = Math.floor(touchStartY - touchClientY);
            lastScrollDistacne = lastTouchY - touchClientY;
            lastTouchY = touchClientY;
            if (moveY * canScroll(currentCanvasLocation) > 0) {
                return;
            }
            cursorMove(moveY)
        }
        // 滑动
        const cursorMove = (value) => {
            currentCanvasLocation = startValue + value;
            window.requestAnimationFrame(draw);
        }
        // 抬起事件
        const canvasTouchEnd = (e) => {
            isDown = false;
            // 直接用最后一次滑动的距离来当做速度
            enableInertiaMove = true;
            ease(lastScrollDistacne);

        }

        // 惯性滑动
        const ease = (target) => {
            if (!enableInertiaMove) {
                return;
            }
            if (target * canScroll(currentCanvasLocation) > 0) {
                return;
            }
            target *= 0.9;
            if (Math.abs(target) < 1 || target * canScroll(currentCanvasLocation) > 0) {
                return
            }
            currentCanvasLocation += Math.floor(target);
            window.requestAnimationFrame(()=>{
                ease(target)
                draw()
            });
        }

        onMounted(() => {
            initCanvas();
        })
        return {
            ...toRefs(state),
            canvasTouchMove,
            canvasTouchEnd,
            canvasTouchStart
        }
    },
});
</script>

<style lang="scss" scoped>
/* 定义颜色变量 */
:root{
  --scale-container: #070707;
  --indicator-value: #14bd73;
  --indicator-arrow: #14bd73;
}

.scale-container{
    height: 90%;
    width: 110px;
    margin: 30px auto 0;
    background-color: var(--scale-container);
    //border-radius: 10px;
    display: flex;
    flex-direction: row;

     //指示器
    .indicator {
        height: 40px;
        width: 60px;
        margin: auto 0;
        display: flex;
        justify-content: space-around;
        align-items: center;
        // 当前值
        .value {
            width: 50px;
            height: 30px;
            margin-right: 5px;
            font-size: 30px;
            font-weight: bolder;
            line-height: 30px;
            color:var(--indicator-value);
            //color:$indicator-value;
            text-align: center;
        }
        // 指示箭头
        .arrow {
            height: 10px;
            width: 10px;
            clip-path: polygon(0 0, 100% 50%, 0 100%);
            background-color: var(--indicator-arrow);
        }
    }
    // 刻度尺
    /*.inner-container{
      width: 50px;
        overflow-x: hidden;
        overflow-y: scroll;
        // 刻度尺背景
        .scale {
            height: 4000px;
            background-color: $scaleColor;
            // 刻度尺刻度
            .tick {
                height: 2px;
                width: 40px;
                background-color: #fff;
                margin: 0 auto;
            }
            .measure{
                height: 40px;
                width: 40px;
                background-color: #14bd73;
            }
        }
    }*/
}
/*.inner-container::-webkit-scrollbar {
    display: none;
}*/
</style>


