<template>
  <div class="report-container">
    <el-card class="report-card">
      <div class="operation-bar hide-print">
        <!-- <el-button text @click="goBack" icon="ArrowLeft" style="font-size: 20px">返回</el-button> -->
        <div class="right-ops">
          <el-button type="success" @click="openExportDialog">导出故障报告</el-button>
        </div>
      </div>

      <div id="report-content" class="report-body">
        <div class="report-header export-item">
          <h1>精细化诊断结果报告</h1>
          <div class="report-meta">
            <span>生成时间：{{ new Date().toLocaleDateString('zh-CN') }}</span>
          </div>
        </div>

        <el-divider border-style="double" class="export-item" />

        <div class="section-block export-item">
          <div class="section-title">一、装备基本信息</div>
          <el-row :gutter="20" class="info-grid">
            <el-col :span="8" v-for="(item, index) in basicInfoItems" :key="index">
              <div class="info-item">
                <span class="label">{{ item.label }}：</span>{{ item.value }}
              </div>
            </el-col>
          </el-row>
        </div>

        <div class="section-block export-item">
          <div class="section-title">二、诊断结果详情</div>
          <el-row :gutter="20" class="conclusion-grid">
            <el-col :span="12" v-for="(item, index) in diagnosisItems" :key="index">
              <div class="conclusion-item">
                <div class="label">{{ item.label }}</div>
                <div class="value" :class="{ 'highlight-red': item.highlight }">{{ item.value }}</div>
              </div>
            </el-col>
          </el-row>
          <el-table :data="subsysList" style="width: 100%; margin-top:20px;" border
            :header-cell-style="headerCellStyle">
            <el-table-column label="装备分系统" prop="subsysName" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="装备部件" align="center" :show-overflow-tooltip="true">
              <template #default="{ row }">
                {{row.partList.map((p: any) => p.partName).join('， ')}}
              </template>
            </el-table-column>
            <el-table-column label="故障原因" align="center" :show-overflow-tooltip="true">
              <template #default="{ row }">
                {{[...new Set(row.partList.map((p: any) => p.failureReason || ''))].join(' ')}}
              </template>
            </el-table-column>
          </el-table>
        </div>

        <div class="section-block export-item">
          <div class="section-title">三、该装备最近一次的故障信息</div>
          <el-row :gutter="20" class="info-grid">
            <el-col :span="8" v-for="(item, index) in recentFaultItems" :key="index">
              <div class="info-item">
                <span class="label">{{ item.label }}：</span>{{ item.value }}
              </div>
            </el-col>
            <el-col :span="24">
              <div class="info-item full-width">
                <span class="label">故障现象：</span>{{ phenomenon }}
              </div>
            </el-col>
          </el-row>
        </div>
      </div>
    </el-card>

    <!-- ================= 导出弹窗 ================= -->
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
<script setup lang="ts">
import { ref, reactive, watch, onMounted, type Ref, computed } from "vue";
import { diagnoseDetailApi } from "@/api/fault/diagnosedetail";
import { useRouter, useRoute } from 'vue-router';
import { useChart } from "@/hooks/useChart";
import { DArrowRight } from '@element-plus/icons-vue';
import { MonitorTypeApi } from '@/api/fault/monitortype';
import { MonitorChartApi } from '@/api/fault/monitorchart';
import { ElLoading, ElMessage } from "element-plus";
import * as echarts from 'echarts';
import html2canvas from 'html2canvas'; // Import html2canvas
import { jsPDF } from 'jspdf'; // Import jsPDF
import { equipDetailApi } from "@/api/fault/equipDetail";
import axios from 'axios'; // 引入 axios 用于发送请求
import { getToken } from "@/utils/auth.js";

const headerCellStyle = {
  backgroundColor: '#F8F8F8',
  color: 'black',
  fontWeight: 'bold'
};

const router = useRouter();
const route = useRoute();
const chartRef5 = ref(null);
const equipCode = ref('');
const unitSimpleName = ref('');
const unitAddress = ref('');
const equipSimpleName = ref('');
const serviceDate = ref('');
const diagnoseTime = ref('');
const fixTiming = ref('');
const failureLocation = ref('');
const fightCode = ref('');
const failureReason = ref('');
const vehicleMotor = ref('');
const manufactureDate = ref('');
const workMile = ref('');
const usingSuggestion = ref('');
const partId = ref('');
const value = ref('');
const inchargeStaffCode = ref("");
const fillStaffCode = ref("");
const handleResult = ref('');
const phenomenon = ref('');
const malfDate = ref('');
const malfType = ref('');
const rectified = ref('');
const subsysList = ref<any[]>([]);
const chartRef1 = ref<HTMLElement | null>(null);
const chartRef2 = ref<HTMLElement | null>(null);
const chartRef3 = ref<HTMLElement | null>(null);
const chartRef4 = ref<HTMLElement | null>(null);
const chartTitles = reactive({
  chart1: '未选择',
  chart2: '未选择',
  chart3: '未选择',
  chart4: '未选择'
});
const value2 = ref<string[]>([]);
const options2 = ref<{ value: string; label: string; measureUnit: string }[]>([]);
const showDatePicker = ref(false);
const startDate = ref<Date | null>(null);
const endDate = ref<Date | null>(null);
let hideTimeout: number | null = null;
const motorLimitYear = ref('');
const serviceLimit = ref('');
const selectedDiagnoseId = ref<number | null>(null);
const equipModelCode = ref('');
const unitPath = ref('');
const exceptionInfo = ref('');
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
    const filename = `精细化诊断报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

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

interface Colors {
  line: string;
  area: string;
  point: string;
}

const defaultTypes: {
  type: string;
  chartRef: Ref<HTMLElement | null>;
  update: any;
  colors: Colors;
}[] = [
    {
      type: '震动传感器',
      chartRef: chartRef1,
      update: null as any,
      colors: { line: '#5470C6', area: '#91CC75', point: '#5470C6' }
    },
    {
      type: '电压传感器',
      chartRef: chartRef2,
      update: null as any,
      colors: { line: '#EE6666', area: '#FAC858', point: '#EE6666' }
    },
    {
      type: '压力传感器',
      chartRef: chartRef3,
      update: null as any,
      colors: { line: '#73C0DE', area: '#3BA272', point: '#73C0DE' }
    },
    {
      type: '温度传感器',
      chartRef: chartRef4,
      update: null as any,
      colors: { line: '#9A60B4', area: '#EA7CCC', point: '#9A60B4' }
    }
  ];
const basicInfoItems = computed(() => [
  { label: '装备统一编码', value: equipCode.value || '暂无数据' },
  { label: '装备战斗编号', value: fightCode.value || '暂无数据' },
  { label: '装备型号', value: equipSimpleName.value || '暂无数据' },
  { label: '所属单位', value: unitSimpleName.value || '暂无数据' },
  { label: '列装时间', value: serviceDate.value || '暂无数据' },
  { label: '服役日期', value: serviceLimit.value || '暂无数据' },
  { label: '大修间隔期内累计行驶里程', value: `${workMile.value || 0} 公里` },
  { label: '当年摩托小时消耗限额', value: `${motorLimitYear.value || 0} 小时` },
  { label: '出厂日期', value: manufactureDate.value || '暂无数据' },
  { label: '中修间隔期内发动机累计消耗摩托小时', value: `${vehicleMotor.value || 0} 小时` },
]);

const diagnosisItems = computed(() => [
  { label: '当前故障概率', value: diagnoseResult.value || '暂无数据', highlight: true },
  { label: '诊断执行时间', value: diagnoseTime.value || '暂无数据' },
  { label: '异常提示信息', value: exceptionInfo.value || '无' },
  { label: '单装使用建议', value: usingSuggestion.value || '无建议' },
  { label: '修理时机建议', value: fixTiming.value || '暂无数据' },
]);

const recentFaultItems = computed(() => [
  { label: '部件编码', value: partId.value || '暂无数据' },
  { label: '故障部位', value: failureLocation.value || '暂无数据' },
  { label: '故障日期', value: malfDate.value || '暂无数据' },
  { label: '故障类型', value: malfType.value || '暂无数据' },
  { label: '是否排除故障', value: rectified.value ? '是' : '否' },
  { label: '填写人代码', value: fillStaffCode.value || '暂无数据' },
  { label: '负责人代码', value: inchargeStaffCode.value || '暂无数据' },
  { label: '故障原因', value: failureReason.value || '暂无数据' },
  { label: '处理结果', value: handleResult.value || '暂无数据' },
]);
const goBack = () => {
  router.push(`/predict/diagnosedetail?equipCode=${encodeURIComponent(equipCode.value)}`);
};

const formatDateToString = (date: Date): string => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  const seconds = String(date.getSeconds()).padStart(2, '0');
  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};

const loadMonitorTypeData = async () => {
  if (!equipCode.value) {
    ElMessage.warning('装备统一编码未选择，请选择装备后重试');
    return;
  }
  try {
    const response = await MonitorTypeApi(equipCode.value);
    if (response.code === 200) {
      options2.value = response.data?.map((item: any) => ({
        label: item.partParamChinese,
        value: item.partParamId,
        measureUnit: item.measureUnit
      }));
      value2.value = response.data?.slice(0, 4).map((item: any) => item.partParamId) || [];
    } else {
      console.error('获取末端感知数据失败:', response.message);
      // ElMessage.error(`加载末端感知数据失败: ${response.message}`);
    }
  } catch (error) {
    console.error('调用 monitorTypeApi 失败:', error);
    // ElMessage.error('加载末端感知数据失败，请稍后重试');
  }
};

const fetchAndRenderChartData = async (
  partParamId: string,
  chartRef: any,
  updateChart: (options: any, withAnimation?: boolean) => Promise<void>,
  useSelectedDates: boolean = false,
  isReplay: boolean = false
) => {
  let retries = 3;
  let lastError = null;
  while (retries > 0) {
    try {
      const params: { equipCode: string; startTimeStr?: string; endTimeStr?: string; partParamId: string } = {
        equipCode: equipCode.value || 'EQ001',
        partParamId: partParamId
      };

      // 只有在明确选择了日期范围时才添加时间参数
      let start, end;
      if (useSelectedDates && startDate.value && endDate.value) {
        start = startDate.value;
        end = endDate.value;
        params.startTimeStr = formatDateToString(start);
        params.endTimeStr = formatDateToString(end);
      } else if (isReplay && startDate.value && endDate.value) {
        // 重放模式也需要时间参数
        start = startDate.value;
        end = endDate.value;
        params.startTimeStr = formatDateToString(start);
        params.endTimeStr = formatDateToString(end);
      }
      // 如果没有选择时间范围，就不发送时间参数，让后端使用默认时间

      const response = await MonitorChartApi(params);
      if (response.code === 200) {
        // ... 图表渲染逻辑保持不变 ...
        const timeStr = response.data?.timeStr.split(',').map((time: string) => time.trim());
        const values = response.data?.valueStr.split(',').map((value: string) => parseFloat(value.trim()));
        const pointsPerDay = 120;
        const typeConfig = defaultTypes.find(t => t.chartRef === chartRef) || defaultTypes[0];
        const colors = typeConfig?.colors || { line: '#ee6666', area: '#ff9999', point: '#ee6666' };
        const paramData = options2.value.find(opt => opt.value === partParamId);
        const displayName = response.data?.paramName || paramData?.label || '未知参数';
        const measureUnit = response.data?.unit || paramData?.measureUnit || '值';

        // Extract unique days from timeStr for x-axis
        const days = [...new Set(timeStr.map((time: string) => time.split(' ')[0]))];
        const timestamps: number[] = timeStr.map((time: string) => new Date(time).getTime());
        const xAxisData = timeStr.map((time: string) => {
          const date = new Date(time);
          return date.toLocaleTimeString('zh-CN', { hour12: false, hour: '2-digit', minute: '2-digit', second: '2-digit' });
        });

        const chartOptions = reactive({
          title: { left: 'center', text: displayName, textStyle: { color: colors.line } },
          tooltip: {
            trigger: 'axis',
            position: function (pt: number[]) { return [pt[0], '10%']; },
            formatter: function (params: any) {
              const dataIndex = params[0].dataIndex;
              const timestamp = timestamps[dataIndex];
              const date = new Date(timestamp);
              const timeStr = date.toLocaleTimeString('zh-CN', { hour12: false, hour: '2-digit', minute: '2-digit', second: '2-digit' });
              const value = values[dataIndex];
              return `${date.toLocaleDateString('zh-CN')} ${timeStr}<br/>${displayName}: ${value} ${measureUnit}`;
            }
          },
          toolbox: { feature: { dataZoom: { yAxisIndex: 'none' }, restore: {}, saveAsImage: {} } },
          xAxis: {
            type: 'category',
            boundaryGap: true,
            data: xAxisData,
            axisLabel: {
              interval: Math.floor(xAxisData.length / days.length) - 1,
              formatter: function (value: string, index: number) {
                return index % Math.floor(xAxisData.length / days.length) === 0 ? timeStr[index].split(' ')[0] : '';
              }
            }
          },
          yAxis: {
            type: 'value',
            boundaryGap: [0, '100%'],
            name: `${displayName} (${measureUnit})`,
            axisLine: { lineStyle: { color: colors.line } }
          },
          dataZoom: [{ type: 'inside', start: 0, end: 100 }, { type: 'slider', start: 0, end: 100, height: 20 }],
          series: [{
            name: displayName,
            type: 'line',
            symbol: 'circle',
            symbolSize: 4,
            sampling: 'lttb',
            itemStyle: { color: colors.line },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: colors.area },
                { offset: 1, color: colors.line }
              ]),
              opacity: 0.5
            },
            data: values
          }]
        });
        await updateChart(chartOptions, isReplay);
        return;
      }
    } catch (error) {
      lastError = error;
      retries--;
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
  console.error(`加载参数 ${partParamId} 数据失败:`, lastError);
  // ElMessage.error(`加载参数 ${options2.value.find(opt => opt.value === partParamId)?.label || partParamId} 数据失败，请稍后重试`);
};

// 修改 loadInitialChartData 函数，确保不发送时间参数
const loadInitialChartData = async () => {
  console.log('Starting loadInitialChartData');
  try {
    await Promise.all(
      defaultTypes.map(({ chartRef, update }, index) => {
        if (update && value2.value[index]) {
          // 初始加载时不使用选择的时间，让后端返回默认数据
          return fetchAndRenderChartData(value2.value[index], chartRef, update, false, false);
        }
        return Promise.resolve();
      })
    );
    console.log('loadInitialChartData completed');
  } catch (error) {
    console.error('加载初始图表数据失败:', error);
  }
};

// 修改 replayData 函数，确保在重放时发送时间参数
const replayData = async () => {
  if (!startDate.value || !endDate.value) {
    ElMessage.error('请选择开始和结束日期');
    return;
  }
  if (startDate.value > endDate.value) {
    ElMessage.error('开始日期不能晚于结束日期');
    return;
  }
  if (!equipCode.value) {
    ElMessage.error('装备统一编码不能为空');
    return;
  }
  if (!value2.value || value2.value.length === 0) {
    ElMessage.error('请选择末端感知数据类型');
    return;
  }
  let loadingInstance: ReturnType<typeof ElLoading.service> | null = null;
  try {
    loadingInstance = ElLoading.service({ lock: true, text: '数据重放中...', spinner: 'el-icon-loading' });
    const selectedIds = value2.value.slice(0, 4);
    await Promise.all(
      selectedIds.map((partParamId, index) => {
        const target = defaultTypes[index];
        if (target && target.update) {
          // 重放时使用选择的时间范围
          return fetchAndRenderChartData(partParamId, target.chartRef, target.update, true, true);
        }
        return Promise.resolve();
      })
    );
    showDatePicker.value = false;
    // ElMessage.success({
    //   message: '数据重放成功',
    //   duration: 2000,
    //   showClose: true
    // });
  } catch (error) {
    console.error('数据重放失败:', error);
    // ElMessage.error({
    //   message: '数据重放失败，请稍后重试',
    //   duration: 3000
    // });
  } finally {
    if (loadingInstance) {
      loadingInstance.close();
    }
  }
};
const handleContainerEnter = () => {
  if (hideTimeout) {
    clearTimeout(hideTimeout);
    hideTimeout = null;
  }
  showDatePicker.value = true;
};

const handleContainerLeave = () => {
  hideTimeout = setTimeout(() => {
    showDatePicker.value = false;
    hideTimeout = null;
  }, 120000);
};



const viewFaultReport = (equipCode: string) => {
  router.push("/deviceinformation/detailinformation?equipCode=" + equipCode);
};

const options = ref<{ value: string; label: string; diagnoseId: number | null }[]>([]);

const storeSessionData = () => {
  if (value.value) {
    safeSessionStorage.set('diagnosedetailEquipCode', value.value);
  }
  if (selectedDiagnoseId.value) {
    safeSessionStorage.set('diagnosedetailDiagnoseId', selectedDiagnoseId.value.toString());
  }
};

const loadData = async (diagnoseId?: number) => {
  const idToUse = diagnoseId || selectedDiagnoseId.value;
  if (!idToUse) {
    console.warn("无有效的诊断ID，跳过 loadData");
    return;
  }
  try {
    const response = await diagnoseDetailApi({ diagnoseId: idToUse });
    const resinfo = await equipDetailApi(value.value);
    motorLimitYear.value = resinfo.data.equipModel?.motorLimitYear || "0";
    workMile.value = resinfo.data.totalWorkMile || "0";
    serviceLimit.value = resinfo.data.serviceLimit;
    vehicleMotor.value = response.data.equipRecord.vehicleMotor || "0";
    fightCode.value=response.data.fightCode || "暂无数据"
    if (response.code === 200) {
      const data = response.data;
      console.log(data);
      equipCode.value = data?.equipCode || "暂无数据";
      unitSimpleName.value = data?.unit?.unitSimpleName || "暂无数据";
      unitAddress.value = data?.unit?.unitAddress || "暂无数据";
      equipSimpleName.value = data?.equipModel?.equipSimpleName || "暂无数据";
      serviceDate.value = data?.serviceDate
        ? data.serviceDate
        : "暂无数据";
      diagnoseTime.value = data?.equipDiag?.diagnoseTime
        ? data.equipDiag.diagnoseTime.split("T")[0]
        : "无诊断时间";
      fixTiming.value = data?.equipDiag?.fixTiming || "暂无数据";
      manufactureDate.value = data?.manufactureDate
        ? data.manufactureDate.split("T")[0]
        : "暂无数据";
      usingSuggestion.value = data?.equipDiag?.usingSuggestion || "无建议";
      failureLocation.value = data?.equipMalf?.failureLocation || "暂无数据";
      failureReason.value = data?.equipMalf?.failureReason || "暂无数据";
      malfType.value = data?.malfRecord?.malfType || "暂无数据";
      inchargeStaffCode.value =
        data?.malfRecord?.inchargeStaffCode || "暂无数据";
      fillStaffCode.value = data?.malfRecord?.fillStaffCode || "暂无数据";
      handleResult.value = data?.malfRecord?.handleResult || "暂无数据";
      phenomenon.value = data?.malfRecord?.phenomenon || "暂无数据";
      rectified.value = data?.malfRecord?.rectified || "暂无数据";
      malfDate.value = data?.malfRecord?.malfDate || "暂无数据";
      exceptionInfo.value = data?.equipDiag?.exceptionInfo || "暂无数据";
      subsysList.value = (data?.equipModel?.subsysList || []).map(
        (subsys: any) => {
          const updatedPartList = subsys.partList.map((part: any) => {
            const diagPart = data.equipDiag?.equipDiagPartList?.find(
              (diag: any) =>
                diag.subsysId === subsys.subsysId &&
                diag.partId === part.partId,
            );
            return {
              ...part,
              failureReason: diagPart ? diagPart.failureReason : "",
            };
          });
          return {
            ...subsys,
            partList: updatedPartList,
          };
        },
      );

      partId.value = subsysList.value[0]?.partList[0]?.partId || "";
      // Update chartRef5 with the current diagnoseId
      const currentDiagnoseId =
        diagnoseId || data.equipDiag?.diagnoseId || data.diagnoseIdLatest;
      if (currentDiagnoseId) {
        const chartOptions = await setChartData5(currentDiagnoseId);
        await updateChartRef5(chartOptions);
      }
      storeSessionData();
    } else {
      console.error("后端返回的数据格式不正确：", response.data);
      // ElMessage.error(`加载数据失败: ${response.message}`);
    }
  } catch (error) {
    console.error("获取历史故障诊断数据失败：", error);
    // ElMessage.error("加载数据失败，请稍后重试");
  }
};

onMounted(async () => {
  console.log('Diagnosedetail.vue 挂载，路由参数:', route.query);

  const queryEquipCode = route.query.equipCode as string | undefined;
  const queryDiagnoseId = route.query.diagnoseId ? parseInt(route.query.diagnoseId as string) : null;
  const sessionId = route.query.sessionId as string | undefined;

  if (queryEquipCode) {
    value.value = queryEquipCode;
    equipCode.value = queryEquipCode;
    selectedDiagnoseId.value = queryDiagnoseId;
    options.value = [{
      value: queryEquipCode,
      label: queryEquipCode,
      diagnoseId: queryDiagnoseId
    }];
    try {
      const resinfo = await equipDetailApi(queryEquipCode);
      console.log('equipDetailApi 响应:', resinfo);
      if (resinfo.data.equipDiag?.diagnoseId && !queryDiagnoseId) {
        selectedDiagnoseId.value = resinfo.data.equipDiag.diagnoseId;
        options.value = [{
          value: queryEquipCode,
          label: queryEquipCode,
          diagnoseId: selectedDiagnoseId.value
        }];
      }
    } catch (error) {
      console.error('获取装备详情失败:', error);
      // ElMessage.error('加载装备详情失败，请稍后重试');
    }
  }

  if (sessionId) {
    const selectedEquipments = safeSessionStorage.get('selectedEquipments');
    if (selectedEquipments) {
      try {
        const equipments = JSON.parse(selectedEquipments);
        console.log('从 sessionStorage 获取的 selectedEquipments:', equipments);
        if (Array.isArray(equipments)) {
          const sessionOptions = equipments.map((item: any) => ({
            value: item.equipCode,
            label: item.equipCode,
            diagnoseId: item.diagnoseId || null,
          }));
          if (queryEquipCode && !sessionOptions.some((opt: any) => opt.value === queryEquipCode)) {
            sessionOptions.unshift({
              value: queryEquipCode,
              label: queryEquipCode,
              diagnoseId: queryDiagnoseId || null
            });
          }
          options.value = sessionOptions;
          if (!queryEquipCode && equipments.length > 0) {
            value.value = equipments[0].equipCode;
            equipCode.value = equipments[0].equipCode;
            selectedDiagnoseId.value = equipments[0].diagnoseId || null;
          }
        }
      } catch (e) {
        console.error('解析 sessionStorage selectedEquipments 失败:', e);
        // ElMessage.error('会话数据解析失败，请重新选择装备');
      }
    }
  }

  if (!queryEquipCode && options.value.length === 0) {
    ElMessage.warning('请先选择装备进行诊断');
  }

  console.log('最终初始化: value=', value.value, 'equipCode=', equipCode.value, 'selectedDiagnoseId=', selectedDiagnoseId.value, 'options=', options.value);
  storeSessionData();

  if (selectedDiagnoseId.value) {
    await loadData(selectedDiagnoseId.value);
  }
  if (equipCode.value) {
    await loadMonitorTypeData();
    await loadInitialChartData();
  }
});

watch(() => safeSessionStorage.get('selectedEquipments'), (newEquipments) => {
  if (newEquipments) {
    try {
      const equipments = JSON.parse(newEquipments);
      if (Array.isArray(equipments)) {
        options.value = equipments.map((item: any) => ({
          value: item.equipCode,
          label: item.equipCode,
          diagnoseId: item.diagnoseId || null,
        }));
        if (!options.value.some(option => option.value === value.value) && equipments.length > 0) {
          value.value = equipments[0].equipCode;
          equipCode.value = equipments[0].equipCode;
          selectedDiagnoseId.value = equipments[0].diagnoseId || null;
          console.log('sessionStorage 更新后设置默认: equipCode=', value.value, 'diagnoseId=', selectedDiagnoseId.value);
        }
        storeSessionData();
        if (selectedDiagnoseId.value) {
          console.log('sessionStorage 更新后调用 loadData，diagnoseId=', selectedDiagnoseId.value);
          loadData(selectedDiagnoseId.value);
        }
        if (equipCode.value) {
          loadMonitorTypeData();
          loadInitialChartData();
        }
      }
    } catch (e) {
      console.error('解析 selectedEquipments 失败:', e);
      // ElMessage.error('会话数据解析失败，请重新选择装备');
    }
  }
}, { immediate: true });

watch(value2, async (newValue) => {
  if (!newValue || newValue.length === 0) {
    chartTitles.chart1 = '未选择';
    chartTitles.chart2 = '未选择';
    chartTitles.chart3 = '未选择';
    chartTitles.chart4 = '未选择';
    for (const { update } of defaultTypes) {
      if (update) {
        await update({
          title: { left: 'center', text: '' },
          xAxis: { type: 'category', boundaryGap: false, data: [] },
          yAxis: { type: 'value', name: '值' },
          series: [{ data: [], type: 'line', areaStyle: {} }],
          grid: { left: '3%', right: '4%', bottom: '10%', containLabel: true }
        });
      }
    }
    return;
  }
  const selectedIds = newValue.slice(0, 4);
  chartTitles.chart1 = options2.value.find(opt => opt.value === selectedIds[0])?.label || '未选择';
  chartTitles.chart2 = options2.value.find(opt => opt.value === selectedIds[1])?.label || '未选择';
  chartTitles.chart3 = options2.value.find(opt => opt.value === selectedIds[2])?.label || '未选择';
  chartTitles.chart4 = options2.value.find(opt => opt.value === selectedIds[3])?.label || '未选择';
  for (let i = 0; i < defaultTypes.length; i++) {
    const { update, chartRef } = defaultTypes[i];
    const partParamId = selectedIds[i] || null;
    if (partParamId && update) {
      await fetchAndRenderChartData(partParamId, chartRef, update, false);
      // ElMessage.success(`${options2.value.find(opt => opt.value === partParamId)?.label || partParamId} 数据加载成功`);
    } else if (update) {
      await update({
        title: { left: 'center', text: '' },
        xAxis: { type: 'category', boundaryGap: false, data: [] },
        yAxis: { type: 'value', name: '值' },
        series: [{ data: [], type: 'line', areaStyle: {} }],
        grid: { left: '3%', right: '4%', bottom: '10%', containLabel: true }
      });
    }
  }
});
const diagnoseResult = ref('');
const setChartData5 = async (diagnoseId: number) => {
  if (!selectedDiagnoseId.value) {
    console.warn('No diagnoseId selected for chart5');
    return { series: [{ type: 'pie', radius: ['50%', '70%'], data: [] }] };
  }
  try {
    const response = await diagnoseDetailApi({ diagnoseId: selectedDiagnoseId.value });
    if (response.code === 200) {
      diagnoseResult.value = response.data.equipDiag?.diagnoseResult;
      const chartOptions = reactive({
        series: [{
          type: 'pie',
          radius: ['50%', '70%'],
          avoidLabelOverlap: false,
          label: { show: false },
          emphasis: { label: { show: false } },
          itemStyle: { color: (colors: { dataIndex: number }) => ['#007BFF', '#E0E0E0'][colors.dataIndex] },
          data: [
            { value: response.data.equipDiag?.diagnoseResult, name: '故障概率' },
            { value: 1 - response.data.equipDiag?.diagnoseResult, name: '正常概率' }
          ]
        }],
        graphic: [{
          type: 'group',
          left: 'center',
          top: '53%',
          bounding: 'raw',
          children: [
            // {
            //     type: 'text',
            //     style: {
            //         text: '预测故障概率',
            //         textAlign: 'center',
            //         textVerticalAlign: 'top',
            //         fontSize: 13,
            //         fill: '#333',
            //     },
            // },
            {
              type: 'text',
              style: {
                // text: `${(response.data.equipDiag?.diagnoseResult * 100).toFixed(2)}%`,
                fontSize: 24,
                textAlign: 'center',
                textVerticalAlign: 'bottom',
                fill: '#FF5E65',
              },
            },
            { type: 'text', style: { text: (response.data.equipDiag?.diagnoseResult) || '无诊断结果', fontSize: 24, textAlign: 'center', textVerticalAlign: 'bottom', fill: '#FF5E65' } }
          ]
        }],
        labelLine: { show: false }
      });
      console.log('chartRef5 options:', chartOptions);
      return chartOptions;
    } else {
      console.error('加载 chart5 数据失败:', response.message);
      return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
    }
  } catch (error) {
    console.error('获取 chart5 数据失败:', error);
    return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
  }
};

const { update: updateChartRef5 } = useChart(chartRef5, () => setChartData5(selectedDiagnoseId.value || 0));

const getInitialOptions = (type: string, colors: Colors) => async () => ({
  title: {
    left: 'center',
    text: type,
    textStyle: { color: colors.line }
  },
  tooltip: {
    trigger: 'axis',
    position: function (pt: number[]) {
      return [pt[0], '10%'];
    }
  },
  toolbox: {
    feature: {
      dataZoom: { yAxisIndex: 'none' },
      restore: {},
      saveAsImage: {}
    }
  },
  xAxis: {
    type: 'category',
    boundaryGap: true,
    data: [],
    axisLabel: {
      interval: 119,
      formatter: function (value: string) {
        return value;
      }
    }
  },
  yAxis: {
    type: 'value',
    name: '值',
    boundaryGap: [0, '100%'],
    axisLine: { lineStyle: { color: colors.line } }
  },
  dataZoom: [
    { type: 'inside', start: 0, end: 100 },
    { type: 'slider', start: 0, end: 100, height: 20 }
  ],
  series: [{
    name: type,
    type: 'line',
    symbol: 'circle',
    symbolSize: 4,
    sampling: 'lttb',
    itemStyle: { color: colors.point },
    areaStyle: {
      color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
        { offset: 0, color: colors.area },
        { offset: 1, color: colors.line }
      ]),
      opacity: 0.5
    },
    data: []
  }],
  grid: {
    left: '5%',
    right: '5%',
    bottom: '15%',
    containLabel: true
  }
});

defaultTypes[0].update = useChart(chartRef1, getInitialOptions('震动传感器', defaultTypes[0].colors)).update;
defaultTypes[1].update = useChart(chartRef2, getInitialOptions('电压传感器', defaultTypes[1].colors)).update;
defaultTypes[2].update = useChart(chartRef3, getInitialOptions('压力传感器', defaultTypes[2].colors)).update;
defaultTypes[3].update = useChart(chartRef4, getInitialOptions('温度传感器', defaultTypes[3].colors)).update;
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
  /* 只保留右侧导出按钮 */
  background: #fff;
  position: sticky;
  top: 0;
  z-index: 100;
}

.right-ops {
  display: flex;
  align-items: center;
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

  .report-meta {
    color: #999;
    font-size: 14px;
  }
}

.section-block {
  margin-bottom: 40px;
  page-break-inside: avoid;
  /* 避免块内分页断开 */
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

.info-grid {
  margin-bottom: 20px;
}

.info-item {
  margin-bottom: 12px;
  font-size: 13px;
  line-height: 1.6;

  .label {
    font-weight: bold;
    color: #555;
  }

  &.full-width {
    margin-top: 15px;
  }
}

.conclusion-grid {
  margin-bottom: 20px;
}

.conclusion-item {
  border: 1px solid #eee;
  padding: 12px;
  margin-bottom: 12px;
  border-radius: 4px;
  background: #fafafa;

  .label {
    font-size: 12px;
    color: #999;
    margin-bottom: 4px;
  }

  .value {
    font-size: 15px;
    font-weight: bold;
    color: #333;
  }

  .highlight-red {
    color: #f56c6c;
    font-size: 18px;
  }
}

/* 表格样式优化（可选，提升可读性） */
:deep(.el-table) {
  margin-top: 20px;
}

:deep(.el-table th) {
  background-color: #f8f9fa !important;
  font-weight: bold;
}

/* 导出优化 */
.export-item {
  page-break-inside: avoid;
}

@media print {
  .hide-print {
    display: none !important;
  }

  .report-container {
    padding: 0;
    background: #fff;
  }

  .report-card {
    width: 100%;
    border: none;
    box-shadow: none;
  }

  .report-body {
    padding: 10mm;
  }
}
</style>