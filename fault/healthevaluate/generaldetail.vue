<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <el-row :gutter="20">
            <el-col :span="1">
              <!-- 添加返回按钮 -->
              <el-button text @click="goBack" icon="ArrowLeft"
                style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
            </el-col>
            <el-col :span="20">
              <h2 class="title-margin">通用化健康评估结果详情</h2>
            </el-col>
          </el-row>
          <el-row>
            <div class="bottom"></div>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="20">
            </el-col>
            <el-col :span="2" v-if="approvalStatus !== '已核准'" style="text-align: right;">
              <el-button type="primary" :disabled="approvalStatus === '待核准' || isSubmitted" @click="handleSubmit">
                提交核准
              </el-button>
            </el-col>
            <el-col :span="2" style="text-align: right;">
              <el-button type="success" @click="viewreport">查看健康报告</el-button>
            </el-col>
          </el-row>
          <el-row :gutter="20" v-if="approvalStatus == '已核准'" style="margin-top: 20px;">
            <el-col :span="6">
              核准人：{{ approvalPerson }}
            </el-col>
            <el-col :span="6">
              核准时间：{{ approvalDate }}
            </el-col>
            <el-col :span="6">
              提交核准时间：{{ approvalSubmissionDate }}
            </el-col>
          </el-row>
          <div class="outer-border" style="margin-top: 10px;">
            <el-row :gutter="20">
              <el-col :span="5">
                <div ref="chartRef2" style="width: auto; height: 175px;"></div>
                <div style="text-align: center; font-weight: bold;">健康指数</div>
                <div class="line"></div>
              </el-col>
              <el-col :span="15">
                <el-row style="margin-top:30px;">
                  <el-col :span="24">
                    <div class="section-title"><el-icon>
                        <Management />
                      </el-icon>基本信息</div>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 20px;">

                  <el-col :span="8">
                    <div>所属单位：{{ unitFullname }}</div>
                  </el-col>
                  <el-col :span="8">
                    <div>装备型号：{{ equipSimpleName }}</div>
                  </el-col>
                  <el-col :span="8">
                    <div>装备统一编码：{{ equipCode }}</div>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 30px;">
                  <el-col :span="8">
                    <div>服役日期：{{ serviceGap }}</div>
                  </el-col>
                  <el-col :span="8">
                    <div>出厂日期：{{ manufactureDate }}</div>
                  </el-col>
                  <el-col :span="8">
                    <div>装备战斗编号：{{ fightCode }}</div>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 20px;"></el-row>
                <el-row>
                  <!-- <el-col :span="1">
                  <div class="info-box"></div>
                </el-col> -->
                  <el-col :span="23">
                    <div class="section-title"><el-icon>
                        <Management />
                      </el-icon>截止到评估时间（{{ assessmentDate }}）的装备状况</div>
                  </el-col>
                </el-row>
              </el-col>
              <el-col :span="4" class="image-col">
                <img src="../../../assets/tanks/tank.png" class="tank-image" />
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="5">
                <el-row>
                  <div class="info">健康等级：{{ equipHealthGrade }}</div>
                </el-row>
                <el-row>
                  <div class="info">评估方式：{{ assessmentMethod }}</div>
                </el-row>
                <el-row>
                  <div class="info">算法模型：{{ algorithm }}</div>
                </el-row>
              </el-col>
              <el-col :span="19">
                <el-row style="margin-top: 20px;">
                  <!-- <el-col :span="6">
                    <div>列装时间：{{ motorLimitYear }}年</div>
                  </el-col> -->
                  <el-col :span="6">
                    <div>故障次数：{{ malfCount }}</div>
                  </el-col>
                  <el-col :span="13"></el-col>
                  <el-col :span="5">
                    <el-button type="primary" plain @click="handleDetail(equipCode)">查看更多装备信息</el-button>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 30px;">
                  <el-col :span="6">
                    <div>修理次数：{{ repairCount }}</div>
                  </el-col>
                  <el-col :span="10">
                    <div>服役后累计大修间隔期内累计行驶里程：{{ totalWorkMile }}公里</div>
                  </el-col>
                  <el-col :span="6"></el-col>
                </el-row>
                <el-row style="margin-top: 20px;">
                  <el-table :data="tableData" :header-cell-style="headerCellStyle" style="width: 100%" border>
                    <el-table-column prop="item" label="实际值" header-align="center" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column prop="value1" label="标准上限值" header-align="center" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column prop="value2" label="偏离" header-align="center" align="center" :show-overflow-tooltip="true">
                      <template #default="{ row }">
                        <div style="display: flex; height: 100%;">
                          <div
                            style="flex: 1; text-align: center; display: flex; align-items: center; justify-content: center;">
                            {{ row.bias.value }}
                          </div>
                          <div
                            style="flex: 1; text-align: center; display: flex; align-items: center; justify-content: center;">
                            <el-tag :type="getTagType(row.bias)" effect="dark" round>
                              <el-icon>
                                <Check />
                              </el-icon>
                            </el-tag>
                          </div>
                        </div>
                      </template>
                    </el-table-column>
                  </el-table>
                </el-row>
                <el-row style="margin-top: 20px; float: right;">
                  <div class="progress-container">
                    <div class="progress-item">
                      <span class="label">范围内</span>
                      <div class="bar blue"></div>
                      <div>|</div>
                    </div>
                    <div class="progress-item">
                      <span class="label">边缘范围内10%</span>
                      <div class="bar green"></div>
                    </div>
                    <div>|</div>
                    <div class="progress-item">
                      <span class="label">边缘范围外10%</span>
                      <div class="bar orange"></div>
                    </div>
                    <div>|</div>
                    <div class="progress-item">
                      <span class="label">范围外</span>
                      <div class="bar red"></div>
                    </div>
                  </div>
                </el-row>
                <div style="clear: both;"></div>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="5"></el-col>
              <el-col :span="19">
                <el-row>
                  <h2>近6月健康指数变化</h2>
                  <div ref="chartRef" style="width: 100%; height: 400px;"></div>
                </el-row>
              </el-col>
            </el-row>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <el-dialog title="选择导出格式" v-model="exportDialogVisible" width="30%" :before-close="handleCloseExportDialog">
      <el-radio-group v-model="exportFormat" class="export-format-radio">
        <el-radio label="pdf" size="large">PDF 格式</el-radio>
        <el-radio label="ofd" size="large">OFD 格式</el-radio>
      </el-radio-group>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="exportDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleExport">确认导出</el-button>
        </span>
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
const viewreport = () => {
  const params = {
    equipCode: equipCode.value || '',
    approvalStatus: approvalStatus.value || '',
    assessmentId: assessmentId.value || ''
  };
  router.push({
    path: "/healthevaluate/generalreport",
    query: params
  });
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
  window.history.back()
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
      equipHealthScore.value = response.data[0].equipAssessment.equipHealthScore || 'N/A'; // Added to match refinedetail.vue
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
    item: `摩托小时（从大修以后的值）|${vehicleMotorAfterMajorFix.value || 'N/A'}`,
     value1: `小修摩托小时：${"250"}|中修摩托小时：${"500"} |大修摩托小时：${"1000"}`,
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
.custom-header-cell {
  background-color: #d9d9db;
  font-weight: bold;
}

.bottom {
  margin-bottom: 20px;
}

.info {
  margin-bottom: 20px;
  display: flex;
  margin-left: 70px;
  margin-top: 20px;
}

.tank-image {
  width: auto;
  height: 90%;
}

.info-box {
  width: 10px;
  height: 20px;
  background-color: #409eff;
  display: flex;
  align-items: center;
  justify-content: center;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
  margin-left: 0;
  color: #409eff;
  display: flex;
  align-items: center;
}

.line {
  content: "";
  display: block;
  width: 85%;
  height: 1px;
  background-color: rgb(177, 174, 174);
  margin: 0 auto;
  margin-top: 10px;
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: Arial, sans-serif;
}

.progress-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

.label {
  font-size: 14px;
  color: #333;
}

.bar {
  height: 10px;
  width: 15px;
  border-radius: 1px;
}

.blue {
  background-color: #007bff;
}

.green {
  background-color: #28a745;
}

.orange {
  background-color: #ffc107;
}

.red {
  background-color: #dc3545;
}

.vertical-line {
  width: 1px;
  height: 104%;
  background-color: #ccc;
  margin-top: -25px;
}

.parallel-line {
  height: 1px;
  width: 100%;
  background-color: #ccc;
}

.header {
  width: 95%;
  font-size: 16px;
  font-weight: bold;
  color: #606266;
  padding: 15px 20px;
  background-color: #f5f7fa;
}

.outer-border {
  border: 2px solid #dfe1e4;
  border-radius: 2px;
  padding: 25px;
}

.blue-square {
  width: 20px;
  height: 10px;
  border: 2px dashed #409eff;
  background-color: transparent;
  display: inline-block;
  vertical-align: middle;
  margin-right: 8px;
}

.equip-code-display {
  display: inline-block;
  width: 240px;
  padding: 8px 12px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  background-color: #fff;
  font-size: 14px;
  color: #606266;
  line-height: 1.5;
  text-align: left;
}

.hide-print {
  display: none !important;
}

@media print {
  body * {
    visibility: hidden;
  }

  .el-card,
  .el-card * {
    visibility: visible;
  }

  .el-card {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    margin: 0;
    padding: 0;
    box-shadow: none;
  }

  .el-button,
  .el-select {
    display: none !important;
  }

  .echarts {
    visibility: visible !important;
  }

  .equip-code-display {
    visibility: visible !important;
    font-size: 14px !important;
    color: #606266 !important;
    border: 1px solid #dcdfe6 !important;
    border-radius: 4px !important;
    background-color: #fff !important;
    padding: 8px 12px !important;
  }
}
</style>