<script setup>
import {ref, onMounted, onBeforeUnmount} from 'vue';
        let timer = null;
        // 保存前一个目标点坐标
        const preTarget=ref({
            x:0,
            y:0
        })
        // 定义雷达画布
        const radar = ref(null);

        // 定义目标信息
        const targetInfo = ref({
            angle: 0,
            distance: 0,
        });

        // 显示目标信息
        const showTarget = ref(false);

        // 绘制雷达图
        const drawRadar = () => {
            const canvas = radar.value;
            if (!canvas.getContext) return;
            const ctx = canvas.getContext('2d');
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            const radius = centerX;

            // 绘制雷达盘
            // ctx.beginPath();
            // ctx.arc(centerX, centerY, radius, 0, Math.PI * 2, true);
            // ctx.strokeStyle = '#035d10';
            // ctx.lineWidth = 2;
            // ctx.stroke();

            // 绘制黑色背景
            ctx.fillStyle = "black";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            //绘制绿色的十字线
            ctx.beginPath();
            ctx.moveTo(0, centerY);
            ctx.lineTo(canvas.width, centerY);
            ctx.moveTo(centerX, 0);
            ctx.lineTo(centerX, canvas.height);
            ctx.setLineDash([])
            ctx.strokeStyle = '#0dff30';
            ctx.lineWidth = 1;
            ctx.stroke();

            // 绘制圆形网格
            for (let i = 0; i < 4; i++) {
                const r = ((i + 1) * radius) / 5;
                ctx.beginPath();
                ctx.arc(centerX, centerY, r, 0, Math.PI * 2, false);
                ctx.strokeStyle = '#0dff30';
                ctx.lineWidth = 1;
                ctx.stroke();
            }

        };

        // 绘制目标
        const drawTarget = () => {
            const canvas = radar.value;
            if (!canvas.getContext) return;
            const ctx = canvas.getContext('2d');
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;

            // 随机生成目标位置
            const angle = Math.random() * Math.PI * 2;
            const distance = Math.random() * canvas.width * 0.4;
            const targetX = centerX + distance * Math.cos(angle);
            const targetY = centerY + distance * Math.sin(angle);
            preTarget.value.x=targetX
            preTarget.value.y=targetY

            // 绘制目标
            ctx.beginPath();
            ctx.fillStyle = '#f00';
            ctx.fillRect(targetX - 5, targetY - 5, 10, 10);
            //绘制目标中心点与雷达中心点的虚线
            ctx.moveTo(targetX, targetY);
            ctx.lineTo(canvas.width / 2, canvas.height / 2);
            ctx.strokeStyle = '#f00';
            ctx.setLineDash([5, 5]);
            ctx.lineWidth = 1;
            ctx.stroke();

            // 更新目标信息，逆时针计算angle 的值，其为弧度制，distance 为像素值，默认1像素为1公里
            targetInfo.value = {
                angle: (360-(angle * 180) / Math.PI).toFixed(2),
                distance: distance.toFixed(2),
            };
            showTarget.value = true;
        };

        // 清除目标
        const clearTarget = () => {
            const canvas = radar.value;
            if (!canvas.getContext) return;
            const ctx = canvas.getContext('2d');
            // 清空画布
            ctx.clearRect(preTarget.value.x - 5, preTarget.value.y - 5, 10, 10);
            drawRadar();
            showTarget.value = false;
        };

        onMounted(() => {
            drawRadar();
            // 每秒绘制一次目标
            timer =setInterval(() => {
                clearTarget();
                drawTarget();
            }, 1000);
        });

        //销毁时清除定时器
        onBeforeUnmount(() => {
            clearInterval(timer)
            timer = null;
            // console.log('销毁定时器了~')
        });

</script>

<template>
    <div class="radar-container">
        <canvas ref="radar" width="500" height="500"></canvas>
        <div class="target-info" v-if="showTarget">
            <div class="target-angle">角度：{{targetInfo.angle}}°</div>
            <div class="target-distance">距离：{{targetInfo.distance}}km</div>
        </div>
    </div>
</template>

<style scoped>
/*设置可视窗口大小*/
.radar-container {
    position: relative;
    width: 700px;
    height: 700px;
    border-radius: 5%;
    background-color: black;
}
/*canvas 位于雷达图的中心*/
canvas{
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
}
/*target-info 位于雷达图的右下角，内容左对齐*/
.target-info {
    position: absolute;
    right: 0;
    bottom: 0;
    margin: 10px;
    text-align: start;
    font-size: 16px;
    color: #ffffff;
}
/*target-angle 与 target-distance 之间有 5px 的间距*/
.target-angle {
    margin-bottom: 5px;
}
</style>
