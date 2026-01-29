<template>
    <el-card>
        <el-row :gutter="15" align="middle">
            <el-col :span="6">
                <el-form>
                    <el-form-item label="预测类型">
                        <el-select v-model="types" placeholder="请选择预测类型" size="large" style="width: 240px">
                            <el-option v-for="item in typeoptions" :key="item.value" :label="item.label"
                                :value="item.value" />
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="6">
                <el-form>
                    <el-form-item label="单位/分队" prop="unitPath">
                                    <el-tree-select v-model="selectedUnit" :data="treeData" :props="treeProps" node-key="unitCode"
                                      placeholder="请选择单位（可搜索）" filterable clearable style="width: 100%" check-strictly
                                      @change="handleUnitSelect" />
                                  </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="12">
                <el-form>
                    <el-form-item>
                        <div class="flex-container">
                            <el-switch v-model="isInputEnabled" @change="handleSwitchChange"
                                class="switch-style"></el-switch>
                            <el-input v-model="inputValue" :disabled="!isInputEnabled" placeholder="输入数字即可"
                                class="input-style" @input="handleInputChange">
                                <template #prepend>
                                    <span>最近</span>
                                </template>
                                <template #append>
                                    <span>天内没有执行预测的装备</span>
                                </template>
                            </el-input>
                            <el-button type="primary" class="ml" @click="handleQuery">查询</el-button>
                        </div>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <el-divider></el-divider>
        <el-row :gutter="20" align="middle">
            <el-col :span="15">

            </el-col>
            <el-col :span="5">
                <el-form>
                    <el-form-item label="装备型号">
                        <el-select placeholder="装备型号" v-model="searchParams.equipModelCode">
                            <el-option-group v-for="group in equipmentTypeOptions" :key="group.label"
                                :label="group.label">
                                <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                                    :value="option.value"></el-option>
                            </el-option-group>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <el-row :gutter="20" align="middle">
            <div style="display: flex; width: 100%; height: 500px;">
                <div ref="leftChartInstance" style="width: 40%; height: 100%; "></div>
                <div ref="rightChartInstance" style="width: 60%; height: 100%; "></div>
            </div>
        </el-row>
        <!-- Add description below the chart -->
        <el-row :gutter="20" align="middle" v-if="chartDescription" style="margin-top: 10px;">
            <el-col :span="24">
                <div class="chart-description">
                    {{ chartDescription }}
                </div>
            </el-col>
        </el-row>
    </el-card>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, nextTick, onUnmounted } from 'vue';
import * as echarts from 'echarts';
import { unitListApi } from '@/api/fault/unitlist';
import { malfregionsubreason } from '@/api/fault/strengthanalyse';
import { ElMessage, ElTree } from 'element-plus';
import { equipmodelApi } from '@/api/fault/equip';

const types = ref('');
const typeoptions = [
    { value: 'true', label: '精细化故障预测' },
];
const handleInputChange = (value: string) => {
    const numValue = parseInt(value);

    // 检查数值是否超过999
    if (numValue > 999) {
        ElMessage.warning('输入的天数不能超过999天');
        inputValue.value = '999'; // 重置为最大值
        searchParams.value.day = 999;
        return;
    }

    // 如果是有效数字，更新搜索参数
    if (!isNaN(numValue) && numValue > 0) {
        searchParams.value.day = numValue;
    } else if (value === '') {
        // 如果输入为空，使用默认值
        searchParams.value.day = 90;
    }
};
watch(types, (newValue) => {
    searchParams.value.isDetailed = newValue === 'true';
});
const selectedUnit = ref('');
const localChartData = ref(null);
const searchParams = ref({
    equipModel: '',
    equipCode: '',
    equipModelCode: '',
    day: 90,
    ifpredicted: false,
    unitPath: '',
    current: 1,
    size: 10,
    isDetailed: false,
});
const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref([]);
const defaultProps = { children: 'childrenUnits', label: 'unitSimpleName' };
const isInputEnabled = ref(true);
const inputValue = ref('90');

const equipmentTypeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);
// Add a ref to store the description
const chartDescription = ref<string>('');
// Chart instances
const leftChartInstance = ref<HTMLElement | null>(null);
const rightChartInstance = ref<HTMLElement | null>(null);

// ECharts instances
let leftChart: echarts.ECharts | null = null;
let rightChart: echarts.ECharts | null = null;

// Dispose of chart instances
const disposeCharts = () => {
    if (leftChart) {
        leftChart.dispose();
        leftChart = null;
    }
    if (rightChart) {
        rightChart.dispose();
        rightChart = null;
    }
};

const renderChart = async (data: any) => {
    await nextTick();
    if (!leftChartInstance.value || !rightChartInstance.value) {
        ElMessage.error('图表容器未找到');
        return;
    }

    try {
        disposeCharts();
        leftChart = echarts.init(leftChartInstance.value);
        rightChart = echarts.init(rightChartInstance.value);
    } catch (error) {
        ElMessage.error('图表初始化失败');
        console.error(error);
        return;
    }

    const colors: Record<string, string> = {
        '发动机漏油': '#ff7f0e',
        '润滑不足': '#2ca02c',
        '电路短路': '#1f77b4',
        '液压泄漏': '#d62728',
        '机械磨损': '#9467bd',
        '其他': '#e377c2',
        '无故障': '#999999',
    };

    // Default data
    const defaultReasons = ['发动机漏油', '润滑不足', '电路短路'];
    const defaultMotorRanges = ['0-100', '100-200', '200-300', '300-400'];
    const defaultPieData = defaultReasons.map(reason => ({
        name: reason,
        value: 1, // Equal slices for default pie
    }));
    const defaultLineSeries = defaultReasons.map(reason => ({
        name: reason,
        type: 'line',
        data: defaultMotorRanges.map(() => 0),
        smooth: true,
        symbol: 'circle',
        symbolSize: 8,
        lineStyle: { color: colors[reason] || '#999' },
        itemStyle: { color: colors[reason] || '#999' },
    }));

    let pieOption, lineOption;

    if (!data || !data.malfRatioChart || !data.malfCountChartList) {
        chartDescription.value = '暂无数据，请选择单位/分队和装备型号后查询';

        // Default Pie Chart
        pieOption = {
            color: defaultReasons.map(reason => colors[reason] || '#999'),
            title: {
                text: '故障原因占比',
                left: 'center',
                top: 10,
                textStyle: { fontSize: 16, fontWeight: 'bold' },
            },
            tooltip: { trigger: 'item', formatter: '{b}: {c} ({d}%)' },
            legend: {
                orient: 'horizontal',
                left: 'center',
                top: 'bottom',
                data: defaultReasons,
                textStyle: { fontSize: 12 },
                itemGap: 10,
                itemWidth: 20,
                itemHeight: 14,
            },
            series: [{
                name: '故障占比',
                type: 'pie',
                radius: ['25%', '45%'],
                center: ['50%', '50%'],
                data: defaultPieData,
                emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
                label: { show: true, formatter: '{b}: {d}%' },
            }],
        };

        // Default Line Chart
        lineOption = {
            color: defaultReasons.map(reason => colors[reason] || '#999'),
            title: {
                text: '故障原因数趋势',
                left: 'center',
                top: 10,
                textStyle: { fontSize: 16, fontWeight: 'bold' },
            },
            tooltip: {
                trigger: 'axis',
                formatter: (params: any) => {
                    const range = params[0].name;
                    let result = `使用强度: ${range}<br/>`;
                    params.forEach((param: any) => {
                        result += `${param.seriesName}: ${param.value}<br/>`;
                    });
                    return result;
                },
            },
            legend: {
                data: defaultReasons,
                top: 60,
                left: 'center',
            },
            grid: { left: '10%', right: '15%', top: '20%', bottom: '15%' },
            xAxis: {
                type: 'category',
                name: '使用强度 (小时)',
                data: defaultMotorRanges,
                axisLabel: { rotate: 45, interval: 0 },
            },
            yAxis: { type: 'value', name: '故障数', min: 0, max: 10 },
            series: defaultLineSeries,
        };
    } else {
        const { malfRatioChart, malfCountChartList, description, chartTitle } = data;
        chartDescription.value = description || '';

        // Pie Chart
        const reasons = malfRatioChart.equipMalfReason.includes(',')
            ? malfRatioChart.equipMalfReason.split(',')
            : [malfRatioChart.equipMalfReason];
        const counts = malfRatioChart.equipMalfCount.includes(',')
            ? malfRatioChart.equipMalfCount.split(',').map(Number)
            : [Number(malfRatioChart.equipMalfCount)];
        const pieData = reasons
            .map((reason: string, index: number) => ({
                name: reason,
                value: counts[index] || 0,
            }))
            .filter((item: { value: number }) => item.value > 0);

        pieOption = {
            color: pieData.map((item: any) => colors[item.name] || '#ff7f0e'),
            title: {
                text: malfRatioChart.chartTitle || '故障原因占比',
                left: 'center',
                top: 10,
                textStyle: { fontSize: 16, fontWeight: 'bold' },
            },
            tooltip: { trigger: 'item', formatter: '{b}: {c} ({d}%)' },
            legend: {
                orient: 'horizontal',
                left: 'center',
                top: 'bottom',
                data: pieData.map((item: any) => item.name),
                textStyle: { fontSize: 12 },
                itemGap: 10,
                itemWidth: 20,
                itemHeight: 14,
            },
            series: [{
                name: '故障占比',
                type: 'pie',
                radius: ['25%', '45%'],
                center: ['50%', '50%'],
                data: pieData,
                emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } },
                label: { show: true, formatter: '{b}: {d}%' },
            }],
        };

        // Line Chart
        const allMotorRanges = malfCountChartList
            .map((item: any) => item.equipMalfMotor.split(','))
            .flat();
        const uniqueMotorRanges = [...new Set(allMotorRanges)];

        const lineSeries = malfCountChartList.map((item: any) => {
            const motorRanges = item.equipMalfMotor.split(',');
            const counts = item.equipMalfCount.split(',').map(Number);
            const data = uniqueMotorRanges.map((range) => {
                const index = motorRanges.indexOf(range);
                return index !== -1 && index < counts.length ? counts[index] : 0;
            });
            return {
                name: item.equipMalfReason,
                type: 'line',
                data,
                smooth: false,
                symbol: 'circle',
                symbolSize: 8,
                lineStyle: { color: colors[item.equipMalfReason] || '#ff7f0e' },
                itemStyle: { color: colors[item.equipMalfReason] || '#ff7f0e' },
            };
        });

        lineOption = {
            color: malfCountChartList.map((item: any) => colors[item.equipMalfReason] || '#ff7f0e'),
            title: {
                text: chartTitle || '故障原因数趋势',
                left: 'center',
                top: 10,
                textStyle: { fontSize: 16, fontWeight: 'bold' },
            },
            tooltip: {
                trigger: 'axis',
                formatter: (params: any) => {
                    const range = params[0].name;
                    let result = `使用强度: ${range}<br/>`;
                    params.forEach((param: any) => {
                        result += `${param.seriesName}: ${param.value}<br/>`;
                    });
                    return result;
                },
            },
            legend: {
                data: malfCountChartList.map((item: any) => item.equipMalfReason),
                top: 60,
                left: 'center',
            },
            grid: { left: '10%', right: '15%', top: '20%', bottom: '15%' },
            xAxis: {
                type: 'category',
                name: '使用强度 (小时)',
                data: uniqueMotorRanges,
                axisLabel: { rotate: 45, interval: 0 },
            },
            yAxis: {
                type: 'value',
                name: '故障数',
                min: 0,

            },
            series: lineSeries,
        };
    }

    try {
        leftChart.setOption(pieOption, true);
        rightChart.setOption(lineOption, true);
        console.log('Charts rendered successfully');
    } catch (error) {
        ElMessage.error('图表渲染失败，请查看控制台');
        console.error(error);
        leftChart.setOption({ title: { text: '图表渲染失败', left: 'center', top: 'center' } });
        rightChart.setOption({ title: { text: '图表渲染失败', left: 'center', top: 'center' } });
    }
};

const resizeCharts = () => {
    if (leftChart) leftChart.resize();
    if (rightChart) rightChart.resize();
};

onMounted(async () => {
    const { data } = await unitListApi();
    treeData.value = data;
    await nextTick();
    renderChart(localChartData.value);
    window.addEventListener('resize', resizeCharts);
});

onUnmounted(() => {
    window.removeEventListener('resize', resizeCharts);
    disposeCharts();
});

watch(localChartData, async (newData) => {
    console.log('localChartData updated:', JSON.stringify(newData, null, 2));
    await renderChart(newData);
});

const updateSearchParamsDay = () => {
    searchParams.value.day = parseInt(inputValue.value) || 90;
};

const handleSwitchChange = (value: boolean) => {
    searchParams.value.ifpredicted = value;
    if (!value) {
        inputValue.value = '';
        searchParams.value.day = 0;
    } else {
        inputValue.value = '90';
        searchParams.value.day = 90;
    }
};

// 在watch部分添加对isDetailed的监听
watch(
    () => searchParams.value.isDetailed,
    async (newValue) => {
        if (selectedUnit.value && searchParams.value.unitPath) {
            try {
                const response = await equipmodelApi(newValue, searchParams.value.unitPath);
                if (response.code === 200 && response.data?.categories) {
                    const groupedOptions = Object.entries(response.data.categories).map(([category, items]) => ({
                        label: category,
                        options: (items as any[]).map((item) => ({
                            label: item.equipSimpleName,
                            value: item.equipModelCode,
                        })),
                    }));
                    equipmentTypeOptions.value = groupedOptions;
                    equipmentCodeOptions.value = groupedOptions;
                    // 清空已选择的装备型号
                    searchParams.value.equipModelCode = '';
                }
            } catch (error) {
                console.error('获取装备型号失败:', error);
                ElMessage.error('获取装备型号失败');
            }
        }
    }
);

// 修改handleNodeClick函数中的equipmodelApi调用
const _deprecated_handleNodeClick = async (nodeData: any) => {
    selectedUnit.value = nodeData.unitSimpleName;
    const unitPath = nodeData.unitPath ? nodeData.unitPath.toString() : '';
    if (!unitPath) {
        ElMessage.error('请选择有效的单位');
        return;
    }
    searchParams.value.unitPath = unitPath;

    // 使用当前的isDetailed值请求装备型号
    const isDetailed = searchParams.value.isDetailed;
    const response = await equipmodelApi(isDetailed, unitPath);
    if (response.code === 200 && response.data?.categories) {
        const groupedOptions = Object.entries(response.data.categories).map(([category, items]) => ({
            label: category,
            options: (items as any[]).map((item) => ({
                label: item.equipSimpleName,
                value: item.equipModelCode,
            })),
        }));
        equipmentTypeOptions.value = groupedOptions;
        equipmentCodeOptions.value = groupedOptions;
        // 清空已选择的装备型号
        searchParams.value.equipModelCode = '';
    }
};
const handleQuery = async () => {
    const params = {
        day: searchParams.value.day,
        equipModelCode: searchParams.value.equipModelCode,
        ifPredicted: isInputEnabled.value,

        unitPath: searchParams.value.unitPath,
    };

    if (!params.unitPath && !params.equipModelCode) {
        ElMessage.warning('请选择单位/分队和装备型号');
        return;
    }
    if (params.unitPath && (!params.equipModelCode)) {
        ElMessage.warning('请选择装备型号');
        return;
    }
    const response = await malfregionsubreason(params);
    if (response.code === 200) {
        ElMessage.success('查询成功');
        localChartData.value = response.data;
    } else {
        ElMessage.error(response.message || '查询失败');
    }
};


// 自动添加的模糊查询相关逻辑
const treeProps = defaultProps;

const handleUnitSelect = (newValue: any) => {
  console.log('选中单位：', newValue)
  // 找到对应的节点
  // 使用 loose type 避免 Tree 类型未定义问题
  const selectedNode = findNodeByLabel(treeData.value, newValue)
  if (!selectedNode) return
  
  if (searchParams.value.equipModelCode !== undefined) searchParams.value.equipModelCode = ''
  if (searchParams.value.equipCode !== undefined) searchParams.value.equipCode = ''
  
  const unitPath = selectedNode.unitPath ? selectedNode.unitPath.toString() : ''
  // 兼容不同的字段名，有的可能是 unitCode
  if (searchParams.value.unitPath !== undefined) searchParams.value.unitPath = unitPath
  else if ((searchParams.value as any).unitCode !== undefined) (searchParams.value as any).unitCode = unitPath // Some files used unitCode

  // 加载装备型号
  loadEquipModels(unitPath)
}

const findNodeByLabel = (nodes: any, label: any) => {
  if (!nodes) return undefined
  for (const node of nodes) {
    if (node.unitSimpleName === label) return node
    if (node.childrenUnits) {
      const found = findNodeByLabel(node.childrenUnits, label)
      if (found) return found
    }
  }
  return undefined
}

// 加载装备型号
// 注意：这里假设了 API 方法名为 equipmodelApi，如果报错请检查 import
const loadEquipModels = async (unitPath: any) => {
  try {
    // 尝试调用 API，API名称在替换时会尝试自动修正
    const response = await equipmodelApi(unitPath)
    if (response.code === 200 && response.data?.categories) {
      const groupedOptions: any[] = []
      Object.entries(response.data.categories).forEach(([category, items]) => {
         if (!Array.isArray(items)) {
          // console.error(`类别 ${category} 的数据不是数组:`, items);
          return;
        }

        const options = items.map((item: any) => ({
          label: item.equipSimpleName,
          value: item.equipModelCode,
        }));

        groupedOptions.push({
          label: category,
          options,
        });
      })

      if (typeof equipmentTypeOptions !== 'undefined') {
        equipmentTypeOptions.value = groupedOptions;
        // console.log('生成的 equipmentTypeOptions:', equipmentTypeOptions.value);
      }
       // 同时也尝试更新 equipmentCodeOptions，部分文件可能用这个
      if (typeof equipmentCodeOptions !== 'undefined') {
          equipmentCodeOptions.value = groupedOptions;
      }
    }
  } catch (e) {
    console.error('加载装备型号失败', e)
  }
}

</script>

<style lang="less" scoped>
.flex-container {
    display: flex;
    align-items: center;
    gap: 10px;
}

/* Style the description */
.chart-description {
    text-align: left;
    font-size: 14px;
    color: #3968ce;
    padding: 10px;
    background-color: #bcd5fc;
    border-radius: 4px;
}

// 树形控件样式
.sub-tree {

    // 鼠标滑过高亮
    ::v-deep .el-tree-node__content:hover {
        background-color: #e6f7ff !important; // 鼠标滑过时的背景颜色
    }

    // 点击后高亮停留
    ::v-deep .el-tree-node.is-current>.el-tree-node__content {
        background-color: #e6f7ff; // 点击后的背景颜色
    }

    font-weight: normal;
}

// 调整分组下拉框的样式
:deep(.el-select-group__title) {
    font-weight: bold; // 分组标题加粗
    color: #409eff; // 分组标题颜色
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px; // 子选项缩进
}
</style>