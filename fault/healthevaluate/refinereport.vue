<template>
  <div class="report-container">
    <el-card class="report-card">
      <div class="operation-bar hide-print">
        <el-button type="success" size="large" @click="openExportDialog">导出健康报告</el-button>
      </div>

      <div id="report-content" class="report-body">
        <!-- 顶部标题 -->
        <div class="report-header export-item">
          <h1>精细化单装装备健康状况评估报告</h1>
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

        <!-- 三、分系统健康状况 -->
        <div class="section-block export-item">
          <div class="section-title">三、分系统健康状况</div>
          <el-table :data="tableData2" border :header-cell-style="headerCellStyle" :span-method="tableSpanMethod"
            class="health-table" row-key="id">
            <el-table-column prop="category" label="分系统" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="module" label="部件" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="coreModule" label="是否核心部件" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="accuracyLevel" label="健康指数" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="performanceLevel" label="健康等级" align="center"  :show-overflow-tooltip="true"/>
          </el-table>
        </div>

        <!-- 四、近6月健康指数变化 -->
        <div class="section-block export-item">
          <div class="section-title">四、近6月健康指数变化</div>
          <div ref="chartRef" class="chart-standard"></div>
        </div>

        <div v-if="suggestionGeneratedStatus !== '未生成'" class="export-item">
          <!-- 五、健康建议 -->
          <div class="section-block">
            <div class="section-title">五、健康建议</div>

            <div class="sub-section export-item">
              <h3>1. 视情维修建议（待维修部件列表）</h3>
              <el-table :data="tableData3" border :header-cell-style="headerCellStyle" class="health-table">
                <el-table-column prop="category" label="分系统"  :show-overflow-tooltip="true"/>
                <el-table-column prop="module" label="部件"  :show-overflow-tooltip="true"/>
                <el-table-column prop="coreModule" label="是否核心部件"  :show-overflow-tooltip="true"/>
                <el-table-column prop="accuracyLevel" label="健康指数"  :show-overflow-tooltip="true"/>
              </el-table>
            </div>

            <el-divider class="export-item" />

            <div class="sub-section export-item">
              <h3>2. 延寿使用建议</h3>
              <el-row :gutter="20">
                <el-col :span="6">
                  <div class="info-item"><span class="label">中修间隔期内发动机累计消耗摩托小时：</span>{{ enginevehicleMotor }} h</div>
                  <div class="info-item"><span class="label">健康指数：</span>{{ equipHealthScore }}</div>
                </el-col>
                <el-col :span="12">
                  <div ref="chartRef4" class="chart-standard"></div>
                </el-col>
                <el-col :span="6">
                  <div class="info-item"><span class="label">是否延寿：</span>{{ isLifeExtend }}</div>
                  <div class="info-item"><span class="label">延寿时间：</span>{{ lifeExtendLength }}</div>
                  <div class="info-item"><span class="label">延寿方式：</span>{{ lifeExtendMethod }}</div>
                </el-col>
              </el-row>
            </div>

            <el-divider class="export-item" />

            <div class="sub-section export-item">
              <h3>3. 作战运用建议</h3>
              <div class="conclusion-text">{{ missionAdvice }}</div>
            </div>

            <el-divider class="export-item" />

            <div class="sub-section export-item">
              <h3>4. 维修保养建议</h3>
              <el-table :data="tableData4" border :header-cell-style="headerCellStyle" class="health-table">
                <el-table-column prop="category" label="分系统"  :show-overflow-tooltip="true"/>
                <el-table-column prop="module" label="部件名称"  :show-overflow-tooltip="true"/>
                <el-table-column prop="accuracyLevel" label="健康指数"  :show-overflow-tooltip="true"/>
                <el-table-column prop="coreModule" label="健康等级"  :show-overflow-tooltip="true"/>
                <el-table-column prop="performanceLevel" label="维修优先级"  :show-overflow-tooltip="true"/>
                <el-table-column prop="manage" label="操作"  :show-overflow-tooltip="true"/>
              </el-table>
            </div>
          </div>
        </div>
      </div>
    </el-card>

    <!-- 导出弹窗 -->
    <el-dialog title="选择导出格式" v-model="exportDialogVisible" width="30%">
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
import { ref, reactive, watch, onMounted, computed, nextTick } from 'vue';
import { useChart } from "@/hooks/useChart";
import { useRoute } from 'vue-router';
import { useRouter } from 'vue-router';
import { moreAssessmentInfo, submit } from '@/api/fault/refinehealth';
import { ElMessage, ElMessageBox, ElLoading } from 'element-plus';
import html2canvas from 'html2canvas';
import { jsPDF } from 'jspdf';
import axios from 'axios';
import { getToken } from "@/utils/auth.js";

const showBasicInfo = ref(false);
const activeTab = ref('historyinformation');
const chartRef = ref(null);
const chartRef2 = ref(null);
const chartRef3 = ref(null);
const chartRef4 = ref(null);
const equipCode = ref('');
const approvalStatus = ref('');
const assessmentId = ref('');
const fightCode = ref('');
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
const bias = ref('');
const equipHealthScore = ref();
const route = useRoute();
const missionAdvice = ref('');
const enginevehicleMotor = ref('');
const engineHealthScore = ref('');
const isLifeExtend = ref('');
const lifeExtendLength = ref('');
const lifeExtendMethod = ref('');
const approvalSubmissionDate = ref('');
const approvalPerson = ref('');
const approvalDate = ref('');
const isSubmitted = ref(false);
const suggestionGeneratedStatus = ref('');
const serviceGap = ref('');
const exportDialogVisible = ref(false);
const exportFormat = ref('pdf');

// 表格合并数据存储
const spanArr = ref<number[]>([]); // 存储每个分系统的行数
const position = ref(0); // 记录分系统开始的位置

// 表格单元格合并方法
const tableSpanMethod = ({ row, column, rowIndex, columnIndex }: any) => {
  // 只合并第一列（分系统列）
  if (columnIndex === 0) {
    const _row = spanArr.value[rowIndex];
    const _col = _row > 0 ? 1 : 0;
    return {
      rowspan: _row,
      colspan: _col
    };
  }
  // 其他列不合并
  return {
    rowspan: 1,
    colspan: 1
  };
};

// 计算表格合并数据
const getSpanArr = (data: any[]) => {
  if (!data || data.length === 0) {
    spanArr.value = [];
    return;
  }

  spanArr.value = [];
  position.value = 0;

  // 按分系统分组计算行数
  for (let i = 0; i < data.length; i++) {
    if (i === 0) {
      // 第一个元素
      spanArr.value.push(1);
    } else {
      // 判断当前分系统是否与上一个相同
      if (data[i].category === data[i - 1].category) {
        spanArr.value[position.value] += 1;
        spanArr.value.push(0);
      } else {
        // 新的分系统
        spanArr.value.push(1);
        position.value = i;
      }
    }
  }
};

// 打开导出格式选择对话框
const openExportDialog = () => {
  exportDialogVisible.value = true;
};

const handleExport = async () => {
  exportDialogVisible.value = false;
  const element = document.getElementById("report-content");
  if (!element) return;

  const loading = ElLoading.service({ text: "正在优化排版并导出..." });
  try {
    const pdf = new jsPDF("p", "mm", "a4");
    const pageWidth = 210;
    const pageHeight = 297;
    const margin = 10;
    const contentWidth = pageWidth - margin * 2;

    const items = element.querySelectorAll(".export-item");
    let currentY = margin;

    for (let i = 0; i < items.length; i++) {
      const item = items[i] as HTMLElement;
      const canvas = await html2canvas(item, { scale: 2, useCORS: true });
      const imgData = canvas.toDataURL("image/jpeg", 1.0);
      const imgHeight = (canvas.height * contentWidth) / canvas.width;

      if (currentY + imgHeight > pageHeight - margin) {
        pdf.addPage();
        currentY = margin;
      }

      pdf.addImage(imgData, "JPEG", margin, currentY, contentWidth, imgHeight);
      currentY += imgHeight + 5;
    }

    const filename = `精细化健康评估报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

    if (exportFormat.value === "pdf") {
      pdf.save(`${filename}.pdf`);
      ElMessage.success("PDF 导出成功");
    } else {
      const pdfBlob = pdf.output("blob");
      const formData = new FormData();
      formData.append("pdfFile", pdfBlob, `${filename}.pdf`);
      const response = await axios.post("/bgd_v1/pdf-to-ofd", formData, {
        responseType: "blob",
        headers: { Authorization: "Bearer " + getToken() }
      });
      const url = URL.createObjectURL(response.data);
      const link = document.createElement("a");
      link.href = url;
      link.download = `${filename}.ofd`;
      link.click();
      ElMessage.success("OFD 导出成功");
    }
  } catch (e) {
    ElMessage.error("导出失败");
    console.error(e);
  } finally {
    loading.close();
  }
};

const goBack = () => {
  window.history.back();
};

const handleSubmit = async () => {
  try {
    // 1. 检查核准状态
    if (approvalStatus.value === '待核准') {
      return;
    }

    // 2. 验证 assessmentId
    if (!assessmentId.value || isNaN(parseInt(assessmentId.value))) {
      ElMessage.error('评估ID无效');
      return;
    }

    // 3. 确认对话框
    await ElMessageBox.confirm('确定要提交核准吗？', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    });

    // 4. 发送请求
    const response = await submit({
      assessmentId: parseInt(assessmentId.value)
    });

    // 5. 处理响应
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

// 存储会话数据的函数
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

const tableData2 = ref<any[]>([]);

const loadData = async () => {
  try {
    const params = {
      assessmentId: assessmentId.value,
    };
    const response = await moreAssessmentInfo(params);
    console.log('API response:', response);
    if (response.code === 200 && response.data && response.data[0] && response.data[0].equipAssessment) {
      equipSimpleName.value = response.data[0].equipSimpleName || 'N/A';
      motorLimitYear.value = response.data[0].motorLimitYear || 'N/A';
      fightCode.value = response.data[0].fightCode || 'N/A';
      equipHealthGrade.value = response.data[0].equipAssessment.equipHealthGrade || 'N/A';
      assessmentMethod.value = response.data[0].equipAssessment.assessmentMethod || 'N/A';
      algorithm.value = response.data[0].equipAssessment.algorithm || 'N/A';
      assessmentDate.value = response.data[0].equipAssessment.assessmentDate || 'N/A';
      approvalSubmissionDate.value = response.data[0].equipAssessment.approvalSubmissionDate || 'N/A';
      approvalDate.value = response.data[0].equipAssessment.approvalDate || 'N/A';
      approvalPerson.value = response.data[0].equipAssessment.approvalPerson || 'N/A';
      equipHealthScore.value = response.data[0].equipAssessment.equipHealthScore || '';
      unitFullname.value = response.data[0].unitFullname || 'N/A';
      manufactureDate.value = response.data[0].manufactureDate || 'N/A';
      serviceDate.value = response.data[0].serviceDate || 'N/A';
      malfCount.value = response.data[0].malfCount || 'N/A';
      repairCount.value = response.data[0].repairCount || 'N/A';
      totalWorkMile.value = response.data[0].totalWorkMile || ' 0.0';
      vehicleMotorAfterMajorFix.value = response.data[0].motorHourAfterMajorFix || '0.0';
      repairRangeMajorFix.value = response.data[0].repairRangeMajorFix || '0';
      repairRangeMiddleFix.value = response.data[0].repairRangeMiddleFix || '0';
      repairRangeMinorFix.value = response.data[0].repairRangeMinorFix || '0';
      bias.value = response.data[0].bias || 'N/A';
      missionAdvice.value = response.data[0].advice.missionAdvice || 'N/A';
      enginevehicleMotor.value = response.data[0].advice.engineMotorHour || '0';
      engineHealthScore.value = response.data[0].advice.engineHealthScore || '0';
      isLifeExtend.value = response.data[0].advice.isLifeExtend || 'N/A';
      serviceGap.value = response.data[0].serviceGap || 'N/A';
      lifeExtendLength.value = response.data[0].advice.lifeExtendLength || 'N/A';
      lifeExtendMethod.value = response.data[0].advice.lifeExtendMethod || 'N/A';
      suggestionGeneratedStatus.value = response.data[0].equipAssessment.suggestionGeneratedStatus || 'N/A';

      // 生成 tableData2，处理分系统数据
      const tableData: any[] = [];
      let idCounter = 1;

      response.data[0].equipAssessment.subsysHealthList?.forEach((subsys: any) => {
        const parts = subsys.partHealthList || [];
        parts.forEach((part: any, partIndex: number) => {
          tableData.push({
            id: idCounter++,
            category: subsys.subsysName || '未知',
            module: part.partName || '未知',
            coreModule: part.isCore || '未知',
            accuracyLevel: part.partHealthScore || '未知',
            performanceLevel: part.partHealthGrade || '未知',
            // 添加排序字段，确保同一分系统的行连续
            sortKey: `${subsys.subsysName}_${partIndex}`
          });
        });
      });

      // 按分系统排序，确保同一分系统的行连续
      tableData.sort((a, b) => a.sortKey.localeCompare(b.sortKey));

      // 移除排序字段
      tableData.forEach(item => delete item.sortKey);

      tableData2.value = tableData;

      // 计算合并数据
      getSpanArr(tableData2.value);

      console.log('tableData2:', tableData2.value);
      console.log('spanArr:', spanArr.value);
    } else {
      console.error('API response missing expected data:', response);
      tableData2.value = [];
    }
    storeSessionData();
  } catch (error) {
    console.error('请求失败:', error);
    tableData2.value = [];
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

//详情跳转
const router = useRouter();
const handleDetail = (equipCode: string) => {
  router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode);
};

// 监听路由参数变化
watch(() => route.query.equipCode, (newEquipCode) => {
  console.log('Watched equipCode change:', newEquipCode);
  if (newEquipCode) {
    equipCode.value = String(newEquipCode);
    console.log('Route updated equipCode:', equipCode.value);
    storeSessionData();
    loadData();
  }
});
watch(() => route.query.approvalStatus, (newApprovalStatus) => {
  console.log('Watched approvalStatus change:', newApprovalStatus);
  if (newApprovalStatus) {
    approvalStatus.value = String(newApprovalStatus);
    console.log('Route updated approvalStatus:', approvalStatus.value);
    storeSessionData();
    loadData();
  }
});
watch(() => route.query.assessmentId, (newAssessmentId) => {
  console.log('Watched assessmentId change:', newAssessmentId);
  if (newAssessmentId) {
    assessmentId.value = String(newAssessmentId);
    console.log('Route updated assessmentId:', assessmentId.value);
    storeSessionData();
    loadData();
  }
});

// 监听表格数据变化，重新计算合并
watch(() => tableData2.value, (newData) => {
  if (newData && newData.length > 0) {
    getSpanArr(newData);
  }
}, { deep: true });

const tableData = computed(() => [
  {
    item: `中修间隔期内发动机累计消耗摩托小时（从大修以后的值）|${vehicleMotorAfterMajorFix.value || '0.0'}`,
    value1: `小修摩托小时：${repairRangeMinorFix.value || '0.0'}|中修摩托小时：${repairRangeMiddleFix.value || '0.0'} |大修摩托小时：${repairRangeMajorFix.value || '0.0'}`,
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

const tableData3 = ref([
  {
    category: '系统1',
    module: '模块1',
    coreModule: '是',
    accuracyLevel: '80',
  },
  {
    category: '系统2',
    module: '模块2',
    coreModule: '否',
    accuracyLevel: '60',
  }
]);
const tableData4 = ref([
  {
    category: '系统1',
    module: '模块1',
    coreModule: '优秀',
    accuracyLevel: '65',
    performanceLevel: '高',
    online: '优秀',
    methodType: '算法1',
    manage: '保养'
  },
]);

// 设置表头样式
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
  equipHealthScore.value = response.data[0].equipAssessment.equipHealthScore ?? 'N/A';
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
        data: [],
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
  chartOptions.series[0].data = [
    { value: equipHealthScore.value, name: '健康指数' },
    { value: 100 - equipHealthScore.value, name: '' },
  ];
  return chartOptions;
};

const setChartData3 = async () => {
  const params = {
    assessmentId: assessmentId.value
  };
  const response = await moreAssessmentInfo(params);
  const healthGradePercent = response.data[0]?.healthGradePercent || {
    "优秀": 0.0,
    "良好": 0.0,
    "一般": 0.0,
    "较差": 0.0,
  };

  const chartOptions: any = reactive({
    legend: {
      top: 'bottom'
    },
    title: {
      text: '部件健康等级占比分布',
      left: 'center'
    },
    tooltip: {
      trigger: 'item'
    },
    series: [
      {
        name: '部件健康等级占比分布',
        type: 'pie',
        radius: '65%',
        data: [
          { value: healthGradePercent["优秀"] * 100, name: '优秀(90-100)' },
          { value: healthGradePercent["良好"] * 100, name: '良好(80-90)' },
          { value: healthGradePercent["一般"] * 100, name: '一般(60-80)' },
          { value: healthGradePercent["较差"] * 100, name: '较差(0-60)' },
        ],
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)'
          }
        }
      }
    ],
  });
  return chartOptions;
};

const setChartData = async () => {
  const params = {
    assessmentId: assessmentId.value
  };
  const response = await moreAssessmentInfo(params);
  const chartData = response.data[0]?.chart || {};

  // 提取后端数据
  const timeArray = chartData.timeArray;
  const equipScore = chartData.equipScore;
  const dipanScore = chartData.dipanScore;
  const tongxinScore = chartData.tongxinScore;
  const dianqiScore = chartData.dianqiScore;
  const wuqiScore = chartData.wuqiScore;
  const huokongScore = chartData.huokongScore;

  const chartOptions: any = reactive({
    tooltip: {
      trigger: 'axis',
    },
    legend: {
      data: ['整体', '底盘系统', '通信系统', '电气系统', '武器系统', '火控系统'],
      top: 'top',
    },
    xAxis: {
      type: 'category',
      data: timeArray,
    },
    yAxis: {
      type: 'value',
      min: 0,
      max: 100,
      interval: 20,
    },
    series: [
      {
        name: '整体',
        type: 'line',
        data: equipScore,
        itemStyle: { color: '#5470C6' },
        smooth: true,
      },
      {
        name: '底盘系统',
        type: 'line',
        data: dipanScore,
        itemStyle: { color: '#91CC75' },
        smooth: true,
      },
      {
        name: '通信系统',
        type: 'line',
        data: tongxinScore,
        itemStyle: { color: '#EE6666' },
        smooth: true,
      },
      {
        name: '电气系统',
        type: 'line',
        data: dianqiScore,
        itemStyle: { color: '#73C0DE' },
        smooth: true,
      },
      {
        name: '武器系统',
        type: 'line',
        data: wuqiScore,
        itemStyle: { color: '#FAC858' },
        smooth: true,
      },
      {
        name: '火控系统',
        type: 'line',
        data: huokongScore,
        itemStyle: { color: '#3BA272' },
        smooth: true,
      },
    ],
  });

  return chartOptions;
};

const setChartData4 = async () => {
  const chartOptions: any = reactive({
    title: {
      text: '发动机最近6个月健康指标曲线',
      left: 'center',
      top: '5px',
      textStyle: {
        fontSize: 20,
        fontWeight: 'bold',
      },
    },
    tooltip: {
      trigger: 'axis',
    },
    xAxis: {
      type: 'category',
      data: ['1月', '2月', '3月', '4月', '5月', '6月'],
    },
    yAxis: {
      type: 'value',
      min: 0,
      height: 100,
      interval: 20,
    },
    series: [
      {
        name: '健康指标',
        type: 'line',
        data: [80, 75, 70, 65, 70, 60],
        itemStyle: { color: '#00C1B3' },
        areaStyle: {
          color: {
            type: 'linear',
            x: 0,
            y: 0,
            x2: 0,
            y2: 1,
            colorStops: [
              { offset: 0, color: 'rgba(0, 193, 179, 0.3)' },
              { offset: 1, color: 'rgba(0, 193, 179, 0)' },
            ],
          },
        },
        smooth: true,
      },
    ]
  });
  return chartOptions;
};

useChart(chartRef, setChartData);
useChart(chartRef2, setChartData2);
useChart(chartRef3, setChartData3);
useChart(chartRef4, setChartData4);
</script>

<style lang="less" scoped>
.report-container {
  padding: 20px;
  background-color: #f0f2f5;
  min-height: 100vh;
}

.report-card {
  width: 310mm;
  margin: 0 auto;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);

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
    margin: 0 0 10px 0;
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

.sub-section h3 {
  font-size: 16px;
  font-weight: bold;
  margin: 20px 0 10px;
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
}

.health-table {
  margin-top: 15px;
  width: 100%;

  :deep(.el-table__header th) {
    background-color: #f8f9fa !important;
    font-weight: bold;
  }

  :deep(.el-table__cell) {
    padding: 8px 0;
  }
}

.chart-standard {
  width: 100%;
  height: 350px;
  margin: 20px 0;
}

.export-format-radio {
  display: flex;
  justify-content: center;
  gap: 30px;
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