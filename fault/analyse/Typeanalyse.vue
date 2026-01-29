<template>
    <el-card>
        <el-row :gutter="24" style="display: flex;">
            <el-col :span="16">
                <el-form style="display: flex; align-items: center;">
                    <el-icon :size="30">
                        <ChatLineSquare />
                    </el-icon>
                    <span style="font-size: 20px; margin-left: 10px;">型号单位分析</span>
                </el-form>
            </el-col>
            <el-col :span="8">
                <el-form style="display: flex; align-items: center;">
                    <el-form-item>
                        <div class="flex-container">
                            <el-switch v-model="isInputEnabled" @change="handleSwitchChange"
                                class="switch-style"></el-switch>
                            <el-input v-model="inputValue" :disabled="!isInputEnabled" placeholder="输入数字即可"
                                class="input-style" @input="handleInputChange">
                                <template #prepend>
                                    <span>分析最近</span>
                                </template>
                                <template #append>
                                    <span>天内的预测记录</span>
                                </template>
                            </el-input>
                        </div>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <el-row :gutter="24" style="display: flex; margin-top: 10px;">
            <el-col :span="24">
                <div style="display: flex; align-items: center; justify-content: center; font-size: 18px;">
                    <span>按型号分析</span>
                    <el-form style="display: flex; align-items: center; margin: 0;">
                        <el-form-item style="margin: 0;">
                            <el-select v-model="selectedUnit" placeholder="请选择单位" style="width: 200px"
                                :popper-append-to-body="false" clearable>
                                <el-option :value="selectedUnit" style="height: auto; padding: 0">
                                    <el-tree ref="modelTreeRef" :data="modelTreeData" :props="defaultTreeProps"
                                        node-key="unitCode" :default-expand-all="false"
                                        @node-click="handleModelTreeNodeClick" class="sub-tree"></el-tree>
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-form>
                    <span>故障预测规律</span>
                </div>
                <div style="display: flex; justify-content: flex-end;">
                    <el-form>
                        <el-form-item label="">
                            <el-select placeholder="请选择对比装备型号" style="width: 600px"
                                v-model="searchParams.equipModelCode" multiple :multiple-limit="5" clearable>
                                <el-option-group v-for="group in equipmentTypeOptions" :key="group.label"
                                    :label="group.label">
                                    <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                                        :value="option.value"></el-option>
                                </el-option-group>
                            </el-select>
                        </el-form-item>
                    </el-form>
                </div>
                <div ref="modelChartRef" style="display: flex; width: 90%; height: 500px; margin: 20px auto;"
                    class="chart1">
                </div>
            </el-col>
            <el-col :span="24">
                <div style="display: flex; align-items: center; justify-content: center; font-size: 18px;">
                    按单位分析故障预测规律
                </div>
                <div style="display: flex; justify-content: flex-end;">
                    <el-form>
                        <el-form-item label="">
                            <el-select v-model="selectedCompareUnits" placeholder="请选择对比单位" style="width: 600px"
                                multiple clearable :multiple-limit="5" @visible-change="handleUnitSelectVisibleChange"
                                :popper-append-to-body="false">
                                <el-option value="" style="height: auto; padding: 0">
                                    <el-tree ref="unitTreeRef" :data="unitTreeData" :props="defaultTreeProps"
                                        node-key="unitCode" :default-expand-all="false"
                                        @node-click="handleUnitTreeNodeClick" class="sub-tree"></el-tree>
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-form>
                </div>
                <div ref="unitChartRef" style="display: flex; width: 90%; height: 500px; margin: 20px auto;"
                    class="chart1">
                </div>
            </el-col>
        </el-row>
    </el-card></template>

<script lang="ts" setup>
import { ref, onMounted, watch, onUnmounted, nextTick,markRaw } from 'vue';
import { ElMessage, ElTree } from 'element-plus';
import * as echarts from 'echarts';
import { equipmodelApi } from '@/api/fault/equipmodel';
import { unitListApi } from '@/api/fault/unitlist';
import { modelprediction, brigadeprediction } from '@/api/fault/typeanalyze';

// 型号分析图表相关变量
const modelChartRef = ref<HTMLDivElement | null>(null);
const modelChartInstance = ref<any>(null);
const modelTreeRef = ref<InstanceType<typeof ElTree>>();
const modelTreeData = ref<any[]>([]);

// 单位分析图表相关变量
const unitChartRef = ref<HTMLDivElement | null>(null);
const unitChartInstance = ref<any>(null);
const unitTreeRef = ref<InstanceType<typeof ElTree>>();
const unitTreeData = ref<any[]>([]);

// 单位选项数据
const unitOptions = ref<{ unitCode: string; unitSimpleName: string; unitPath: string }[]>([]);

// 共用配置
const defaultTreeProps = {
    children: 'childrenUnits',
    label: 'unitSimpleName'
};

// 搜索参数
const searchParams = ref({
    equipModel: '',
    equipCode: '',
    equipModelCode: [] as string[],
    days: 90,
    isAllRecords: false,
    unitPath: '',
    unit: '',
    units: '',
    current: 1,
    size: 10,
    isDetailed: false,
});

// 表单数据
const selectedUnit = ref<string>('');
const selectedCompareUnits = ref<string[]>([]);
const selectedUnitPaths = ref<string[]>([]);
const equipmentTypeOptions = ref<{ label: string; options: { label: string; value: string }[] }[]>([]);

// 开关和输入框
const isInputEnabled = ref(true);
const inputValue = ref('90');

// 输入框变化处理
const handleInputChange = (value: string) => {
    const numValue = parseInt(value);

    // 检查数值是否超过999
    if (numValue > 999) {
        ElMessage.warning('输入的天数不能超过999天');
        inputValue.value = '999'; // 重置为最大值
        searchParams.value.days = 999;
        return;
    }

    // 如果是有效数字，更新搜索参数
    if (!isNaN(numValue) && numValue > 0) {
        searchParams.value.days = numValue;
    } else if (value === '') {
        // 如果输入为空，使用默认值
        searchParams.value.days = 90;
    }

    // 刷新图表
    refreshModelChart();
    refreshUnitChart();
};


// 型号分析图表初始化
const initModelChart = async () => {
    await nextTick();
    if (!modelChartRef.value) return;

    if (modelChartInstance.value) {
        modelChartInstance.value.dispose();
        modelChartInstance.value = null;
    }

    try {
        const rawlnstance = echarts.init(modelChartRef.value);
        modelChartInstance.value = markRaw(rawlnstance);
        const options = await setModelChartData();
        modelChartInstance.value.setOption(options);
        window.addEventListener('resize', handleModelChartResize);
    } catch (error) {
        console.error('型号分析图表初始化失败:', error);
    }
};

// 单位分析图表初始化
const initUnitChart = async () => {
    await nextTick();
    if (!unitChartRef.value) return;

    if (unitChartInstance.value) {
        unitChartInstance.value.dispose();
        unitChartInstance.value = null;
    }

    try {
         const rawlnstance = echarts.init(unitChartRef.value);
         unitChartInstance.value = markRaw(rawlnstance);
        const options = await setUnitChartData();
        unitChartInstance.value.setOption(options);
        window.addEventListener('resize', handleUnitChartResize);
    } catch (error) {
        console.error('单位分析图表初始化失败:', error);
    }
};

// 设置型号分析图表数据
const setModelChartData = async () => {
    const params = {
        days: searchParams.value.days,
        equipModel: searchParams.value.equipModel,
        isAllRecords: isInputEnabled.value,
        unit: searchParams.value.unit,
    };

    console.log('API Parameters:', params);

    if (!params.equipModel || !params.unit) {
        return getEmptyChartOptions();
    }

    try {
        const response = await modelprediction(params);
        if (!response?.data || Object.keys( response.data).length===0) {
            return getEmptyChartOptions();
        }

        const modelData = response.data;

        if (Object.keys(modelData).length === 0) {
            console.warn('No model data available in response');
            ElMessage.warning('没有可用的型号数据');
            return getEmptyChartOptions();
        }

        const modelNames = Object.keys(modelData);
        const seriesData = {
            repairPredicted: [] as number[],
            diagnosedFault: [] as number[],
            faultPredicted: [] as number[],
            faultPossible: [] as number[],
            faultOccurred: [] as number[],
        };

        modelNames.forEach(model => {
            const modelInfo = modelData[model] || {};
            seriesData.repairPredicted.push(Number(modelInfo.repairPredictedCount) || 0);
            seriesData.diagnosedFault.push(Number(modelInfo.diagnosedFaultCount) || 0);
            seriesData.faultPredicted.push(Number(modelInfo.predictedFaultCount ) || 0);
            seriesData.faultPossible.push(Number(modelInfo.likelyFaultCount ) || 0);
            seriesData.faultOccurred.push(Number(modelInfo.actualFaultCount) || 0);
        });

        return {
            tooltip: {
                trigger: 'axis',
                formatter: (params: any) => {
                    let result = `${params[0].name}<br>`;
                    params.forEach((item: any) => {
                        result += `${item.seriesName}: ${item.value}<br>`;
                    });
                    return result;
                },
            },
            legend: {
                data: [
                    '故障已发生装备数',
                    '故障可能发生装备数',
                    '进行过故障预测的装备数',
                    '进行过送修预测装备数',
                    '诊断有故障的装备数',
                ],
                top: 10,
            },
            xAxis: {
                name: '装备型号',
                type: 'category',
                data: modelNames,
                axisLabel: {
                    interval: 0,
                    rotate: 0,
                },
            },
            yAxis: {
                name: '装备数量',
                type: 'value',
            },
            series: [
                {
                    name: '故障已发生装备数',
                    type: 'line',
                    data: seriesData.faultOccurred,
                    itemStyle: { color: '#FF6B6B' },
                },
                {
                    name: '故障可能发生装备数',
                    type: 'line',
                    data: seriesData.faultPossible,
                    itemStyle: { color: '#4ECDC4' },
                },
                {
                    name: '进行过故障预测的装备数',
                    type: 'line',
                    data: seriesData.faultPredicted,
                    itemStyle: { color: '#45B7D1' },
                },
                {
                    name: '进行过送修预测装备数',
                    type: 'line',
                    data: seriesData.repairPredicted,
                    itemStyle: { color: '#FFA07A' },
                },
                {
                    name: '诊断有故障的装备数',
                    type: 'line',
                    data: seriesData.diagnosedFault,
                    itemStyle: { color: '#98D8C8' },
                },
            ],
            grid: {
                left: '5%',
                right: '5%',
                bottom: '10%',
                containLabel: true,
            },
        };
    } catch (error) {
        console.error('获取图表数据失败:', error);
        ElMessage.error(`获取图表数据失败: ${error instanceof Error ? error.message : String(error)}`);
        return getEmptyChartOptions();
    }
};

// 设置单位分析图表数据
const setUnitChartData = async () => {
    if (selectedUnitPaths.value.length === 0) {
        return getEmptyChartOptions();
    }

    const params = {
        days: searchParams.value.days,
        isAllRecords: isInputEnabled.value,
        unit: searchParams.value.units,
    };

    console.log('Unit Chart API Parameters:', params);

    try {
        const response = await brigadeprediction(params);
        if ( !response.data) {
            console.error('Invalid API response: response or response.data is missing', response);
            return getEmptyChartOptions();
        }

        const unitData = response.data;

        if (Object.keys(unitData).length === 0) {
            console.warn('No unit data available in response');
            return getEmptyChartOptions();
        }

        // 确保只显示 selectedUnitPaths 中的单位
        const unitNames = selectedUnitPaths.value
            .map((unitCode) => {
                const unit = unitOptions.value.find((opt) => opt.unitCode === unitCode);
                return unit ? unit.unitSimpleName : unitCode;
            }); // 只保留有数据的单位

        const seriesData = {
            repairPredicted: [] as number[],
            diagnosedFault: [] as number[],
            faultPredicted: [] as number[],
            faultPossible: [] as number[],
            faultOccurred: [] as number[],
        };

        unitNames.forEach((unitName) => {
            const unitInfo = (unitData && unitData[unitName]) || {};
            seriesData.repairPredicted.push(Number(unitInfo.repairPredictedCount) || 0);
            seriesData.diagnosedFault.push(Number(unitInfo.diagnosedFaultCount ) || 0);
            seriesData.faultPredicted.push(Number(unitInfo.predictedFaultCount ) || 0);
            seriesData.faultPossible.push(Number(unitInfo.likelyFaultCount ) || 0);
            seriesData.faultOccurred.push(Number(unitInfo.actualFaultCount ) || 0);
        });

        return {
            tooltip: {
                trigger: 'axis',
                formatter: (params: any) => {
                    let result = `${params[0].name}<br>`;
                    params.forEach((item: any) => {
                        result += `${item.seriesName}: ${item.value}<br>`;
                    });
                    return result;
                },
            },
            legend: {
                data: [
                    '故障已发生装备数',
                    '故障可能发生装备数',
                    '进行过故障预测的装备数',
                    '进行过送修预测装备数',
                    '诊断有故障的装备数',
                ],
                top: 10,
            },
            xAxis: {
                name: '单位名称',
                type: 'category',
                data: unitNames, // 只显示当前选中的单位
                axisLabel: {
                    interval: 0,
                    rotate: 0,
                },
            },
            yAxis: {
                name: '装备数量',
                type: 'value',
            },
            series: [
                {
                    name: '故障已发生装备数',
                    type: 'line',
                    data: seriesData.faultOccurred,
                    itemStyle: { color: '#FF6B6B' },
                },
                {
                    name: '故障可能发生装备数',
                    type: 'line',
                    data: seriesData.faultPossible,
                    itemStyle: { color: '#4ECDC4' },
                },
                {
                    name: '进行过故障预测的装备数',
                    type: 'line',
                    data: seriesData.faultPredicted,
                    itemStyle: { color: '#45B7D1' },
                },
                {
                    name: '进行过送修预测装备数',
                    type: 'line',
                    data: seriesData.repairPredicted,
                    itemStyle: { color: '#FFA07A' },
                },
                {
                    name: '诊断有故障的装备数',
                    type: 'line',
                    data: seriesData.diagnosedFault,
                    itemStyle: { color: '#98D8C8' },
                },
            ],
            grid: {
                left: '5%',
                right: '5%',
                bottom: '10%',
                containLabel: true,
            },
        };
    } catch (error) {
        console.error('获取单位图表数据失败:', error);
        ElMessage.error(`获取单位图表数据失败: ${error instanceof Error ? error.message : String(error)}`);
        return getEmptyChartOptions();
    }
};

// 刷新型号分析图表
const refreshModelChart = async () => {
    if (!modelChartInstance.value) {
        await initModelChart();
        return;
    }

    try {
        const options = await setModelChartData();
        modelChartInstance.value.setOption(options, {notMerge: true});
    } catch (error) {
        console.error('型号分析图表刷新失败:', error);
        ElMessage.error('型号分析图表数据刷新失败');
    }
};

// 刷新单位分析图表
const refreshUnitChart = async () => {
    if (!unitChartInstance.value) {
        await initUnitChart();
        return;
    }

    // 如果没有选择对比单位，显示空图表
    if (selectedUnitPaths.value.length === 0) {
        unitChartInstance.value.clear();
        unitChartInstance.value.setOption(getEmptyChartOptions(), {notMerge: true});
        return;
    }

    try {
        const options = await setUnitChartData();
        unitChartInstance.value.clear(); // 清除旧数据
        unitChartInstance.value.setOption(options, {
            notMerge: true,
            lazyUpdate: false,
        });
    } catch (error) {
        console.error('单位分析图表刷新失败:', error);
        unitChartInstance.value.clear();
        unitChartInstance.value.setOption(getEmptyChartOptions(), {
            notMerge: true,
            lazyUpdate: false,
        });
        ElMessage.error('单位分析图表数据刷新失败');
    }
};

// 型号树节点点击处理
const handleModelTreeNodeClick = async (nodeData: any) => {
    const unitName = nodeData.unitSimpleName;
    const unitPath = nodeData.unitPath ? nodeData.unitPath.toString() : '';

    if (!unitPath) {
        ElMessage.error('请选择有效的单位');
        return;
    }

    selectedUnit.value = unitName;
    searchParams.value.unit = nodeData.unitCode;
    searchParams.value.unitPath = unitPath;
    searchParams.value.equipModelCode = [];
    searchParams.value.equipModel = '';

    try {
        const response = await equipmodelApi(unitPath);
        if (response.code === 200 && response.data?.categories) {
            equipmentTypeOptions.value = Object.entries(response.data.categories).map(([category, items]) => ({
                label: category,
                options: (items as any[]).map(item => ({
                    label: item.equipSimpleName,
                    value: item.equipModelCode,
                })),
            }));
        }
        await refreshModelChart();
    } catch (error) {
        console.error('获取装备型号失败:', error);
    }
};

// 单位树节点点击处理
const handleUnitTreeNodeClick = async (nodeData: any) => {
    const { unitCode, unitSimpleName } = nodeData;

    // Check if unit is already selected
    const index = selectedCompareUnits.value.indexOf(unitSimpleName);
    if (index > -1) {
        // Remove unit
        selectedCompareUnits.value.splice(index, 1);
        selectedUnitPaths.value.splice(index, 1);
        unitOptions.value = unitOptions.value.filter((unit) => unit.unitCode !== unitCode);
    } else if (selectedCompareUnits.value.length < 6) {
        // Add unit, limit to 6
        selectedCompareUnits.value.push(unitSimpleName);
        selectedUnitPaths.value.push(unitCode);
        unitOptions.value.push({ unitCode, unitSimpleName, unitPath: nodeData.unitPath || '' });
    } else {
        ElMessage.warning('最多只能选择6个单位进行对比');
        return;
    }

    // Update units parameter
    searchParams.value.units = selectedUnitPaths.value.join(',');

    // Refresh unit chart
    await refreshUnitChart();
};

// 单位选择框显示/隐藏处理
const handleUnitSelectVisibleChange = (visible: boolean) => {
    if (visible) {
        setTimeout(() => {
            unitTreeRef.value?.setCurrentKey();
        }, 0);
    }
};

// 空图表配置
const getEmptyChartOptions = () => ({
    tooltip: { trigger: 'axis' },
    legend: {
        data: [
            '故障已发生装备数',
            '故障可能发生装备数',
            '进行过故障预测的装备数',
            '进行过送修预测装备数',
            '诊断有故障的装备数'
        ],
        top: 10,
    },
    xAxis: {
        name: '装备型号',
        type: 'category',
        data: [],
        axisLabel: { formatter: '{value}' }
    },
    yAxis: {
        name: '装备数量',
        type: 'value',
        axisLabel: { formatter: '{value}' }
    },
    series: [
        {
            name: '故障已发生装备数',
            type: 'line',
            data: [],
            itemStyle: { color: '#FF6B6B' },
            smooth: true
        },
        {
            name: '故障可能发生装备数',
            type: 'line',
            data: [],
            itemStyle: { color: '#4ECDC4' },
            smooth: true
        },
        {
            name: '进行过故障预测的装备数',
            type: 'line',
            data: [],
            itemStyle: { color: '#45B7D1' },
            smooth: true
        },
        {
            name: '进行过送修预测装备数',
            type: 'line',
            data: [],
            itemStyle: { color: '#FFA07A' },
            smooth: true
        },
        {
            name: '诊断有故障的装备数',
            type: 'line',
            data: [],
            itemStyle: { color: '#98D8C8' },
            smooth: true
        }
    ]
});

// 窗口大小变化处理
const handleModelChartResize = () => {
    if (modelChartInstance.value) {
        modelChartInstance.value.resize();
    }
};

const handleUnitChartResize = () => {
    if (unitChartInstance.value) {
        unitChartInstance.value.resize();
    }
};

// 开关变化处理
const handleSwitchChange = (value: boolean) => {
    if (!value) {
        inputValue.value = '';
        searchParams.value.days = 90;
    } else {
        // 开启时检查输入值是否超过999
        const numValue = parseInt(inputValue.value);
        if (numValue > 999) {
            ElMessage.warning('输入的天数不能超过999天');
            inputValue.value = '999';
            searchParams.value.days = 999;
        } else {
            inputValue.value = inputValue.value || '90';
            searchParams.value.days = parseInt(inputValue.value) || 90;
        }
    }
    refreshModelChart();
    refreshUnitChart();
};

// Helper function to flatten unit tree into unitOptions
const flattenUnitTree = (units: any[]): { unitCode: string; unitSimpleName: string; unitPath: string }[] => {
    let result: { unitCode: string; unitSimpleName: string; unitPath: string }[] = [];
    units.forEach((unit) => {
        result.push({
            unitCode: unit.unitCode,
            unitSimpleName: unit.unitSimpleName,
            unitPath: unit.unitPath || '',
        });
        if (unit.childrenUnits && unit.childrenUnits.length > 0) {
            result = result.concat(flattenUnitTree(unit.childrenUnits));
        }
    });
    return result;
};

// Helper function to find a unit in the tree by unitSimpleName
const findUnitInTree = (tree: any[], unitSimpleName: string): any | null => {
    for (const node of tree) {
        if (node.unitSimpleName === unitSimpleName) {
            return node;
        }
        if (node.childrenUnits && node.childrenUnits.length > 0) {
            const found = findUnitInTree(node.childrenUnits, unitSimpleName);
            if (found) return found;
        }
    }
    return null;
};

// 监听变化
watch(
    [selectedUnit, () => searchParams.value.equipModelCode],
    async () => {
        searchParams.value.equipModel = searchParams.value.equipModelCode.join(',');
        await refreshModelChart();
    },
    { deep: true }
);

watch(
    selectedCompareUnits,
    async (newVal) => {
        if (newVal.length === 0) {
            selectedUnitPaths.value = [];
            unitOptions.value = [];
            searchParams.value.units = '';
            if (unitChartInstance.value) {
                unitChartInstance.value.clear();
                unitChartInstance.value.setOption(getEmptyChartOptions(), {
                    notMerge: true,
                    lazyUpdate: false,
                });
            }
            return;
        }

        // Sync selectedUnitPaths and unitOptions with newVal
        const newUnitPaths: string[] = [];
        const newUnitOptions: { unitCode: string; unitSimpleName: string; unitPath: string }[] = [];

        newVal.forEach((unitName) => {
            const existingUnit = unitOptions.value.find((unit) => unit.unitSimpleName === unitName);
            if (existingUnit) {
                newUnitPaths.push(existingUnit.unitCode);
                newUnitOptions.push(existingUnit);
            } else {
                // Find unit in unitTreeData
                const unitNode = findUnitInTree(unitTreeData.value, unitName);
                if (unitNode) {
                    newUnitPaths.push(unitNode.unitCode);
                    newUnitOptions.push({
                        unitCode: unitNode.unitCode,
                        unitSimpleName: unitNode.unitSimpleName,
                        unitPath: unitNode.unitPath || '',
                    });
                }
            }
        });

        selectedUnitPaths.value = newUnitPaths;
        unitOptions.value = newUnitOptions;
        searchParams.value.units = selectedUnitPaths.value.join(',');

        await refreshUnitChart();
    },
    { deep: true }
);

// 组件生命周期
onMounted(async () => {
    try {
        const { data } = await unitListApi();
        modelTreeData.value = data;
        unitTreeData.value = JSON.parse(JSON.stringify(data));
        unitOptions.value = flattenUnitTree(data);
        await initModelChart();
        await initUnitChart();
    } catch (error) {
        console.error('初始化失败:', error);
        ElMessage.error('初始化失败');
    }
});

onUnmounted(() => {
    if (modelChartInstance.value) {
        modelChartInstance.value.dispose();
        window.removeEventListener('resize', handleModelChartResize);
    }
    if (unitChartInstance.value) {
        unitChartInstance.value.dispose();
        window.removeEventListener('resize', handleUnitChartResize);
    }
});
</script>

<style lang="less" scoped>
.chart1 {
    border: 2px solid #ccc;
    border-radius: 20px;
}

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

.title-margin {
    margin-bottom: 5px;
}

.input-style {
    width: 80%;
}

.sub-tree {
    max-height: 300px;

    width: 100%;
    padding: 10px;
    font-weight: normal;
}

:deep(.el-select-group__title) {
    font-weight: bold;
    color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px;
}
</style>