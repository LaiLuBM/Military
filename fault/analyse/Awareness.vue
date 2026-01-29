<template>
    <div>
        <el-row :gutter="10">
            <el-col :span="17">
                <el-card>
                    <h2 style="font-size: 20px;">态势-型号装备预测规律看板</h2>
                    <el-row>
                        <el-col :span="12">
                            <div style="margin-top: 20px;width: 100%">
                                <el-radio-group v-model="radio1" size="large" style="width: 100%"
                                    @change="handleRadio1Change">
                                    <el-radio-button label="型号装备预测总量" value="total" />
                                    <el-radio-button label="型号装备故障概率发生趋势" value="trend" />
                                </el-radio-group>
                                <!-- 第一个图表容器 -->
                                <div ref="chart1" style="width: 100%; height: 350px; margin-top: 20px;"></div>
                            </div>
                        </el-col>
                        <el-col :span="12">
                            <div style="margin-top: 20px;width: 100%">
                                <el-radio-group v-model="radio2" size="large" style="width: 100%"
                                    @change="handleRadio2Change">
                                    <el-radio-button label="型号装备故障发生时间" value="time" />
                                    <el-radio-button label="型号装备故障概率发生分系统趋势" value="system" />
                                </el-radio-group>
                                <!-- 第二个图表容器 -->
                                <div ref="chart2" style="width: 100%; height: 350px; margin-top: 20px;"></div>
                            </div>
                        </el-col>
                    </el-row>
                    <el-row style="margin-top: 10px;">
                        <el-col :span="24">
                            <div style="margin-top: 20px;width: 100%" class="three">
                                <el-radio-group v-model="radio3" size="large" style="width: 100%"
                                    @change="handleRadio3Change">
                                    <el-radio-button label="型号装备故障发生趋势" value="faulttrend" />
                                    <el-radio-button label="型号装备故障发生部件比例" value="part" />
                                    <el-radio-button label="型号装备故障发生原因" value="reason" />
                                </el-radio-group>
                                <!-- 第三个图表容器，动态显示单个图表或三个饼图 -->
                                <div style="width: 100%; height: 300px; margin-top: 20px; display: flex; ">
                                    <div ref="chart3" style="width: 100%; height: 100%;"
                                        v-show="radio3 === 'faulttrend'">
                                    </div>
                                    <!-- 为part准备的三张饼图 -->
                                    <div v-show="radio3 === 'part'" style="width: 100%; height: 100%; display: flex;">
                                        <div ref="chart3_part1" style="width:300px; height: 300px;"></div>
                                        <div ref="chart3_part2" style="width:400px; height: 300px;margin-left: 50px;">
                                        </div>
                                        <div ref="chart3_part3" style="width:300px; height: 300px;margin-left: 60px;">
                                        </div>
                                    </div>

                                    <!-- 为reason准备的三张饼图 -->
                                    <div v-show="radio3 === 'reason'" style="width: 100%; height: 100%; display: flex;">
                                        <div ref="chart3_reason1" style="width: 300px; height: 300px;"></div>
                                        <div ref="chart3_reason2" style="width:400px; height: 300px;margin-left: 50px;">
                                        </div>
                                        <div ref="chart3_reason3" style="width:300px; height: 300px;margin-left: 60px;">
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </el-col>
                    </el-row>
                </el-card>
            </el-col>
            <el-col :span="7">
                <el-card>
                    <h2 style="font-size: 20px;">态势-单装故障看板</h2>
                    <div
                        style="width: 100%;margin-top: 20px;background-color: gainsboro;font-size: 20px;padding-left: 10px;">
                        装备基本信息</div>
                    <el-row style="margin-top: 10px;">
                        <el-col :span="12">
                            <div>装备统一编码：{{ equipCode }}</div>
                        </el-col>
                        <el-col :span="12">
                            <div>装备型号：{{ equipSimpleName }}</div>
                        </el-col>
                        <el-col :span="12" style="margin-top: 10px;">
                            <div>所属单位:{{ unitSimpleName }}</div>
                        </el-col>
                        <el-col :span="12" style="margin-top: 10px;">
                            <div>所属分队:{{ unitSimpleName }}</div>
                        </el-col>
                        <el-col :span="12" style="margin-top: 10px;">
                            <div>列装时间：{{ serviceDate }}</div>
                        </el-col>
                        <el-col :span="12" style="margin-top: 10px;">
                            <div>中修间隔期内发动机累计消耗摩托小时：{{ equipRecord.vehicleMotor }}</div>
                        </el-col>
                        <el-col :span="12" style="margin-top: 10px;">
                            <div>大修间隔期内累计行驶里程：{{ equipRecord.workMile }}</div>
                        </el-col>
                    </el-row>
                    <div
                        style="width: 100%;margin-top: 20px;background-color: gainsboro;font-size: 20px;padding-left: 10px;">
                        故障预测</div>
                    <el-row style="margin-top: 10px;">
                        <el-col :span="24">
                            <div>故障类型：{{ equipMalf.failureType || '无数据' }}</div>
                        </el-col>
                    </el-row>
                    <div style="background-color:gainsboro ;width: 100%;height: 3px;margin-top: 10px;"></div>
                    <div ref="chart4" style="width: 100%;margin-top: -20px;height: 300px;"></div>
                    <div
                        style="width: 100%;margin-top: -20px;background-color: gainsboro;font-size: 20px;padding-left: 10px;">
                        送修预测</div>
                    <el-row style="margin-top: 10px;">
                        <el-col :span="4">
                            <el-tag>{{ repairType }}</el-tag>
                        </el-col>
                        <el-col :span="12">
                            <div>未来送修时间：{{ repairTime }}</div>
                        </el-col>
                    </el-row>
                    <ul class="ranking-list">
                        <li class="ranking-item">
                            <span class="rank">1</span>
                            <span class="store-name">历史故障总次数</span>
                            <span class="sales" style="margin-right: 130px;">{{ predictCount }}</span>
                        </li>
                        <li class="ranking-item">
                            <span class="rank">2</span>
                            <span class="store-name">最近一次大修时间</span>
                            <span class="sales">{{ malfDate || '无记录' }}</span>
                        </li>
                        <li class="ranking-item">
                            <span class="rank">3</span>
                            <span class="store-name">故障预测次数（近 3 个月）</span>
                            <span class="sales">{{ predictMalfCount }}</span>
                            <span class="report-link">
                                <a href="#" style="color: #409eff; text-decoration: none;">查看报告</a>
                            </span>
                        </li>
                        <li class="ranking-item">
                            <span class="rank">4</span>
                            <span class="store-name">送修预测次数（近 3 个月）</span>
                            <span class="sales">{{ predictFixCount }}</span>
                            <span class="report-link">
                                <a href="#" style="color: #409eff; text-decoration: none;">查看报告</a>
                            </span>
                        </li>
                        <li class="ranking-item">
                            <span class="rank">5</span>
                            <span class="store-name">诊断次数（近 3 个月）</span>
                            <span class="sales">{{ diagnosisCount }}</span>
                            <span class="report-link">
                                <a href="#" style="color: #409eff; text-decoration: none;">查看报告</a>
                            </span>
                        </li>
                    </ul>
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>

<script setup lang="ts">
import { nextTick, onMounted, reactive, ref } from 'vue';
import { useChart } from '@/hooks/useChart';
import { faultAwarenessApi } from '@/api/fault/fault-awareness';

// Radio bindings
const radio1 = ref('total');
const radio2 = ref('time');
const radio3 = ref('faulttrend');
// 绑定 API 中的计数变量
const predictCount = ref<number | undefined>(undefined);
const malfCount = ref<number | undefined>(undefined);
const malfDate = ref<string | null | undefined>(undefined);
const predictMalfCount = ref<number | undefined>(undefined);
const diagnosisCount = ref<number | undefined>(undefined);
const predictFixCount = ref<number | undefined>(undefined);
const equipCode = ref('')
const unitSimpleName = ref('')
const equipSimpleName = ref('')
const serviceDate = ref('')
const repairTime = ref('')
const repairType = ref('')
// 绑定 equipMalf 数据
const equipMalf = ref<{
    predictTime?: string;
    failureTiming?: string;
    failureProbability?: number;
    failureLocation?: string;
    failureType?: string;
    failureReason?: string;
    monthProbability?: string;
}>({});
const equipRecord = ref<{
    vehicleMotor?: string;
    workMile?: string;
}>({});

// Define types for backend data
interface TypeCountChart {
    type: string;
    count: string;
}

interface TypeMalfTrendChart {
    month: string;
    type: string;
    probability: string;
}

interface TypeMalfChart {
    type: string;
    count: string;
    year: string;
}

interface TypeSubsysMalfChartList {
    type: string;
    subsysList: string;
    count: string;
}

interface TypeServiceDateChart {
    type: string;
    serviceYear: string;
    probability: string;
}

interface TypePartMalfChart {
    type: string;
    partList: string;
    count: string;
}

interface TypeReasonMalfChart {
    type: string;
    reasonList: string;
    count: string;
}

interface ChartData {
    typeCountChart: TypeCountChart | null;
    typeMalfTrendChartList: TypeMalfTrendChart[];
    typeMalfChartList: TypeMalfChart[];
    typeSubsysMalfChartList: TypeSubsysMalfChartList[];
    typeServiceDateChartList: TypeServiceDateChart[];
    typePartMalfChartList: TypePartMalfChart[];
    typeReasonMalfChartList: TypeReasonMalfChart[];
}

// Chart references
const chart1 = ref<HTMLElement | null>(null);
const chart2 = ref<HTMLElement | null>(null);
const chart3 = ref<HTMLElement | null>(null);
const chart4 = ref<HTMLElement | null>(null);
const chart3_part1 = ref<HTMLElement | null>(null);
const chart3_part2 = ref<HTMLElement | null>(null);
const chart3_part3 = ref<HTMLElement | null>(null);
const chart3_reason1 = ref<HTMLElement | null>(null);
const chart3_reason2 = ref<HTMLElement | null>(null);
const chart3_reason3 = ref<HTMLElement | null>(null);

// Store backend data
const chartData = reactive<ChartData>({
    typeCountChart: null,
    typeMalfTrendChartList: [],
    typeMalfChartList: [],
    typeSubsysMalfChartList: [],
    typeServiceDateChartList: [],
    typePartMalfChartList: [],
    typeReasonMalfChartList: [],
});

const param = {
    unitPath: '/11000/11001/',
};

// Fetch backend data
const fetchChartData = async () => {
    try {
        const res = await faultAwarenessApi(param);
        if (res.code === 200) {
            chartData.typeCountChart = res.data?.typeCountChart;
            chartData.typeMalfTrendChartList = res.data?.typeMalfTrendChartList;
            chartData.typeMalfChartList = res.data?.typeMalfChartList;
            chartData.typeSubsysMalfChartList = res.data?.typeSubsysMalfChartList;
            chartData.typeServiceDateChartList = res.data?.typeServiceDateChartList;
            chartData.typePartMalfChartList = res.data?.typePartMalfChartList;
            chartData.typeReasonMalfChartList = res.data?.typeReasonMalfChartList;
            // 更新计数变量
            predictCount.value = res.data?.equipInfo?.predictCount;
            malfCount.value = res.data?.equipInfo?.malfCount;
            malfDate.value = res.data?.equipInfo?.malfDate;
            predictMalfCount.value = res.data?.equipInfo?.predictMalfCount;
            diagnosisCount.value = res.data?.equipInfo?.diagnosisCount;
            predictFixCount.value = res.data?.equipInfo?.predictFixCount;
            equipCode.value = res.data?.equipInfo?.equip?.equipCode;
            serviceDate.value = res.data?.equipInfo?.equip?.serviceDate;
            unitSimpleName.value = res.data?.equipInfo?.equip?.unit?.unitSimpleName;
            equipSimpleName.value = res.data?.equipInfo?.equip?.equipModel?.equipSimpleName;
            repairTime.value = res.data?.equipInfo?.equip?.equipRepair?.repairTime;
            repairType.value = res.data?.equipInfo?.equip?.equipRepair?.repairType;

            // 更新 equipMalf 数据
            equipMalf.value = {
                predictTime: res.data?.equipInfo?.equip?.equipMalf?.predictTime,
                failureTiming: res.data?.equipInfo?.equip?.equipMalf?.failureTiming,
                failureProbability: res.data?.equipInfo?.equip?.equipMalf?.failureProbability,
                failureLocation: res.data?.equipInfo?.equip?.equipMalf?.failureLocation,
                failureType: res.data?.equipInfo?.equip?.equipMalf?.failureType,
                failureReason: res.data?.equipInfo?.equip?.equipMalf?.failureReason,
                monthProbability: res.data?.equipInfo?.equip?.equipMalf?.monthProbability,
            };
            equipRecord.value = {
                workMile: res.data?.equipInfo?.equip?.equipRecord?.workMile,
                vehicleMotor: res.data?.equipInfo?.equip?.equipRecord?.vehicleMotor,

            };
        }
    } catch (error) {
        console.error('Failed to fetch chart data:', error);
    }
};

// Chart 1: 型号装备预测总量 & 型号装备故障概率发生趋势
const getChartData1 = (value: string) => {
    if (value === 'total' && chartData.typeCountChart) {
        return {
            tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } },
            legend: {},
            grid: { left: '3%', right: '20%', containLabel: true },
            xAxis: {
                type: 'value',
                boundaryGap: [0, 0.01],
                name: '最近30天\n预测装备数',
                nameTextStyle: { fontSize: 14, padding: [0, 0, 10, 0] },
            },
            yAxis: {
                type: 'category',
                data: [chartData.typeCountChart.type || '未知型号'],
                name: '型号'
            },
            series: [
                {
                    name: '数量',
                    type: 'bar',
                    data: [parseInt(chartData.typeCountChart.count) || 0],
                },
            ],
        };
    } else if (value === 'trend' && chartData.typeMalfTrendChartList.length) {
        const months = chartData.typeMalfTrendChartList[0].month.split(',');
        return {
            tooltip: { trigger: 'axis' },
            legend: { data: chartData.typeMalfTrendChartList.map((item) => item.type) },
            grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
            toolbox: { feature: { saveAsImage: {} } },
            xAxis: {
                type: 'category',
                boundaryGap: false,
                data: months,
            },
            yAxis: { type: 'value', name: '故障概率' },
            series: chartData.typeMalfTrendChartList.map((item) => ({
                name: item.type,
                type: 'line',
                data: item.probability.split(',').map(Number).map((val: any) => parseFloat(val) * 100 || 0),
            })),
        };
    }
    // 默认配置
    return {
        title: { text: '暂无数据', left: 'center' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: [] },
        yAxis: { type: 'value' },
        series: [{ name: '无数据', type: 'bar', data: [] }],
    };
};

// Chart 2: 型号装备故障发生时间 & 型号装备故障概率发生分系统趋势
const getChartData2 = (value: string) => {
    if (value === 'time' && chartData.typeMalfChartList.length) {
        const currentYear = new Date().getFullYear();
        const years = Array.from({ length: 10 }, (_, i) => currentYear - 9 + i);
        return {
            tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } },
            legend: { data: chartData.typeMalfChartList.map((item) => item.type) },
            grid: { left: '3%', right: '20%', containLabel: true },
            xAxis: {
                type: 'category',
                data: years,
                name: '过去10年实际\n故障发生时间',
                nameTextStyle: { fontSize: 14, padding: [0, 0, 10, 0] },
            },
            yAxis: {
                type: 'value',
                name: '故障次数',
                min: 0,
                interval: 1,
                axisLabel: { formatter: '{value}' },
                nameGap: -1
            },
            series: chartData.typeMalfChartList.map((item) => ({
                name: item.type,
                type: 'bar',
                barGap: 0,
                data: item.count.split(',').map(Number).map(val => val || 0),
            })),
        };
    } else if (value === 'system' && chartData.typeSubsysMalfChartList?.length) {
        const fixedSubsystems = ['电子系统', '底盘系统', '电气系统', '通信系统', '其他系统'];
        const colorPalette = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd'];
        const seriesData = chartData.typeSubsysMalfChartList.map((item, index) => {
            const subsystems = item.subsysList.split(',');
            const counts = item.count.split(',').map(Number);
            const mappedValues = fixedSubsystems.map((fixedSubsys) => {
                if (fixedSubsys === '其他系统') {
                    let otherCount = 0;
                    subsystems.forEach((subsys, idx) => {
                        if (!['电子系统', '底盘系统', '电气系统', '通信系统'].includes(subsys)) {
                            otherCount += counts[idx] || 0;
                        }
                    });
                    return otherCount;
                } else {
                    const idx = subsystems.indexOf(fixedSubsys);
                    return idx !== -1 && idx < counts.length ? counts[idx] : 0;
                }
            });
            return {
                value: mappedValues,
                name: item.type,
                lineStyle: { width: 2, color: colorPalette[index % colorPalette.length] },
                symbol: 'circle',
                symbolSize: 6,
            };
        });
        const maxCount = Math.max(...seriesData.flatMap((series) => series.value), 1);
        return {
            color: colorPalette,
            title: { text: '子系统故障占比', left: 'center', top: 10, textStyle: { fontSize: 16, fontWeight: 'bold' } },
            tooltip: {
                trigger: 'item',
                formatter: (params: any) => {
                    const { name, value } = params.data;
                    return `${name}<br/>${fixedSubsystems
                        .map((subsys, index) => `${subsys}: ${value[index]}`)
                        .join('<br/>')}`;
                },
            },
            legend: {
                data: chartData.typeSubsysMalfChartList.map((item) => item.type), type: 'scroll',
                bottom: 10,
            },
            radar: {
                shape: 'polygon',
                center: ['50%', '50%'],
                radius: '60%',
                indicator: fixedSubsystems.map((subsys) => ({ name: subsys, max: maxCount + 1 })),
                splitNumber: 5,
                splitArea: { show: true, areaStyle: { color: ['#f0f0f0', '#ffffff'] } },
                splitLine: { lineStyle: { color: '#d3d3d3' } },
                axisLine: { lineStyle: { color: '#d3d3d3' } },
                name: { textStyle: { color: '#333', fontSize: 12 } },
            },
            series: [{ name: 'Subsystem Faults', type: 'radar', data: seriesData }],
        };
    }
    return {
        title: { text: '暂无数据', left: 'center' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: [] },
        yAxis: { type: 'value' },
        series: [{ name: '无数据', type: 'bar', data: [] }],
    };
};

// Chart 3: Updated to use typeServiceDateChartList, typePartMalfChartList, and typeReasonMalfChartList
const getChartData3 = (value: string) => {
    if (value === 'faulttrend' && chartData.typeServiceDateChartList?.length) {
        const serviceYears = chartData.typeServiceDateChartList[0].serviceYear.split(',');
        const colorPalette = ['#2f7ed8', '#00c4b4', '#ff851b'];
        const series = chartData.typeServiceDateChartList.map((item, index) => ({
            name: item.type,
            type: 'line',
            smooth: true,
            areaStyle: { opacity: 0.12 },
            emphasis: { focus: 'series' },
            symbol: 'circle',
            symbolSize: 10,
            lineStyle: { width: 2 },
            itemStyle: { color: colorPalette[index % colorPalette.length] },
            data: item.probability.split(',').map((prob: string) => parseFloat(prob) * 100 || 0),
        }));
        const maxProbability = Math.max(...series.flatMap((s) => s.data), 100);
        return {
            tooltip: { trigger: 'axis', axisPointer: { type: 'cross', label: { backgroundColor: '#6a7985' } } },
            legend: { data: chartData.typeServiceDateChartList.map((item) => item.type), top: '0%' },
            toolbox: {},
            grid: { left: '3%', right: '6%', bottom: '3%', containLabel: true },
            xAxis: [{ type: 'category', boundaryGap: false, name: '', data: serviceYears }],
            yAxis: [{ type: 'value', name: '故障率/%', min: 0, max: Math.ceil(maxProbability / 20) * 20, interval: 20 }],
            series,
        };
    } else if (value === 'part' && chartData.typePartMalfChartList?.length) {
        const pieCharts = chartData.typePartMalfChartList.slice(0, 3).map((item) => {
            const parts = item.partList.split(',');
            const counts = item.count.split(',').map(Number);
            const data = parts.map((part, i) => ({
                name: part,
                value: counts[i] || 0,
            }));
            return {
                title: { text: item.type, left: 'center' },
                tooltip: { trigger: 'item' },
                legend: {
                    type: 'scroll',
                    bottom: 10,
                },
                series: [
                    {
                        name: '部件故障',
                        type: 'pie',
                        radius: '50%',
                        center: ['50%', '50%'],
                        data,
                        emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
                        grid: { bottom: '10%' },
                    },
                ],
            };
        });
        while (pieCharts.length < 3) {
            pieCharts.push({
                title: { text: '无数据', left: 'center' },
                tooltip: { trigger: 'item' },
                legend: { orient: 'horizontal', bottom: 'bottom' },
                series: [
                    {
                        name: '部件故障',
                        type: 'pie',
                        radius: '65%',
                        center: ['50%', '40%'],
                        data: [],
                        emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
                    },
                ],
            });
        }
        return pieCharts;
    } else if (value === 'reason' && chartData.typeReasonMalfChartList?.length) {
        const pieCharts = chartData.typeReasonMalfChartList.slice(0, 3).map((item) => {
            const reasons = item.reasonList.split(',');
            const counts = item.count.split(',').map(Number);
            const data = reasons.map((reason, i) => ({
                name: reason,
                value: counts[i] || 0,
            }));
            return {
                title: { text: item.type, left: 'center' },
                tooltip: { trigger: 'item' },
                legend: {
                    type: 'scroll',
                    bottom: 10,
                },
                series: [
                    {
                        name: '故障原因',
                        type: 'pie',
                        radius: ['40%', '65%'],
                        data,
                        emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
                    },
                ],
            };
        });
        while (pieCharts.length < 3) {
            pieCharts.push({
                title: { text: '无数据', left: 'center' },
                tooltip: { trigger: 'item' },
                legend: { orient: 'vertical', bottom: 'bottom' },
                series: [
                    {
                        name: '故障原因',
                        type: 'pie',
                        radius: ['40%', '65%'],
                        data: [],
                        emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
                    },
                ],
            });
        }
        return pieCharts;
    }
    return {
        title: { text: '暂无数据', left: 'center' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: [] },
        yAxis: { type: 'value' },
        series: [{ name: '无数据', type: 'bar', data: [] }],
    };
};

// Chart 4: Keep existing logic

const setChartData4 = async () => {
    try {
        const months = ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '8月', '9月', '10月', '11月', '12月'];
        let probabilities = new Array(12).fill(0);

        if (equipMalf.value && equipMalf.value.monthProbability) {
            probabilities = equipMalf.value.monthProbability.split(',').map((val: string) => {
                const num = parseFloat(val.trim());
                return isNaN(num) ? 0 : num * 100;
            });
        }

        return {
            title: {
                left: 'center',
                top: 10,
                text: '月度故障概率趋势',
                textStyle: { fontSize: 16, fontWeight: 'bold' },
            },
            tooltip: { trigger: 'axis', formatter: '{b}: {c}%' },
            xAxis: { type: 'category', data: months, name: '月份' },
            yAxis: { type: 'value', name: '故障概率 (%)', min: 0, max: 100, interval: 20 },
            series: [
                {
                    name: '故障概率',
                    data: probabilities,
                    type: 'line',
                    smooth: true,
                    areaStyle: { opacity: 0.12 },
                },
            ],
        };
    } catch (error) {
        console.error('Error in setChartData4:', error);
        return {
            title: { text: '暂无数据', left: 'center' },
            xAxis: { type: 'category', data: [] },
            yAxis: { type: 'value' },
            series: [{ name: '无数据', type: 'line', data: [] }],
        };
    }
};

// Initialize and update charts
const { update: update1 } = useChart(chart1, async () => getChartData1(radio1.value));
const { update: update2 } = useChart(chart2, async () => getChartData2(radio2.value));
const { update: update3 } = useChart(chart3, async () => getChartData3(radio3.value));
const { update: update4 } = useChart(chart4, async () => await setChartData4());
const { update: update3_part1 } = useChart(chart3_part1, async () => (getChartData3('part') as any[])[0]);
const { update: update3_part2 } = useChart(chart3_part2, async () => (getChartData3('part') as any[])[1]);
const { update: update3_part3 } = useChart(chart3_part3, async () => (getChartData3('part') as any[])[2]);
const { update: update3_reason1 } = useChart(chart3_reason1, async () => (getChartData3('reason') as any[])[0]);
const { update: update3_reason2 } = useChart(chart3_reason2, async () => (getChartData3('reason') as any[])[1]);
const { update: update3_reason3 } = useChart(chart3_reason3, async () => (getChartData3('reason') as any[])[2]);

// Handle radio1 change
const handleRadio1Change = async (value: string) => {
    const options = getChartData1(value);
    await update1(options);
};

// Handle radio2 change
const handleRadio2Change = async (value: string) => {
    const options = getChartData2(value);
    await update2(options);
};

// Handle radio3 change
const handleRadio3Change = async (value: string) => {
    if (value === 'faulttrend') {
        await update3(getChartData3(value), true);
    } else if (value === 'part') {
        await Promise.all([
            update3_part1((getChartData3(value) as any[])[0], true),
            update3_part2((getChartData3(value) as any[])[1], true),
            update3_part3((getChartData3(value) as any[])[2], true),
        ]);
    } else if (value === 'reason') {
        await Promise.all([
            update3_reason1((getChartData3(value) as any[])[0], true),
            update3_reason2((getChartData3(value) as any[])[1], true),
            update3_reason3((getChartData3(value) as any[])[2], true),
        ]);
    }
};

// 在 onMounted 中改进初始化逻辑
onMounted(async () => {
    // 先获取数据
    await fetchChartData();
    // 等待 DOM 渲染
    await nextTick();

    try {
        // 简化初始化，不做过多的条件检查
        const initPromises = [
            update1(getChartData1(radio1.value), true).catch(err =>
                console.warn('Chart1 init failed:', err)
            ),
            update2(getChartData2(radio2.value), true).catch(err =>
                console.warn('Chart2 init failed:', err)
            ),
        ];

        // 根据当前 radio3 的值初始化对应的图表
        if (radio3.value === 'faulttrend') {
            initPromises.push(
                update3(getChartData3(radio3.value), true).catch(err =>
                    console.warn('Chart3 init failed:', err)
                )
            );
        } else if (radio3.value === 'part') {
            const partCharts = getChartData3(radio3.value) as any[];
            initPromises.push(
                update3_part1(partCharts[0] || getEmptyPieOptions(), true).catch(err =>
                    console.warn('Chart3_part1 init failed:', err)
                ),
                update3_part2(partCharts[1] || getEmptyPieOptions(), true).catch(err =>
                    console.warn('Chart3_part2 init failed:', err)
                ),
                update3_part3(partCharts[2] || getEmptyPieOptions(), true).catch(err =>
                    console.warn('Chart3_part3 init failed:', err)
                )
            );
        } else if (radio3.value === 'reason') {
            const reasonCharts = getChartData3(radio3.value) as any[];
            initPromises.push(
                update3_reason1(reasonCharts[0] || getEmptyPieOptions(), true).catch(err =>
                    console.warn('Chart3_reason1 init failed:', err)
                ),
                update3_reason2(reasonCharts[1] || getEmptyPieOptions(), true).catch(err =>
                    console.warn('Chart3_reason2 init failed:', err)
                ),
                update3_reason3(reasonCharts[2] || getEmptyPieOptions(), true).catch(err =>
                    console.warn('Chart3_reason3 init failed:', err)
                )
            );
        }

        // 初始化 chart4
        const chart4Options = await setChartData4();
        initPromises.push(
            update4(chart4Options, true).catch(err =>
                console.warn('Chart4 init failed:', err)
            )
        );

        await Promise.all(initPromises);
    } catch (error) {
        console.error('Chart initialization failed:', error);
    }
});

// 添加空图表配置的辅助函数
const getEmptyPieOptions = () => ({
    title: { text: '暂无数据', left: 'center' },
    tooltip: { trigger: 'item' },
    series: [{
        name: '无数据',
        type: 'pie',
        radius: '50%',
        data: [],
        emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
    }]
});
</script>

<style lang="less" scoped>
.el-radio-button {
    width: 48%;

    :deep(.el-radio-button__inner) {
        width: 100%;
        padding: 12px 10px;
    }
}

.el-radio-button__inner {
    background-color: #ffffff;
    color: #606266;
}

.el-radio-button__orig-radio:checked+.el-radio-button__inner {
    background-color: #409eff !important;
    color: #ffffff !important;
    box-shadow: -1px 0 0 0 #409eff;
}

.el-radio-button__inner:hover {
    color: #409eff;
}

.three .el-radio-button {
    width: 33%;
}

.el-tag {
    background-color: rgb(200, 245, 245);
    color: rgb(76, 231, 172);
    font-weight: bold;
}

.ranking-list {
    .ranking-item {
        display: flex;
        justify-content: space-between;
        padding: 10px;

        .rank {
            display: inline-block;
            font-weight: bold;
            color: #666;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
        }

        .store-name {
            flex-grow: 1;
            padding: 0 10px;
        }

        .sales {
            color: #666;
            margin-right: 60px;
        }
    }

    .ranking-item:nth-child(odd) {
        background-color: gainsboro;
    }

    margin-top: 20px;
}
</style>