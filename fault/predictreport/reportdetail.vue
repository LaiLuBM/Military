<template>
<div>
  <el-card>
    <!-- 基本信息 -->
    <el-card>
      <div class="top">
        <el-row>
          <el-col :span="1">
            <el-button text @click="goBack" icon="ArrowLeft" style="font-size: 30px; padding: 0;" />
          </el-col>
          <el-col :span="19" style="padding-left: 10px; display: flex; align-items: center;">
            <h2 class="title-margin" style="margin: 0;">故障结果报告</h2>
          </el-col>
          <el-col :span="4" class="center-content">
            <template v-if="reportStatus === '待审核'">
              <el-button type="success" size="large" @click="reportId && submit(reportId)">提交审核</el-button>
            </template>
            <el-button type="success" size="large" @click="openExportDialog">生成版式报告</el-button>
          </el-col>
        </el-row>
        <el-row style="margin-top: 10px;">
          <el-col :span="20">
            <el-row style="margin-top: 20px;">
              <el-col :span="8">
                <div>所属单位：{{ equipSimpleName || '未知' }}</div>
              </el-col>
              <el-col :span="8">
                <div>装备统一编码：{{ equipCode }}</div>
              </el-col>
              <el-col :span="8">
                <div>报告生成时间：{{ reportGenerationTime || '' }}</div>
              </el-col>
            </el-row>
            <el-row style="margin-top: 20px;">
              <el-col :span="8">
                <div>装备型号：{{ equipModelName || '未知' }}</div>
              </el-col>
              <el-col :span="8">
                <div>服役日期：{{ serviceLimit || '未知' }}</div>
              </el-col>
              <el-col :span="8">
               <div>报告状态：{{ reportStatus }}</div>
              </el-col>
            </el-row>
            <el-row style="margin-top: 20px;">
              <el-col :span="8">
                <div>大修间隔期内累计行驶里程：{{ workMile || '未知' }}公里</div>
              </el-col>
              <el-col :span="8">
               <div>中修间隔期内发动机累计消耗摩托小时：{{ vehicleMotor || '未知' }}小时</div>
              </el-col>
              <el-col :span="8">
               <div>列装时间：{{ serviceDate || '未知' }}</div>
              </el-col>
            </el-row>
            <el-row style="margin-top: 20px;">
              <el-col :span="8">
               <div>当年摩托小时消耗限额：{{ motorLimitYear || '未知' }}小时</div>
              </el-col>
              <el-col :span="8">
               <div>出厂日期：{{ manufactureDate || '未知' }}</div>
              </el-col>     
            </el-row>      
          </el-col>
          <el-col :span="4" class="image-col">
            <img src="../../../assets/tanks/tank.png" class="tank-image" />
          </el-col>
        </el-row>
      </div>
    </el-card>
    <div>
      <!-- 通用化故障预测 -->
      <el-card v-show="isGeneralPrediction">
        <div class="top">
          <el-row>
            <el-col :span="20">
              <h2 style="text-align: center;">通用化故障预测</h2>
            </el-col>
          </el-row>
          <el-row style="margin-top: 20px;">
            <el-col :span="6">
              <div ref="chartRef20" style="width: 65%;height: 200px;"></div>
            </el-col>
            <el-col :span="18">
              <el-row>
                <el-col :span="10">
                  <div style="margin-top: 40px;">故障最可能出现的时机：{{ failureTiming || '未知' }}</div>
                </el-col>
                <el-col :span="10">
                  <div style="margin-top: 40px; text-align: left;">
                    故障预测执行时间：{{ predictTime || '未知' }}
                  </div>
                </el-col>
                <el-col :span="10">
                  <div style="margin-top: 40px;">故障类型：{{ failureType || '未知' }}</div>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
          <el-divider></el-divider>
          <el-row style="margin-top: 30px;">
            <el-col :span="10"></el-col>
            <el-col :span="14">
              <h2>未来12个月故障概率</h2>
            </el-col>
            <el-col :span="24">
              <div ref="chartRef10" style="width: 100%;height: 500px;"></div>
            </el-col>
          </el-row>
        </div>
      </el-card>
    </div>

    <!-- 精细化故障预测 -->
    <el-card v-show="isFiningDetail">
      <div class="top">
        <el-row>
          <el-col :span="20">
            <h2 style="text-align: center;">精细化故障预测</h2>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="24">
            <el-row>
              <el-col :span="6">
                <div ref="chartRef5" style="width: 65%; height: 200px;"></div>
              </el-col>
              <el-col :span="18">
                <el-row>
                  <div style="margin-top: 10px;">
                    故障发生系统：{{ faultySystems.join('，') || '无' }}
                  </div>
                </el-row>
                <el-row style="margin-top: 10px;">
                  <div style="margin-top: 30px;">故障最可能出现的月份：{{ failureTiming || '未知' }}</div>
                </el-row>
                <el-row>
                  <div style="margin-top: 40px; text-align: left;">
                    故障类型：{{ failureType || '未知' }}
                  </div>
                </el-row>
                <el-row>
                  <el-col :span="12">
                    <div style="margin-top: 40px; text-align: left;">
                      故障预测执行时间：{{ equips.equipMalfDetail?.predictTime || '无' }}
                    </div>
                  </el-col>
                  <el-col :span="12">
                    <div style="margin-top: 40px; text-align: left;">
                      数据采样时间：{{formatTime(equips.equipMalfDetail?.predictTime,5)  || '无' }}
                    </div>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <el-divider></el-divider>
            <div class="mid">
              <el-row>
                <el-col :span="6">
                  <el-radio-group v-model="selectedOption" @change="handleOptionChange" class="group">
                    <el-radio-button label="整机">整机</el-radio-button>
                    <el-radio-button label="分系统">分系统</el-radio-button>
                    <el-radio-button label="部件">部件</el-radio-button>
                  </el-radio-group>
                  <el-tree ref="treeRef" :data="filteredTreeData" :props="defaultProps" node-key="partId"
                    :default-expand-all="false" class="sub-tree" @node-click="handleNodeClick">
                    <template #default="{ data }">
                      <span>
                        {{ data.label || '' }}
                        <span v-if="data.faultProbability" style="color: #FF5E65; margin-left: 10px;">
                          (故障概率: {{ data.faultProbability || '' }})
                        </span>
                        <span v-if="data.faultMonth" style="color: #007BFF; margin-left: 10px;">
                          (月份: {{ data.faultMonth || '' }})
                        </span>
                      </span>
                    </template>
                  </el-tree>
                </el-col>
                <el-col :span="15">
                  <h2>{{ chartTitle }}</h2>
                  <div ref="chartRef6" class="chart"></div>
                </el-col>
              </el-row>
            </div>
          </el-col>
        </el-row>
        <el-row style="margin-top: 10px;">
          <el-col :span="10">
            <el-form class="moduan">
              <el-form-item label="末端感知数据">
                <el-select v-model="value2" placeholder="请选择" style="width: 240px" clearable multiple
                  :multiple-limit="4">
                  <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
                </el-select>
              </el-form-item>
            </el-form>
          </el-col>
          <el-col :span="10"></el-col>
          <el-col :span="4">
            <div class="replay-container" @mouseleave="handleContainerLeave" @mouseenter="handleContainerEnter">
              <el-button type="primary" size="large">
                重放数据
              </el-button>
              <el-card class="date-picker-card" v-show="showDatePicker">
                <el-form>
                  <el-form-item label="开始日期">
                    <el-date-picker v-model="startDate" type="date" placeholder="选择开始日期" :style="{ width: '100%' }" />
                  </el-form-item>
                  <el-form-item label="结束日期">
                    <el-date-picker v-model="endDate" type="date" placeholder="选择结束日期" :style="{ width: '100%' }" />
                  </el-form-item>
                  <el-form-item>
                    <el-button type="primary" size="small" @click="replayData">确定</el-button>
                    <el-button size="small" @click="showDatePicker = false">取消</el-button>
                  </el-form-item>
                </el-form>
              </el-card>
            </div>
          </el-col>
        </el-row>

        <el-row>
          <el-col :span="24">
            <el-card class="info-card">
              <div class="section-title"></div>
              <el-row :gutter="20">
                <el-col :span="12">
                  <div class="chart-container">
                    <div class="chart-title">{{ chartTitles.chart1 }}</div>
                    <div ref="chartRef1" class="chart-placeholder"></div>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="chart-container">
                    <div class="chart-title">{{ chartTitles.chart2 }}</div>
                    <div ref="chartRef2" class="chart-placeholder"></div>
                  </div>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <el-col :span="12">
                  <div class="chart-container">
                    <div class="chart-title">{{ chartTitles.chart3 }}</div>
                    <div ref="chartRef3" class="chart-placeholder"></div>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="chart-container">
                    <div class="chart-title">{{ chartTitles.chart4 }}</div>
                    <div ref="chartRef4" class="chart-placeholder"></div>
                  </div>
                </el-col>
              </el-row>
            </el-card>
          </el-col>
        </el-row>
      </div>
    </el-card>

    <!-- 精细化诊断 -->
    <el-card v-show="isDiagnoseDetail">
      <div class="topinfo">
        <el-row>
          <el-col :span="20">
            <h2 style="text-align: center;">精细化诊断</h2>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="6">
            <div ref="chartRef15" style="width: 65%;height: 200px;"></div>
          </el-col>
          <el-col :span="18">
            <el-table :data="subsysList" style="width: 100%;" border :header-cell-style="headerCellStyle">
              <el-table-column label="故障分系统" prop="subsysName" :show-overflow-tooltip="true"></el-table-column>
              <el-table-column label="故障部件" :show-overflow-tooltip="true">
                <template #default="{ row }">
                  {{row.partList.map((p: any) => p.partName).join(', ')}}
                </template>
              </el-table-column>
              <el-table-column label="故障原因" :show-overflow-tooltip="true">
                <template #default="{ row }">
                  {{[...new Set(row.partList.map((p: any) => p.failureReason || ''))].join(' ')}}
                </template>
              </el-table-column>
            </el-table>
          </el-col>
        </el-row>
        <el-row style="margin-top: 15px;">
          <el-col :span="8">
            <div style="margin-top: 10px;">单装使用建议：{{ usingSuggestion }}</div>
          </el-col>
          <el-col :span="4">
            <div style="margin-top: 10px;">修理时机建议：{{ fixTiming }}</div>
          </el-col>
          <el-col :span="6">
            <div style="margin-top: 10px;">异常提示信息：{{ exceptionInfo || '无' }}</div>
          </el-col>
          <el-col :span="6">
            <div style="margin-top: 10px;">故障诊断执行时间：{{ diagnoseTime }}
            </div>
          </el-col>
        </el-row>
      </div>
    </el-card>

    <!-- 修理时机预测 -->
    <el-card v-show="isRepairDetail">
      <div class="topinfo">
        <el-row>
          <el-col :span="20">
            <h2 style="text-align: center;">修理时机故障预测</h2>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="21">
            <h3 class="section-title">装备最近一次故障信息</h3>
          </el-col>
        </el-row>
        <el-row class="info-row" v-if="malfRecords.malfRecordId">
          <el-col :span="6">
            <div class="info-item"><strong>单装故障记录ID:</strong> {{ malfRecords.malfRecordId }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>单装履历记录ID:</strong> {{ malfRecords.equipRecordId }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>故障日期:</strong> {{ malfRecords.malfDate }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>故障类型:</strong> {{ malfRecords.malfCategoryDictId }}</div>
          </el-col>
        </el-row>
        <el-row class="info-row" v-if="malfRecords.malfRecordId">
          <el-col :span="6">
            <div class="info-item"><strong>部件编码:</strong> {{ malfRecords.partId }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>故障部位:</strong> {{ malfRecords.malfLocation }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>故障现象:</strong> {{ malfRecords.phenomenon }}</div>
          </el-col>
        </el-row>
        <el-row class="info-row" v-if="malfRecords.malfRecordId">
          <el-col :span="6">
            <div class="info-item"><strong>故障原因:</strong> {{ malfRecords.reason }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>处理结果:</strong> {{ malfRecords.handleResult }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item">
              <strong>
                <span class="info-label">是否排除故障</span>
              </strong>
              <el-radio-group class="radio-group" v-model="isFixed" :disabled="true">
                <el-radio label="是"></el-radio>
                <el-radio label="否"></el-radio>
              </el-radio-group>
            </div>
          </el-col>
        </el-row>
        <el-row class="info-row" v-if="malfRecords.malfRecordId">
          <el-col :span="6">
            <div class="info-item"><strong>填写人代码:</strong> {{ malfRecords.fillStaffCode }}</div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>负责人代码:</strong> {{ malfRecords.inchargeStaffCode }}</div>
          </el-col>
        </el-row>
        <el-row class="info-row" v-if="malfRecords.malfRecordId">
          <el-col :span="6">
            <div class="info-item"><strong>备注:</strong> 无</div>
          </el-col>
        </el-row>
        <el-row v-else>
          <el-col :span="24">
            <div class="info-item">暂无故障记录</div>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="21">
            <h3 class="section-title">修理时机预测结果</h3>
          </el-col>
        </el-row>
        <el-row class="info-row">
          <el-col :span="6">
            <div class="info-item"><strong>修理类别:</strong> {{ repairequips.equipRepair.repairType || '未知'
            }}
            </div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>送修时间:</strong> {{ repairequips.equipRepair.repairTime || '未知'
            }}
            </div>
          </el-col>
          <el-col :span="6">
            <div class="info-item"><strong>预测执行时间:</strong> {{ repairequips.equipRepair.predictTime ||
              '未知' }}
            </div>
          </el-col>
        </el-row>
      </div>
    </el-card>

  </el-card>
  <!-- 导出格式选择对话框 -->
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
import { ref, reactive, onMounted, nextTick, computed, watch, type Ref } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { ElLoading, ElMessage, type ElTree } from 'element-plus';
import * as echarts from 'echarts';
import { submitreview } from '@/api/fault/report';
import { useChart } from '@/hooks/useChart';
import { chartDataApi3 } from '@/api/fault/detail';
import { equipDetailApi } from '@/api/fault/equipDetail';
import { diagnoseDetailApi } from '@/api/fault/diagnosedetail';
import { MonitorTypeApi } from '@/api/fault/monitortype';
import { MonitorChartApi } from '@/api/fault/monitorchart';
import { RepairdetailApi } from '@/api/fault/repairdetail';
import html2canvas from 'html2canvas';
import { jsPDF } from 'jspdf';
import axios from 'axios'; // 引入 axios 用于发送请求
import { getToken } from "@/utils/auth.js";
import { reportdetail } from '@/api/fault/report';


const route = useRoute();

const equipCode = ref('');
const predictId = ref<number | null>(null);
const diagnoseId = ref<number | null>(null);
const repairId = ref<number | null>(null);
const predictIdDetail = ref<number | null>(null);
const faultPredictionType = ref('');
const reportGenerationTime = ref('');
const isGeneralPrediction = ref(false);
const isFiningDetail = ref(false);
const isDiagnoseDetail = ref(false);
const isRepairDetail = ref(false);
const reportStatus = ref('');
const reportId = ref<number | null>(null);
// 添加 determineActiveSections 函数
const determineActiveSections = () => {
  const predictionTypes = faultPredictionType.value.split(',');

  isGeneralPrediction.value = !!equips.value.hasGeneralPrediction ||
    predictionTypes.includes('通用化');
  isFiningDetail.value = !!equips.value.hasDetailedPrediction ||
    predictionTypes.includes('精细化');
  isDiagnoseDetail.value = !!diagnoseId.value;
  isRepairDetail.value = !!repairId.value;
};


const safeSessionStorage = {
  set(key: string, value: string): void {
    try {
      sessionStorage.setItem(key, value);
    } catch (e) {
      console.error('SessionStorage存储失败:', e);
    }
  },
  get(key: string): string | null {
    try {
      return sessionStorage.getItem(key);
    } catch (e) {
      console.error('SessionStorage读取失败:', e);
      return null;
    }
  },
};

const storeSessionData = () => {
  if (equipCode.value) {
    safeSessionStorage.set('reportDetailEquipCode', equipCode.value);
    if (predictId.value) safeSessionStorage.set('reportDetailPredictId', predictId.value.toString());
    if (predictIdDetail.value) safeSessionStorage.set('reportpredictIdDetail', predictIdDetail.value.toString());
    if (diagnoseId.value) safeSessionStorage.set('reportDetailDiagnoseId', diagnoseId.value.toString());
    if (repairId.value) safeSessionStorage.set('reportDetailRepairId', repairId.value.toString());
    if (reportGenerationTime.value) safeSessionStorage.set('reportGenerationTime', reportGenerationTime.value.toString());
    if (faultPredictionType.value) safeSessionStorage.set('reportDetailFaultPredictionType', faultPredictionType.value);
    if (reportId.value) safeSessionStorage.set('reportId', reportId.value.toString());
  }
};
onMounted(async () => {
  // 从路由查询参数获取必要的ID
  equipCode.value = route.query.equipCode as string || '';
  predictId.value = route.query.predictId ? parseInt(route.query.predictId as string) : null;
  predictIdDetail.value = route.query.predictIdDetail && route.query.predictIdDetail !== 'null'
    ? parseInt(route.query.predictIdDetail as string)
    : null;
  diagnoseId.value = route.query.diagnoseId && route.query.diagnoseId !== 'null'
    ? parseInt(route.query.diagnoseId as string)
    : null;
  repairId.value = route.query.repairId && route.query.repairId !== 'null'
    ? parseInt(route.query.repairId as string)
    : null;
  faultPredictionType.value = route.query.faultPredictionType as string || '';
  reportGenerationTime.value = route.query.reportGenerationTime as string || '';
  reportId.value = route.query.reportId && route.query.reportId !== 'null'
    ? parseInt(route.query.reportId as string)
    : null;
  const res = await reportdetail({ reportId: reportId.value });
  reportStatus.value = res.data.reportStatus;
  reportGenerationTime.value=res.data.reportGenerationTime;
  console.log('reportstatus:', reportStatus.value);
  console.log('路由参数详情:', {
    equipCode: equipCode.value,
    predictId: predictId.value,
    predictIdDetail: predictIdDetail.value,
    faultPredictionType: faultPredictionType.value
  });

  // 等待DOM更新
  await nextTick();

  // 确定激活的模块
  determineActiveSections();

  console.log('激活状态:', {
    isGeneralPrediction: isGeneralPrediction.value,
    isFiningDetail: isFiningDetail.value,
    isDiagnoseDetail: isDiagnoseDetail.value,
    isRepairDetail: isRepairDetail.value
  });

  // 并行加载所有数据
  await loadAllData();
});
// 新增统一的加载函数
const loadAllData = async () => {
  const loadingPromises = [];

  // 基础数据加载
  if (equipCode.value) {
    loadingPromises.push(loadData());
  }

  // 根据模块状态加载相应数据
  if (isGeneralPrediction.value && predictId.value) {
    loadingPromises.push(loadgeneralData());
  }

  if (isFiningDetail.value && predictIdDetail.value) {
    loadingPromises.push(loadfinningData());
    loadingPromises.push(loadMonitorTypeData());
  }

  if (isDiagnoseDetail.value && diagnoseId.value) {
    loadingPromises.push(loaddiagnoseData());
  }

  if (isRepairDetail.value && repairId.value) {
    loadingPromises.push(loadrepairData());
  }

  // 并行执行所有加载任务
  await Promise.allSettled(loadingPromises);

  // 再次等待DOM更新，确保所有图表容器都已渲染
  await nextTick();

  // 加载图表数据
  await loadAllCharts();
};

// 新增统一的图表加载函数
const loadAllCharts = async () => {
  const chartPromises = [];

  if (isGeneralPrediction.value) {
    chartPromises.push(updateChart10(await setChartData10()));
    chartPromises.push(updateChart20(await setChartData20()));
  }

  if (isFiningDetail.value) {
    chartPromises.push(updateChartRef5(await setChartData5()));
    chartPromises.push(loadInitialChartData());

    // 等待树数据加载完成后初始化图表
    if (treeData.value.length > 0) {
      await nextTick();
      chartPromises.push(handleNodeClick(treeData.value[0]));
    }
  }

  if (isDiagnoseDetail.value) {
    chartPromises.push(updateChartRef15(await setChartData15()));
  }

  await Promise.allSettled(chartPromises);
};

const goBack = () => {
  window.history.back();
};


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
    const filename = `故障报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

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

const formatTime = (timeStr,add=0)=>{
   if(!timeStr|| typeof timeStr!=="string"||timeStr.trim()===''){
       return "无"
   }

   let isoStr = timeStr.trim().replace(/\s+/g,'T')
   const date=new Date(isoStr)

   if(add!==0){
       date.setSeconds(date.getSeconds()+add)
   }

   const yyyy=date.getFullYear();
   const mm=String(date.getMonth()+1).padStart(2,'0')
   const dd=String(date.getDate()).padStart(2,'0')
   const hh=String(date.getHours()).padStart(2,'0')
   const mi=String(date.getMinutes()).padStart(2,'0')
   const ss=String(date.getSeconds()).padStart(2,'0')

   return `${yyyy}-${mm}-${dd} ${hh}-${mi}-${ss}`
}
const submit = async (reportId: number) => {
  try {
    const res = await submitreview(reportId);
    if (res.code === 200) {
      ElMessage.success('提交成功');
      reportStatus.value = res.data.reportStatus;
    }
  } catch (error) {
    console.error(error);
  }
};
watch(isGeneralPrediction, (newVal) => {
  if (newVal) {
    nextTick(() => {
      if (chartRef10.value && chartInstance10.value) {
        chartInstance10.value.resize();
      }
      if (chartRef20.value && chartInstance20.value) {
        chartInstance20.value.resize();
      }
    });
  }
});

watch(isFiningDetail, (newVal) => {
  if (newVal) {
    nextTick(() => {
      if (chartRef5.value && chartInstanceForRef5.value) {
        chartInstanceForRef5.value.resize();
      }
      if (chartRef6.value && chartInstanceForRef6.value) {
        chartInstanceForRef6.value.resize();
      }
      if (chartRef1.value && instance1.value) {
        chartInstanceForRef5.value.resize();
      }
      if (chartRef2.value && instance2.value) {
        chartInstanceForRef6.value.resize();
      }
      if (chartRef3.value && instance3.value) {
        chartInstanceForRef5.value.resize();
      }
      if (chartRef4.value && instance4.value) {
        chartInstanceForRef6.value.resize();
      }
    });
  }
});

watch(isDiagnoseDetail, (newVal) => {
  if (newVal) {
    nextTick(() => {
      if (chartRef15.value && chartInstanceForRef15.value) {
        chartInstanceForRef15.value.resize();
      }
    });
  }
});
const headerCellStyle = {
  backgroundColor: '#F8F8F8',
  color: 'black',
  fontWeight: 'bold'
};

// 通用化
const chartRef10 = ref<HTMLElement | null>(null);
const chartRef20 = ref<HTMLElement | null>(null);
const equipModelName = ref(''); // 装备型号
const equipSimpleName = ref(''); // 所属单位
const serviceDate = ref('');
const predictTime = ref('');
const failureType = ref('');
const vehicleMotor = ref('');
const manufactureDate = ref('');
const workMile = ref('');
const motorLimitYear = ref('');
const serviceLimit = ref('');
const maxIndex = ref<number>();
const failureTiming = ref('');
const monthProbability = ref('');
const predictMonth = ref<number>(6); // 默认6月
const predictYear = ref<number>(2025); // 默认2025年

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
const { update: updateChart10, instance: chartInstance10 } = useChart(chartRef10, async () => await setChartData10());
const { update: updateChart20, instance: chartInstance20 } = useChart(chartRef20, async () => await setChartData20());
const setChartData10 = async () => {
  if (!predictId.value) {
    return;
  }
  try {
    // 检查predictId.value是否有效
    const res = await chartDataApi3(false, predictId.value);
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

const setChartData20 = async () => {
  if (!predictId.value) {
    return;
  }
  try {
    const res = await chartDataApi3(false, predictId.value);
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
                text: `${(res.data?.equipMalf?.failureProbability * 100)}%`,
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
  if (!equipCode) {
    return;
  }
  try {
    const resinfo = await equipDetailApi(equipCode.value);
    // 修改这里：装备型号使用 equipModelName
    equipModelName.value = resinfo.data?.equipModel?.equipSimpleName || '暂无数据';
    // 所属单位保持不变
    equipSimpleName.value = resinfo.data?.unit?.unitSimpleName || '暂无数据';
    serviceDate.value = resinfo.data?.serviceDate || '未知';
    predictTime.value = resinfo.data?.equipMalf?.predictTime || '暂无数据';
    failureType.value = resinfo.data?.equipMalf?.failureType || '暂无数据';
    vehicleMotor.value = resinfo.data?.equipRecord?.vehicleMotor || '未知';
    manufactureDate.value = resinfo.data?.manufactureDate || '暂无数据';
    failureTiming.value = resinfo.data?.equipMalf?.failureTiming || '暂无数据';
    monthProbability.value = resinfo.data?.equipMalf?.monthProbability || '暂无数据';
    workMile.value = resinfo.data?.totalWorkMile || '未知';
    motorLimitYear.value = resinfo.data?.equipModel?.motorLimitYear || '未知';
    serviceLimit.value = resinfo.data?.serviceLimit || '暂无数据';
    determineActiveSections();
    storeSessionData();
  } catch (error) {
    console.error('请求失败:', error);
    ElMessage.error('加载数据失败，请稍后重试');
  }
};


const loadgeneralData = async () => {
  if (!predictId.value) {
    return;
  }
  try {
    const res = await chartDataApi3(false, predictId.value);

    const probabilities = failureTiming.value.split(',').map(Number);
    maxIndex.value = probabilities.indexOf(Math.max(...probabilities)) + 1;
    if (predictTime.value) {
      const predictDate = new Date(predictTime.value);
      predictMonth.value = predictDate.getMonth() + 2;
      predictYear.value = predictDate.getFullYear();
      if (predictMonth.value > 12) {
        predictMonth.value -= 12;
        predictYear.value += 1;
      }
    }
    determineActiveSections();
    await updateChart10(await setChartData10());
    await updateChart20(await setChartData20());
    storeSessionData();
  } catch (error) {
    console.error('请求失败:', error);
  }
};



// 精细化
const selectedOption = ref('整机');
const chartTitle = ref('整机未来12个月故障概率');
const equipData = ref<any>(null);
const equipCodeOptions = ref<{ value: string; label: string; predictId: number | null }[]>([]);
const chartRef1 = ref<HTMLElement | null>(null);
const chartRef2 = ref<HTMLElement | null>(null);
const chartRef3 = ref<HTMLElement | null>(null);
const chartRef4 = ref<HTMLElement | null>(null);
const chartTitles = reactive({
  chart1: '未选择',
  chart2: '未选择',
  chart3: '未选择',
  chart4: '未选择',
});
const chartRef5 = ref<HTMLElement | null>(null);
const chartRef6 = ref<HTMLElement | null>(null);
const value2 = ref<string[]>([]);
const options = ref<{ value: string; label: string; measureUnit: string }[]>([]);
const showDatePicker = ref(false);
const startDate = ref<Date | null>(null);
const endDate = ref<Date | null>(null);
let hideTimeout: number | null = null;
const treeData = ref<Tree[]>([]);
const defaultProps = { children: 'children', label: 'label' };
const selectedEquipCode = ref('');

interface Colors {
  line: string;
  area: string;
  point: string;
}

const defaultTypes: {
  type: string;
  chartRef: Ref<HTMLElement | null>;
  update: ((options: any, withAnimation?: boolean) => Promise<void>) | null;
  instance: Ref<echarts.ECharts | null> | null;  // 新增
  colors: Colors;
}[] = [
    {
      type: '震动传感器',
      chartRef: chartRef1,
      update: null,
      instance: null,  // 初始化为 null
      colors: { line: '#5470C6', area: '#91CC75', point: '#5470C6' }
    },
    {
      type: '电压传感器',
      chartRef: chartRef2,
      update: null,
      instance: null,
      colors: { line: '#EE6666', area: '#FAC858', point: '#EE6666' }
    },
    {
      type: '压力传感器',
      chartRef: chartRef3,
      update: null,
      instance: null,
      colors: { line: '#73C0DE', area: '#3BA272', point: '#73C0DE' }
    },
    {
      type: '温度传感器',
      chartRef: chartRef4,
      update: null,
      instance: null,
      colors: { line: '#9A60B4', area: '#EA7CCC', point: '#9A60B4' }
    },
  ];
interface Subsys { subsysId: string; subsysName: string; partList: Part[]; }
interface Part { partId: string; partName: string; }
interface Tree { label: string; monthProbability: string; level: string; children?: Tree[]; subsysId?: string; partId?: string; faultProbability?: string; faultMonth?: string; }
interface EquipMalfPart { predictPartId: number; partId: string; monthProbability: string; failureProbability: number; }
interface EquipMalfSubsys { predictSubsysId: number; subsysId: string; monthProbability: string; failureProbability: number; equipMalfPartList: EquipMalfPart[]; }
interface EquipMalfDetail { predictId: number; equipCode: string; predictTime: string; modelAlgorithm: string; modelVersion: string; failureTiming: string; failureProbability: number; failureLocation: string; failureType: string; failureReason: string; monthProbability: string; userId: string; equipMalfSubsysList: EquipMalfSubsys[]; }

const equips = ref<{
  equipCode: string; equipModelCode: string; unitCode: string; manufactureDate: string; serviceDate: string;
  equipModel: { equipModelCode: string; equipFullname: string; equipSimpleName: string; equipCategory: string; motorLimitYear: string; subsysList: Subsys[]; };
  unit: { unitCode: string; unitPath: string; parentUnitCode: string; unitFullname: string; unitAddress: string; longitude: string; latitude: string; elevation: string; unitType: string; unitSimpleName: string; };
  equipRecord: { equipRecordId: string; equipCode: string; recordStartDate: string; workMile: string; vehicleMotor: string; fireTimes: string; fireAmmo: string; majorFixTimes: string; mediumFixTimes: string; minorFixTimes: string; };
  equipMalfDetail: EquipMalfDetail;
  // 添加缺失的属性
  hasGeneralPrediction?: boolean;
  hasDetailedPrediction?: boolean;
  hasDiagnoseReport?: boolean;
  hasRepairReport?: boolean;
}>({
  equipCode: '', equipModelCode: '', unitCode: '', manufactureDate: '', serviceDate: '',
  equipModel: { equipModelCode: '', equipFullname: '', equipSimpleName: '', equipCategory: '', motorLimitYear: '', subsysList: [] },
  unit: { unitCode: '', unitPath: '', parentUnitCode: '', unitFullname: '', unitAddress: '', longitude: '', latitude: '', elevation: '', unitType: '', unitSimpleName: '' },
  equipRecord: { equipRecordId: '', equipCode: '', recordStartDate: '', workMile: '', vehicleMotor: '', fireTimes: '', fireAmmo: '', majorFixTimes: '', mediumFixTimes: '', minorFixTimes: '' },
  equipMalfDetail: { predictId: 0, equipCode: '', predictTime: '', modelAlgorithm: '', modelVersion: '', failureTiming: '', failureProbability: 0, failureLocation: '', failureType: '', failureReason: '', monthProbability: '', userId: '', equipMalfSubsysList: [] },
  // 初始化缺失的属性
  hasGeneralPrediction: false,
  hasDetailedPrediction: false,
  hasDiagnoseReport: false,
  hasRepairReport: false
});

const faultySystems = computed(() => {
  const subsysList = equips.value.equipModel.subsysList || [];
  const malfSubsysList = equips.value.equipMalfDetail.equipMalfSubsysList || [];
  return malfSubsysList
    .filter(subsys => subsys.failureProbability > 0)
    .map(subsys => {
      const matchedSubsys = subsysList.find(s => s.subsysId === subsys.subsysId);
      return matchedSubsys ? matchedSubsys.subsysName : subsys.subsysId;
    });
});

const loadMonitorTypeData = async () => {
  if (!predictIdDetail.value) {
    return;
  }
  try {
    const response = await MonitorTypeApi(equipCode.value);
    if (response.code === 200) {
      options.value = response.data?.map((item: any) => ({
        label: item.partParamChinese,
        value: item.partParamId,
        measureUnit: item.measureUnit
      }));
      value2.value = response.data?.slice(0, 4).map((item: any) => item.partParamId) || [];
    } else {
      console.error('获取末端感知数据失败:', response.message);
      ElMessage.error(`加载末端感知数据失败: ${response.message}`);
    }
  } catch (error) {
    console.error('调用 monitorTypeApi 失败:', error);
    ElMessage.error('加载末端感知数据失败，请稍后重试');
  }
};

const formatDateToString = (date: Date): string => {
  const year = date.getFullYear(); const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0'); const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0'); const seconds = String(date.getSeconds()).padStart(2, '0');
  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
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
        equipCode: equipCode.value || '',
        partParamId: partParamId
      };

      // 只有在明确选择了日期范围时才添加时间参数
      if (useSelectedDates && startDate.value && endDate.value) {
        params.startTimeStr = formatDateToString(startDate.value);
        params.endTimeStr = formatDateToString(endDate.value);
      } else if (isReplay && startDate.value && endDate.value) {
        // 重放模式也需要时间参数
        params.startTimeStr = formatDateToString(startDate.value);
        params.endTimeStr = formatDateToString(endDate.value);
      }
      // 如果没有选择时间范围，就不发送时间参数，让后端使用默认时间

      const response = await MonitorChartApi(params);
      if (response.code === 200) {
        const timeStr = response.data?.timeStr.split(',').map((time: string) => time.trim());
        const values = response.data?.valueStr.split(',').map((value: string) => parseFloat(value.trim()));
        const typeConfig = defaultTypes.find(t => t.chartRef === chartRef) || defaultTypes[0];
        const colors = typeConfig?.colors || { line: '#ee6666', area: '#ff9999', point: '#ee6666' };
        const paramData = options.value.find(opt => opt.value === partParamId);
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
            boundaryGap: false,
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
  ElMessage.error(`加载参数 ${options.value.find(opt => opt.value === partParamId)?.label || partParamId} 数据失败，请稍后重试`);
};
const loadInitialChartData = async () => {
  if (!predictIdDetail.value) {
    return;
  }
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
  if (!selectedEquipCode.value) {
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
    ElMessage.success({
      message: '数据重放成功',
      duration: 2000,
      showClose: true
    });
  } catch (error) {
    console.error('数据重放失败:', error);
    ElMessage.error({
      message: '数据重放失败，请稍后重试',
      duration: 3000
    });
  } finally {
    if (loadingInstance) {
      loadingInstance.close();
    }
  }
};

watch(value2, async (newValue) => {
  await initSensorCharts();
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
  chartTitles.chart1 = options.value.find(opt => opt.value === selectedIds[0])?.label || '未选择';
  chartTitles.chart2 = options.value.find(opt => opt.value === selectedIds[1])?.label || '未选择';
  chartTitles.chart3 = options.value.find(opt => opt.value === selectedIds[2])?.label || '未选择';
  chartTitles.chart4 = options.value.find(opt => opt.value === selectedIds[3])?.label || '未选择';
  for (let i = 0; i < defaultTypes.length; i++) {
    const { update, chartRef } = defaultTypes[i];
    const partParamId = selectedIds[i] || null;
    if (partParamId && update) {
      await fetchAndRenderChartData(partParamId, chartRef, update, false);
      ElMessage.success(`${options.value.find(opt => opt.value === partParamId)?.label || partParamId} 数据加载成功`);
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
const filteredTreeData = computed(() => {
  if (selectedOption.value === '整机') return treeData.value.map(node => ({ ...node, children: undefined }));
  else if (selectedOption.value === '分系统') return treeData.value.map(node => ({ ...node, children: (node.children || []).map(subsys => ({ ...subsys, children: undefined })) }));
  else if (selectedOption.value === '部件') return treeData.value;
  return [];
});

const handleOptionChange = async () => {
  console.log('当前选项:', selectedOption.value);
  if (treeData.value.length > 0) await handleNodeClick(treeData.value[0]);
  chartTitle.value = selectedOption.value === "整机" ? '整机及未来12个月故障概率' : selectedOption.value === '分系统' ? '分系统未来12个月故障概率' : '部件未来12个月故障概率';
};
const handleNodeClick = async (node: Tree) => {
  console.log('选中节点:', node.label, '层级:', node.level, '当前选项:', selectedOption.value);
  const isLeaf = !node.children || node.children.length === 0;
  let series: { name: string; type: string; data: number[] }[] = [];
  if (selectedOption.value === '整机') {
    const wholeNode = treeData.value[0];
    const probabilities = wholeNode.monthProbability ? wholeNode.monthProbability.split(',').map((prob: string) => parseFloat(prob) * 100) : new Array(12).fill(0);
    series.push({ name: `${wholeNode.label.split('(')[0].trim()}故障概率`, type: 'line', data: probabilities });
    const sortedSubsys = [...equips.value.equipMalfDetail.equipMalfSubsysList].sort((a, b) => b.failureProbability - a.failureProbability).slice(0, 2);
    sortedSubsys.forEach(subsys => {
      const subsysNode = wholeNode.children?.find(child => child.subsysId === subsys.subsysId);
      if (subsysNode) {
        const subsysProb = subsys.monthProbability ? subsys.monthProbability.split(',').map((prob: string) => parseFloat(prob) * 100) : new Array(12).fill(0);
        series.push({ name: `${subsysNode.label.split('(')[0].trim()}故障概率`, type: 'line', data: subsysProb });
      }
    });
    const allParts = sortedSubsys.flatMap(subsys => subsys.equipMalfPartList).sort((a, b) => b.failureProbability - a.failureProbability).slice(0, 2);
    allParts.forEach(part => {
      const subsysNode = wholeNode.children?.find(child => child.children?.some((p: any) => p.partId === part.partId));
      const partNode = subsysNode?.children?.find((p: any) => p.partId === part.partId);
      if (partNode) {
        const partProb = part.monthProbability ? part.monthProbability.split(',').map((prob: string) => parseFloat(prob) * 100) : new Array(12).fill(0);
        series.push({ name: `${partNode.label.split('(')[0].trim()}故障概率`, type: 'line', data: partProb });
      }
    });
  } else if (isLeaf) {
    const probabilities = node.monthProbability ? node.monthProbability.split(',').map((prob: string) => parseFloat(prob) * 100) : new Array(12).fill(0);
    series.push({ name: `${node.label.split('(')[0].trim()}故障概率`, type: 'line', data: probabilities });
    chartTitle.value = `${node.label.split('(')[0].trim()}未来12个月故障概率`;
  } else {
    node.children?.forEach(child => {
      const childName = child.label.split('(')[0].trim();
      const probabilities = child.monthProbability ? child.monthProbability.split(',').map((prob: string) => parseFloat(prob) * 100) : new Array(12).fill(0);
      series.push({ name: `${childName}故障概率`, type: 'line', data: probabilities });
    });
    chartTitle.value = `${node.label.split('(')[0].trim()}${node.level === 'whole' ? '分系统' : '部件'}未来12个月故障概率`;
  }
  const chartOptions = generateChartOptions(series);
  await updateChart(chartOptions);
};
const setChartData5 = async () => {
  if (!predictIdDetail.value) {
    return;
  }
  try {
    const response = await chartDataApi3(true, predictIdDetail.value);
    if (response.code === 200) {
      const chartOptions = reactive({
        series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], avoidLabelOverlap: false, label: { show: false }, emphasis: { label: { show: false } }, itemStyle: { color: (colors: { dataIndex: number }) => ['#007BFF', '#E0E0E0'][colors.dataIndex] }, data: [response.data?.equipMalfDetail?.failureProbability, 1 - response.data?.equipMalfDetail?.failureProbability] }],
        graphic: [{ type: 'group', left: 'center', top: '53%', bounding: 'raw', children: [{ type: 'text', style: { text: '预测故障概率', textAlign: 'center', textVerticalAlign: 'top', fontSize: 13, fill: '#333' } }, { type: 'text', style: { text: `${(response.data?.equipMalfDetail?.failureProbability * 100)}%`, fontSize: 24, textAlign: 'center', textVerticalAlign: 'bottom', fill: '#FF5E65' } }] }],
        labelLine: { show: false },
      });
      return chartOptions;
    } else {
      ElMessage.error(`加载 chart5 数据失败: ${response.message}`);
      return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
    }
  } catch (error) {
    console.error('获取 chart5 数据失败:', error);
    ElMessage.error('加载 chart5 数据失败，请稍后重试');
    return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
  }
};
const { update: updateChartRef5, instance: chartInstanceForRef5 } = useChart(chartRef5, setChartData5);
const loadfinningData = async () => {
  if (!predictIdDetail.value) {
    return;
  }
  try {
    const response = await chartDataApi3(true, predictIdDetail.value);
    vehicleMotor.value = response.data.equipRecord.vehicleMotor || '未知';
    if (response.code === 200) {
      const data = response.data;
      equipData.value = data;
      const predictTime = data.equipMalfDetail.predictTime;
      const predictDate = new Date(predictTime);
      predictMonth.value = predictDate.getMonth() + 2;
      predictYear.value = predictDate.getFullYear();
      if (predictMonth.value > 12) { predictMonth.value -= 12; predictYear.value += 1; }
      treeData.value = [{ label: `主机 (${data?.equipMalfDetail?.failureTiming || ''} - 故障概率 ${(data?.equipMalfDetail?.failureProbability * 100).toFixed(2)}%)`, monthProbability: data?.equipMalfDetail?.monthProbability, level: 'whole', children: data?.equipModel?.subsysList.map((subsys: any) => { const subsysMalf = data?.equipMalfDetail?.equipMalfSubsysList.find((s: any) => s.subsysId === subsys.subsysId); return { label: `${subsys.subsysName || ''} (故障概率 ${(subsysMalf?.failureProbability * 100).toFixed(2)}%)`, monthProbability: subsysMalf?.monthProbability || '', level: 'subsystem', subsysId: subsys.subsysId, children: subsys.partList.map((part: any) => { const partMalf = subsysMalf?.equipMalfPartList.find((p: any) => p.partId === part.partId); return { label: `${part.partName || ''} (故障概率 ${(partMalf?.failureProbability * 100).toFixed(2)}%)`, monthProbability: partMalf?.monthProbability || '', level: 'part', partId: part.partId }; }) }; }) }];
      equips.value = { ...data };
      // 修改这里：装备型号使用 equipModelName
      equipModelName.value = data?.equipModel?.equipSimpleName || '';
      // 所属单位保持不变
      equipSimpleName.value = data?.unit?.unitSimpleName || '';
      serviceDate.value = data?.serviceDate || '';
      failureType.value = data?.equipMalfDetail?.failureType || '';
      manufactureDate.value = data?.manufactureDate || '';
      failureTiming.value = data?.equipMalfDetail?.failureTiming || '';
      determineActiveSections();
      storeSessionData();
      await nextTick();
      if (treeData.value.length > 0) await handleNodeClick(treeData.value[0]);
      const chartOptions = await setChartData5();
      await updateChartRef5(chartOptions);
    } else {
      console.error('获取数据失败:', response.message);
      ElMessage.error(`加载数据失败: ${response.message}`);
    }
  } catch (error) {
    console.error('API 调用失败:', error);
    ElMessage.error('加载数据失败，请稍后重试');
  }
};
const generateChartOptions = (series: { name: string; type: string; data: number[] }[]) => {
  const monthLabels = generateMonthLabels(predictMonth.value || 6, predictYear.value || 2025);
  return {
    tooltip: { trigger: 'axis', formatter: function (params: any) { const month = monthLabels[params[0].dataIndex]; let result = `${month}<br/>`; params.forEach((item: any) => { result += `${item.seriesName}: ${item.value.toFixed(2)}%<br/>`; }); return result; } },
    legend: { data: series.map(s => s.name), top: '5%' },
    grid: { left: '3%', right: '4%', bottom: '10%', containLabel: true },
    toolbox: { feature: { saveAsImage: {} } },
    xAxis: { type: 'category', boundaryGap: false, data: monthLabels, axisLabel: { fontSize: 12 } },
    yAxis: { name: '故障概率(%)', type: 'value', min: 0, max: 100, interval: 20, axisLabel: { formatter: '{value}' } },
    series
  };
};

const { update: updateChartRef, instance: chartInstanceForRef6 } = useChart(chartRef6, async () => {
  const wholeNode = treeData.value[0] || { level: 'whole', monthProbability: '', label: '' } as Tree;
  const name = wholeNode.label.split('(')[0].trim();
  const probabilities = wholeNode.monthProbability ? wholeNode.monthProbability.split(',').map((prob: string) => parseFloat(prob) * 100) : new Array(12).fill(0);
  const series = [{ name: `${name}故障概率`, type: 'line', data: probabilities }];
  return generateChartOptions(series);
});

const updateChart = async (chartOptions: any) => {
  if (!chartRef6.value) { console.error('chartRef 未挂载'); return; }
  try { console.log('Updating chart with options:', chartOptions); await updateChartRef(chartOptions); } catch (error) { console.error('更新图表失败:', error); }
};

const getInitialOptions = (type: string, colors: Colors) => async () => ({
  title: { left: 'center', text: type, textStyle: { color: colors.line } },
  tooltip: { trigger: 'axis', position: function (pt: number[]) { return [pt[0], '10%']; } },
  toolbox: { feature: { dataZoom: { yAxisIndex: 'none' }, restore: {}, saveAsImage: {} } },
  xAxis: { type: 'category', boundaryGap: false, data: [], axisLabel: { interval: 119, formatter: function (value: string) { return value; } } },
  yAxis: { type: 'value', name: type === '震动传感器' ? '震动 (mm/s)' : type === '电压传感器' ? '电压 (V)' : type === '压力传感器' ? '压力 (Pa)' : '温度 (℃)', boundaryGap: [0, '100%'], axisLine: { lineStyle: { color: colors.line } } },
  dataZoom: [{ type: 'inside', start: 0, end: 100 }, { type: 'slider', start: 0, end: 100, height: 20 }],
  series: [{ name: type, type: 'line', symbol: 'circle', symbolSize: 4, sampling: 'lttb', itemStyle: { color: colors.point }, areaStyle: { color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{ offset: 0, color: colors.area }, { offset: 1, color: colors.line }]), opacity: 0.5 }, data: [] }],
  grid: { left: '5%', right: '5%', bottom: '15%', containLabel: true }
});
const initSensorCharts = async () => {
  await nextTick(); // 确保 DOM 已渲染
  if (defaultTypes[0].instance) return; // 防止重复初始化

  const { update: update1, instance: instance1 } = useChart(chartRef1, getInitialOptions('震动传感器', defaultTypes[0].colors));
  const { update: update2, instance: instance2 } = useChart(chartRef2, getInitialOptions('电压传感器', defaultTypes[1].colors));
  const { update: update3, instance: instance3 } = useChart(chartRef3, getInitialOptions('压力传感器', defaultTypes[2].colors));
  const { update: update4, instance: instance4 } = useChart(chartRef4, getInitialOptions('温度传感器', defaultTypes[3].colors));

  // 赋值
  defaultTypes[0].update = update1; defaultTypes[0].instance = instance1;
  defaultTypes[1].update = update2; defaultTypes[1].instance = instance2;
  defaultTypes[2].update = update3; defaultTypes[2].instance = instance3;
  defaultTypes[3].update = update4; defaultTypes[3].instance = instance4;

  // 强制 resize
  [instance1, instance2, instance3, instance4].forEach(inst => inst.value?.resize());
};
// 精细化诊断
const chartRef15 = ref(null);
const usingSuggestion = ref('');
const exceptionInfo = ref('');
const diagnoseTime = ref('');
const fixTiming = ref('');
const subsysList = ref<any[]>([]);

const loaddiagnoseData = async () => {
  if (!diagnoseId.value) {
    return;
  }
  try {
    const idToUse = diagnoseId.value;
    const response = await diagnoseDetailApi({ diagnoseId: idToUse });
    if (response.code === 200) {
      const data = response.data;

      // 更新诊断信息
      diagnoseTime.value = data?.equipDiag?.diagnoseTime
        ? data.equipDiag.diagnoseTime.split("T")[0]
        : "无诊断时间";
      fixTiming.value = data?.equipDiag?.fixTiming || "暂无数据";
      usingSuggestion.value = data?.equipDiag?.usingSuggestion || "无建议";
      exceptionInfo.value = data?.equipDiag?.exceptionInfo || "暂无数据";

      // 更新子系统列表
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
      const chartOptions = await setChartData15();
      await updateChartRef15(chartOptions);
      determineActiveSections();
    } else {
      console.error("后端返回的数据格式不正确：", response);
    };
  } catch (error) {
    console.error("请求失败：", error);
  };
}
const setChartData15 = async () => {
  if (!diagnoseId.value) {
    return;
  }
  try {
    const idToUse = diagnoseId.value;
    console.log("idToUse", idToUse);
    const response = await diagnoseDetailApi({ diagnoseId: idToUse });
    if (response.code === 200) {
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
      return chartOptions;
    } else {
      console.error('加载 chart15 数据失败:', response.message);
      return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
    }
  } catch (error) {
    console.error('获取 chart15 数据失败:', error);
    return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
  }
};

const { update: updateChartRef15, instance: chartInstanceForRef15 } = useChart(chartRef15, () => setChartData15());

// 修理时机

const repairequips = ref({
  equipCode: "",
  equipModelCode: "",
  unitCode: "",
  manufactureDate: "",
  serviceDate: "",
  equipModel: {
    equipModelCode: "",
    equipFullname: "",
    equipSimpleName: "",
    equipCategory: "",
    motorLimitYear: "",
    motorControl: false,
    suppliers: "",
  },
  unit: {
    unitCode: "",
    unitPath: "",
    parentUnitCode: "",
    unitFullname: "",
    unitAddress: "",
    longitude: "",
    latitude: "",
    elevation: "",
    unitType: "",
    unitSimpleName: "",
  },
  equipRecord: {
    equipRecordId: "",
    equipCode: "",
    recordStartDate: "",
    workMile: "",
    vehicleMotor: "",
    fireTimes: "",
    fireAmmo: "",
    majorFixTimes: "",
    mediumFixTimes: "",
    minorFixTimes: "",
  },
  equipRepair: {
    repairId: '',
    userId: "",
    equipCode: "",
    predictTime: "",
    repairType: "",
    repairTime: "",
  }
});
const isFixed = ref('是');
const malfRecords = ref({
  malfRecordId: "",
  equipRecordId: "",
  equipCode: "",
  malfDate: "",
  malfCategoryDictId: "",
  partId: "",
  malfLocation: "",
  phenomenon: "",
  reason: "",
  handleResult: "",
  fillStaffCode: "",
  inchargeStaffCode: "",
  rectified: "",
});

const loadrepairData = async () => {
  if (!repairId.value) {
    return;
  }
  try {
    const response = await RepairdetailApi(repairId.value);

    if (response.code === 200) {
      repairequips.value = response.data;
      storeSessionData();
      determineActiveSections();
    } else {
      console.error('获取数据失败:', response.message);
      ElMessage.error(`加载数据失败: ${response.message}`);
    }
  } catch (error) {
    console.error('API 调用失败:', error);
    ElMessage.error('加载数据失败，请稍后重试');
  }
};

</script>

<style scoped>
.top {
  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
}

.topinfo {
  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
}

.lastinfo {
  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: 10px;
}

.title {
  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
}

.bar {
  width: 5px;
  height: 20px;
  background-color: #409EFF;
  margin-right: 10px;
}

.fl {
  float: left;
}

.describe {
  font-size: 18px;
  font-weight: bold;
  color: #409EFF;
}

.clear {
  clear: both;
}

.center-content {
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.image-col {
  display: flex;
  justify-content: center;
  align-items: center;
}

.tank-image {
  width: auto;
  height: 90%;
}

.chart-container {
  width: 100%;
  height: 200px;
}

.line-container {
  width: 100%;
  height: 300px;
}

.group {
  margin-top: 20px;
  margin-left: 20px;
}

.sub-tree {
  margin-top: 20px;
  margin-left: 20px;
}

.replay-container {
  position: relative;
  display: inline-block;
  margin-right: 10px;
}

.date-picker-card {
  position: absolute;
  top: 100%;
  right: 0;
  z-index: 1000;
  width: 300px;
  padding: 10px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  margin-top: 5px;
}

.moduan {
  :deep(.el-select-group__title) {
    font-size: 20px;
    font-weight: bold;
  }
}

.info-row {
  margin-bottom: 10px;
}

.info-item {
  margin-bottom: 5px;
}

/* 新增样式 */
.chart-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  color: #909399;
  font-size: 14px;
  height: 400px;
  width: 100%;
  min-height: 0;
  min-width: 0;
  overflow: hidden;
}

.chart {
  width: 100%;
  height: 400px;
  border-left: #999494 1px solid;
}

.chart-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chart-title {
  font-size: 20px;
  font-weight: bold;
  color: #409eff;
  margin-bottom: 10px;
  background-color: #bdddfd;
  height: 50px;
  line-height: 50px;
  padding-left: 5px;
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
}
</style>