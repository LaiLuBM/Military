<template>
  <div class="report-container">
    <el-card class="report-card">
      <div class="operation-bar hide-print">
        <el-button type="success" size="large" @click="openExportDialog">导出健康报告</el-button>
      </div>

      <div class="report-body">
        <!-- 顶部标题 -->
        <div class="report-header export-item">
          <h1>通用化单装装备健康状况评估报告</h1>
          <div class="report-meta" v-if="approvalStatus == '已核准'">
            <span>核准人：{{ approvalPerson }}</span>　
            <span>核准时间：{{ approvalDate }}</span>　
            <span>提交核准时间：{{ approvalSubmissionDate }}</span>
          </div>
        </div>

        <el-divider border-style="double" class="export-item" />

        <!-- 一、基本信息 -->
        <div class="section-block export-item">
          <div class="section-title">一、基本信息</div>
          <el-row :gutter="20" class="info-grid">
            <el-col :span="8">
              <div class="info-item">
                <span class="label">装备统一编码：</span>{{ equipCode }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">装备战斗编号：</span>{{ fightCode }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">装备型号：</span>{{ equipSimpleName }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">所属单位：</span>{{ unitFullname }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">服役日期：</span>{{ serviceGap }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">出厂日期：</span>{{ manufactureDate }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">故障次数：</span>{{ malfCount }}
              </div>
            </el-col>
            <el-col :span="8">
              <div class="info-item">
                <span class="label">修理次数：</span>{{ repairCount }}
              </div>
            </el-col>
            <el-col :span="16">
              <div class="info-item">
                <span class="label">服役后累计大修间隔期内累计行驶里程：</span>{{ totalWorkMile }} 公里
              </div>
            </el-col>
            
          </el-row>
        </div>

        <!-- 二、综合评估摘要 -->
        <div class="section-block export-item">
          <div class="section-title">二、综合评估摘要</div>
          <div class="conclusion-text">
            <p>总体健康分数：{{ equipHealthScore }}分 | 总体健康等级：{{ equipHealthGrade }}。</p>
          </div>
        </div>

        <!-- 三、近6月健康指数变化 -->
        <div class="section-block export-item">
          <div class="section-title">三、近6月健康指数变化</div>
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



<script lang="ts" setup>
import { ref, reactive, watch, onMounted, computed } from 'vue';
import { useChart } from "@/hooks/useChart";
import { moreAssessmentInfo, submit } from '@/api/fault/refinehealth';
import { useRoute, useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { ElMessage, ElMessageBox } from 'element-plus';
import html2canvas from 'html2canvas';
import { jsPDF } from 'jspdf';
import axios from 'axios'; // 引入 axios 用于发送请求
import { getToken } from "@/utils/auth.js";

const showBasicInfo = ref(false);
const activeTab = ref('historyinformation');
const chartRef = ref(null);
const chartRef2 = ref(null);
const equipCode = ref('');
const fightCode = ref('');
const approvalStatus = ref('');
const assessmentId = ref('');
const equipSimpleName = ref('');
const motorLimitYear = ref('');
const equipHealthGrade = ref('');
const assessmentMethod = ref('');
const algorithm = ref('');
const assessmentDate = ref('');
const unitFullname = ref('');
const manufactureDate = ref('');
const serviceDate = ref('');
const malfCount = ref('');
const repairCount = ref('');
const totalWorkMile = ref('');
const vehicleMotorAfterMajorFix = ref('');
const repairRangeMajorFix = ref('');
const repairRangeMinorFix = ref('');
const repairRangeMiddleFix = ref('');
const approvalSubmissionDate = ref('');
const approvalPerson = ref('');
const approvalDate = ref('');
const bias = ref('');
const equipHealthScore = ref('N/A');
const serviceGap = ref('');
const isSubmitted = ref(false);
const route = useRoute();
const router = useRouter();
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

    const canvas = await html2canvas(element, {
      scale: 2,
      useCORS: true,
      allowTaint: false,
      backgroundColor: '#ffffff',
      scrollY: -window.scrollY,
      ignoreElements: el => el.classList.contains('el-button')
    });

    buttons.forEach(btn => btn.classList.remove('hide-print'));

    const imgData = canvas.toDataURL('image/jpeg', 0.85); // 高质量 JPEG
    const imgWidth = 210; // A4 width (mm)
    const pageHeight = 297; // A4 height (mm)
    const imgHeight = (canvas.height * imgWidth) / canvas.width;

    const pdf = new jsPDF('p', 'mm', 'a4');
    const filename = `通用化健康评估报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

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

const goBack = () => {
  // addTab("通用化健康评估", "/healthevaluate/generalhealthassessments", "Warning");
  // setCurrentTab("通用化健康评估", "/healthevaluate/generalhealthassessments");
  router.push('/healthevaluate/generalhealth');
};

const handleSubmit = async () => {
  try {
    if (approvalStatus.value === '待核准') {
      return;
    }

    if (!assessmentId.value || isNaN(parseInt(assessmentId.value))) {
      ElMessage.error('评估ID无效');
      return;
    }

    await ElMessageBox.confirm('确定要提交核准吗？', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    });

    const response = await submit({
      assessmentId: parseInt(assessmentId.value)
    });

    if (response.code === 200) {
      ElMessage.success('提交核准成功');
      isSubmitted.value = true;
      approvalStatus.value = '待核准';
      storeSessionData();
    } else {
      ElMessage.error(response.message || '提交核准失败');
    }
  } catch (error) {
    if (error !== 'cancel') {
      console.error('提交核准失败:', error);
      ElMessage.error('提交核准失败');
    }
  }
};

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

const storeSessionData = () => {
  if (equipCode.value) {
    safeSessionStorage.set('generalfaultdetailEquipCode', equipCode.value);
  }
  if (approvalStatus.value) {
    safeSessionStorage.set('generalfaultdetailApprovalStatus', approvalStatus.value);
  }
  if (assessmentId.value) {
    safeSessionStorage.set('generalfaultdetailAssessmentId', assessmentId.value);
  }
};

const loadData = async () => {
  try {
    const params = {
      assessmentId: assessmentId.value
    };
    const response = await moreAssessmentInfo(params);
    if (response.code === 200 && response.data && response.data[0] && response.data[0].equipAssessment) {
      equipSimpleName.value = response.data[0].equipSimpleName || 'N/A';
      motorLimitYear.value = response.data[0].motorLimitYear || 'N/A';
      fightCode.value=response.data[0].fightCode || 'N/A'
      equipHealthGrade.value = response.data[0].equipAssessment.equipHealthGrade || 'N/A';
      assessmentMethod.value = response.data[0].equipAssessment.assessmentMethod || 'N/A';
      algorithm.value = response.data[0].equipAssessment.algorithm || 'N/A';
      assessmentDate.value = response.data[0].equipAssessment.assessmentDate || 'N/A';
      approvalSubmissionDate.value = response.data[0].equipAssessment.approvalSubmissionDate || 'N/A';
      approvalDate.value = response.data[0].equipAssessment.approvalDate || 'N/A';
      approvalPerson.value = response.data[0].equipAssessment.approvalPerson || 'N/A';
      unitFullname.value = response.data[0].unitFullname || 'N/A';
      manufactureDate.value = response.data[0].manufactureDate || 'N/A';
      serviceDate.value = response.data[0].serviceDate || 'N/A';
      malfCount.value = response.data[0].malfCount || 'N/A';
      repairCount.value = response.data[0].repairCount || 'N/A';
      totalWorkMile.value = response.data[0].totalWorkMile || '未知';
      vehicleMotorAfterMajorFix.value = response.data[0].equipAssessment.motorHourAssessed || 'N/A';
      repairRangeMajorFix.value = response.data[0].repairRangeMajorFix || 'N/A';
      repairRangeMiddleFix.value = response.data[0].repairRangeMiddleFix || 'N/A';
      repairRangeMinorFix.value = response.data[0].repairRangeMinorFix || 'N/A';
      bias.value = response.data[0].bias || 'N/A';
      equipHealthScore.value = response.data[0].equipAssessment.equipHealthScore || '';
      isSubmitted.value = response.data[0].approvalStatus === '已核准';
      serviceGap.value = response.data[0].serviceGap || 'N/A';
      tableData2.value = response.data[0].equipAssessment.subsysList?.flatMap((subsys: any) =>
        subsys.partList?.map((part: any) => ({
          category: subsys.subsysName || '未知',
          module: part.partName || '未知',
          coreModule: part.isCorePart || '未知',
          accuracyLevel: part.partHealthScore || '未知',
          performanceLevel: part.partHealthGrade || '未知',
        })) || []
      ) || [];
    } else {
      console.error('API response missing expected data:', response);
    }
    storeSessionData();
  } catch (error) {
    console.error('请求失败:', error);
  }
};

onMounted(() => {
  const queryEquipCode = route.query.equipCode;
  const queryApprovalStatus = route.query.approvalStatus;
  const queryAssessmentId = route.query.assessmentId;
  if (queryEquipCode) {
    equipCode.value = String(queryEquipCode);
  } else {
    const storageEquipCode = safeSessionStorage.get('generalfaultdetailEquipCode');
    equipCode.value = storageEquipCode || '';
  }
  if (queryApprovalStatus) {
    approvalStatus.value = String(queryApprovalStatus);
  } else {
    const storageApprovalStatus = safeSessionStorage.get('generalfaultdetailApprovalStatus');
    approvalStatus.value = storageApprovalStatus || '';
  }
  if (queryAssessmentId) {
    assessmentId.value = String(queryAssessmentId);
  } else {
    const storageAssessmentId = safeSessionStorage.get('generalfaultdetailAssessmentId');
    assessmentId.value = storageAssessmentId || '';
  }
  storeSessionData();
  loadData();
});

const handleDetail = (equipCode: string) => {
  router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode);
};

watch(() => route.query.equipCode, (newEquipCode) => {
  if (newEquipCode) {
    equipCode.value = String(newEquipCode);
    storeSessionData();
    loadData();
  }
});
watch(() => route.query.approvalStatus, (newApprovalStatus) => {
  if (newApprovalStatus) {
    approvalStatus.value = String(newApprovalStatus);
    storeSessionData();
    loadData();
  }
});
watch(() => route.query.assessmentId, (newAssessmentId) => {
  if (newAssessmentId) {
    assessmentId.value = String(newAssessmentId);
    storeSessionData();
    loadData();
  }
});

const tableData = computed(() => [
  {
    item: `中修间隔期内发动机累计消耗摩托小时（从大修以后的值）|${vehicleMotorAfterMajorFix.value || 'N/A'}`,
    value1: `小修摩托小时：${repairRangeMinorFix.value || 'N/A'}|中修摩托小时：${repairRangeMiddleFix.value || 'N/A'} |大修摩托小时：${repairRangeMajorFix.value || 'N/A'}`,
    value2: `${bias.value || 'N/A'}`,
    bias: {
      value: bias.value || 'N/A'
    }
  }
]);

const getTagType = (status: any) => {
  const statusStr = String(status?.value || status || '');
  if (statusStr.includes('范围内') && !statusStr.includes('边缘')) {
    return 'primary';
  } else if (statusStr.includes('边缘范围内10%')) {
    return 'success';
  } else if (statusStr.includes('边缘范围外10%')) {
    return 'warning';
  } else if (statusStr.includes('范围外')) {
    return 'danger';
  }
  return 'info';
};

const tableData2 = ref([]);

const headerCellStyle = {
  backgroundColor: '#d9d9db',
  color: 'black',
  fontWeight: 'bold'
};

const setChartData2 = async () => {
  const params = {
    assessmentId: assessmentId.value
  };
  const response = await moreAssessmentInfo(params);
  equipHealthScore.value = response.data[0]?.equipAssessment?.equipHealthScore ?? 'N/A';
  const chartOptions: any = reactive({
    series: [
      {
        name: "健康指数",
        type: "pie",
        radius: ["50%", "70%"],
        avoidLabelOverlap: false,
        label: { show: false },
        emphasis: { label: { show: false } },
        itemStyle: {
          color: function (colors: { dataIndex: number }) {
            return ["#007BFF", "#E0E0E0"][colors.dataIndex];
          },
        },
        data: [
          { value: equipHealthScore.value, name: '健康指数' },
          { value: 100 - Number(equipHealthScore.value), name: '' },
        ],
      },
    ],
    graphic: [
      {
        type: "group",
        left: "center",
        top: "56%",
        bounding: "raw",
        children: [
          {
            type: "text",
            style: {
              text: `${equipHealthScore.value}`,
              fontSize: 24,
              textAlign: "center",
              textVerticalAlign: "bottom",
              fill: "#007BFF",
            },
          },
        ],
      },
    ],
    labelLine: { show: false },
    data: [],
  });
  console.log('setChartData2 - equipHealthScore:', equipHealthScore.value);
  return chartOptions;
};

const setChartData = async () => {
  // Fetch data from the API
  const params = {
    assessmentId: assessmentId.value
  };
  const response = await moreAssessmentInfo(params);

  // Extract timeArray and equipScore from the response
  const timeArray = response.data[0]?.chart?.timeArray;
  // Format equipScore data to retain two decimal places
  const equipScore = response.data[0]?.chart?.equipScore?.map((score: number) =>
    typeof score === 'number' ? parseFloat(score.toFixed(2)) : score
  );

  const chartOptions: any = reactive({
    tooltip: {
      trigger: 'axis',
    },
    legend: {
      data: ['整机'],
      top: 'top',
    },
    xAxis: {
      type: 'category',
      data: timeArray, // Use timeArray from backend response
      axisLabel: {
        fontSize: 12,
        color: '#606266',
      },
    },
    yAxis: {
      type: 'value',
      min: 0,
      max: 100,
      interval: 20,
      axisLabel: {
        fontSize: 12,
        color: '#606266',
      },
    },
    series: [
      {
        name: '整机',
        type: 'line',
        data: equipScore,
        itemStyle: { color: '#5470C6' },
        smooth: false,
        lineStyle: {
          width: 2,
          type: 'solid'
        },
        showSymbol: false,
        symbol: 'circle',
        symbolSize: 8,
      },
    ],
  });
  return chartOptions;
};

useChart(chartRef, setChartData);
useChart(chartRef2, setChartData2);
</script>

<style lang="less" scoped>
/* 与 Finingdetailreport.vue 完全一致的报告样式 */

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

.report-meta {
  margin-top: 15px;
  font-size: 14px;
  color: #666;
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
  font-size: 15px;
  line-height: 1.8;
  margin-top: 10px;
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