<template>
    <div>
        <el-row :gutter="20">
            <el-col :span="24">
                <div class="top">
                    <el-row>
                        <el-col :span="1">
                            <!-- 添加返回按钮 -->
                            <el-button text @click="goBack" icon="ArrowLeft"
                                style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
                        </el-col>
                        <el-col :span="15">
                            <h2 class="title-margin">精细化故障预测结果详情</h2>
                        </el-col>
                        <el-col :span="8" class="center-content">
                            <el-button type="warning" size="large"
                                @click="viewReport(selectedEquipCode)">查看故障报告</el-button>
                        </el-col>
                    </el-row>
                     <el-row style="margin-top:10px;margin-bottom:10px">
            <el-col :span="24">
              <el-tabs v-model="selectedEquipCode" type="card" @tab-change="onEquipCodeChange"> 
                <el-tab-pane
                v-for="item in equipCodeOptions"
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
                            <div ref="chartRef5" style="width: 65%; height: 200px;"></div>
                        </el-col>
                        <el-col :span="18">
                            <el-row style="margin-top: 15px;">
                                <el-col :span="7">
                                    <div><strong>所属单位：</strong>{{ unitSimpleName }}</div>
                                </el-col>
                                <el-col :span="6">
                                    <div><strong>装备型号：</strong>{{ equips.equipModel.equipSimpleName }}</div>
                                </el-col>
                                <el-col :span="6">
                                    <div><strong>所属分队：</strong>{{ unitSimpleName }}</div>
                                </el-col>
                            </el-row>
                            <el-row style="margin-top: 15px;">
                                <el-col :span="7">
                                    <div><strong>装备统一编码：</strong>
                                        <!-- <el-select v-model="selectedEquipCode"
                                            :placeholder="selectedEquipCode || '请选择装备统一编码'" style="width: 150px"
                                            @change="onEquipCodeChange">
                                            <el-option v-for="item in equipCodeOptions" :key="item.value"
                                                :label="item.label" :value="item.value" />
                                        </el-select> -->
                                        <span style="color:#409EFF;font-weight:bold;">
                                             {{selectedEquipCode|| '请选择装备统一编码' }}
                                        </span>
                                    </div>
                                </el-col>
                                 <el-col :span="7">
                     <div><strong>装备战斗编号：</strong>{{ fightCode}}</div>
                                    </el-col>
                            </el-row>

                            <el-row style="margin-top: 15px;">
                                <el-col :span="7">
                                    <div><strong>列装时间：</strong>{{ serviceDate }}</div>
                                </el-col>
                                <el-col :span="6">
                                    <div><strong>服役日期：</strong>{{ serviceLimit }}</div>
                                </el-col>
                            </el-row>
                            <el-row style="margin-top: 15px;">
                                <el-col :span="7">
                                    <div><strong>大修间隔期内累计行驶里程：</strong>{{ workMile }}公里</div>
                                </el-col>
                                <el-col :span="8">
                                    <div><strong>中修间隔期内发动机累计消耗摩托小时：</strong>{{ vehicleMotor }}小时</div>
                                </el-col>
                                <el-col :span="6">
                                    <div><strong>当年摩托小时消耗限额：</strong>{{ motorLimitYear }}小时</div>
                                </el-col>
                            </el-row>
                            <el-row style="margin-top: 15px;">
                                <el-col :span="6">
                                    <div><strong>出厂日期：</strong>{{ manufactureDate }}</div>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                    <el-divider border-style="double" />
                    <el-row style="margin-top: 15px;">
                        <el-col :span="10">
                            <div style="margin-top: 10px;">
                                <strong>故障发生系统：</strong>{{ faultySystems.join('，') || '无' }}
                            </div>
                        </el-col>
                        <el-col :span="14"></el-col>
                        <el-col :span="6" style="margin-top: 10px;">
                            <div style="margin-top: 10px;"><strong>故障最可能出现的月份：</strong>{{ failureTiming }}</div>
                        </el-col>
                        <el-col :span="18">
                            <div style="margin-top: 15px; text-align: left;">
                                <strong>故障类型：</strong>{{ failureType }}
                            </div>
                        </el-col>
                        <el-col :span="6">
                            <div style="margin-top: 20px; text-align: left;">
                                <strong>故障预测执行时间：</strong>{{ equips.equipMalfDetail.predictTime }}
                            </div>
                        </el-col>
                        <el-col :span="10">
                            <div style="margin-top: 20px; text-align: left;">
                                <strong>数据采样时间：</strong>{{formatTime(equips.equipMalfDetail.predictTime,5)}}
                            </div>
                        </el-col>
                        <el-col :span="3">
                            <el-button type="primary" text @click="viewFaultReport(selectedEquipCode)">查看更多装备信息<el-icon>
                                    <DArrowRight />
                                </el-icon></el-button>
                        </el-col>
                    </el-row>
                </div>
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
                        <el-col :span="18">
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
                            <el-option v-for="item in options" :key="item.value" :label="item.label"
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
import { ref, onMounted, reactive, computed, nextTick, watch, type Ref } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { chartDataApi3 } from '@/api/fault/detail';
import { useChart } from '@/hooks/useChart';
import { ElLoading, ElMessage, type ElTree } from 'element-plus';
import { MonitorTypeApi } from '@/api/fault/monitortype';
import { MonitorChartApi } from '@/api/fault/monitorchart';
import * as echarts from 'echarts';
import { equipDetailApi } from '@/api/fault/equipDetail';
import { DArrowRight } from '@element-plus/icons-vue';

const router = useRouter();
const route = useRoute();
const selectedEquipCode = ref('');
const fightCode = ref('');
const equipCodeOptions = ref<{ value: string; label: string; predictId: number | null }[]>([]);
const selectedOption = ref('整机');
const chartTitle = ref('整机未来12个月故障概率');
const treeData = ref<Tree[]>([]);
const unitSimpleName = ref<string>('');
const serviceDate = ref<string>('');
const failureType = ref<string>('');
const vehicleMotor = ref<string>('');
const manufactureDate = ref<string>('');
const workMile = ref<string>('');
const failureTiming = ref<string>('');
const chartRef5 = ref<HTMLElement | null>(null);
const chartRef6 = ref<HTMLElement | null>(null);
const equipData = ref<any>(null);
const predictMonth = ref<number>(6);
const predictYear = ref<number>(2025);
const motorLimitYear = ref('');
const serviceLimit = ref('');
const chartRef1 = ref<HTMLElement | null>(null);
const chartRef2 = ref<HTMLElement | null>(null);
const chartRef3 = ref<HTMLElement | null>(null);
const chartRef4 = ref<HTMLElement | null>(null);
const selectedPredictId = ref<number | null>(null);
const chartTitles = reactive({
    chart1: '未选择',
    chart2: '未选择',
    chart3: '未选择',
    chart4: '未选择',
});
const value2 = ref<string[]>([]);
const options = ref<{ value: string; label: string; measureUnit: string }[]>([]);
const showDatePicker = ref(false);
const startDate = ref<Date | null>(null);
const endDate = ref<Date | null>(null);
let hideTimeout: number | null = null;

const viewReport = (equipCode: string) => {
    router.push({
        path: '/predict/finingdetailreport',
        query: {
            predictId: selectedPredictId.value,
            equipCode: selectedEquipCode.value,
        }
    });
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

   return `${yyyy}-${mm}-${dd} ${hh}:${mi}:${ss}`
}
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
        { type: '震动传感器', chartRef: chartRef1, update: null as any, colors: { line: '#5470C6', area: '#91CC75', point: '#5470C6' } },
        { type: '电压传感器', chartRef: chartRef2, update: null as any, colors: { line: '#EE6666', area: '#FAC858', point: '#EE6666' } },
        { type: '压力传感器', chartRef: chartRef3, update: null as any, colors: { line: '#73C0DE', area: '#3BA272', point: '#73C0DE' } },
        { type: '温度传感器', chartRef: chartRef4, update: null as any, colors: { line: '#9A60B4', area: '#EA7CCC', point: '#9A60B4' } },
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
}>({
    equipCode: '', equipModelCode: '', unitCode: '', manufactureDate: '', serviceDate: '',
    equipModel: { equipModelCode: '', equipFullname: '', equipSimpleName: '', equipCategory: '', motorLimitYear: '', subsysList: [] },
    unit: { unitCode: '', unitPath: '', parentUnitCode: '', unitFullname: '', unitAddress: '', longitude: '', latitude: '', elevation: '', unitType: '', unitSimpleName: '' },
    equipRecord: { equipRecordId: '', equipCode: '', recordStartDate: '', workMile: '', vehicleMotor: '', fireTimes: '', fireAmmo: '', majorFixTimes: '', mediumFixTimes: '', minorFixTimes: '' },
    equipMalfDetail: { predictId: 0, equipCode: '', predictTime: '', modelAlgorithm: '', modelVersion: '', failureTiming: '', failureProbability: 0, failureLocation: '', failureType: '', failureReason: '', monthProbability: '', userId: '', equipMalfSubsysList: [] }
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

const goBack = () => {
    window.history.back()
};

const safeSessionStorage = {
    set(key: string, value: string): void { try { sessionStorage.setItem(key, value); } catch (e) { console.error('SessionStorage存储失败:', e); } },
    get(key: string): string { try { return sessionStorage.getItem(key) || ''; } catch (e) { console.error('SessionStorage读取失败:', e); return ''; } }
};

const storeSessionData = () => {
    if (selectedEquipCode.value && selectedPredictId.value) {
        safeSessionStorage.set('generalfaultdetailEquipCode', selectedEquipCode.value);
        safeSessionStorage.set('generalfaultdetailPredictId', selectedPredictId.value.toString());
    }
};

const loadMonitorTypeData = async () => {
    if (!selectedEquipCode.value) {
        // ElMessage.warning('装备统一编码未选择，请选择装备后重试');
        return;
    }
    try {
        const response = await MonitorTypeApi(selectedEquipCode.value);
        if (response.code === 200) {
            options.value = response.data?.map((item: any) => ({
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
                equipCode: selectedEquipCode.value || 'EQ001',
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
    // ElMessage.error(`加载参数 ${options.value.find(opt => opt.value === partParamId)?.label || partParamId} 数据失败，请稍后重试`);
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
    chartTitles.chart1 = options.value.find(opt => opt.value === selectedIds[0])?.label || '未选择';
    chartTitles.chart2 = options.value.find(opt => opt.value === selectedIds[1])?.label || '未选择';
    chartTitles.chart3 = options.value.find(opt => opt.value === selectedIds[2])?.label || '未选择';
    chartTitles.chart4 = options.value.find(opt => opt.value === selectedIds[3])?.label || '未选择';
    for (let i = 0; i < defaultTypes.length; i++) {
        const { update, chartRef } = defaultTypes[i];
        const partParamId = selectedIds[i] || null;
        if (partParamId && update) {
            await fetchAndRenderChartData(partParamId, chartRef, update, false);
            // ElMessage.success(`${options.value.find(opt => opt.value === partParamId)?.label || partParamId} 数据加载成功`);
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



const treeRef = ref<InstanceType<typeof ElTree>>();
const defaultProps = { children: 'children', label: 'label' };

const viewFaultReport = (equipCode: string) => {
    router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode);
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

const generateMonthLabels = (startMonth: number, startYear: number): string[] => {
    const labels: string[] = [];
    for (let i = 0; i < 12; i++) {
        const month = (startMonth - 1 + i) % 12 + 1;
        const year = startYear + Math.floor((startMonth - 1 + i) / 12);
        labels.push(`${year}-${month.toString().padStart(2, '0')}`);
    }
    return labels;
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
    if (!selectedPredictId.value) {
        console.warn('No predictId selected for chart5');
        return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
    }
    try {
        const response = await chartDataApi3(true, selectedPredictId.value);
        if (response.code === 200) {
            const chartOptions = reactive({
                series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], avoidLabelOverlap: false, label: { show: false }, emphasis: { label: { show: false } }, itemStyle: { color: (colors: { dataIndex: number }) => ['#007BFF', '#E0E0E0'][colors.dataIndex] }, data: [response.data?.equipMalfDetail?.failureProbability, 1 - response.data?.equipMalfDetail?.failureProbability] }],
                graphic: [{ type: 'group', left: 'center', top: '53%', bounding: 'raw', children: [{ type: 'text', style: { text: '预测故障概率', textAlign: 'center', textVerticalAlign: 'top', fontSize: 13, fill: '#333' } }, { type: 'text', style: { text: `${(response.data?.equipMalfDetail?.failureProbability * 100).toFixed(2)}%`, fontSize: 24, textAlign: 'center', textVerticalAlign: 'bottom', fill: '#FF5E65' } }] }],
                labelLine: { show: false },
            });
            console.log('setChartData5: predictId=', selectedPredictId.value, 'chartOptions=', chartOptions);
            return chartOptions;
        } else {
            // ElMessage.error(`加载 chart5 数据失败: ${response.message}`);
            return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
        }
    } catch (error) {
        console.error('获取 chart5 数据失败:', error);
        // ElMessage.error('加载 chart5 数据失败，请稍后重试');
        return { series: [{ name: '预测故障概率', type: 'pie', radius: ['50%', '70%'], data: [] }] };
    }
};
const { update: updateChartRef5 } = useChart(chartRef5, setChartData5);

const loadData = async () => {
    console.log('loadData called with predictId:', selectedPredictId.value);
    if (!selectedPredictId.value) {
        ElMessage.warning('请先选择有效的预测ID');
        return;
    }
    try {
        const response = await chartDataApi3(true, selectedPredictId.value);
        const resinfo = await equipDetailApi(selectedEquipCode.value);
        motorLimitYear.value = response.data.equipModel?.motorLimitYear || '0';
        workMile.value = resinfo.data.totalWorkMile || '0';
        serviceLimit.value = resinfo.data.serviceLimit;
        fightCode.value=response.data?.fightCode || "暂无数据"
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
            unitSimpleName.value = data?.unit?.unitSimpleName || '';
            serviceDate.value = data?.serviceDate || '';
            failureType.value = data?.equipMalfDetail?.failureType || '';
            manufactureDate.value = data?.manufactureDate || '';
            failureTiming.value = data?.equipMalfDetail?.failureTiming || '';
            storeSessionData();
            await nextTick();
            if (treeData.value.length > 0) await handleNodeClick(treeData.value[0]);
            const chartOptions = await setChartData5();
            await updateChartRef5(chartOptions);
        } else {
            console.error('获取数据失败:', response.message);
            // ElMessage.error(`加载数据失败: ${response.message}`);
        }
    } catch (error) {
        console.error('API 调用失败:', error);
        // ElMessage.error('加载数据失败，请稍后重试');
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

const { update: updateChartRef } = useChart(chartRef6, async () => {
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

const onEquipCodeChange = (newEquipCode: string) => {
    const selectedOption = equipCodeOptions.value.find(option => option.value === newEquipCode);
    if (selectedOption) {
        selectedPredictId.value = selectedOption.predictId;
        console.log('onEquipCodeChange: selectedEquipCode=', newEquipCode, 'selectedPredictId=', selectedPredictId.value);
        loadData();
    } else {
        console.warn('No matching option found for equipCode:', newEquipCode);
    }
};

onMounted(async () => {
    console.log(1111111,formatTime( equips.value.equipMalfDetail.predictTime,5))
    console.log('Finingdetail.vue 挂载，路由参数:', route.query);

    const queryEquipCode = route.query.equipCode as string | undefined;
    const queryPredictId = route.query.predictId ? parseInt(route.query.predictId as string) : null;
    const sessionId = route.query.sessionId as string | undefined;

    if (queryEquipCode) {
        selectedEquipCode.value = queryEquipCode;
    }

    if (sessionId) {
        const selectedEquipments = safeSessionStorage.get('selectedEquipments');
        if (selectedEquipments) {
            try {
                const equipments = JSON.parse(selectedEquipments);
                equipCodeOptions.value = equipments.map((item: any) => ({
                    value: item.equipCode,
                    label: `${item.equipCode}`,
                    predictId: item.predictId || null,
                }));
                if (queryEquipCode) {
                    const matchedEquip = equipments.find((item: any) => item.equipCode === queryEquipCode);
                    if (matchedEquip) {
                        selectedPredictId.value = matchedEquip.predictId || null;
                    } else {
                        const resinfo = await equipDetailApi(queryEquipCode);
                        equipCodeOptions.value.push({
                            value: queryEquipCode,
                            label: queryEquipCode,
                            predictId: resinfo.data.equipMalf?.predictId || null,
                        });
                        selectedPredictId.value = resinfo.data.equipMalf?.predictId || null;
                    }
                } else if (equipments.length > 0) {
                    selectedEquipCode.value = equipments[0].equipCode;
                    selectedPredictId.value = equipments[0].predictId || null;
                }
            } catch (e) {
                console.error('解析 selectedEquipments 失败:', e);
                // ElMessage.error('会话数据解析失败，请重新选择装备');
            }
        }
    } else if (queryEquipCode) {
        selectedEquipCode.value = queryEquipCode;
        try {
            const resinfo = await equipDetailApi(queryEquipCode);
            equipCodeOptions.value = [{
                value: queryEquipCode,
                label: queryEquipCode,
                predictId: resinfo.data.equipMalf?.predictId || null,
            }];
            selectedPredictId.value = queryPredictId !== null ? queryPredictId : resinfo.data.equipMalf?.predictId || null;
        } catch (error) {
            console.error('获取装备详情失败:', error);
            // ElMessage.error('加载装备详情失败，请稍后重试');
            equipCodeOptions.value = [{
                value: queryEquipCode,
                label: queryEquipCode,
                predictId: null,
            }];
        }
    } else {
        equipCodeOptions.value = [];
        // ElMessage.warning('请先选择装备进行预测');
    }

    console.log('初始 selectedEquipCode:', selectedEquipCode.value);
    console.log('初始 selectedPredictId:', selectedPredictId.value);
    console.log('equipCodeOptions:', equipCodeOptions.value);
    storeSessionData();

    if (selectedEquipCode.value) {
        await loadMonitorTypeData();
        if (selectedPredictId.value) {
            await loadData();
        }
        await loadInitialChartData();
    }
});

watch(() => safeSessionStorage.get('selectedEquipments'), (newEquipments) => {
    if (newEquipments) {
        try {
            const equipments = JSON.parse(newEquipments);
            equipCodeOptions.value = equipments.map((item: any) => ({
                value: item.equipCode,
                label: `${item.equipCode}`,
                predictId: item.predictId || null,
            }));
            if (!equipCodeOptions.value.some(option => option.value === selectedEquipCode.value) && equipments.length > 0) {
                selectedEquipCode.value = equipments[0].equipCode;
                const selectedOption = equipCodeOptions.value.find(option => option.value === selectedEquipCode.value);
                selectedPredictId.value = selectedOption ? selectedOption.predictId : null;
            }
            storeSessionData();
            loadMonitorTypeData();
            loadInitialChartData();
        } catch (e) {
            console.error('解析 selectedEquipments 失败:', e);
            // ElMessage.error('会话数据解析失败，请重新选择装备');
        }
    }
}, { immediate: true });

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

.top {
    border: #999494 1px solid;
    padding-left: 30px;
    padding-top: 30px;
    padding-bottom: 20px;
}

.mid {
    margin-top: 20px;
    border: #999494 1px solid;
    padding-left: 30px;
    padding-top: 30px;
    height: 500px;
}

.sub-tree {
    height: 400px;
    overflow-y: auto;
}

.group {
    margin-left: 50px;
}

h2 {
    margin-left: 300px;
    margin-bottom: 30px;
}

.chart {
    width: 100%;
    height: 400px;
    border-left: #999494 1px solid;
}

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