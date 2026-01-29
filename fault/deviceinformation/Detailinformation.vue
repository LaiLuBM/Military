```vue
<template>
    <el-card class="main-card">
        <el-row :gutter="20">
            <!-- 基础信息 -->
            <el-col :span="24">
                <el-card class="info-card">
                    <el-row>
                        <el-col :span="1">
                            <!-- 返回按钮 -->
                            <el-button text icon="Back" style="font-size: 30px; padding: 0; margin-left: 15px;"
                                @click="goBack" />
                        </el-col>
                        <el-col :span="20">
                            <h2>装备详情</h2>
                        </el-col>
                    </el-row>
                    <el-row>
                        <el-col :span="5">
                            <div class="section-title">基础信息</div>
                        </el-col>
                        <el-col :span="15" />
                        <el-col :span="4">
                            <el-dropdown>
                                <el-button class="section-title" style="border: none;">
                                    更多业务活动信息<el-icon>
                                        <ArrowRight />
                                    </el-icon>
                                </el-button>
                                <template #dropdown>
                                    <el-dropdown-menu class="custom-dropdown-menu">
                                        <el-dropdown-item @click="usage">单装使用消耗</el-dropdown-item>
                                        <el-dropdown-item @click="maintain">单装保养记录</el-dropdown-item>
                                        <el-dropdown-item @click="fault">单装故障记录</el-dropdown-item>
                                        <el-dropdown-item @click="repair">单装修理记录</el-dropdown-item>
                                    </el-dropdown-menu>
                                </template>
                            </el-dropdown>
                        </el-col>
                    </el-row>
                    <el-row :gutter="20">
                        <el-col :span="24">
                            <div class="info-container">
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">所属单位</span>
                                        </div>
                                        <el-input v-model="deviceInfo.unitFullname" readonly placeholder="请输入所属单位"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">装备型号</span>
                                        </div>
                                        <el-input v-model="deviceInfo.equipSimpleName" readonly placeholder="请输入装备型号"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">装备统一编码</span>
                                        </div>
                                        <el-input v-model="deviceInfo.equipCode" readonly placeholder="请输入装备统一编码"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">所属分队</span>
                                        </div>
                                        <el-input v-model="deviceInfo.unitFullname" readonly placeholder="请输入所属单位"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">是否精细化装备</span>
                                        </div>
                                        <el-radio-group v-model="deviceInfo.isRefinedEquipment" disabled>
                                            <el-radio label="是">是</el-radio>
                                            <el-radio label="否">否</el-radio>
                                        </el-radio-group>
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">列装时间</span>
                                        </div>
                                        <el-input v-model="deviceInfo.serviceDate" readonly placeholder="请输入列装时间"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">服役日期</span>
                                        </div>
                                        <el-input v-model="deviceInfo.servicePeriod" readonly placeholder="请输入服役日期"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">当年摩托小时消耗限额</span>
                                        </div>
                                        <el-input v-model="deviceInfo.motorLimitYear" readonly placeholder="请输入当年摩托小时消耗限额"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">出厂日期</span>
                                        </div>
                                        <el-input v-model="deviceInfo.manufactureDate" readonly placeholder="请输入出厂日期"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">装备剩余里程</span>
                                        </div>
                                        <el-input v-model="deviceInfo.residueMile" readonly placeholder="请输入装备剩余寿命"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">装备战备储备标准</span>
                                        </div>
                                        <el-input v-model="deviceInfo.inspectionStandard" readonly
                                            placeholder="请输入装备检验标准" class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">年度修理指标</span>
                                        </div>
                                        <el-input v-model="deviceInfo.repairCategory" readonly placeholder="请输入年度修理指标"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                            </div>
                        </el-col>
                    </el-row>
                    <div class="section-title">技术状况及末端感知数据对比</div>
                    <el-row>
                        <el-col :span="6">
                            <div style="margin: 10px 5px;"><strong>技术状况</strong></div>
                        </el-col>
                        <el-col :span="18">
                            <div class="progress-container">
                                <div class="progress-item">
                                    <span class="label">范围内</span>
                                    <div class="bar blue" />
                                    <div>|</div>
                                </div>
                                <div class="progress-item">
                                    <span class="label">边缘范围内10%</span>
                                    <div class="bar green" />
                                </div>
                                <div>|</div>
                                <div class="progress-item">
                                    <span class="label">边缘范围外10%</span>
                                    <div class="bar orange" />
                                </div>
                                <div>|</div>
                                <div class="progress-item">
                                    <span class="label">范围外</span>
                                    <div class="bar red" />
                                </div>
                            </div>
                        </el-col>
                    </el-row>
                    <el-row>
                        <el-col :span="24">
                            <el-table :data="data" border :header-cell-style="headerCellStyle">
                                <el-table-column label="实际值" prop="real" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column label="标准上限值" prop="standard" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column label="偏离" align="center" :show-overflow-tooltip="true">
                                    <template #default="{ row }">
                                        <div style="display: flex; height: 100%;">
                                            <div
                                                style="flex: 1; text-align: center; border-right: 1px solid #ebeef5; display: flex; align-items: center; justify-content: center;">
                                                {{ row.departure.status }}
                                            </div>
                                            <div
                                                style="flex: 1; text-align: center; display: flex; align-items: center; justify-content: center;">
                                                <el-tag :type="getTagType(row.departure.status)" effect="dark" round>
                                                    {{ row.departure.value }}
                                                </el-tag>
                                            </div>
                                        </div>
                                    </template>
                                </el-table-column>
                            </el-table>
                        </el-col>
                    </el-row>
                    <el-row :gutter="20">
                        <el-col :span="24">
                            <el-table :data="tableData" border :header-cell-style="headerCellStyle"
                                style="width: 100%; margin-top: 20px" :span-method="objectSpanMethod">
                                <el-table-column prop="system" label="分系统" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column prop="component" label="部件" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column prop="parameter" label="运行参数项" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column prop="actualValue" label="实际值" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column prop="referenceValue" label="参考值" align="center"  :show-overflow-tooltip="true"/>
                                <el-table-column prop="deviation" label="偏离" align="center" :show-overflow-tooltip="true">
                                    <template #default="{ row }">
                                        <div style="display: flex; height: 100%;">
                                            <div
                                                style="flex: 1; text-align: center; border-right: 1px solid #ebeef5; display: flex; align-items: center; justify-content: center;">
                                                {{ row.deviation.status }}
                                            </div>
                                            <div
                                                style="flex: 1; text-align: center; display: flex; align-items: center; justify-content: center;">
                                                <el-tag :type="getTagType(row.deviation.status)" effect="dark" round>
                                                    {{ row.deviation.value }}
                                                </el-tag>
                                            </div>
                                        </div>
                                    </template>
                                </el-table-column>
                            </el-table>
                        </el-col>
                    </el-row>
                    <!-- 单装履历信息 -->
                    <div class="section-title">单装履历信息</div>
                    <el-row :gutter="20">
                        <el-col :span="24">
                            <div class="info-container">
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">单装履历记录ID</span>
                                        </div>
                                        <el-input v-model="deviceInfo.faultRecordId" readonly placeholder="请输入履历记录ID"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">装备唯一识别码(单装流水)</span>
                                        </div>
                                        <el-input v-model="deviceInfo.historyRecordId" readonly placeholder="请输入装备唯一识别码"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">装属分队</span>
                                        </div>
                                        <el-input v-model="deviceInfo.unitFullname" readonly placeholder="请输入装履分队"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">战斗编号</span>
                                        </div>
                                        <el-input v-model="deviceInfo.equipSimpleName" readonly placeholder="请输入战斗编号"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">单装负责人代码</span>
                                        </div>
                                        <el-input v-model="deviceInfo.partCode" readonly placeholder="请输入单装负责人代码"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">启用时间</span>
                                        </div>
                                        <el-input v-model="deviceInfo.sendRepairDate" readonly placeholder="请输入启用时间"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">大修间隔期内累计行驶里程</span>
                                        </div>
                                        <el-input v-model="deviceInfo.workmile" readonly placeholder="请输入大修间隔期内累计行驶里程"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">摩托小时</span>
                                        </div>
                                        <el-input v-model="deviceInfo.motorHour" readonly placeholder="请输入摩托小时"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">加电时间</span>
                                        </div>
                                        <el-input v-model="deviceInfo.faultResult" readonly placeholder="请输入加电时间"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">架设次数</span>
                                        </div>
                                        <el-input v-model="deviceInfo.faultCategory" readonly placeholder="请输入架设次数"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">发射次数</span>
                                        </div>
                                        <el-input v-model="deviceInfo.responsibleCode" readonly placeholder="请输入发射次数"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">发射实弹</span>
                                        </div>
                                        <el-input v-model="deviceInfo.remark" readonly placeholder="请输入发射实弹"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                                <el-row :gutter="10">
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">大修次数</span>
                                        </div>
                                        <el-input v-model="deviceInfo.majorFixTimes" readonly placeholder="请输入大修次数"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">中修次数</span>
                                        </div>
                                        <el-input v-model="deviceInfo.mediumFixTimes" readonly placeholder="请输入中修次数"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">小修次数</span>
                                        </div>
                                        <el-input v-model="deviceInfo.minorFixTimes" readonly placeholder="请输入小修次数"
                                            class="input-box" />
                                    </el-col>
                                    <el-col :span="6">
                                        <div class="info-item">
                                            <span class="info-label">填发单位代码</span>
                                        </div>
                                        <el-input v-model="deviceInfo.formUnitCode" readonly placeholder="请输入填表单位代码"
                                            class="input-box" />
                                    </el-col>
                                </el-row>
                            </div>
                        </el-col>
                    </el-row>
                    <!-- 分系统及部件 -->
                    <!-- 分系统及部件 -->
                    <div class="section-title">分系统及部件</div>
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-timeline style="max-width: 600px; margin-left: 0">
                                <el-timeline-item v-for="(subsys, index) in subsystemActivities" :key="subsys.subsysId"
                                    :type="getTimelineType(index)" :size="'large'">
                                    {{ getSubsystemContent(subsys) }}
                                </el-timeline-item>
                            </el-timeline>
                        </el-col>
                    </el-row>
                    <!-- 末端感知数据 -->
                    <div v-show="isde">
                        <div class="section-title">末端感知数据</div>
                        <el-row :gutter="20">
                            <el-col :span="10">
                                <el-form class="moduan">
                                    <el-form-item label="末端感知数据">
                                        <el-select v-model="value2" placeholder="请选择" style="width: 240px" clearable
                                            multiple :multiple-limit="4">
                                            <el-option v-for="item in options" :key="item.value" :label="item.label"
                                                :value="item.value" />
                                        </el-select>
                                    </el-form-item>
                                </el-form>
                            </el-col>
                            <el-col :span="10"></el-col>
                            <el-col :span="4">
                                <div class="replay-container" @mouseleave="handleContainerLeave"
                                    @mouseenter="handleContainerEnter">
                                    <el-button type="primary" size="large">重放数据</el-button>
                                    <el-card class="date-picker-card" v-show="showDatePicker">
                                        <el-form>
                                            <el-form-item label="开始日期">
                                                <el-date-picker v-model="startDate" type="date" placeholder="选择开始日期"
                                                    style="width: 100%" />
                                            </el-form-item>
                                            <el-form-item label="结束日期">
                                                <el-date-picker v-model="endDate" type="date" placeholder="选择结束日期"
                                                    style="width: 100%" />
                                            </el-form-item>
                                            <el-form-item>
                                                <el-button type="primary" size="small"
                                                    @click="replayData">确定</el-button>
                                                <el-button size="small" @click="showDatePicker = false">取消</el-button>
                                            </el-form-item>
                                        </el-form>
                                    </el-card>
                                </div>
                            </el-col>
                        </el-row>
                        <el-row :gutter="20">
                            <el-col :span="12">
                                <div class="chart-container">
                                    <div class="chart-title">{{ chartTitles.chart1 }}</div>
                                    <div ref="chartRef1" class="chart-placeholder" />
                                </div>
                            </el-col>
                            <el-col :span="12">
                                <div class="chart-container">
                                    <div class="chart-title">{{ chartTitles.chart2 }}</div>
                                    <div ref="chartRef2" class="chart-placeholder" />
                                </div>
                            </el-col>
                        </el-row>
                        <el-row :gutter="20">
                            <el-col :span="12">
                                <div class="chart-container">
                                    <div class="chart-title">{{ chartTitles.chart3 }}</div>
                                    <div ref="chartRef3" class="chart-placeholder" />
                                </div>
                            </el-col>
                            <el-col :span="12">
                                <div class="chart-container">
                                    <div class="chart-title">{{ chartTitles.chart4 }}</div>
                                    <div ref="chartRef4" class="chart-placeholder" />
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                </el-card>
            </el-col>
        </el-row>
    </el-card>
</template>

<script lang="ts" setup>
import { ref, onMounted, reactive, watch, nextTick, type Ref, computed } from 'vue';
import { equipDetailApi } from '@/api/fault/equipDetail';
import { useRoute, useRouter } from 'vue-router';
import { useChart } from '@/hooks/useChart';
import { MonitorTypeApi } from '@/api/fault/monitortype';
import { MonitorChartApi } from '@/api/fault/monitorchart';
import { ElLoading, ElMessage } from 'element-plus';
import { techConditionApi } from '@/api/fault/techCondition';
import * as echarts from 'echarts';

interface Colors {
    line: string;
    area: string;
    point: string;
}

interface User {
    system: string;
    component: string;
    parameter: string;
    actualValue: string;
    referenceValue: string;
    deviation: { status: string; value: string };
}


interface SubsystemActivity {
    subsysId: string;
    subsysName: string;
    subsysSpec: string;
    coreSubsys: boolean;
    partList: {
        partId: string;
        partName: string;
        partSpec: string;
        corePart: boolean;
        maint: boolean;
    }[];
}

const router = useRouter();
const route = useRoute();
let isde = ref()


const equipCode = ref<string>('100434'); // Updated to match backend data
const deviceInfo = ref({
    equipCode: '',
    equipSimpleName: '',
    unitFullname: '',
    unitAddress: '',
    isRefinedEquipment: '',
    serviceDate: '',
    servicePeriod: '',
    fireTimes: '',
    manufactureDate: '',
    motorLimitYear: '',
    inspectionStandard: '',
    repairCategory: '',
    faultRecordId: '',
    historyRecordId: '',
    partCode: '',
    sendRepairDate: '',
    equipCategory: '',
    motorHour: '',
    faultResult: '',
    faultCategory: '',
    responsibleCode: '',
    remark: '',
    majorFixTimes: '',
    mediumFixTimes: '',
    minorFixTimes: '',
    formUnitCode: '',
    workmile: '',
    residueMile: '',
    equipModel: {
        subsysList: []
    }
});

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

const options = ref<{ value: string; label: string; measureUnit: string }[]>([]);
const value2 = ref<string[]>([]);
const showDatePicker = ref(false);
const startDate = ref<Date | null>(null);
const endDate = ref<Date | null>(null);
let hideTimeout: number | null = null;

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
    if (equipCode.value) safeSessionStorage.set('detailInforEquipCode', equipCode.value);
};

// New function to load and transform techCondition data
const loadTechConditionData = async () => {
    try {
        const res = await techConditionApi(equipCode.value);
        if (res.code === 200) {
            tableData.value = res?.data?.subsysList.flatMap((subsys: any) =>
                subsys.partList.flatMap((part: any) =>
                    part.paramList.map((param: any) => ({
                        system: subsys.subsysName,
                        component: part.partName,
                        parameter: param.paramName,
                        actualValue: param.actualValue !== null ? param.actualValue.toString() : '无数据',
                        referenceValue: param.valueScope,
                        deviation: {
                            status: param.deviationRate,
                            value: param.deviationRate.includes('范围内') ? '√' : param.deviationRate === '无数据' ? '-' : '!',
                        },
                    }))
                )
            );
        } else {
            ElMessage.warning('该装备暂无技术状况数据 ' + res.message);
        }
    } catch (error) {
        console.error('获取技术状况数据失败:', error);
        ElMessage.warning('该装备暂无技术状况数据');
    }
};

// 修改 loadData 函数，确保 equipModel 数据正确加载
const loadData = async () => {
    try {
        const res = await equipDetailApi(equipCode.value);
        if (res.code === 200) {
            deviceInfo.value = {
                equipCode: res.data?.equipCode || '暂无数据',
                equipSimpleName: res.data?.equipModel?.equipSimpleName || '暂无数据',
                unitFullname: res.data?.unit?.unitFullname || '暂无数据',
                unitAddress: res.data?.unit?.unitAddress || '暂无数据',
                motorHour:res.data?.equipRecord?.vehicleMotor? `${res.data?.equipRecord?.vehicleMotor}小时` : '暂无数据',
                serviceDate:res.data?.serviceDate? `${res.data?.serviceDate}` : '暂无数据',
                manufactureDate: res.data?.manufactureDate || '暂无数据',
                equipCategory: res.data?.equipModel?.equipCategory || '暂无数据',
                majorFixTimes:res.data?.equipRecord?.majorFixTimes? `${res.data?.equipRecord?.majorFixTimes}次` : '暂无数据',
                mediumFixTimes: res.data?.equipRecord?.mediumFixTimes?`${res.data?.equipRecord?.mediumFixTimes}次` : '暂无数据',
                minorFixTimes: res.data?.equipRecord?.minorFixTimes?`${res.data?.equipRecord?.minorFixTimes}次` : '暂无数据',
                motorLimitYear: res.data?.equipModel?.motorLimitYear?`${res.data.equipModel.motorLimitYear}小时` : "暂无数据",
                isRefinedEquipment: res.data?.isDetailedEquip ? '是' : '否',
                servicePeriod: res.data?.serviceLimit || '暂无数据',
                inspectionStandard: res.data?.readinessStandard || '暂无数据',
                fireTimes: res.data?.equipRecord?.fireTimes?`${res.data?.equipRecord?.fireTimes}公里` : '暂无数据',
                faultRecordId: res.data?.equipRecord?.equipRecordId || '暂无数据',
                historyRecordId: res.data?.equipCode || '暂无数据',
                faultCategory: res.data?.usageRecord?.erectTimes?`${res.data?.usageRecord?.erectTimes}次` : '暂无数据',
                partCode: res.data?.partCode || '暂无数据',
                faultResult: res.data?.usageRecord?.elecHour?`${res.data?.usageRecord?.elecHour}小时` : '暂无数据',
                responsibleCode:res.data?.usageRecord?.fireTimes? `${res.data?.usageRecord?.fireTimes}次` : '暂无数据',
                remark: res.data?.usageRecord?.fireAmt?`${res.data?.usageRecord?.fireAmt}发` : '暂无数据',
                repairCategory: res.data?.annualRepairIndicator || '暂无数据',
                sendRepairDate: res.data?.equipRecord?.recordStartDate || '暂无数据',
                formUnitCode: res.data?.unit?.unitCode || '暂无数据',
                workmile:res.data?.usageRecord?.workMile? `${res.data?.usageRecord?.workMile}公里` : '暂无数据',
                residueMile: res.data?.usageRecord?.residueMile?`${res.data?.usageRecord?.residueMile}公里` : '暂无数据',
                equipModel: res.data?.equipModel || null,
            };

            if (deviceInfo.value.isRefinedEquipment === "是") {
                isde.value = true;
            } else {
                isde.value = false;
            }

            storeSessionData();
            console.log('分系统数据:', deviceInfo.value.equipModel?.subsysList);
        } else {
            ElMessage.error('获取数据失败: ' + res.message);
        }
    } catch (error) {
        console.error('API 调用失败:', error);
    }
};

const handleContainerEnter = () => {
    if (hideTimeout) clearTimeout(hideTimeout);
    showDatePicker.value = true;
};

const handleContainerLeave = () => {
    hideTimeout = setTimeout(() => {
        showDatePicker.value = false;
        hideTimeout = null;
    }, 120000);
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
                partParamId: partParamId,
            };

            if (useSelectedDates && startDate.value && endDate.value) {
                params.startTimeStr = formatDateToString(startDate.value);
                params.endTimeStr = formatDateToString(endDate.value);
            } else if (isReplay && startDate.value && endDate.value) {
                params.startTimeStr = formatDateToString(startDate.value);
                params.endTimeStr = formatDateToString(endDate.value);
            }

            const response = await MonitorChartApi(params);
            if (response.code === 200) {
                const timeStr = response.data?.timeStr.split(',').map((time: string) => time.trim());
                const values = response.data?.valueStr.split(',').map((value: string) => parseFloat(value.trim()));
                const typeConfig = defaultTypes.find(t => t.chartRef === chartRef) || defaultTypes[0];
                const colors = typeConfig?.colors || { line: '#ee6666', area: '#ff9999', point: '#ee6666' };
                const paramData = options.value.find(opt => opt.value === partParamId);
                const displayName = response.data?.paramName || paramData?.label || '未知参数';
                const measureUnit = response.data?.unit || paramData?.measureUnit || '值';

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
                        },
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
                            },
                        },
                    },
                    yAxis: {
                        type: 'value',
                        boundaryGap: [0, '100%'],
                        name: `${displayName} (${measureUnit})`,
                        axisLine: { lineStyle: { color: colors.line } },
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
                                { offset: 1, color: colors.line },
                            ]),
                            opacity: 0.5,
                        },
                        data: values,
                    }],
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
    try {
        await Promise.all(
            defaultTypes.map(({ chartRef, update }, index) => {
                if (update && value2.value[index]) {
                    return fetchAndRenderChartData(value2.value[index], chartRef, update, false, false);
                }
                return Promise.resolve();
            })
        );
    } catch (error) {
        console.error('加载初始图表数据失败:', error);
    }
};

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
    const loadingInstance = ElLoading.service({ lock: true, text: '数据重放中...', spinner: 'el-icon-loading' });
    try {
        const selectedIds = value2.value.slice(0, 4);
        await Promise.all(
            selectedIds.map((partParamId, index) => {
                const target = defaultTypes[index];
                if (target?.update) {
                    return fetchAndRenderChartData(partParamId, target.chartRef, target.update, true, true);
                }
                return Promise.resolve();
            })
        );
        showDatePicker.value = false;
        ElMessage.success('数据重放成功');
    } catch (error) {
        console.error('数据重放失败:', error);
        ElMessage.error('数据重放失败，请稍后重试');
    } finally {
        loadingInstance.close();
    }
};

const loadMonitorTypeData = async () => {
    if (!equipCode.value) {
        ElMessage.warning('装备统一编码未选择，请选择装备后重试');
        return;
    }
    try {
        const response = await MonitorTypeApi(equipCode.value);
        if (response.code === 200) {
            options.value = response.data?.map((item: any) => ({
                label: item.partParamChinese,
                value: item.partParamId,
                measureUnit: item.measureUnit,
            }));
            value2.value = response.data?.slice(0, 4).map((item: any) => item.partParamId) || [];
        } else {
            ElMessage.error(`加载末端感知数据失败: ${response.message}`);
        }
    } catch (error) {
        console.error('调用 MonitorTypeApi 失败:', error);
        ElMessage.error('加载末端感知数据失败，请稍后重试');
    }
};

const goBack = () => {
    router.go(-1);
};

const usage = () => {
    router.push({ path: '/deviceinformation/usageConsumption', query: { equipCode: deviceInfo.value.equipCode, businessActivity: '单装使用消耗' } });
};

const maintain = () => {
    router.push({ path: '/deviceinformation/maintenanceRecord', query: { equipCode: deviceInfo.value.equipCode, businessActivity: '单装保养记录' } });
};

const repair = () => {
    router.push({ path: '/deviceinformation/repairRecord', query: { equipCode: deviceInfo.value.equipCode, businessActivity: '单装修理记录' } });
};

const fault = () => {
    router.push({ path: '/deviceinformation/faultRecord', query: { equipCode: deviceInfo.value.equipCode, businessActivity: '单装故障记录' } });
};

const getTagType = (status: string) => {
    if (status.includes('范围内')) return 'success';
    if (status.includes('边缘范围内')) return 'warning';
    if (status.includes('边缘范围外')) return 'danger';
    return 'info';
};

const objectSpanMethod = ({ rowIndex }: { rowIndex: number }) => {
    const componentGroups: { system: string; component: string; startIndex: number; count: number }[] = [];
    let currentSystem = tableData.value[0].system;
    let currentComponent = tableData.value[0].component;
    let startIndex = 0;
    let count = 1;

    for (let i = 1; i < tableData.value.length; i++) {
        if (tableData.value[i].system === currentSystem && tableData.value[i].component === currentComponent) {
            count++;
        } else {
            componentGroups.push({ system: currentSystem, component: currentComponent, startIndex, count });
            currentSystem = tableData.value[i].system;
            currentComponent = tableData.value[i].component;
            startIndex = i;
            count = 1;
        }
    }
    componentGroups.push({ system: currentSystem, component: currentComponent, startIndex, count });

    const group = componentGroups.find(g => rowIndex >= g.startIndex && rowIndex < g.startIndex + g.count);
    if (group && rowIndex === group.startIndex) {
        return { rowspan: group.count, colspan: 1 };
    }
    return { rowspan: 0, colspan: 0 };
};

// Reactive tableData to allow dynamic updates
const tableData = ref<User[]>([]);

const data = [
    {
        real: '摩托小时（从大修以后的值计算）',
        standard: '小修摩托小时',
        departure: { status: '小修-范围内', value: '√' },
    },
];

// 替换原来的 activities 为动态计算的 subsystemActivities
const subsystemActivities = computed<SubsystemActivity[]>(() => {
    if (!deviceInfo.value.equipModel?.subsysList) {
        return [];
    }
    return deviceInfo.value.equipModel.subsysList;
});

// 获取时间线类型（根据索引交替显示不同颜色）
const getTimelineType = (index: number): string => {
    const types = ['primary', 'success', 'warning', 'danger', 'info'];
    return types[index % types.length];
};
// 生成分系统显示内容（保持原来的格式）
const getSubsystemContent = (subsys: SubsystemActivity): string => {
    const partNames = subsys.partList.map(part => part.partName).join('、');
    return `${subsys.subsysName}——${partNames}`;
};

const headerCellStyle = {
    backgroundColor: '#F8F8F8',
    color: 'black',
    fontWeight: 'bold',
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
                    grid: { left: '3%', right: '4%', bottom: '10%', containLabel: true },
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
        } else if (update) {
            await update({
                title: { left: 'center', text: '' },
                xAxis: { type: 'category', boundaryGap: false, data: [] },
                yAxis: { type: 'value', name: '值' },
                series: [{ data: [], type: 'line', areaStyle: {} }],
                grid: { left: '3%', right: '4%', bottom: '10%', containLabel: true },
            });
        }
    }
});

// 在 watch 中监听 isde 变化，重新调整图表尺寸
watch(isde, (newValue) => {
    if (newValue) {
        // 使用 nextTick 确保 DOM 已经更新
        nextTick(() => {
            // 重新初始化所有图表
            defaultTypes.forEach(({ chartRef, update }) => {
                if (chartRef.value && update) {
                    const chartInstance = echarts.getInstanceByDom(chartRef.value);
                    if (chartInstance) {
                        chartInstance.resize();
                    }
                }
            });
        });
    }
}, { immediate: true });

const getInitialOptions = (type: string, colors: Colors) => async () => ({
    title: { left: 'center', text: type, textStyle: { color: colors.line } },
    tooltip: { trigger: 'axis', position: function (pt: number[]) { return [pt[0], '10%']; } },
    toolbox: { feature: { dataZoom: { yAxisIndex: 'none' }, restore: {}, saveAsImage: {} } },
    xAxis: { type: 'category', boundaryGap: false, data: [], axisLabel: { interval: 119 } },
    yAxis: {
        type: 'value',
        name: type === '震动传感器' ? '震动 (mm/s)' : type === '电压传感器' ? '电压 (V)' : type === '压力传感器' ? '压力 (Pa)' : '温度 (°C)',
        boundaryGap: [0, '100%'],
        axisLine: { lineStyle: { color: colors.line } },
    },
    dataZoom: [{ type: 'inside', start: 0, end: 100 }, { type: 'slider', start: 0, end: 100, height: 20 }],
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
                { offset: 1, color: colors.line },
            ]),
            opacity: 0.5,
        },
        data: [],
    }],
    grid: { left: '5%', right: '5%', bottom: '15%', containLabel: true },
});

defaultTypes[0].update = useChart(chartRef1, getInitialOptions('震动传感器', defaultTypes[0].colors)).update;
defaultTypes[1].update = useChart(chartRef2, getInitialOptions('电压传感器', defaultTypes[1].colors)).update;
defaultTypes[2].update = useChart(chartRef3, getInitialOptions('压力传感器', defaultTypes[2].colors)).update;
defaultTypes[3].update = useChart(chartRef4, getInitialOptions('温度传感器', defaultTypes[3].colors)).update;

onMounted(async () => {
    equipCode.value = (route.query.equipCode as string) || safeSessionStorage.get('detailInforEquipCode') || '100434';
    await nextTick();
    await loadMonitorTypeData();
    await loadInitialChartData();
    storeSessionData();
    await loadData();
    await loadTechConditionData(); // Load tech condition data on mount
});

watch(() => route.query.equipCode, async (newEquipCode) => {
    if (newEquipCode && newEquipCode !== equipCode.value) {
        equipCode.value = newEquipCode as string;
        storeSessionData();
        await loadData();
        await loadMonitorTypeData();
        await loadInitialChartData();
        await loadTechConditionData(); // Reload tech condition data when equipCode changes
    }
});
</script>

<style lang="less" scoped>
.main-card {
    background-color: #e9eef3;
    border: none;
    box-shadow: none;
}

.info-card {
    margin-bottom: 20px;
    border: 1px solid #dcdcdc;
    border-radius: 4px;
    background-color: #fff;
}

.section-title {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 15px;
    color: #409eff;
    padding: 10px;
    display: flex;
    align-items: center;
}

.info-container {
    display: flex;
    flex-direction: column;
}

.info-item {
    display: flex;
    justify-content: space-between;
    padding: 10px 0;
}

.info-label {
    font-weight: bold;
    color: #606266;
}

.input-box {
    width: 50%;
    margin-bottom: 10px;
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

.section-title:hover {
    background-color: #f5f7fa;
}

::v-deep(.custom-dropdown-menu) {
    border: none !important;
}

::v-deep(.custom-dropdown-menu .el-dropdown-menu__item) {
    padding: 8px 20px;
}

::v-deep(.custom-dropdown-menu .el-dropdown-menu__item:hover) {
    background-color: #f5f7fa;
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

.progress-container {
    display: flex;
    align-items: center;
    gap: 10px;
    font-family: Arial, sans-serif;
    float: right;
}

.progress-item {
    display: flex;
    align-items: center;
    gap: 5px;
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

::v-deep(.el-radio-group .el-radio.is-disabled) {
    .el-radio__input .el-radio__inner {
        background-color: #E6F7FF !important;
    }

    &.is-checked .el-radio__inner {
        background-color: #409EFF !important;
    }

    .el-radio__label {
        color: black !important;
    }
}

.moduan {
    :deep(.el-select-group__title) {
        font-size: 20px;
        font-weight: bold;
    }
}
</style>
```