<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <el-row :gutter="20">
            <el-col :span="1">
              <el-button text @click="goBack" icon="ArrowLeft"
                style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
            </el-col>
            <el-col :span="20">
              <h2 class="title-margin">精细化健康评估结果详情</h2>
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
          <el-row :gutter="20" v-if="approvalStatus == '已核准'" style="margin-top: 10px;">
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
          <div class="outer-border" style="margin-top: 5px;">
            <el-row :gutter="20">
              <el-col :span="5" style="margin-top: 20px;">
                <div ref="chartRef2" style="width: auto; height: 175px;"></div>
                <div style="text-align: center; font-weight: bold;">健康指数</div>
                <!-- <div class="line"></div> -->
              </el-col>
              <el-col :span="15">
                <el-row style="margin-top:20px;">
                  <el-col :span="24">
                    <div class="section-title"><el-icon>
                        <Management />
                      </el-icon>基本信息</div>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 20px;">
                  <el-col :span="9">
                    <div>所属单位：{{ unitFullname }}</div>
                  </el-col>
                  <el-col :span="7">
                    <div>装备型号：{{ equipSimpleName }}</div>
                  </el-col>
                  <el-col :span="8">
                    <div>装备统一编码：{{ equipCode }}</div>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 30px;">
                  <el-col :span="9">
                    <div>服役日期：{{ serviceGap }}</div>
                  </el-col>
                  <el-col :span="7">
                    <div>出厂日期：{{ manufactureDate }}</div>
                  </el-col>
                  <el-col :span="8">
                     <div>装备战斗编号：{{ fightCode }}</div>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 30px;"></el-row>
                <el-row>
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
                  <!-- <div class="info">健康等级：{{ equipHealthGrade }}</div> -->
                </el-row>
                <!-- <el-row>
                  <div class="info">评估方式：{{ assessmentMethod }}</div>
                </el-row>
                <el-row>
                  <div class="info">算法模型：{{ algorithm }}</div>
                </el-row> -->
              </el-col>
              <el-col :span="19">
                <el-row style="margin-top: 20px;">
                  <!-- <el-col :span="6">
                    <div>列装时间：{{ motorLimitYear }}年</div>
                  </el-col>
                  <el-col :span="6" style="margin-left: auto;">
                    <div>故障次数：{{ malfCount }}</div>
                  </el-col> -->
                  <el-col :span="12">
                    <div>故障次数：{{ malfCount }}</div>
                  </el-col>
                  <el-col :span="7"></el-col>
                  <el-col :span="5">
                    <el-button type="primary" plain @click="handleDetail(equipCode)">查看更多装备信息</el-button>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 20px;">
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
            <el-divider />
            <el-row :gutter="20">
              <el-col :span="7">
                <div ref="chartRef3" style="width: auto; height: 500px;"></div>
              </el-col>
              <el-col :span="1">
                <div class="vertical-line"></div>
              </el-col>
              <el-col :span="16">
                <el-row>
                  <h2>分系统健康指数概况</h2>
                </el-row>
                <!-- 表格 -->
                <el-table :data="tableData2" border :header-cell-style="headerCellStyle"
                  style="width: 100%; margin-bottom: 20px;" :span-method="tableSpanMethod">
                  <el-table-column prop="category" label="分系统" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="module" label="部件" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="coreModule" label="是否核心部件" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="accuracyLevel" label="健康指数" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="performanceLevel" label="健康等级" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                </el-table>
                <el-row>
                  <h2>近6月健康指数变化</h2>
                  <div ref="chartRef" style="width: 100%; height: 400px;"></div>
                </el-row>
              </el-col>
            </el-row>
            <el-row>
              <div class="parallel-line"></div>
            </el-row>
            <el-row :gutter="20">
              <div class="bottom"></div>
            </el-row>
            <div v-if="suggestionGeneratedStatus !== '未生成'">
              <el-row :gutter="20">
                <el-col :span="10">
                  <div class="section-title">健康建议<el-icon>
                      <DArrowRight />
                    </el-icon></div>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <div class="header">
                  <div class="fl" style="color: #4079eb; margin-right: 5px;">1/</div>
                  <div class="fl">视情维修建议（待维修部件列表）</div>
                  <div style="clear: both;"></div>
                </div>
              </el-row>
              <el-row :gutter="20">
                <div class="bottom"></div>
              </el-row>
              <el-row :gutter="20">
                <el-table :data="tableData3" border :header-cell-style="headerCellStyle"
                  style="width: 95%; margin-bottom: 20px; margin: 0 auto">
                  <el-table-column prop="category" label="分系统" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="module" label="部件" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="coreModule" label="是否核心部件" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="accuracyLevel" label="健康指数" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                </el-table>
              </el-row>
              <el-row :gutter="20">
                <div class="bottom"></div>
              </el-row>
              <el-row :gutter="20">
                <div class="header">
                  <div class="fl" style="color: #4079eb; margin-right: 5px;">2/</div>
                  <div class="fl">延寿使用建议</div>
                  <div style="clear: both;"></div>
                </div>
              </el-row>
              <el-row :gutter="20">
                <div class="bottom"></div>
                <el-col :span="6">
                  <h3 style="text-align: center; margin-top: 30px;">发动机状况</h3>
                  <div class="line" style="text-align: center; margin-top: 30px;"></div>
                  <div style="text-align: center; margin-top: 20px;">
                    中修间隔期内发动机累计消耗摩托小时：{{ enginevehicleMotor }}h
                  </div>
                  <div style="text-align: center; margin-top: 20px;">
                    健康指数：{{ equipHealthScore }}
                  </div>
                </el-col>
                <el-col :span="11">
                  <div ref="chartRef4" style="width: 100%; height: 300px; margin-top: 30px;"></div>
                </el-col>
                <el-col :span="7">
                  <div class="right-panel">
                    <h3 style="color: #4079eb; text-align: center; margin-top: 30px;">延寿建议</h3>
                    <div class="line" style="text-align: center; margin-top: 30px;"></div>
                    <div style="text-align: center; margin-top: 20px;">
                      是否延寿：{{ isLifeExtend }}
                    </div>
                    <div style="text-align: center; margin-top: 20px;">
                      延寿时间：{{ lifeExtendLength }}
                    </div>
                    <div style="text-align: center; margin-top: 20px;">
                      延寿方式：{{ lifeExtendMethod }}
                    </div>
                  </div>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <div class="bottom"></div>
              </el-row>
              <el-row :gutter="20">
                <div class="header">
                  <div class="fl" style="color: #4079eb; margin-right: 5px;">3/</div>
                  <div class="fl">作战运用建议</div>
                  <div style="clear: both;"></div>
                </div>
                <el-col :span="24">
                  <div class="bottom">{{ missionAdvice }}</div>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <div class="bottom"></div>
              </el-row>
              <el-row :gutter="20">
                <div class="header">
                  <div class="fl" style="color: #4079eb; margin-right: 5px;">4/</div>
                  <div class="fl">维修保养建议（维修保养建议生成）</div>
                  <div style="clear: both;"></div>
                </div>
              </el-row>
              <el-row :gutter="20">
                <div class="bottom"></div>
              </el-row>
              <el-row :gutter="20">
                <el-table :data="tableData4" border :header-cell-style="headerCellStyle"
                  style="width: 95%; margin-bottom: 20px; margin: 0 auto">
                  <el-table-column prop="category" label="分系统" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="module" label="部件名称" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="accuracyLevel" label="健康指数" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="coreModule" label="健康等级" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="performanceLevel" label="维修优先级" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                  <el-table-column prop="manage" label="操作" header-align="center" align="center"  :show-overflow-tooltip="true"/>
                </el-table>
              </el-row>
            </div>
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
import { useRoute } from 'vue-router';
import { useRouter } from 'vue-router';
import { moreAssessmentInfo, submit } from '@/api/fault/refinehealth';
import { ElMessage, ElMessageBox } from 'element-plus';
import html2canvas from 'html2canvas';
import jsPDF from 'jspdf';
import axios from 'axios'; // 引入 axios 用于发送请求
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
const isSubmitted = ref(false); // Track submission status
const suggestionGeneratedStatus = ref('');
const serviceGap = ref('');
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
    path: "/healthevaluate/refinereport",
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
    const filename = `精细化健康评估报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

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
      assessmentId: parseInt(assessmentId.value) // 使用 parseInt 更安全
    });

    // 5. 处理响应
    if (response.code === 200) {
      ElMessage.success('提交核准成功');
      isSubmitted.value = true;
      approvalStatus.value = '待核准'; // 更新为"待核准"状态
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

const tableData2 = ref([]);

// 表格单元格合并方法
const tableSpanMethod = ({ row, column, rowIndex, columnIndex }) => {
  if (columnIndex === 0) {
    // 分系统列
    if (rowIndex === row.rowIndex) {
      return {
        rowspan: row.rowspan,
        colspan: 1,
      };
    } else {
      return {
        rowspan: 0,
        colspan: 0,
      };
    }
  }
  return {
    rowspan: 1,
    colspan: 1,
  };
};

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

      // 生成 tableData2，保留每行一个部件的结构，添加 rowIndex 和 rowspan
      const subsysMap = new Map();
      let currentRowIndex = 0;
      tableData2.value = response.data[0].equipAssessment.subsysHealthList?.flatMap((subsys: any) => {
        const parts = subsys.partHealthList?.map((part: any, partIndex: number) => ({
          category: subsys.subsysName || '未知',
          module: part.partName || '未知',
          coreModule: part.isCore || '未知',
          accuracyLevel: part.partHealthScore || '未知',
          performanceLevel: part.partHealthGrade || '未知',
          rowIndex: partIndex === 0 ? currentRowIndex : null, // 仅第一行记录 rowIndex
          rowspan: partIndex === 0 ? (subsys.partHealthList?.length || 1) : 0, // 第一行设置 rowspan
        })) || [];
        currentRowIndex += subsys.partHealthList?.length || 1;
        return parts;
      }) || [];

      console.log('tableData2:', tableData2.value);
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
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;
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

const tableData = computed(() => [
  {
    item: `中修间隔期内发动机累计消耗摩托小时（从大修以后的值）|${vehicleMotorAfterMajorFix.value || '0.0'}`,
    value1: `小修摩托小时：${"250"}|中修摩托小时：${"500"} |大修摩托小时：${"1000"}`,
    value2: `${bias.value || 'N/A'}`,
    bias: {
      value: bias.value || 'N/A' // Ensure bias is an object with a value property
    }
  }
]);

const getTagType = (status: any) => {
  const statusStr = String(status?.value || status || '');
  if (statusStr.includes('范围内') && !statusStr.includes('边缘')) {
    return 'primary'; // 蓝色
  } else if (statusStr.includes('边缘范围内10%')) {
    return 'success'; // 绿色
  } else if (statusStr.includes('边缘范围外10%')) {
    return 'warning'; // 黄色
  } else if (statusStr.includes('范围外')) {
    return 'danger'; // 红色
  }
  return 'info'; // 默认灰色
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
  backgroundColor: '#d9d9db', // 表头背景颜色
  color: 'black', // 表头字体颜色
  fontWeight: 'bold' // 表头字体加粗
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
        itemStyle: { color: '#5470C6' }, // 蓝色
        smooth: true,
      },
      {
        name: '底盘系统',
        type: 'line',
        data: dipanScore,
        itemStyle: { color: '#91CC75' }, // 绿色
        smooth: true,
      },
      {
        name: '通信系统',
        type: 'line',
        data: tongxinScore,
        itemStyle: { color: '#EE6666' }, // 红色
        smooth: true,
      },
      {
        name: '电气系统',
        type: 'line',
        data: dianqiScore,
        itemStyle: { color: '#73C0DE' }, // 青色
        smooth: true,
      },
      {
        name: '武器系统',
        type: 'line',
        data: wuqiScore,
        itemStyle: { color: '#FAC858' }, // 黄色
        smooth: true,
      },
      {
        name: '火控系统',
        type: 'line',
        data: huokongScore,
        itemStyle: { color: '#3BA272' }, // 深绿色
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

<style scoped>
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
  margin: 0 auto;
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
  margin-left: 0px;
  color: #409eff;
  display: flex;
  align-items: center;
}

.line {
  content: '';
  display: block;
  width: 70%;
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
  height: 102%;
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

/* 添加打印样式 */
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

  /* 隐藏按钮 */
  .el-button {
    display: none !important;
  }

  /* 确保图表可见 */
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