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
              <h2 class="title-margin">通用化故障预测结果详情</h2>
            </el-col>
            <el-col :span="8" class="center-content">
              <el-button type="warning" size="large" @click="viewReport()">查看故障报告</el-button>
            </el-col>
          </el-row>
          <el-row style="margin-top:10px;margin-bottom:20px">
            <el-col :span="24">
              <el-tabs v-model="selectedEquipCode" type="card" > 
                <el-tab-pane
                v-for="item in equipCodeOptions"
                :key="item.value"
                :label="item.label"
                :name="item.value"
                >
                </el-tab-pane>
              </el-tabs>
            </el-col>

          </el-row>
          <el-row>
            <el-col :span="6">
              <div ref="chartRef2" style="width: 65%;height: 200px;"></div>
            </el-col>
            <el-col :span="18">
              <el-row style="margin-top: 15px;">
                <el-col :span="7">
                  <div><strong>所属单位：</strong>{{ equipSimpleName }}</div>
                </el-col>
                <el-col :span="8">
                  <div><strong>装备型号：</strong>{{ unitSimpleName }}</div>
                </el-col>
                <el-col :span="6">
                  <div><strong>所属分队：</strong>{{ equipSimpleName }}</div>
                </el-col>
              </el-row>
              <el-row style="margin-top: 15px;">
                <el-col :span="7">
                  <div><strong>装备统一编码：</strong>
                    <!-- <el-select v-model="selectedEquipCode" :placeholder="selectedEquipCode || '请选择装备统一编码'"
                      style="width: 150px" @change="loadData">
                      <el-option v-for="item in equipCodeOptions" :key="item.value" :label="item.label"
                        :value="item.value" />
                    </el-select> -->
                    <span style="color:#409EFF;font-weight:bold;margin-left:5px">
                      {{selectedEquipCode|| '请选择装备统一编码' }}
                    </span>
                  </div>
                </el-col>
                <el-col :span="7">
                  <div><strong>装备战斗编号：</strong>{{ fightCode }}</div>
                </el-col>
              </el-row>
              <el-row style="margin-top: 15px;">
                <el-col :span="7">
                  <div><strong>列装时间：</strong>{{ serviceDate }}</div>
                </el-col>
                <el-col :span="6">
                  <div><strong>服役日期：</strong>{{ serviceLimit }}</div>
                </el-col>
              </el-row>
              <el-row style="margin-top: 15px;">
                <el-col :span="7">
                  <div><strong>大修间隔期内累计行驶里程：</strong>{{ workMile }}公里</div>
                </el-col>
                <el-col :span="8">
                  <div><strong>中修间隔期内发动机累计消耗摩托小时：</strong>{{ vehicleMotor }}小时</div>
                </el-col>
                <el-col :span="6">
                  <div><strong>当年摩托小时消耗限额：</strong>{{ motorLimitYear }}小时</div>
                </el-col>
              </el-row>
              <el-row style="margin-top: 15px;">
                <el-col :span="6">
                  <div><strong>出厂日期：</strong>{{ manufactureDate }}</div>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
          <el-divider border-style="double" />
          <el-row style="margin-top: 15px;">
            <el-col :span="6">
              <div style="margin-top: 10px;"><strong>故障最可能出现的时机：</strong>{{ failureTiming }}</div>
            </el-col>
            <el-col :span="6">
              <div style="margin-top: 10px;"><strong>故障类型：</strong>{{ failureType }}</div>
            </el-col>
            <el-col :span="6">
              <div style="margin-top: 10px; text-align: left;margin-left: -80px;">
                <strong>故障预测执行时间：</strong>{{ predictTime }}
              </div>
            </el-col>
            <el-col :span="3">
              <div style="margin-top: 10px;margin-left: -40px;"><strong>故障部件(系统)：</strong>{{ failureLocation }}</div>
            </el-col>
            <el-col :span="3">
              <el-button type="primary" text @click="viewFaultReport">查看更多装备信息<el-icon>
                  <DArrowRight />
                </el-icon></el-button>
            </el-col>
          </el-row>
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
import { ref, watch, onMounted } from "vue";
import { useChart } from "@/hooks/useChart";
import { chartDataApi3 } from "@/api/fault/detail";
import { useRouter, useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import { equipDetailApi } from '@/api/fault/equipDetail';

// 路由和状态管理
const router = useRouter();
const route = useRoute();
const chartRef = ref<HTMLElement | null>(null);
const chartRef2 = ref<HTMLElement | null>(null);
const unitSimpleName = ref('');
const equipSimpleName = ref('');
const serviceDate = ref('');
const predictTime = ref('');
const failureType = ref('');
const vehicleMotor = ref('');
const manufactureDate = ref('');
const fightCode = ref('');
const workMile = ref('');
const motorLimitYear = ref('');
const failureLocation = ref('');
const serviceLimit = ref('');
const maxIndex = ref<number>();
const failureTiming = ref('');
const monthProbability = ref('');
const isDetailed = ref(false);
const selectedEquipCode = ref(''); // 当前选中的装备统一编码
const selectedPredictId = ref<number | null>(null); // 当前选中的预测ID
const predictMonth = ref<number>(6); // 默认6月
const predictYear = ref<number>(2025); // 默认2025年

// 存储所有可选的装备统一编码和预测ID
const equipCodeOptions = ref<{ value: string; label: string; predictId: number | null }[]>([]);

// 安全处理 sessionStorage
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

// 存储当前选中的装备统一编码和预测ID到 sessionStorage
const storeSessionData = () => {
  if (selectedEquipCode.value && selectedPredictId.value) {
    safeSessionStorage.set('generalfaultdetailEquipCode', selectedEquipCode.value);
    safeSessionStorage.set('generalfaultdetailPredictId', selectedPredictId.value.toString());
  }
};

const goBack = () => {
 window.history.back();
};

const viewFaultReport = () => {
  if (selectedEquipCode.value) {
    // router.push("/devcieinformation/detailinformation?equipCode=" + selectedEquipCode.value);
      router.push("/deviceinformation/Detailinformation?equipCode=" + selectedEquipCode.value);
  }
};

const viewReport = () => {
  router.push({
    path: '/predict/detailreport',
    query: {
      predictId: selectedPredictId.value,
      equipCode: selectedEquipCode.value,
    }
  });
  // router.push(`/predict/detailreport?predictId=${selectedPredictId.value}`);
};

const generateMonthLabels = (startMonth: number, startYear: number): string[] => {
  const labels: string[] = [];
  for (let i = 0; i < 12; i++) {
    const month = (startMonth - 1 + i) % 12 + 1;
    const year = startYear + Math.floor((startMonth - 1 + i) / 12);
    labels.push(`${year}-${month.toString().padStart(2, '0')}`);
  }
  return labels;
};

// 图表更新函数
const { update: updateChart } = useChart(chartRef, async () => await setChartData());
const { update: updateChart2 } = useChart(chartRef2, async () => await setChartData2());

const setChartData = async () => {
  if (!selectedPredictId.value) {
    console.warn('No predictId selected for setChartData');
    return {
      tooltip: { trigger: 'axis', formatter: () => '无数据' },
      xAxis: { type: 'category', data: generateMonthLabels(predictMonth.value, predictYear.value) },
      yAxis: { type: 'value', name: '故障概率(%)' },
      series: [{ name: '故障概率', type: 'line', data: [] }],
    };
  }

  try {
    const res = await chartDataApi3(isDetailed.value, selectedPredictId.value);
    const chartOptions = {
      tooltip: {
        trigger: 'axis',
        formatter: function (params: any) {
          const month = generateMonthLabels(predictMonth.value, predictYear.value)[params[0].dataIndex];
          return `${month}<br/>故障概率: ${params[0].value.toFixed(2)}%`;
        },
      },
      legend: {
        data: ['故障概率'],
      },
      xAxis: {
        name: '月份',
        type: 'category',
        boundaryGap: false,
        data: generateMonthLabels(predictMonth.value, predictYear.value),
      },
      yAxis: {
        name: '故障概率(%)',
        type: 'value',
        mim: 0,
        max: 100,
        interval: 20,
        axisLabel: {
          formatter: '{value}',
        },
      },
      series: [
        {
          name: '故障概率',
          type: 'line',
          data: res.data?.equipMalf?.monthProbability.split(',').map((value: string) => Number(value) * 100),
          lineStyle: {
            width: 1,
          },
          itemStyle: {
            color: 'blue',
            shadowBlur: 5,
            shadowColor: 'rgba(0,255,0,0.5)',
          },
          smooth: false,
        },
      ],
    };

    return chartOptions;
  } catch (error) {
    console.error('获取 setChartData 数据失败:', error);
    ElMessage.error('加载图表数据失败，请稍后重试');
    return {
      tooltip: { trigger: 'axis', formatter: () => '无数据' },
      xAxis: { type: 'category', data: generateMonthLabels(predictMonth.value, predictYear.value) },
      yAxis: { type: 'value', name: '故障概率(%)' },
      series: [{ name: '故障概率', type: 'line', data: [] }],
    };
  }
};

const setChartData2 = async () => {
  if (!selectedPredictId.value) {
    console.warn('No predictId selected for setChartData2');
    return {
      series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }],
      graphic: [
        {
          type: 'group',
          left: 'center',
          top: '53%',
          children: [{ type: 'text', style: { text: '无数据', textAlign: 'center', fontSize: 13, fill: '#333' } }],
        },
      ],
    };
  }

  try {
    const res = await chartDataApi3(isDetailed.value, selectedPredictId.value);
    const chartOptions = {
      series: [
        {
          name: '预测故障概率',
          type: 'pie',
          radius: ['50%', '70%'],
          avoidLabelOverlap: false,
          label: { show: false },
          emphasis: { label: { show: false } },
          itemStyle: {
            color: function (colors: { dataIndex: number }) {
              return ['#007BFF', '#E0E0E0'][colors.dataIndex];
            },
          },
          data: [
            res.data?.equipMalf?.failureProbability || 0,
            1 - (res.data?.equipMalf?.failureProbability || 0),
          ],
        },
      ],
      graphic: [
        {
          type: 'group',
          left: 'center',
          top: '53%',
          bounding: 'raw',
          children: [
            {
              type: 'text',
              style: {
                text: '预测故障概率',
                textAlign: 'center',
                textVerticalAlign: 'top',
                fontSize: 13,
                fill: '#333',
              },
            },
            {
              type: 'text',
              style: {
                text: `${(res.data?.equipMalf?.failureProbability * 100).toFixed(2)}%`,
                fontSize: 24,
                textAlign: 'center',
                textVerticalAlign: 'bottom',
                fill: '#FF5E65',
              },
            },
          ],
        },
      ],
      labelLine: { show: false },
    };

    return chartOptions;
  } catch (error) {
    console.error('获取 setChartData2 数据失败:', error);
    ElMessage.error('加载图表数据失败，请稍后重试');
    return {
      series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }],
      graphic: [
        {
          type: 'group',
          left: 'center',
          top: '53%',
          children: [{ type: 'text', style: { text: '无数据', textAlign: 'center', fontSize: 13, fill: '#333' } }],
        },
      ],
    };
  }
};

const loadData = async () => {
  if (!selectedPredictId.value) {
    ElMessage.warning('请先选择有效的预测ID');
    return;
  }

  try {
    const res = await chartDataApi3(isDetailed.value, selectedPredictId.value);
    const resinfo = await equipDetailApi(selectedEquipCode.value);
    unitSimpleName.value = res.data?.equipModel?.equipSimpleName || '暂无数据';
    equipSimpleName.value = res.data?.unit?.unitSimpleName || '暂无数据';
    fightCode.value=res.data?.fightCode || "暂无数据"
    serviceDate.value = res.data?.serviceDate || '未知';
    predictTime.value = res.data?.equipMalf?.predictTime || '暂无数据';
    failureType.value = res.data?.equipMalf?.failureType || '暂无数据';
    vehicleMotor.value = res.data?.equipRecord?.vehicleMotor || '未知';
    manufactureDate.value = res.data?.manufactureDate || '暂无数据';
    failureTiming.value = res.data?.equipMalf?.failureTiming || '暂无数据';
    monthProbability.value = res.data?.equipMalf?.monthProbability || '暂无数据';
    const probabilities = failureTiming.value.split(',').map(Number);
    maxIndex.value = probabilities.indexOf(Math.max(...probabilities)) + 1;
    workMile.value = resinfo.data?.totalWorkMile || '未知';
    motorLimitYear.value = res.data?.equipModel?.motorLimitYear || '未知';
    serviceLimit.value = resinfo.data?.serviceLimit || '暂无数据';
    failureLocation.value = res.data?.equipMalf?.failureLocation || '未知'

    if (predictTime.value) {
      const predictDate = new Date(predictTime.value);
      predictMonth.value = predictDate.getMonth() + 2;
      predictYear.value = predictDate.getFullYear();
      if (predictMonth.value > 12) {
        predictMonth.value -= 12;
        predictYear.value += 1;
      }
    }

    await updateChart(await setChartData());
    await updateChart2(await setChartData2());
    storeSessionData();
  } catch (error) {
    console.error('请求失败:', error);
    ElMessage.error('加载数据失败，请稍后重试');
  }
};

onMounted(async () => {
  // 优先使用路由中的 equipCode
  const queryEquipCode = route.query.equipCode;
  const queryEquipSimpleName = route.query.equipSimpleName
    ? decodeURIComponent(String(route.query.equipSimpleName))
    : '';
  const queryPredictId = route.query.predictId ? parseInt(route.query.predictId as string) : null;

  if (queryEquipCode) {
    selectedEquipCode.value = String(queryEquipCode);
  }

  // 从 sessionStorage 获取批量预测的装备信息
  const selectedEquipments = safeSessionStorage.get('selectedEquipments');
  if (selectedEquipments) {
    const equipments = JSON.parse(selectedEquipments);
    equipCodeOptions.value = [...new Set(equipments.map((item: any) => item.equipCode))].map((code: any) => {
      const equip = equipments.find((e: any) => e.equipCode === code);
      return {
        value: code,
        label: `${code} `,
        predictId: equip?.predictId || null,
      };
    });

    if (queryEquipCode && !equipCodeOptions.value.some(option => option.value === queryEquipCode)) {
      try {
        const resinfo = await equipDetailApi(String(queryEquipCode));
        const equipSimpleNameFromApi = resinfo.data.equipModel?.equipSimpleName || queryEquipSimpleName || '未知型号';
        const latestPredictId = resinfo.data.equipMalf?.predictId || null;
        equipCodeOptions.value.push({
          value: String(queryEquipCode),
          label: `${queryEquipCode}`,
          predictId: latestPredictId,
        });
        selectedPredictId.value = latestPredictId;
      } catch (error) {
        console.error('获取装备详情失败:', error);
        ElMessage.error('加载装备详情失败，请稍后重试');
        equipCodeOptions.value.push({
          value: String(queryEquipCode),
          label: `${queryEquipCode} `,
          predictId: null,
        });
      }
    }

    if (!queryEquipCode && equipments.length > 0) {
      selectedEquipCode.value = equipments[0].equipCode;
      selectedPredictId.value = equipments[0].predictId || null;
    }
  } else if (queryEquipCode) {
    try {
      const resinfo = await equipDetailApi(String(queryEquipCode));
      const equipSimpleNameFromApi = resinfo.data.equipModel?.equipSimpleName || queryEquipSimpleName || '未知型号';
      const latestPredictId = resinfo.data.equipMalf?.predictId || null;
      equipCodeOptions.value = [{
        value: String(queryEquipCode),
        label: `${queryEquipCode} `,
        predictId: latestPredictId,
      }];
      selectedPredictId.value = latestPredictId;
    } catch (error) {
      console.error('获取装备详情失败:', error);
      ElMessage.error('加载装备详情失败，请稍后重试');
      equipCodeOptions.value = [{
        value: String(queryEquipCode),
        label: `${queryEquipCode} `,
        predictId: null,
      }];
    }
  } else {
    equipCodeOptions.value = [];
    ElMessage.warning('请先选择装备进行预测');
  }

  // 如果路由中提供了 predictId，则优先使用
  if (queryPredictId) {
    selectedPredictId.value = queryPredictId;
    const option = equipCodeOptions.value.find(opt => opt.value === selectedEquipCode.value);
    if (option) {
      option.predictId = queryPredictId;
    } else {
      try {
        const resinfo = await equipDetailApi(String(queryEquipCode));
        const equipSimpleNameFromApi = resinfo.data.equipModel?.equipSimpleName || queryEquipSimpleName || '未知型号';
        equipCodeOptions.value.push({
          value: String(queryEquipCode),
          label: `${queryEquipCode}`,
          predictId: queryPredictId,
        });
      } catch (error) {
        console.error('获取装备详情失败:', error);
        ElMessage.error('加载装备详情失败，请稍后重试');
        equipCodeOptions.value.push({
          value: String(queryEquipCode),
          label: `${queryEquipCode} `,
          predictId: queryPredictId,
        });
      }
    }
  }

  console.log('初始 selectedEquipCode:', selectedEquipCode.value);
  console.log('初始 selectedPredictId:', selectedPredictId.value);
  console.log('equipCodeOptions:', equipCodeOptions.value);
  storeSessionData();
  await loadData();
});

watch(() => safeSessionStorage.get('selectedEquipments'), (newEquipments) => {
  if (newEquipments) {
    const equipments = JSON.parse(newEquipments);
    equipCodeOptions.value = equipments.map((item: any) => ({
      value: item.equipCode,
      label: `${item.equipCode}`,
      predictId: item.predictId || null,
    }));

    if (!equipCodeOptions.value.some(option => option.value === selectedEquipCode.value)) {
      selectedEquipCode.value = equipments.length > 0 ? equipments[0].equipCode : '';
      selectedPredictId.value = equipments.length > 0 ? equipments[0].predictId || null : null;
    }

    storeSessionData();
    if (selectedPredictId.value) loadData();
  }
}, { immediate: true });

watch(() => safeSessionStorage.get('selectedPredictions'), (newPredictions) => {
  if (newPredictions) {
    const predictions = JSON.parse(newPredictions);
    equipCodeOptions.value = predictions.map((item: any) => ({
      value: item.equipCode,
      label: `${item.equipCode}`,
      predictId: item.predictId,
    }));

    if (route.query.sessionId && !equipCodeOptions.value.some(option => option.value === selectedEquipCode.value)) {
      if (predictions.length > 0) {
        selectedEquipCode.value = predictions[0].equipCode;
        selectedPredictId.value = predictions[0].predictId;
      }
    }

    storeSessionData();
    if (selectedPredictId.value) loadData();
  }
}, { immediate: true });

// Watch for changes in selectedEquipCode to update selectedPredictId
watch(selectedEquipCode, (newEquipCode) => {
  const selectedOption = equipCodeOptions.value.find(option => option.value === newEquipCode);
  selectedPredictId.value = selectedOption ? selectedOption.predictId : null;
  storeSessionData();
  if (selectedPredictId.value) {
    loadData();
  } else {
    // ElMessage.warning('所选装备没有有效的预测ID');
  }
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

:deep(.el-tabs__header){
  margin-bottom: 0;
}
:deep(.el-tabs__item){
  font-weight: bold;
}
:deep(.el-tabs__item.is-active){
  color: #409EEF;
  background-color: #f5f7fa;
}
</style>