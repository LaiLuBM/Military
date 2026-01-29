<template>
    <div class="report-container">
        <el-card class="report-card">
            <div class="operation-bar hide-print">
                <div class="right-ops">
                    <el-button type="success" @click="openExportDialog">导出故障报告</el-button>
                </div>
            </div>

            <div id="report-content" class="report-body">
                <div class="report-header export-item">
                    <h1>修理时机故障预测报告</h1>
                    <div class="report-meta">
                        <span>生成时间：{{ new Date().toLocaleDateString('zh-CN') }}</span>
                    </div>
                </div>

                <el-divider border-style="double" class="export-item" />

                <div class="section-block export-item">
                    <div class="section-title">一、装备信息</div>
                    <el-row :gutter="20" class="info-grid">
                        <el-col :span="8" v-for="(item, index) in equipInfoItems" :key="index">
                            <div class="info-item">
                                <span class="label">{{ item.label }}：</span>{{ item.value }}
                            </div>
                        </el-col>
                    </el-row>
                </div>

                <div class="section-block export-item">
                    <div class="section-title">二、装备最近一次故障信息</div>
                    <div v-if="malfRecords.malfRecordId">
                        <el-row :gutter="20" class="info-grid">
                            <el-col :span="8" v-for="(item, index) in recentFaultItems" :key="index">
                                <div class="info-item">
                                    <span class="label">{{ item.label }}：</span>{{ item.value }}
                                </div>
                            </el-col>
                            <el-col :span="24">
                                <div class="info-item full-width">
                                    <span class="label">故障现象：</span>{{ malfRecords.phenomenon || '暂无数据' }}
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                    <div v-else class="no-data">
                        暂无数据故障记录
                    </div>
                </div>

                <div class="section-block export-item">
                    <div class="section-title">三、修理时机预测结果</div>
                    <el-row :gutter="20" class="conclusion-grid">
                        <el-col :span="8" v-for="(item, index) in repairPredictionItems" :key="index">
                            <div class="conclusion-item">
                                <div class="label">{{ item.label }}</div>
                                <div class="value" :class="{ 'highlight-red': item.highlight }">{{ item.value }}</div>
                            </div>
                        </el-col>
                    </el-row>
                </div>
            </div>
        </el-card>

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
import { ref, onMounted, computed } from 'vue';
import { RepairdetailApi } from '@/api/fault/repairdetail';
import { useRoute, useRouter } from 'vue-router';
import html2canvas from 'html2canvas';
import { jsPDF } from 'jspdf';
import { ElMessage } from 'element-plus';
import { equipDetailApi } from '@/api/fault/equipDetail';
import axios from 'axios';
import { getToken } from "@/utils/auth.js";

const route = useRoute();
const router = useRouter();

const equipCode = ref('');
const motorLimitYear = ref('');
const serviceLimit = ref('');
const workMile = ref('');
const repairId = ref('');
const isFixed = ref('是');

const formatRepairTime = (timeStr: string | undefined) => {
    if (!timeStr) return '暂无数据';
    if (timeStr.includes('天')) return timeStr;
    const match = timeStr.match(/(\d+)\s*周/);
    if (match && match[1]) {
        const weeks = parseInt(match[1]);
        return `${weeks * 7}天`;
    }
    return timeStr;
};

const equips = ref<any>({
    equipCode: "",
    fightCode:"",
    manufactureDate: "",
    serviceDate: "",
    equipModel: { equipSimpleName: "", motorLimitYear: "" },
    unit: { unitSimpleName: "", unitType: "" },
    equipRecord: { vehicleMotor: "0", majorFixTimes: "0" },
});

const equipRepairs = ref({
    repairId: 0,
    predictTime: "",
    repairType: "",
    repairTime: "",
});

const malfRecords = ref<any>({
    malfRecordId: "",
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

// 计算属性
const equipInfoItems = computed(() => [
    { label: '所属单位', value: equips.value.unit?.unitSimpleName || '暂无数据' },
    { label: '装备型号', value: equips.value.equipModel?.equipSimpleName || '暂无数据' },
    { label: '装备统一编码', value: equips.value.equipCode || 'N/A' },
    { label: '装备战斗编号', value: equips.value.fightCode || '暂无数据' },
    { label: '列装时间', value: equips.value.serviceDate || '暂无数据' },
    { label: '服役日期', value: serviceLimit.value || '暂无数据' },
    { label: '大修间隔期内累计行驶里程', value: `${workMile.value || 0} 公里` },
    { label: '中修间隔期内发动机累计消耗摩托小时', value: `${equips.value.equipRecord?.vehicleMotor || '0'} 小时` },
    { label: '当年摩托小时消耗限额', value: `${motorLimitYear.value || 0} 小时` },
    { label: '出厂日期', value: equips.value.manufactureDate || '暂无数据' },
    // { label: '装备剩余寿命', value: `${equips.value.equipModel?.motorLimitYear || '暂无数据'} 年` },
    { label: '装备战备储备标准', value: equips.value.unit?.unitType || '暂无数据' },
    { label: '年度修理指标', value: `${equips.value.equipRecord?.majorFixTimes || '0'} 次` },
]);

const recentFaultItems = computed(() => [
    { label: '单装故障记录ID', value: malfRecords.value.malfRecordId || '暂无数据' },
    { label: '故障日期', value: malfRecords.value.malfDate || '暂无数据' },
    { label: '故障类型', value: malfRecords.value.malfCategoryDictId || '暂无数据' },
    { label: '部件编码', value: malfRecords.value.partId || '暂无数据' },
    { label: '故障部位', value: malfRecords.value.malfLocation || '暂无数据' },
    { label: '故障原因', value: malfRecords.value.reason || '暂无数据' },
    { label: '处理结果', value: malfRecords.value.handleResult || '暂无数据' },
    { label: '是否排除故障', value: isFixed.value },
    { label: '填写人代码', value: malfRecords.value.fillStaffCode || '暂无数据' },
    { label: '负责人代码', value: malfRecords.value.inchargeStaffCode || '暂无数据' },
    { label: '备注', value: '无' },
]);

const repairPredictionItems = computed(() => [
    { label: '修理类别', value: equipRepairs.value.repairType || '暂无数据', highlight: true },
    { label: '送修时间', value: formatRepairTime(equipRepairs.value.repairTime), highlight: true },
    { label: '预测执行时间', value: equipRepairs.value.predictTime || '暂无数据' },
]);

const exportDialogVisible = ref(false);
const exportFormat = ref('pdf');

const openExportDialog = () => {
    exportDialogVisible.value = true;
};

const handleExport = async () => {
    exportDialogVisible.value = false;

    try {
        ElMessage.info(`正在生成${exportFormat.value.toUpperCase()}报告，请稍候...`);

        const element = document.querySelector('.el-card') as HTMLElement;
        if (!element) throw new Error('无法找到 .el-card 元素');

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

        const imgData = canvas.toDataURL('image/jpeg', 0.85);
        const imgWidth = 210;
        const pageHeight = 297;
        const imgHeight = (canvas.height * imgWidth) / canvas.width;

        const pdf = new jsPDF('p', 'mm', 'a4');
        const filename = `修理时机预测报告_${equipCode.value || 'unknown'}_${new Date().toISOString().slice(0, 10)}`;

        let heightLeft = imgHeight;
        let position = 0;

        pdf.addImage(imgData, 'JPEG', 0, position, imgWidth, imgHeight);
        heightLeft -= pageHeight;

        while (heightLeft > 0) {
            position = heightLeft - imgHeight;
            pdf.addPage();
            pdf.addImage(imgData, 'JPEG', 0, position, imgWidth, imgHeight);
            heightLeft -= pageHeight;
        }

        if (exportFormat.value === 'pdf') {
            pdf.save(`${filename}.pdf`);
            ElMessage.success('PDF 导出成功');
            return;
        }

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
                pdf.save(`${filename}.pdf`);
            }
        }
    } catch (error) {
        console.error('导出失败:', error);
        ElMessage.error('导出失败，请重试');
    }
};

const safeSessionStorage = {
    set(key: string, value: string): void {
        try { sessionStorage.setItem(key, value); } catch (e) { console.error('SessionStorage存储失败:', e); }
    },
    get(key: string): string {
        try { return sessionStorage.getItem(key) || ''; } catch (e) { console.error('SessionStorage读取失败:', e); return ''; }
    }
};

const storeSessionData = () => {
    if (equipCode.value) {
        safeSessionStorage.set('repairDetailEquipCode', equipCode.value);
    }
};

onMounted(async () => {
    const queryEquipCode = route.query.equipCode;
    const queryRepairId = route.query.repairId;

    if (queryRepairId) repairId.value = String(queryRepairId);
    if (queryEquipCode) {
        equipCode.value = String(queryEquipCode);
    } else {
        const storageEquipCode = safeSessionStorage.get('repairDetailEquipCode');
        equipCode.value = storageEquipCode || '';
    }

    storeSessionData();

    try {
        const response = await RepairdetailApi(repairId.value);
        const resinfo = await equipDetailApi(equipCode.value);
        motorLimitYear.value = resinfo.data.equipModel?.motorLimitYear || '0';
        workMile.value = resinfo.data.usageRecord?.workMile || '0';
        serviceLimit.value = resinfo.data.serviceLimit;
        if (response.code === 200) {
            const data = response.data;
            equips.value = data;

            equipRepairs.value = data.equipRepair;

            if (data.malfRecords?.length > 0) {
                malfRecords.value = data.malfRecords[0];
                isFixed.value = malfRecords.value.rectified === '1' ? '是' : '否';
            } else {
                malfRecords.value = {
                    malfRecordId: "", malfDate: "", malfCategoryDictId: "", partId: "", malfLocation: "",
                    phenomenon: "", reason: "", handleResult: "", fillStaffCode: "", inchargeStaffCode: "", rectified: ""
                };
            }
            storeSessionData();
        }
    } catch (error) {
        console.error('API 调用失败:', error);
    }
});
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

.no-data {
    text-align: center;
    color: #999;
    font-size: 14px;
    padding: 20px;
}

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