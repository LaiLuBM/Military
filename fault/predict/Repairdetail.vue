<template>
    <div>
        <el-card>
            <el-row :gutter="20">
                <el-col :span="24">
                    <!-- 标题和按钮 -->
                    <el-row>
                        <el-col :span="1">
                            <el-button text @click="goBack" icon="ArrowLeft"
                                style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
                        </el-col>
                        <el-col :span="15">
                            <h2 class="title-margin">修理时机故障预测结果详情</h2>
                        </el-col>
                        <el-col :span="8" class="center-content">
                            <el-button type="warning" size="large" @click="viewReport()">查看故障报告</el-button>
                        </el-col>
                    </el-row>
 <el-row style="margin-top:10px;margin-bottom:10px">
            <el-col :span="24">
              <el-tabs v-model="selectedEquipCode" type="card" > 
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

                    <!-- 装备信息 -->
                    <el-card class="mt card-container">
                        <div>
                            <el-row>
                                <el-col :span="24">
                                    <div class="section-title">装备信息</div>

                                    <el-row class="info-row" type="flex" justify="space-between">
                                        <el-col :span="18">
                                            <el-row class="info-row">
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>所属单位:</strong> {{
                                                        equips.unit.unitSimpleName || '未知' }}</div>
                                                </el-col>
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>装备型号:</strong> {{
                                                        equips.equipModel.equipSimpleName || '未知' }}</div>
                                                </el-col>

                                                <el-col :span="8">
                                                    <div class="info-item"><strong>所属分队:</strong> {{
                                                        equips.unit.unitFullname || '未知' }}</div>
                                                </el-col>
                                            </el-row>
                                            <el-row class="info-item">
                                                <el-col :span="8">
                                                     <div>
                                                <strong>装备统一编码:</strong>
                                                <!-- <el-select v-model="selectedEquipCode"
                                                    :placeholder="selectedEquipCode || '请选择装备统一编码'" style="width: 150px"
                                                    @change="loadData">
                                                    <el-option v-for="item in equipCodeOptions" :key="item.value"
                                                        :label="item.label" :value="item.value" />
                                                </el-select> -->
                                                 <span style="color:#409EFF;font-weight:bold;margin-left:5px">
                      {{selectedEquipCode|| '请选择装备统一编码' }}
                    </span>
                                            </div>
                                                </el-col>
                                                <el-col :span="7">
                                                    <div class="info-item"><strong>装备战斗编号：</strong>{{equips.fightCode || "未知"}}</div>
                                                </el-col>
                                            </el-row>
                                           

                                            <el-row class="info-row">
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>列装时间:</strong> {{ equips.serviceDate
                                                        ?
                                                        equips.serviceDate : '未知' }}</div>
                                                </el-col>
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>服役日期:</strong> {{ serviceLimit ||
                                                        '未知' }}
                                                    </div>
                                                </el-col>
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>大修间隔期内累计行驶里程:</strong> {{ workMile || '未知'
                                                    }}公里
                                                    </div>
                                                </el-col>
                                            </el-row>
                                            <el-row class="info-row">
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>中修间隔期内发动机累计消耗摩托小时:</strong> {{
                                                        equips.equipRecord.vehicleMotor ?
                                                            equips.equipRecord.vehicleMotor + ' 小时' : '未知' }}</div>
                                                </el-col>
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>当年摩托小时消耗限额:</strong> {{ motorLimitYear ?
                                                        motorLimitYear + '小时' : '未知' }}</div>
                                                </el-col>
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>列装时间:</strong> {{
                                                        equips.manufactureDate
                                                        || '未知' }}</div>
                                                </el-col>
                                            </el-row>
                                            <el-row class="info-row">
                                                <!-- <el-col :span="8">
                                                    <div class="info-item"><strong>装备剩余寿命:</strong> {{
                                                        equips.equipModel.motorLimitYear ?
                                                            equips.equipModel.motorLimitYear + ' 年' : '未知' }}</div>
                                                </el-col> -->
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>装备战备储备标准:</strong> {{
                                                        equips.unit.unitType
                                                        || '未知' }}</div>
                                                </el-col>
                                                <el-col :span="8">
                                                    <div class="info-item"><strong>年度修理指标:</strong> {{
                                                        equips.equipRecord.majorFixTimes ?
                                                            equips.equipRecord.majorFixTimes + ' 次' : '未知' }}</div>
                                                </el-col>
                                            </el-row>
                                        </el-col>
                                        <el-col :span="6" class="image-col">
                                            <img src="../../../assets/tanks/tank.png" class="tank-image" />
                                        </el-col>
                                    </el-row>
                                </el-col>
                            </el-row>
                        </div>
                    </el-card>
                    <!-- 装备最近一次故障信息 -->
                    <el-card class="mt card-container">
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
                    </el-card>
                    <!-- 修理时机预测结果 -->
                    <el-card class="mt card-container">
                        <el-row>
                            <el-col :span="21">
                                <h3 class="section-title">修理时机预测结果</h3>
                            </el-col>
                        </el-row>
                        <el-row class="info-row">
                            <el-col :span="6">
                                <div class="info-item"><strong>修理类别:</strong> {{ equips.equipRepair.repairType || '未知'
                                }}
                                </div>
                            </el-col>
                            <el-col :span="6">
                                <div class="info-item"><strong>送修时间:</strong>{{ formatRepairTime(equips.equipRepair.repairTime) || '暂无' }}
                                </div>
                            </el-col>
                            <el-col :span="6">
                                <div class="info-item"><strong>预测执行时间:</strong> {{ equips.equipRepair.predictTime ||
                                    '未知' }}
                                </div>
                            </el-col>
                            <el-col :span="6">
                                <el-button type="primary" text @click="viewFaultReport">查看更多装备信息<el-icon>
                                        <DArrowRight />
                                    </el-icon></el-button>
                            </el-col>
                        </el-row>
                    </el-card>
                </el-col>
            </el-row>
        </el-card>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, onActivated } from 'vue';
import { RepairdetailApi } from '@/api/fault/repairdetail';
import { useRoute, useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';
import { equipDetailApi } from '@/api/fault/equipDetail';

const router = useRouter();
const route = useRoute();

const selectedEquipCode = ref('');
const selectedRepairId = ref<number | null>(null);
const equipCodeOptions = ref<{ value: string; label: string; repairId: number | null }[]>([]);
const isFixed = ref('是');

const motorLimitYear = ref('');
const serviceLimit = ref('');
const workMile = ref('');
const formatRepairTime = (timeStr: string | undefined) => {
    if (!timeStr) return '';

    // 如果已经是天数格式（包含"天"），直接返回
    if (timeStr.includes('天')) return timeStr;

    // 尝试匹配周数
    const match = timeStr.match(/(\d+)\s*周/);
    if (match && match[1]) {
        const weeks = parseInt(match[1]);
        return `${weeks * 7}天`;
    }

    // 如果没有匹配到周数，返回原值
    return timeStr;
};
const equips = ref({
    equipCode: "",
    fightCode:"",
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
    }
};

const storeSessionData = () => {
    if (selectedEquipCode.value && selectedRepairId.value) {
        safeSessionStorage.set('repairDetailEquipCode', selectedEquipCode.value);
        safeSessionStorage.set('repairDetailRepairId', selectedRepairId.value.toString());
    }
};

const viewReport = () => {
    if (selectedEquipCode.value && selectedRepairId.value) {
        router.push({
            path: '/predict/repairreport',
            query: {
                repairId: selectedRepairId.value,
                equipCode: selectedEquipCode.value,
            }
        });
    } else {
        ElMessage.warning('请先选择有效的装备统一编码和修理ID');
    }
};

const goBack = () => {
    window.history.back()
};

const viewFaultReport = () => {
    if (selectedEquipCode.value) {
        router.push("/deviceinformation/detailinformation?equipCode=" + selectedEquipCode.value);
    }
};

const loadData = async () => {
    console.log('正在加载数据，装备统一编码:', selectedEquipCode.value, '修理ID:', selectedRepairId.value);
    if (!selectedRepairId.value || !selectedEquipCode.value) {
        console.warn(`无法加载数据：装备 ${selectedEquipCode.value} 的修理ID为空`);
        ElMessage.warning(`无法加载数据：缺少有效的装备统一编码或修理ID`);
        return;
    }
    try {
        const response = await RepairdetailApi(selectedRepairId.value);
        const resinfo = await equipDetailApi(selectedEquipCode.value);

        if (response.code === 200) {
            equips.value = response.data;
            motorLimitYear.value = resinfo.data?.equipModel?.motorLimitYear || '未知';
            workMile.value = resinfo.data?.totalWorkMile || '未知';
            serviceLimit.value = resinfo.data?.serviceLimit || '暂无数据';
            storeSessionData();
        } else {
            console.error('获取数据失败:', response.message);
            ElMessage.error(`加载数据失败: ${response.message}`);
        }
    } catch (error) {
        console.error('API 调用失败:', error);
        ElMessage.error('加载数据失败，请稍后重试');
    }
};

const initializeData = async () => {
    console.log('初始化数据，路由参数:', route.query);

    // Step 1: 获取路由参数
    const queryEquipCode = route.query.equipCode ? String(route.query.equipCode) : '';
    const queryRepairId = route.query.repairId ? parseInt(route.query.repairId as string) : null;

    // Step 2: 初始化下拉框选项
    equipCodeOptions.value = [];

    // Step 3: 从 sessionStorage 获取批量预测的装备信息
    const selectedEquipmentsJson = safeSessionStorage.get('selectedEquipments');
    let sessionEquipments: any[] = [];

    if (selectedEquipmentsJson) {
        try {
            sessionEquipments = JSON.parse(selectedEquipmentsJson);
            if (!Array.isArray(sessionEquipments)) {
                console.warn('sessionStorage selectedEquipments 不是有效数组');
                sessionEquipments = [];
            }
        } catch (error) {
            console.error('解析 sessionStorage selectedEquipments 失败:', error);
            sessionEquipments = [];
        }

        // 构建下拉框选项，仅使用 equipCode
        equipCodeOptions.value = [...new Set(sessionEquipments.map((item: any) => item.equipCode))].map((code: any) => {
            const equip = sessionEquipments.find((e: any) => e.equipCode === code);
            return {
                value: code,
                label: code,
                repairId: equip?.repairId || null,
            };
        });
    }

    // Step 4: 处理单独点击“预测详情”的情况
    if (queryEquipCode) {
        if (!equipCodeOptions.value.some(option => option.value === queryEquipCode)) {
            try {
                const resinfo = await equipDetailApi(queryEquipCode);
                const latestRepairId = resinfo.data?.equipRepair?.repairId || queryRepairId;
                equipCodeOptions.value.push({
                    value: queryEquipCode,
                    label: queryEquipCode,
                    repairId: latestRepairId,
                });
                if (!selectedRepairId.value) {
                    selectedRepairId.value = latestRepairId;
                }
            } catch (error) {
                console.error('获取装备详情失败:', error);
                ElMessage.error('加载装备详情失败，将使用路由参数');
                equipCodeOptions.value.push({
                    value: queryEquipCode,
                    label: queryEquipCode,
                    repairId: queryRepairId,
                });
            }
        }
        selectedEquipCode.value = queryEquipCode;
        selectedRepairId.value = queryRepairId;
    } else if (sessionEquipments.length > 0) {
        selectedEquipCode.value = sessionEquipments[0].equipCode;
        selectedRepairId.value = sessionEquipments[0].repairId || null;
    } else {
        ElMessage.warning('请先选择装备进行预测');
    }

    console.log('初始 selectedEquipCode:', selectedEquipCode.value);
    console.log('初始 selectedRepairId:', selectedRepairId.value);
    console.log('equipCodeOptions:', equipCodeOptions.value);

    // Step 5: 存储并加载数据
    storeSessionData();
    if (selectedEquipCode.value && selectedRepairId.value) {
        await loadData();
    }
};

onMounted(() => {
    console.log('Repairdetail组件已挂载');
    initializeData();
});

onActivated(() => {
    console.log('Repairdetail组件已激活');
    initializeData();
});

watch(() => safeSessionStorage.get('selectedEquipments'), (newEquipments) => {
    if (newEquipments) {
        let sessionEquipments: any[] = [];
        try {
            sessionEquipments = JSON.parse(newEquipments);
            if (!Array.isArray(sessionEquipments)) {
                console.warn('sessionStorage selectedEquipments 不是有效数组');
                sessionEquipments = [];
            }
        } catch (error) {
            console.error('解析 sessionStorage selectedEquipments 失败:', error);
            sessionEquipments = [];
        }

        equipCodeOptions.value = sessionEquipments.map((item: any) => ({
            value: item.equipCode,
            label: item.equipCode,
            repairId: item.repairId || null,
        }));

        const currentFromRoute = route.query.equipCode === selectedEquipCode.value;
        if (!currentFromRoute && !equipCodeOptions.value.some(option => option.value === selectedEquipCode.value)) {
            if (sessionEquipments.length > 0) {
                selectedEquipCode.value = sessionEquipments[0].equipCode;
                selectedRepairId.value = sessionEquipments[0].repairId || null;
            }
        }

        storeSessionData();
        if (selectedRepairId.value && selectedEquipCode.value) {
            loadData();
        }
    }
}, { immediate: true });

watch(selectedEquipCode, (newEquipCode) => {
    const selectedOption = equipCodeOptions.value.find(option => option.value === newEquipCode);
    selectedRepairId.value = selectedOption ? selectedOption.repairId : null;
    storeSessionData();
    if (selectedRepairId.value && selectedEquipCode.value) {
        loadData();
    }
});
</script>

<style lang="less" scoped>
.tank-image {
    width: 100%;
    max-width: 200px;
    height: auto;
    object-fit: contain;
    margin-right: 100px;
}

.image-col {
    display: flex;
    justify-content: flex-end;
    align-items: center;
}

.page-title {
    font-size: 24px;
    font-weight: bold;
    color: #333;
    margin-bottom: 20px;
}

.report-button {
    background-color: #e6a23c;
    color: #fff;
    font-weight: bold;
}

.card-container {
    margin-bottom: 20px;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.section-title {
    font-size: 18px;
    font-weight: bold;
    color: #333;
    margin-bottom: 15px;
}

.info-item {
    font-size: 14px;
    color: #666;
    margin-bottom: 10px;
}

.info-row {
    margin-bottom: 15px;
}

.center-content {
    display: flex;
    justify-content: center;
    align-items: center;
}

.fl {
    float: left;
}

.fr {
    float: right;
}

.clear {
    clear: both;
}

:deep(.el-radio-group .el-radio.is-disabled) {
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