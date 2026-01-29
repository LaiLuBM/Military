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
                    <h1>精细化故障预测结果报告</h1>
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
                    <div class="section-title">二、故障预测结果</div>
                    <el-row :gutter="20" class="conclusion-grid">
                        <el-col :span="12" v-for="(item, index) in conclusionItems" :key="index">
                            <div class="conclusion-item">
                                <div class="label">{{ item.label }}</div>
                                <div class="value" :class="{ 'highlight-red': item.highlight }">{{ item.value }}</div>
                            </div>
                        </el-col>
                    </el-row>
                </div>

                <div class="section-block export-item">
                    <div class="section-title">三、整机未来12个月故障概率趋势</div>
                    <div ref="wholeChartRef" class="chart-standard"></div>
                </div>

                <div class="section-block export-item">
                    <div class="section-title">四、主要分系统故障概率对比 (Top 5)</div>
                    <div ref="subsysChartRef" class="chart-standard"></div>
                </div>

                <div class="section-block">
                    <div class="section-title export-item">五、重点分系统及部件详细分析</div>
                    <div class="parts-grid-container">
                        <div v-for="(item, index) in partChartsData" :key="index"
                            class="part-chart-wrapper export-item">
                            <div class="sub-chart-header">
                                {{ index + 1 }}. {{ item.subsysName }} 部件故障概率
                            </div>
                            <div :ref="(el) => setPartChartRef(el, index)" class="chart-small"></div>
                        </div>
                    </div>
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
import { ref, onMounted, computed, nextTick, onBeforeUnmount } from "vue";
import { useRouter, useRoute } from "vue-router";
import { chartDataApi3 } from "@/api/fault/detail";
import { equipDetailApi } from "@/api/fault/equipDetail";
import { ElMessage, ElLoading } from "element-plus";
import * as echarts from "echarts";
import html2canvas from "html2canvas";
import { jsPDF } from "jspdf";
import axios from "axios";
import { getToken } from "@/utils/auth.js";

const router = useRouter();
const route = useRoute();

// === 数据变量 ===
const selectedEquipCode = ref("");
const selectedPredictId = ref<number | null>(null);
const displayUnitName = ref("");
const equips = ref<any>({
    equipModel: { equipSimpleName: "" },
    equipRecord: {},
    equipMalfDetail: { failureProbability: 0, predictTime: "", monthProbability: "" },
});
const fightCode = ref("");
const serviceDate = ref("");
const serviceLimit = ref("");
const workMile = ref("");
const vehicleMotor = ref("");
const manufactureDate = ref("");
const failureTiming = ref("");
const failureType = ref("");
const faultySystems = ref<string[]>([]);
const predictMonth = ref<number>(6);
const predictYear = ref<number>(2025);

const wholeChartRef = ref<HTMLElement | null>(null);
const subsysChartRef = ref<HTMLElement | null>(null);
const partChartRefs = ref<(HTMLElement | null)[]>([]);
let charts: echarts.ECharts[] = [];
const partChartsData = ref<{ subsysName: string; series: any[] }[]>([]);

const exportDialogVisible = ref(false);
const exportFormat = ref("pdf");

// === 计算属性（统一信息展示）===
const basicInfoItems = computed(() => [
    { label: "所属单位", value: displayUnitName.value || "暂无数据" },
    { label: "装备型号", value: equips.value.equipModel?.equipSimpleName || "暂无" },
    { label: "装备统一编码", value: selectedEquipCode.value || "N/A" },
    { label: "装备战斗编号", value: fightCode.value || "暂无" },
    { label: "列装时间", value: serviceDate.value || "暂无" },
    { label: "服役日期", value: serviceLimit.value || "暂无" },
    { label: "累计大修间隔期内累计行驶里程", value: `${workMile.value || 0} 公里` },
    { label: "中修间隔期内发动机累计消耗摩托小时", value: `${vehicleMotor.value || 0} 小时` },
    { label: "出厂日期", value: manufactureDate.value || "暂无" },
]);

const conclusionItems = computed(() => [
    { label: "当前故障概率", value: formatFailureProbability(), highlight: true },
    { label: "预测执行时间", value: equips.value.equipMalfDetail?.predictTime || "暂无" },
    { label: "故障发生时机", value: failureTiming.value || "暂无", highlight: true },
    { label: "预测故障类型", value: failureType.value || "暂无", highlight: true },
    // { label: "涉及系统", value: faultySystems.value.join("，") || "无" },
]);

const formatFailureProbability = () => {
    const probability = equips.value.equipMalfDetail?.failureProbability;
    if (probability === undefined || probability === null) return "暂无数据";
    return `${(probability * 100).toFixed(2)}%`;
};

// === 工具函数（保持原有逻辑不变）===
const parseProbString = (str: string | null | undefined): number[] => {
    if (!str) return new Array(12).fill(0);
    return str.split(",").map((s) => {
        const val = parseFloat(s.trim());
        return isNaN(val) ? 0 : parseFloat((val * 100).toFixed(2));
    });
};

const setPartChartRef = (el: any, index: number) => {
    if (el) partChartRefs.value[index] = el;
};

const getMonthLabels = () => {
    const labels: string[] = [];
    const startMonth = predictMonth.value;
    const startYear = predictYear.value;
    for (let i = 0; i < 12; i++) {
        const m = ((startMonth - 1 + i) % 12) + 1;
        const y = startYear + Math.floor((startMonth - 1 + i) / 12);
        labels.push(`${y}-${m.toString().padStart(2, "0")}`);
    }
    return labels;
};

const parseUnitInfo = (unitName: string) => {
    if (!unitName) {
        displayUnitName.value = "暂无数据";
        return;
    }
    const battalionPattern = /(.+战区)(.+军)(.+旅)(.+营)/;
    const brigadePattern = /(.+战区)(.+军)(.+旅)/;
    const battalionMatch = unitName.match(battalionPattern);
    const brigadeMatch = unitName.match(brigadePattern);

    if (battalionMatch) {
        const [, theater, army, brigade, battalion] = battalionMatch;
        displayUnitName.value = `${theater}${army}${brigade}`;
    } else if (brigadeMatch) {
        const [, theater, army, brigade] = brigadeMatch;
        displayUnitName.value = `${theater}${army}${brigade}`;
    } else {
        displayUnitName.value = unitName;
    }
};

const renderAllCharts = async () => {
    await nextTick();
    const detail = equips.value.equipMalfDetail;
    const monthLabels = getMonthLabels();

    // 整机图
    if (wholeChartRef.value) {
        const chart = echarts.init(wholeChartRef.value);
        charts.push(chart);
        chart.setOption({
            tooltip: { trigger: "axis" },
            grid: { top: 30, bottom: 30, left: 50, right: 30 },
            xAxis: { type: "category", data: monthLabels, axisLabel: { interval: 0 } },
            yAxis: { type: "value", name: "概率(%)", max: 100 },
            series: [{
                name: "整机故障概率",
                type: "line",
                data: parseProbString(detail.monthProbability),
                smooth: true,
                itemStyle: { color: "#409EFF" },
                lineStyle: { width: 3 },
                areaStyle: { color: "rgba(64,158,255,0.2)" }
            }]
        });
    }

    // 分系统数据准备
    const modelSubsysList = equips.value.equipModel?.subsysList || [];
    const enrichedSubsys = (detail.equipMalfSubsysList || [])
        .map((sys: any) => {
            const match = modelSubsysList.find((s: any) => s.subsysId === sys.subsysId);
            return { ...sys, name: match ? match.subsysName : sys.subsysId, probArray: parseProbString(sys.monthProbability) };
        })
        .sort((a: any, b: any) => b.failureProbability - a.failureProbability);

    const top5Subsys = enrichedSubsys.slice(0, 5);

    // 分系统图
    if (subsysChartRef.value) {
        const chart = echarts.init(subsysChartRef.value);
        charts.push(chart);
        chart.setOption({
            legend: { top: 0 },
            grid: { top: 40, bottom: 30, left: 50, right: 30 },
            xAxis: { type: "category", data: monthLabels, axisLabel: { interval: 0 } },
            yAxis: { type: "value"},
            series: top5Subsys.map((sys: any) => ({
                name: sys.name,
                type: "line",
                data: sys.probArray,
                smooth: true
            }))
        });
    }

    // 部件图数据
    partChartsData.value = top5Subsys.map((sys: any) => {
        const series = [{
            name: `${sys.name} (总体)`,
            type: "line",
            data: sys.probArray,
            lineStyle: { type: "dashed", width: 2 },
            itemStyle: { color: "#303133" },
            symbol: "none",
            z: 10,
        }];

        if (sys.equipMalfPartList) {
            const sysModel = modelSubsysList.find((s: any) => s.subsysId === sys.subsysId);
            const partModelList = sysModel ? sysModel.partList : [];
            sys.equipMalfPartList.forEach((part: any) => {
                const pMatch = partModelList.find((p: any) => p.partId === part.partId);
                series.push({
                    name: pMatch ? pMatch.partName : part.partId,
                    type: "line",
                    data: parseProbString(part.monthProbability),
                    smooth: true
                });
            });
        }
        return { subsysName: sys.name, series };
    });

    await nextTick();
    partChartsData.value.forEach((data, index) => {
        const el = partChartRefs.value[index];
        if (el) {
            const chart = echarts.init(el);
            charts.push(chart);
            chart.setOption({
                tooltip: { trigger: "axis" },
                legend: { type: "scroll", bottom: 0 },
                grid: { top: 20, bottom: 60, left: 40, right: 20 },
                xAxis: { type: "category", data: monthLabels, axisLabel: { interval: 0 } },
                yAxis: { type: "value"},
                series: data.series
            });
        }
    });
};

const loadData = async () => {
    if (!selectedPredictId.value || !selectedEquipCode.value) return;

    const loading = ElLoading.service({ text: "生成报告中..." });
    try {
        const [chartRes, infoRes] = await Promise.all([
            chartDataApi3(true, selectedPredictId.value!),
            equipDetailApi(selectedEquipCode.value)
        ]);

        if (chartRes.code === 200) {
            equips.value = chartRes.data;
            const detail = chartRes.data.equipMalfDetail;

            const predictTime = new Date(detail.predictTime);
            predictMonth.value = predictTime.getMonth() + 2;
            predictYear.value = predictTime.getFullYear();
            if (predictMonth.value > 12) {
                predictMonth.value -= 12;
                predictYear.value += 1;
            }

            parseUnitInfo(chartRes.data.unit?.unitSimpleName);
            fightCode.value = chartRes.data.fightCode || "暂无数据";
            serviceDate.value = chartRes.data.serviceDate;
            manufactureDate.value = chartRes.data.manufactureDate;
            vehicleMotor.value = chartRes.data.equipRecord?.vehicleMotor || "0";
            failureTiming.value = detail.failureTiming;
            failureType.value = detail.failureType;
            workMile.value = infoRes.data.totalWorkMile || "0";
            serviceLimit.value = infoRes.data.serviceLimit;

            const sysList = chartRes.data.equipModel.subsysList || [];
            faultySystems.value = (detail.equipMalfSubsysList || [])
                .filter((s: any) => s.failureProbability > 0.1)
                .map((s: any) => (sysList.find((m: any) => m.subsysId === s.subsysId)?.subsysName || s.subsysId));

            await renderAllCharts();
        }
    } catch (e) {
        ElMessage.error("报告生成失败");
        console.error(e);
    } finally {
        loading.close();
    }
};

const openExportDialog = () => (exportDialogVisible.value = true);

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

        const filename = `精细化故障预测报告_${selectedEquipCode.value}`;

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
    } finally {
        loading.close();
    }
};

onMounted(() => {
    const q = route.query;
    if (q.equipCode) selectedEquipCode.value = q.equipCode as string;
    if (q.predictId) selectedPredictId.value = Number(q.predictId);
    if (selectedEquipCode.value && selectedPredictId.value) loadData();
});

onBeforeUnmount(() => charts.forEach(c => c.dispose()));
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

.info-item {
    margin-bottom: 12px;
    font-size: 13px;

    .label {
        font-weight: bold;
    }
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
    height: 280px;
    margin-bottom: 10px;
}

.part-chart-wrapper {
    width: 100%;
    border: 1px solid #eee;
    padding: 15px;
    margin-bottom: 20px;
    box-sizing: border-box;
}

.sub-chart-header {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 10px;
    color: #409eff;
}

.chart-small {
    width: 100%;
    height: 320px;
}

@media print {
    .hide-print {
        display: none;
    }

    .report-card {
        width: 100%;
        border: none;
    }
}
</style>