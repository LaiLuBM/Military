<template>
  <div class="report-container">
    <el-card class="report-card">
      <div class="operation-bar hide-print">
        <el-button type="success" size="large" @click="openExportDialog">
          导出故障预测报告
        </el-button>
      </div>

      <div class="report-body">
        <!-- 顶部标题 -->
        <div class="report-header export-item">
          <h1>通用化单装装备故障预测报告</h1>
        </div>

        <el-divider border-style="double" class="export-item" />

        <!-- 一、基本信息 -->
        <div class="section-block export-item">
          <div class="section-title">一、基本信息</div>
          <el-row :gutter="20" class="info-grid">
            <el-col :span="8">
              <div class="info-item">
                <span class="label">装备统一编码：</span>{{ equipCode || 'N/A' }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">装备型号：</span>{{ unitSimpleName }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">所属单位：</span>{{ equipSimpleName }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">列装时间：</span>{{ serviceDate }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">服役日期：</span>{{ serviceLimit }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">大修间隔期内累计行驶里程：</span>{{ workMile }} 公里
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">中修间隔期内发动机累计消耗摩托小时：</span>{{ vehicleMotor }} 小时
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">当年摩托小时消耗限额：</span>{{ motorLimitYear }} 小时
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">出厂日期：</span>{{ manufactureDate }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">预测报告时间：</span>{{ predictTime }}
              </div>
            </el-col>
             <el-col :span="8">
               <div class="info-item">
                <span class="label">装备战斗编号：</span>{{ fightCode }}
              </div>
              
                 
                </el-col>
          </el-row>
        </div>

        <!-- 二、故障预测结果 -->
        <div class="section-block export-item">
          <div class="section-title">二、故障预测结果</div>
          <div class="conclusion-text">
            <el-row :gutter="20">
              <el-col :span="12">
                <div class="conclusion-item">
                  <div class="label">当前故障概率</div>
                  <div class="value highlight-red">{{ currentProbability }}%</div>
                </div>
              </el-col>
              <el-col :span="12">
                <div class="conclusion-item">
                  <div class="label">故障部件（系统）</div>
                  <div class="value">{{ failureLocation }}</div>
                </div>
              </el-col>
              <el-col :span="12">
                <div class="conclusion-item">
                  <div class="label">故障类型</div>
                  <div class="value">{{ failureType }}</div>
                </div>
              </el-col>
              <el-col :span="12">
                <div class="conclusion-item">
                  <div class="label">故障最可能出现的时机</div>
                  <div class="value highlight-red">{{ failureTiming }}</div>
                </div>
              </el-col>
            </el-row>
          </div>
        </div>

        <!-- 三、未来12个月故障概率 -->
        <div class="section-block export-item">
          <div class="section-title">三、未来 12 个月故障概率</div>
          <div ref="chartRef" class="chart-standard"></div>
        </div>
      </div>
    </el-card>

    <!-- 导出弹窗保持不变 -->
    <el-dialog title="选择导出格式" v-model="exportDialogVisible" width="30%">
      <el-radio-group v-model="exportFormat">
        <el-radio label="pdf">PDF 格式</el-radio>
        <el-radio label="ofd">OFD 格式</el-radio>
      </el-radio-group>
      <template #footer>
        <el-button @click="exportDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleExport">确认导出</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, reactive, onMounted } from "vue";
import { useChart } from "@/hooks/useChart";
import { chartDataApi3 } from "@/api/fault/detail";
import { useRouter, useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import html2canvas from 'html2canvas';
import { jsPDF } from 'jspdf';
import { equipDetailApi } from '@/api/fault/equipDetail';
import axios from 'axios'; // 引入 axios 用于发送请求
import { getToken } from "@/utils/auth.js";

const router = useRouter();
const route = useRoute();
const chartRef = ref(null);
const unitSimpleName = ref('');
const equipSimpleName = ref('');
const serviceDate = ref('');
const predictTime = ref('');
const failureType = ref('');
const vehicleMotor = ref('');
const fightCode = ref('');
const manufactureDate = ref('');
const failureLocation = ref('');
const workMile = ref('');
const maxIndex = ref();
const failureTiming = ref('');
const monthProbability = ref('');
const currentProbability = ref(''); // 新增：当前故障概率
const isDetailed = ref(false);
const equipCode = ref('');
const predictId = ref('');
const motorLimitYear = ref('');
const serviceLimit = ref('');
const exportDialogVisible = ref(false); // 控制导出对话框显示
const exportFormat = ref('pdf'); // 默认导出格式为 PDF

// 打开导出格式选择对话框
const openExportDialog = () => {
  exportDialogVisible.value = true;
};

// 关闭导出对话框
const handleCloseExportDialog = () => {
  exportDialogVisible.value = false;
};

const handleExport = async () => {
  exportDialogVisible.value = false;

  try {
    ElMessage.info(`正在生成${exportFormat.value.toUpperCase()}报告，请稍候...`);

    const element = document.querySelector('.el-card') as HTMLElement;
    if (!element) throw new Error('无法找到 .el-card 元素');

    // 隐藏按钮
    const buttons = element.querySelectorAll('.el-button');
    buttons.forEach(btn => btn.classList.add('hide-print'));

    // 忽略返回按钮和导出按钮
    const canvas = await html2canvas(element, {
      scale: 2,
      useCORS: true,
      allowTaint: false,
      backgroundColor: '#ffffff',
      scrollY: -window.scrollY,
      ignoreElements: el => el.classList.contains('el-button') || el.classList.contains('center-content') // 额外的排除
    });

    buttons.forEach(btn => btn.classList.remove('hide-print'));

    const imgData = canvas.toDataURL('image/jpeg', 0.85); // 高质量 JPEG
    const imgWidth = 210; // A4 width (mm)
    const pageHeight = 297; // A4 height (mm)
    const imgHeight = (canvas.height * imgWidth) / canvas.width;

    const pdf = new jsPDF('p', 'mm', 'a4');
    const filename = `通用化故障预测报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

    // === 修复：正确分页 ===
    let heightLeft = imgHeight;
    let position = 0;

    // 第一页
    pdf.addImage(imgData, 'JPEG', 0, position, imgWidth, imgHeight);
    heightLeft -= pageHeight;

    // 后续页（修复 position 计算）
    while (heightLeft > 0) {
      position = heightLeft - imgHeight; // 正确：负值偏移
      pdf.addPage();
      pdf.addImage(imgData, 'JPEG', 0, position, imgWidth, imgHeight);
      heightLeft -= pageHeight;
    }

    if (exportFormat.value === 'pdf') {
      pdf.save(`${filename}.pdf`);
      ElMessage.success('PDF 导出成功');
      return;
    }

    // === OFD 转换 ===
    if (exportFormat.value === 'ofd') {
      const pdfBlob = pdf.output('blob');
      const formData = new FormData();
      formData.append('pdfFile', pdfBlob, `${filename}.pdf`);

      try {
        const response = await axios.post('/bgd_v1/pdf-to-ofd', formData, {
          responseType: 'blob',
          timeout: 180000,
          headers: {
            'Authorization': 'Bearer ' + getToken(),
            'Content-Type': 'multipart/form-data'
          }
        });

        if (response.status === 200) {
          const url = URL.createObjectURL(response.data);
          const link = document.createElement('a');
          link.href = url;
          link.download = `${filename}.ofd`;
          link.click();
          URL.revokeObjectURL(url);
          ElMessage.success('OFD 导出成功');
        }
      } catch (error: any) {
        console.error('OFD 转换失败:', error);
        ElMessage.warning('OFD 转换失败，自动回退为 PDF 下载');
        pdf.save(`${filename}.pdf`); // 回退
      }
    }
  } catch (error) {
    console.error('导出失败:', error);
    ElMessage.error('导出失败，请重试');
  }
};


// 安全存储函数
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

// 存储 session 数据
const storeSessionData = () => {
  if (equipCode.value) {
    safeSessionStorage.set('generalfaultdetailEquipCode', equipCode.value);
  }
  if (predictId.value) {
    safeSessionStorage.set('generalfaultdetailPredictId', predictId.value);
  }
};

// 生成未来12个月的日期（YYYY-MM 格式）
const generateMonths = (startDate: string): string[] => {
  const date = new Date(startDate);
  const startYear = date.getFullYear();
  const startMonth = date.getMonth();
  const result: string[] = [];

  for (let i = 0; i < 12; i++) {
    const monthIndex = (startMonth + i) % 12;
    const yearOffset = Math.floor((startMonth + i) / 12);
    const year = startYear + yearOffset;
    const month = (monthIndex + 1).toString().padStart(2, '0');
    result.push(`${year}-${month}`);
  }
  return result;
};

// 加载数据
const loadData = async () => {
  try {
    const res = await chartDataApi3(isDetailed.value, predictId.value);
    const resinfo = await equipDetailApi(equipCode.value);
    unitSimpleName.value = res.data.equipModel.equipSimpleName || '';
    equipSimpleName.value = res.data.unit.unitSimpleName || ''; // 假设单位和分队都使用 unitSimpleName
    serviceDate.value = res.data.serviceDate || '';
    predictTime.value = res.data.equipMalf.predictTime || '';
    fightCode.value=res.data.fightCode || "暂无数据"
    failureType.value = res.data.equipMalf.failureType || '';
    vehicleMotor.value = res.data.equipRecord.vehicleMotor || '';
    manufactureDate.value = res.data.manufactureDate || '';
    currentProbability.value = (res.data?.equipMalf?.failureProbability * 100).toFixed(2);
    // 计算最可能的故障时机
    const probabilitiesForTiming = failureTiming.value.split(',').map(Number);
    maxIndex.value = probabilitiesForTiming.indexOf(Math.max(...probabilitiesForTiming)) + 1;

    const predictDate = new Date(predictTime.value);
    const failureMonthIndex = (predictDate.getMonth() + maxIndex.value) % 12;
    const failureYearOffset = Math.floor((predictDate.getMonth() + maxIndex.value) / 12);
    const failureYear = predictDate.getFullYear() + failureYearOffset;
    const failureMonth = (failureMonthIndex === 0 ? 12 : failureMonthIndex).toString().padStart(2, '0');
    failureTiming.value = `${failureYear}年${failureMonth}月`;

    workMile.value = resinfo.data.usageRecord?.workMile || '0';
    motorLimitYear.value = resinfo.data.equipModel?.motorLimitYear || '0';
    serviceLimit.value = resinfo.data.serviceLimit;
    failureLocation.value = res.data?.equipMalf?.failureLocation || '未知';
    storeSessionData();
  } catch (error) {
    console.error('请求失败:', error);
  }
};

// 挂载时优先使用查询参数中的 equipCode
onMounted(() => {
  const queryEquipCode = route.query.equipCode;
  const queryPredictId = route.query.predictId;
  if (queryEquipCode) {
    equipCode.value = String(queryEquipCode);
  } else {
    const storageEquipCode = safeSessionStorage.get('generalfaultdetailEquipCode');
    equipCode.value = storageEquipCode || '';
  }
  if (queryPredictId) {
    predictId.value = String(queryPredictId);
  } else {
    const storagePredictId = safeSessionStorage.get('generalfaultdetailPredictId');
    predictId.value = storagePredictId || '';
  }
  storeSessionData();
  loadData();
});

// 监听路由查询参数变化
watch(() => route.query.equipCode, (newEquipCode) => {
  if (newEquipCode) {
    equipCode.value = String(newEquipCode);
    storeSessionData();
    loadData();
  }
});

// 设置折线图数据（未来12个月故障概率）
const setChartData = async () => {
  const chartOptions: any = reactive({
    title: {
      left: 'center',
    },
    tooltip: {
      trigger: 'axis',
      formatter: (params: any) => {
        const data = params[0];
        return `${data.name}<br/>故障概率: ${data.value}%`;
      }
    },
    legend: { data: ['故障概率'] },
    grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
    xAxis: {
      name: '月份',
      type: 'category',
      boundaryGap: false,
      data: []
    },
    yAxis: {
      name: '故障概率(%)',
      type: 'value',
      min: 0,
      max: 100,
      axisLabel: { formatter: '{value}%' }
    },
    series: [
      {
        name: '故障概率',
        type: 'line',
        data: [],
        lineStyle: { width: 2 },
        itemStyle: {
          color: "blue",
        },
        smooth: false
      },
    ]
  });

  const res = await chartDataApi3(isDetailed.value, predictId.value);

  monthProbability.value = res.data.equipMalf.monthProbability || '';
  const testArray: number[] = monthProbability.value
    .split(',')
    .map(Number)
    .map(value => parseFloat((value * 100).toFixed(2)));

  chartOptions.series[0].data = testArray;

  // 设置当前故障概率为第一个月的概率
  if (testArray.length > 0) {
    currentProbability.value = testArray[0].toFixed(2);
  }

  // 使用 predictTime 生成动态月份（完整的 YYYY-MM 格式）
  const predictTimeValue = res.data.equipMalf.predictTime || '';
  chartOptions.xAxis.data = generateMonths(predictTimeValue); // 完整的 YYYY-MM 格式

  return chartOptions;
};

useChart(chartRef, setChartData);
</script>

<style lang="less" scoped>
/* 与 Finingdetailreport.vue 完全一致的报告样式体系 */

.report-container {
  padding: 20px;
  background-color: #f0f2f5;
  min-height: 100vh;
}

.report-card {
  width: 310mm;
  margin: 0 auto;

  :deep(.el-card__body) {
    padding: 0;
  }
}

.operation-bar {
  padding: 10px 20px;
  border-bottom: 1px solid #ebeef5;
  display: flex;
  justify-content: flex-end;
  background: #fff;
  position: sticky;
  top: 0;
  z-index: 100;
}

.report-body {
  padding: 20mm;
  background: #fff;
}

.report-header {
  text-align: center;
  margin-bottom: 30px;

  h1 {
    font-size: 24px;
    color: #333;
  }
}

.section-block {
  margin-bottom: 30px;
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  border-left: 4px solid #409eff;
  padding-left: 10px;
  line-height: 32px;
  background: #f8f9fa;
  margin-bottom: 15px;
}

.info-grid .info-item {
  margin-bottom: 12px;
  font-size: 13px;

  .label {
    font-weight: bold;
  }
}

.conclusion-text {
  margin-top: 10px;
}

.conclusion-item {
  border: 1px solid #eee;
  padding: 12px;
  margin-bottom: 10px;

  .label {
    font-size: 12px;
    color: #999;
  }

  .value {
    font-size: 15px;
    font-weight: bold;
  }

  .highlight-red {
    color: #f56c6c;
  }
}

.chart-standard {
  width: 100%;
  height: 350px;
  /* 统一图表高度，便于导出排版 */
  margin: 20px 0;
}

@media print {
  .hide-print {
    display: none;
  }

  .report-card {
    width: 100%;
    border: none;
  }

  body {
    background: white;
  }
}
</style>