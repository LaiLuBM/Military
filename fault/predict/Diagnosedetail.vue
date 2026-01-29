<template>
    <div>
        <el-row :gutter="20">
            <el-col :span="24">
                <el-card>
                    <div class="topinfo">
                        <el-row>
                            <el-col :span="1">
                                <el-button text @click="goBack" icon="ArrowLeft"
                                    style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
                            </el-col>
                            <el-col :span="15">
                                <h2 class="title-margin">精细化诊断结果详情</h2>
                            </el-col>
                            <el-col :span="8" class="center-content">
                                <el-button type="warning" size="large" @click="viewReport">查看故障报告</el-button>
                            </el-col>
                        </el-row>
                          <el-row style="margin-top:10px;margin-bottom:10px">
                                 <el-col :span="24">
                                     <el-tabs v-model="value" type="card" @tab-change="handleEquipChange"> 
                                         <el-tab-pane
                                         v-for="item in options"
                                            :key="item.value"
                                            :label="item.label"
                                            :name="item.value"
                                            >
                                         </el-tab-pane>
                                        </el-tabs>
                                     </el-col>
                        </el-row>
                         <el-row>
                            <el-col :span="6">
                                <div ref="chartRef5" style="width: 65%;height: 200px;"></div>
                            </el-col>
                            <el-col :span="18">
                                <el-row style="margin-top: 35px;">
                                    <el-col :span="8">
                                <div><strong>装备统一编码：</strong>&nbsp;&nbsp;
                                    <!-- <el-select v-model="value" :placeholder="value || '请选择装备统一编码'" style="width: 240px"
                                        @change="handleEquipChange">
                                        <el-option v-for="item in options" :key="item.value" :label="item.label"
                                            :value="item.value" />
                                    </el-select> -->
                                       <span style="color:#409EFF;font-weight:bold;margin-left:10px">
                      {{value|| '请选择装备统一编码' }}
                    </span>
                                </div>
                                </el-col>
                                <el-col :span="7">
                     <div><strong>装备战斗编号：</strong>{{ fightCode}}</div>
                                    </el-col>
                                    </el-row>
                                <el-row style="margin-top: 15px;">
                                    <el-col :span="8">
                                        <div><strong>装备型号：</strong>{{ equipSimpleName }}</div>
                                    </el-col>
                                    <el-col :span="6">
                                        <div><strong>所属单位：</strong>{{ unitSimpleName }}</div>
                                    </el-col>
                                    <el-col :span="7" style="margin-left: 80px;">
                                        <div><strong>所属分队：</strong>{{ unitSimpleName }}</div>
                                    </el-col>
                                </el-row>
                                <el-row style="margin-top: 15px;">
                                    <el-col :span="8">
                                        <div><strong>列装时间：</strong>{{ serviceDate }}</div>
                                    </el-col>
                                    <el-col :span="6">
                                        <div><strong>服役日期：</strong>{{ serviceLimit }}</div>
                                    </el-col>
                                    <el-col :span="7" style="margin-left: 80px;">
                                        <div><strong>大修间隔期内累计行驶里程：</strong>{{ workMile }}公里</div>
                                    </el-col>
                                </el-row>
                                <el-row style="margin-top: 15px;">
                                    <el-col :span="8">
                                        <div><strong>中修间隔期内发动机累计消耗摩托小时：</strong>{{ vehicleMotor }}小时</div>
                                    </el-col>
                                    <el-col :span="6">
                                        <div style="text-align: left;">
                                            <strong>当年摩托小时消耗限额：</strong>{{ motorLimitYear }}小时
                                        </div>
                                    </el-col>
                                    <el-col :span="6" style="margin-left: 80px;">
                                        <div><strong>出厂日期：</strong>{{ manufactureDate }}</div>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                        <el-divider border-style="double" />
                        <el-table :data="subsysList" style="width: 100%;" border :header-cell-style="headerCellStyle">
                            <el-table-column label="装备分系统" prop="subsysName" align="center" :show-overflow-tooltip="true"></el-table-column>
                            <el-table-column label="装备部件" align="center" :show-overflow-tooltip="true">
                                <template #default="{ row }">
                                    {{row.partList.map((p: any) => p.partName).join(', ')}}
                                </template>
                            </el-table-column>
                            <el-table-column label="故障原因" align="center" :show-overflow-tooltip="true">
                                <template #default="{ row }">
                                    {{[...new Set(row.partList.map((p: any) => p.failureReason || ''))].join(' ')}}
                                </template>
                            </el-table-column>
                        </el-table>
                        <el-row style="margin-top: 15px;">
                            <el-col :span="20">
                                <div style="margin-top: 10px;">单装使用建议：{{ usingSuggestion }}</div>
                            </el-col>
                            <el-col :span="4">
                                <el-button type="primary" text @click="viewFaultReport(equipCode)">查看更多装备信息<el-icon>
                                        <DArrowRight />
                                    </el-icon></el-button>
                            </el-col>
                            <el-col :span="24">
                                <div style="margin-top: 20px;">修理时机建议：{{ fixTiming }}</div>
                            </el-col>
                            <el-col :span="24" style="margin-top: 20px;">
                                <div>异常提示信息：{{ exceptionInfo || '无' }}</div>
                            </el-col>
                            <el-col :span="24">
                                <div style="margin-top: 20px;">
                                    故障诊断执行时间：{{ diagnoseTime }}
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                    <div class="title">
                        <div class="bar fl"></div>
                        <div class="fl describe">该装备最近一次的故障信息</div>
                        <div style="clear: both;"></div>
                    </div>
                    <div class="lastinfo">
                        <el-row style="margin-top: 20px;margin-left: 50px;">
                            <el-col :span="8">
                                <div>部件编码：{{ partId }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>故障部位：{{ malfLocation }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>故障日期：{{ malfDate }}</div>
                            </el-col>
                        </el-row>
                        <el-row style="margin-top: 20px;margin-left: 50px;">
                            <el-col :span="8">
                                <div>故障类型：{{ malfType }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>是否排除故障：{{ rectified ? '是' : '否' }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>填写人代码：{{ fillStaffCode }}</div>
                            </el-col>
                        </el-row>
                        <el-row style="margin-top: 20px;margin-left: 50px;">
                            <el-col :span="8">
                                <div>负责人代码：{{ inchargeStaffCode }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>故障原因：{{ failureReason }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>处理结果：{{ handleResult }}</div>
                            </el-col>
                        </el-row>
                        <el-row style="margin-top: 20px;margin-left: 50px;">
                            <el-col :span="8">
                                <div>故障现象：{{ phenomenon }}</div>
                            </el-col>
                            <el-col :span="8">
                                <div>备注：无</div>
                            </el-col>
                        </el-row>
                    </div>
                </el-card>
            </el-col>
        </el-row>
        <el-row style="margin-top: 10px;">
            <el-col :span="10">
                <el-form class="moduan">
                    <el-form-item label="末端感知数据">
                        <el-select v-model="value2" placeholder="请选择" style="width: 240px" clearable multiple
                            :multiple-limit="4">
                            <el-option v-for="item in options2" :key="item.value" :label="item.label"
                                :value="item.value" />
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
                                <el-date-picker v-model="startDate" type="date" placeholder="选择开始日期"
                                    :style="{ width: '100%' }" />
                            </el-form-item>
                            <el-form-item label="结束日期">
                                <el-date-picker v-model="endDate" type="date" placeholder="选择结束日期"
                                    :style="{ width: '100%' }" />
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
</template>

<script setup lang="ts">
import { ref, reactive, watch, onMounted, type Ref } from "vue";
import { diagnoseDetailApi } from "@/api/fault/diagnosedetail";
import { useRouter, useRoute } from 'vue-router';
import { useChart } from "@/hooks/useChart";
import { DArrowRight } from '@element-plus/icons-vue';
import { MonitorTypeApi } from '@/api/fault/monitortype';
import { MonitorChartApi } from '@/api/fault/monitorchart';
import { ElLoading, ElMessage } from "element-plus";
import * as echarts from 'echarts';
import { equipDetailApi } from "@/api/fault/equipDetail";

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
const failureReason = ref('');
const vehicleMotor = ref('');
const fightCode = ref('');
const manufactureDate = ref('');
const workMile = ref('');
const usingSuggestion = ref('');
const partId = ref('');
const value = ref('');
const inchargeStaffCode = ref("");
const fillStaffCode = ref("");
// const userId = ref('');
const handleResult = ref('');
const malfLocation=ref('')
const phenomenon = ref('');
const reason=ref('')
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

// 修改 options 的数据结构，确保包含 diagnoseId
interface OptionItem {
    value: string;
    label: string;
    diagnoseId: number | null;
}

const options = ref<OptionItem[]>([]);

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

const goBack = () => {
    window.history.back();
};

const viewReport = () => {
    router.push({
        path: '/predict/diagnosereport',
        query: {
            diagnoseId: selectedDiagnoseId.value,
            equipCode: equipCode.value,
        }
    });
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
                //  图表渲染逻辑
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
    // console.error(`加载参数 ${partParamId} 数据失败:`, lastError);
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
        //     message: '数据重放成功',
        //     duration: 2000,
        //     showClose: true
        // });
    } catch (error) {
        console.error('数据重放失败:', error);
        // ElMessage.error({
        //     message: '数据重放失败，请稍后重试',
        //     duration: 3000
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
    router.push("/predict/deviceinformation/Detailinformation?equipCode=" + equipCode);
};

// const options = ref<{ value: string; label: string; diagnoseId: number | null }[]>([]);

const storeSessionData = () => {
    if (equipCode.value && selectedDiagnoseId.value) {
        safeSessionStorage.set('diagnoseDetailEquipments', JSON.stringify([{
            equipCode: equipCode.value,
            diagnoseId: selectedDiagnoseId.value,
        }]));
    }
};


const handleEquipChange = async (selectedEquipCode: string) => {
    console.log('装备选择变更:', selectedEquipCode);

    // 从选项中查找对应的 diagnoseId
    const selectedOption = options.value.find(option => option.value === selectedEquipCode);

    if (selectedOption) {
        equipCode.value = selectedEquipCode;
        selectedDiagnoseId.value = selectedOption.diagnoseId;

        console.log('变更后: equipCode=', equipCode.value, 'diagnoseId=', selectedDiagnoseId.value);

        storeSessionData();

        // 调用 loadData 时传递 diagnoseId 和 equipCode
        if (selectedDiagnoseId.value) {
            await loadData(selectedDiagnoseId.value, selectedEquipCode);
        } else {
            console.warn('选中的装备没有诊断ID');
            ElMessage.warning('该装备暂无诊断数据');
            return;
        }

        // 加载监控类型数据和图表数据
        await loadMonitorTypeData();
        await loadInitialChartData();

    } else {
        console.warn('未找到对应的选项:', selectedEquipCode);
        ElMessage.warning('未找到对应的装备信息');
    }
};


// 修改 loadData 函数，接受 equipCode 参数
const loadData = async (diagnoseId?: number, equipCodeParam?: string) => {
    const idToUse = diagnoseId || selectedDiagnoseId.value;
    const codeToUse = equipCodeParam || equipCode.value;

    if (!idToUse) {
        // console.warn("无有效的诊断ID，跳过 loadData");
        // ElMessage.warning("无有效的诊断ID，无法加载数据");
        return;
    }

    if (!codeToUse) {
        // console.warn("无有效的装备统一编码，跳过 loadData");
        // ElMessage.warning("无有效的装备统一编码，无法加载数据");
        return;
    }

    let loadingInstance: ReturnType<typeof ElLoading.service> | null = null;

    try {
        loadingInstance = ElLoading.service({
            lock: true,
            text: '加载诊断详情中...',
            spinner: 'el-icon-loading'
        });

        console.log('调用 diagnoseDetailApi，参数:', { diagnoseId: idToUse, equipCode: codeToUse });
        const response = await diagnoseDetailApi({ diagnoseId: idToUse });
        console.log('diagnoseDetailApi 响应:', response);

        console.log('调用 equipDetailApi，参数:', codeToUse);
        const resinfo = await equipDetailApi(codeToUse);
        console.log('equipDetailApi 响应:', resinfo);

        // 更新装备基本信息
        motorLimitYear.value = response.data.equipModel?.motorLimitYear || "0";
        workMile.value = resinfo.data.totalWorkMile || "0";
        serviceLimit.value = resinfo.data.serviceLimit || "暂无数据";
        vehicleMotor.value = response.data.equipRecord?.vehicleMotor || "0";
        fightCode.value=response.data.fightCode || "暂无数据"

        if (response.code === 200) {
            const data = response.data;
            console.log('诊断详情数据:', data);

            // 更新装备统一编码
            equipCode.value = codeToUse;

            // 更新单位信息
            unitSimpleName.value = data?.unit?.unitSimpleName || "暂无数据";
            unitAddress.value = data?.unit?.unitAddress || "暂无数据";

            // 更新装备型号信息
            equipSimpleName.value = data?.equipModel?.equipSimpleName || "暂无数据";
            equipModelCode.value = data?.equipModel?.equipModelCode || "暂无数据";

            // 更新服役信息
            serviceDate.value = data?.serviceDate
                ? data.serviceDate 
                : "暂无数据";

            // 更新生产信息
            manufactureDate.value = data?.manufactureDate
                ? data.manufactureDate.split("T")[0]
                : "暂无数据";

            // 更新诊断信息
            diagnoseTime.value = data?.equipDiag?.diagnoseTime
                ? data.equipDiag.diagnoseTime.split("T")[0]
                : "无诊断时间";
            fixTiming.value = data?.equipDiag?.fixTiming || "暂无数据";
            usingSuggestion.value = data?.equipDiag?.usingSuggestion || "无建议";
            exceptionInfo.value = data?.equipDiag?.exceptionInfo || "暂无数据";

            // 更新故障信息
            malfLocation.value = data?.malfRecord?.malfLocation || "暂无数据";
            reason.value = data?.malfRecord?.reason || "暂无数据";

            // 更新故障记录信息
            malfType.value = data?.malfRecord?.malfType || "暂无数据";
            inchargeStaffCode.value = data?.malfRecord?.inchargeStaffCode || "暂无数据";
            fillStaffCode.value = data?.malfRecord?.fillStaffCode || "暂无数据";
            handleResult.value = data?.malfRecord?.handleResult || "暂无数据";
            phenomenon.value = data?.malfRecord?.phenomenon || "暂无数据";
            rectified.value = data?.malfRecord?.rectified ? '是' : '否';
            malfDate.value = data?.malfRecord?.malfDate
                ? data.malfRecord.malfDate.split("T")[0]
                : "暂无数据";

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
                //     const relatedDiagParts = (data.equipDiag?.equipDiagPartList||[]).filter((diag: any) =>diag.subsysId === subsys.subsysId);
                //     const allReasons = [...new Set(relatedDiagParts.map((d:any)=>d.failureReason).filter(Boolean))];
                //     const updatedPartList = subsys.partList.map((part:any)=>{const exactDiog = relatedDiagParts.find((d:any)=>d.partId === part.partId);
                //     return {
                //         ...part,
                //         failureReason:exactDiog?.failureReason || (allReasons.length>0?allReasons.join(';'):""),
                //     };
                //     });
                //     return {
                //         ...subsys,
                //         partList:updatedPartList,
                //         _subsysFailureReason: allReasons.join(';'),
                //     };
                },
            );

            // 获取部件ID（用于显示）
            partId.value = subsysList.value[0]?.partList[0]?.partId || "";

            // 更新饼图数据
            const currentDiagnoseId = diagnoseId || data.equipDiag?.diagnoseId || data.diagnoseIdLatest;
            if (currentDiagnoseId) {
                const chartOptions = await setChartData5(currentDiagnoseId);
                await updateChartRef5(chartOptions);
            }

            storeSessionData();

            // ElMessage.success({
            //     message: '诊断详情加载成功',
            //     duration: 2000,
            //     showClose: true
            // });
        } else {
            console.error("后端返回的数据格式不正确：", response);
            // ElMessage.error(`加载数据失败: ${response.message || '未知错误'}`);
        }
    } catch (error) {
        console.error("获取历史故障诊断数据失败：", error);
        // ElMessage.error({
        //     message: '加载数据失败，请稍后重试',
        //     duration: 3000
        // });
    } finally {
        if (loadingInstance) {
            loadingInstance.close();
        }
    }
};

// 添加这个 watch —— 这是你缺失的核心！
watch(value2, async (newValue) => {
    if (!newValue || newValue.length === 0) {
        chartTitles.chart1 = '未选择';
        chartTitles.chart2 = '未选择';
        chartTitles.chart3 = '未选择';
        chartTitles.chart4 = '未选择';

        // 清空所有图表
        for (const { update } of defaultTypes) {
            if (update) {
                await update({
                    title: { left: 'center', text: '' },
                    series: [{ data: [], type: 'line' }]
                });
            }
        }
        return;
    }

    const selectedIds = newValue.slice(0, 4);

    // 更新标题
    chartTitles.chart1 = options2.value.find(opt => opt.value === selectedIds[0])?.label || '未选择';
    chartTitles.chart2 = options2.value.find(opt => opt.value === selectedIds[1])?.label || '未选择';
    chartTitles.chart3 = options2.value.find(opt => opt.value === selectedIds[2])?.label || '未选择';
    chartTitles.chart4 = options2.value.find(opt => opt.value === selectedIds[3])?.label || '未选择';

    // 重新渲染图表
    for (let i = 0; i < defaultTypes.length; i++) {
        const { update, chartRef } = defaultTypes[i];
        const partParamId = selectedIds[i];
        if (partParamId && update) {
            await fetchAndRenderChartData(partParamId, chartRef, update, false, false);
        } else if (update) {
            // 清空该图表
            await update({
                title: { text: '' },
                series: [{ data: [] }]
            });
        }
    }
}, { immediate: false });
// 在 Diagnosedetail.vue 中添加或修改以下函数


// 修改 onMounted，添加清理逻辑
const triggerUpdate = ref(0);

onMounted(() => {
    initPage();
    // 监听 sessionStorage 变化（哪怕是其他页面改的）
    window.addEventListener('storage', handleStorageChange);
});

onActivated(() => {
    console.log('Diagnosedetail.vue 被激活（keep-alive）');
    initPage();
});

onDeactivated(() => {
    // 可选：离开时清理，防止内存泄漏
});

onBeforeUnmount(() => {
    window.removeEventListener('storage', handleStorageChange);
});

const handleStorageChange = (e: StorageEvent) => {
    if (e.key === 'refinedDiagnosisEquipments') {
        triggerUpdate.value++;
        nextTick(() => initPage());
    }
};

// 核心初始化函数（被 onMounted / onActivated / storage 事件共用）
const initPage = async () => {
    console.log('initPage 执行，当前路由参数:', route.query);

    const queryEquipCode = route.query.equipCode as string | undefined;
    const queryDiagnoseId = route.query.diagnoseId ? parseInt(route.query.diagnoseId as string) : null;
    const shouldClearCache = route.query.clearCache === 'true';

    // 重置状态
    options.value = [];
    value.value = '';
    equipCode.value = '';
    selectedDiagnoseId.value = null;

    let equipments: any[] = [];

    // 情况1：本次是全新批量诊断（带 clearCache=true）→ 优先用最新 sessionStorage
    if (shouldClearCache) {
        const stored = safeSessionStorage.get('refinedDiagnosisEquipments');
        if (stored) {
            try {
                equipments = JSON.parse(stored);
            } catch (e) {
                console.error('解析 refinedDiagnosisEquipments 失败', e);
            }
        }
        // 如果 session 里没数据，兜底用路由参数（极少发生）
        if (equipments.length === 0 && queryEquipCode) {
            equipments = [{ equipCode: queryEquipCode, diagnoseId: queryDiagnoseId }];
        }
    }
    // 情况2：从历史记录或其他页面点进来 → 只用路由参数
    else if (queryEquipCode) {
        equipments = [{ equipCode: queryEquipCode, diagnoseId: queryDiagnoseId }];
    }
    // 情况3：异常情况，尝试读取旧 session（基本不会走到）
    else {
        const stored = safeSessionStorage.get('refinedDiagnosisEquipments');
        if (stored) {
            try {
                equipments = JSON.parse(stored);
            } catch (e) { }
        }
    }

    // 关键：完全覆盖，不合并！
    if (equipments.length > 0) {
        options.value = equipments.map(item => ({
            value: item.equipCode,
            label: item.equipCode,
            diagnoseId: item.diagnoseId || null
        }));

        // 默认选中第一台（最新诊断批次的第一台）
        const first = equipments[0];
        value.value = first.equipCode;
        equipCode.value = first.equipCode;
        selectedDiagnoseId.value = first.diagnoseId || null;
    }

    // 重新加载数据
    if (selectedDiagnoseId.value && equipCode.value) {
        await loadData(selectedDiagnoseId.value, equipCode.value);
        await loadMonitorTypeData();
        await loadInitialChartData();
    } else if (equipCode.value) {
        await loadMonitorTypeData();
        await loadInitialChartData();
    }
};
// 监听路由参数变化，确保路由变化时能及时更新
watch(() => route.query, async (newQuery) => {
    const newEquipCode = newQuery.equipCode as string | undefined;
    const newDiagnoseId = newQuery.diagnoseId ? parseInt(newQuery.diagnoseId as string) : null;

    if (newEquipCode && newEquipCode !== value.value) {
        console.log('路由参数变化，更新装备数据:', newEquipCode, newDiagnoseId);

        value.value = newEquipCode;
        equipCode.value = newEquipCode;
        selectedDiagnoseId.value = newDiagnoseId;

        // 更新选项，确保路由参数装备在第一位
        if (!options.value.some(opt => opt.value === newEquipCode)) {
            options.value.unshift({
                value: newEquipCode,
                label: newEquipCode,
                diagnoseId: newDiagnoseId
            });
        }

        storeSessionData();
        loadData()

        if (selectedDiagnoseId.value) {
            await loadData(selectedDiagnoseId.value);
        }
        if (equipCode.value) {
            await loadMonitorTypeData();
            await loadInitialChartData();
        }
    }
}, { immediate: true });

const setChartData5 = async (diagnoseId: number) => {
    if (!selectedDiagnoseId.value) {
        console.warn('No diagnoseId selected for chart5');
        return { series: [{ type: 'pie', radius: ['50%', '70%'], data: [] }] };
    }
    try {
        const response = await diagnoseDetailApi({ diagnoseId: selectedDiagnoseId.value });
        if (response.code === 200) {
            const diagnoseResult = response.data.equipDiag?.diagnoseResult;

            // 判断是否为装备正常
            const displayText = diagnoseResult === '装备正常' ? '装备技术状况正常' : (diagnoseResult || '无诊断结果');

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
                                text: displayText,
                                fontSize: 24,
                                textAlign: 'center',
                                textVerticalAlign: 'bottom',
                                fill: '#FF5E65',
                            },
                        }
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
.center-content {
    display: flex;
    justify-content: center;
    align-items: center;
}

.bold-text {
    font-weight: bold;
}

.left-align {
    text-align: left;
}

.chart-container {
    width: 100%;
}

.chart-placeholder {
    display: flex;
    justify-content: center;
    align-items: center;
    color: #909399;
    font-size: 14px;
    height: 400px;
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

.bar {
    width: 10px;
    height: 15px;
    background-color: #409eff;
}

.describe {
    color: #409eff;
    font-weight: bold;
    margin-left: 5px;
}

.title {
    line-height: 15px;
    margin-top: 40px;
    margin-bottom: 15px;
    margin-left: 30px;
}

.fl {
    float: left;
}

.topinfo,
.lastinfo {
    border: #909399 1px solid;
    padding: 10px;
}

:deep(.el-tabs__header){
  margin-bottom: 0;
}
:deep(.el-tabs__item){
  font-weight: bold;
}
:deep(.el-tabs__item.is-active){
  color: #409EEF;
  background-color: #f5f7fa;
}
</style>