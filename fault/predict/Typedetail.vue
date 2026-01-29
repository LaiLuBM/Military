<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <el-row>
            <el-col :span="1">
              <!-- 添加返回按钮 -->
              <el-button text @click="goBack" icon="ArrowLeft"
                style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
            </el-col>
            <el-col :span="15">
              <h2 class="title-margin">型号故障预测详情</h2>
            </el-col>
          </el-row>
          <el-row style="margin-top: 30px;">
            <el-col :span="1"></el-col>
            <el-col :span="6">
              <div>所属单位：{{ unitName }}</div>
            </el-col>
            <el-col :span="6">
              <div>所属分队：{{ unitName }}</div>
            </el-col>
            <el-col :span="6">
              <div>预测执行时间：{{ predictTime }}</div>
            </el-col>
            <el-col :span="5"></el-col>
          </el-row>
          <el-row style="margin-top: 30px;">
            <el-col :span="1"></el-col>
            <el-col :span="6">
              <div>装备型号：{{ equipSimpleName }}</div>
            </el-col>
            <el-col :span="6">
              <div>装备总数：{{ equipNumber }}台</div>
            </el-col>
            <el-col :span="11"></el-col>
          </el-row>
          <el-divider border-style="double" />
          <el-row style="margin-top: 50px;">
            <el-col :span="24">
              <div ref="chartRef" style="width: 100%;height: 500px;"></div>
            </el-col>
          </el-row>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>


<script setup lang="ts">
import { ref, watch, onMounted, nextTick } from "vue";
import { useChart } from "@/hooks/useChart";
import { predictDetail } from "@/api/fault/type";
import { reactive } from "vue";
import { useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';

const router = useRouter();
const chartRef = ref(null);
const unitCode = ref('');
const unitName = ref('');
const predictTime = ref('');
const timeStamp = ref('');
const equipModelCode = ref('');
const monthMalfRate = ref('');
const monthMalfNumber = ref('');
const equipNumber = ref('');
const equipSimpleName = ref('');
const chartInstance = ref<any>(null);
// 先声明但不初始化 useChart，等 setChartData 定义后再初始化
let chartHook: any = null;

const goBack = () => {
 window.history.back();
}

// Safe session storage
const safeSessionStorage = {
  set(key: string, value: string): void {
    try {
      sessionStorage.setItem(key, value);
    } catch (e) {
      console.error('SessionStorage存储失败:', e);
    }
  },
  get(key: string): string {
    try {
      return sessionStorage.getItem(key) || '';
    } catch (e) {
      console.error('SessionStorage读取失败:', e);
      return '';
    }
  }
};

// Store session data
const storeSessionData = () => {
  if (timeStamp.value) {
    safeSessionStorage.set('timeStamp', timeStamp.value);
  }
};

const getChar = async () => {
  try {
    if (!timeStamp.value) {
      ElMessage.error('缺少时间戳，无法加载预测详情');
      return;
    }
    console.log('获取数据，timeStamp:', timeStamp.value);

    const res = await predictDetail({ timeStamp: timeStamp.value });
    unitCode.value = res.data.unitCode || '';
    predictTime.value = res.data.predictTime || '';
    equipModelCode.value = res.data.equipModelCode || '';
    equipNumber.value = res.data.equipNumber || '';
    unitName.value = res.data.unitName || '';
    equipSimpleName.value = res.data.equipSimpleName || '';
    monthMalfRate.value = res.data.monthMalfRate || '';
    monthMalfNumber.value = res.data.monthMalfNumber || '';
    storeSessionData();

    // 数据获取完成后，强制刷新图表
    await refreshChart();
  } catch (error) {
    console.error('请求预测详情失败:', error);
    ElMessage.error('加载预测详情失败，请稍后重试');
  }
};

// 新增：专门刷新图表的方法
const refreshChart = async () => {
  try {
    console.log('刷新图表...');
    const options = await setChartData();
    // 使用 useChart 返回的 update 方法更新图表
    if (chartHook && chartHook.update) {
      await chartHook.update(options, true); // true 表示使用动画
      console.log('图表刷新完成');
    } else {
      console.warn('chartHook 尚未初始化');
    }
  } catch (error) {
    console.error('刷新图表失败:', error);
  }
};

// Handle route query and session storage
onMounted(() => {
  console.log('页面加载');
  const querytimeStamp = router.currentRoute.value.query.timeStamp;
  if (querytimeStamp) {
    timeStamp.value = String(querytimeStamp);
  }
  storeSessionData();
  getChar();
});

// 监听路由参数变化
watch(() => router.currentRoute.value.query.timeStamp, async (newTimeStamp) => {
  console.log('路由时间戳变化:', newTimeStamp);
  if (newTimeStamp) {
    const oldTimeStamp = timeStamp.value;
    timeStamp.value = String(newTimeStamp);

    if (oldTimeStamp !== timeStamp.value) {
      console.log('时间戳变化，重新获取数据:', timeStamp.value);
      storeSessionData();
      // 等待 DOM 更新后执行
      nextTick(async () => {
        await getChar();
      });
    }
  }
}, { immediate: true });

// 原来的 setChartData 函数保持不变
const setChartData = async () => {
  console.log('生成图表配置...');

  // 如果数据为空，先获取数据
  if (!monthMalfRate.value || !monthMalfNumber.value || !predictTime.value) {
    if (timeStamp.value) {
      try {
        const res = await predictDetail({ timeStamp: timeStamp.value });
        predictTime.value = res.data.predictTime || '';
        monthMalfRate.value = res.data.monthMalfRate || '';
        monthMalfNumber.value = res.data.monthMalfNumber || '';
      } catch (error) {
        console.error('获取图表数据失败:', error);
      }
    }
  }

  const generateMonthLabels = (predictTime: string): string[] => {
    if (!predictTime) {
      console.warn('predictTime is empty or undefined, using default months');
      const currentDate = new Date();
      const year = currentDate.getFullYear();
      const month = currentDate.getMonth() + 1;
      const labels: string[] = [];
      for (let i = 0; i < 12; i++) {
        const newMonth = (month + i - 1) % 12 + 1;
        const newYear = year + Math.floor((month + i - 1) / 12);
        labels.push(`${newYear}-${newMonth.toString().padStart(2, '0')}`);
      }
      return labels;
    }

    let startDate: Date;
    try {
      startDate = new Date(predictTime);
      if (isNaN(startDate.getTime())) {
        console.warn('Invalid predictTime format, using default months:', predictTime);
        const currentDate = new Date();
        const year = currentDate.getFullYear();
        const month = currentDate.getMonth() + 1;
        const labels: string[] = [];
        for (let i = 0; i < 12; i++) {
          const newMonth = (month + i - 1) % 12 + 1;
          const newYear = year + Math.floor((month + i - 1) / 12);
          labels.push(`${newYear}-${newMonth.toString().padStart(2, '0')}`);
        }
        return labels;
      }
    } catch (error) {
      console.warn('Failed to parse predictTime, using default months:', predictTime, error);
      const currentDate = new Date();
      const year = currentDate.getFullYear();
      const month = currentDate.getMonth() + 1;
      const labels: string[] = [];
      for (let i = 0; i < 12; i++) {
        const newMonth = (month + i - 1) % 12 + 1;
        const newYear = year + Math.floor((month + i - 1) / 12);
        labels.push(`${newYear}-${newMonth.toString().padStart(2, '0')}`);
      }
      return labels;
    }

    const labels: string[] = [];
    const startYear = startDate.getFullYear();
    const startMonth = startDate.getMonth() + 1;
    for (let i = 0; i < 12; i++) {
      const newMonth = (startMonth + i - 1) % 12 + 1;
      const newYear = startYear + Math.floor((startMonth + i - 1) / 12);
      labels.push(`${newYear}-${newMonth.toString().padStart(2, '0')}`);
    }
    return labels;
  };

  const chartOptions: any = reactive({
    title: {
      left: 'center',
      text: "整机未来12个月故障装备及故障率分布",
    },
    tooltip: {
      trigger: 'axis',
    },
    legend: {
      data: ['故障率', '故障装备数量'],
      bottom: 10,
    },
    xAxis: {
      name: '',
      type: 'category',
      boundaryGap: false,
      data: generateMonthLabels(predictTime.value)
    },
    yAxis: [
      {
        name: '故障率(%)',
        type: 'value',
        axisLabel: {
          formatter: '{value}'
        },
        position: 'left',
        min: 0,
        max: 100
      },
      {
        name: '故障装备数（台）',
        type: 'value',
        axisLabel: {
          formatter: '{value}'
        },
        position: 'right',
        min: 0
      }
    ],
    series: [
      {
        name: '故障率',
        type: 'line',
        data: [],
        lineStyle: {
          width: 2
        },
        itemStyle: {
          color: "blue",
        },
        smooth: false,
        yAxisIndex: 0
      },
      {
        name: '故障装备数量',
        type: 'line',
        data: [],
        lineStyle: {
          width: 2
        },
        itemStyle: {
          color: "green",
        },
        smooth: false,
        yAxisIndex: 1
      }
    ]
  });

  // Convert data to arrays
  const testArray1: number[] = monthMalfRate.value
    ? monthMalfRate.value.split(',').map(Number).map(value => Number((value * 100).toFixed(2)))
    : [];
  const testArray2: number[] = monthMalfNumber.value
    ? monthMalfNumber.value.split(',').map(Number)
    : [];

  chartOptions.series[0].data = testArray1;
  chartOptions.series[1].data = testArray2;

  console.log('图表配置生成完成:', chartOptions);
  return chartOptions;
};

// 在 setChartData 定义后才初始化 useChart
chartHook = useChart(chartRef, setChartData, (instance: any) => {
  chartInstance.value = instance;
  console.log('图表实例已保存');
});
</script>

<style lang="less" scoped>
.center-content {
  display: flex;
  justify-content: center;
  align-items: center;
}

.bold-text {
  font-weight: bold;
}

.left-align {
  text-align: left;
}

.title-margin {
  margin-bottom: 5px;
}
</style>