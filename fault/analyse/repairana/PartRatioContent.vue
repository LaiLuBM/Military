<template>
  <el-card>
    <el-row :gutter="15" align="middle">
      <el-col :span="6">
        <el-form>
          <el-form-item label="预测类型">
            <el-select v-model="types" placeholder="请选择预测类型" size="large" style="width: 240px">
              <el-option v-for="item in typeoptions" :key="item.value" :label="item.label" :value="item.value" />
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
              <el-switch v-model="isInputEnabled" @change="handleSwitchChange" class="switch-style"></el-switch>
              <el-input v-model="inputValue" :disabled="!isInputEnabled" placeholder="输入数字即可" class="input-style"
                @input="handleInputChange">
                <template #prepend>
                  <span>最近</span>
                </template>
                <template #append>
                  <span>天内的预测记录</span>
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
      <el-col :span="12">
      </el-col>
      <el-col :span="8">
        <el-form>
          <el-form-item label="装备型号">
            <el-select placeholder="装备型号" v-model="searchParams.equipModelCode">
              <el-option-group v-for="group in equipmentTypeOptions" :key="group.label" :label="group.label">
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
    <el-row :gutter="20" align="middle" v-if="chartDescription && chartDescription !== '。'" style="margin-top: 10px;">
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
import { repairpart } from '@/api/fault/repairanalyse';
import { ElMessage, ElTree } from 'element-plus';
import { equipmodelApi } from '@/api/fault/equip';
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
const types = ref('');
const typeoptions = [
  { value: 'true', label: '精细化故障预测' },
];
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
  ifUnpredicted: false,
  unitPath: '',
  current: 1,
  size: 10,
  isDetailed: false,
});
const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref([]);
const defaultProps = { children: 'childrenUnits', label: 'unitSimpleName' };
const isInputEnabled = ref(false);
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

const generateRandomColors = (count: number): string[] => {
  const colors: string[] = [];
  for (let i = 0; i < count; i++) {
    const hue = Math.floor(Math.random() * 360);
    const saturation = 70 + Math.floor(Math.random() * 30); // 70-100%
    const lightness = 40 + Math.floor(Math.random() * 30); // 40-70%
    colors.push(`hsl(${hue}, ${saturation}%, ${lightness}%)`);
  }
  return colors;
};

const renderChart = async (data: any) => {
  // Dispose of existing charts to prevent memory leaks
  disposeCharts();

  // Validate chart DOM elements
  if (!leftChartInstance.value || !rightChartInstance.value) {
    console.error('Chart DOM element not found');
    ElMessage.error('无法渲染图表：DOM元素未找到');
    return;
  }

  // Wait for DOM to be ready
  await nextTick();

  // Initialize ECharts
  try {
    leftChart = echarts.init(leftChartInstance.value);
    rightChart = echarts.init(rightChartInstance.value);
  } catch (error) {
    console.error('Chart initialization failed:', error);
    ElMessage.error('图表初始化失败');
    return;
  }

  // Extract chart data
  const { malfRatioChart, malfCountChartList, description, chartTitle } = data;
  chartDescription.value = description || '';

  // Process pie chart data (malfRatioChart)
  const pieParts: string[] = (malfRatioChart.equipMalfPart || '').split(',').map((part: string) => part.trim()).filter(Boolean);
  const pieCounts: number[] = (malfRatioChart.equipMalfCount || '').split(',').map((count: string) => {
    const num = parseFloat(count.trim());
    return isNaN(num) || num < 0 || num > 1000 ? 0 : num;
  });
  const pieData = pieParts.map((part: string, index: number) => ({
    name: part,
    value: pieCounts[index] || 0,
  }));

  // Process bar chart data (malfCountChartList)
  const repairStatuses: string[] = (malfCountChartList[0]?.repairStatus || '').split(',').map((status: string) => status.trim()).filter(Boolean);

  // Generate random colors based on the number of unique parts
  const uniqueParts = [...new Set([...pieParts, ...malfCountChartList.map((item: { equipMalfPart: string }) => item.equipMalfPart)])];
  const randomColors = generateRandomColors(uniqueParts.length);
  const colorMap: Record<string, string> = uniqueParts.reduce((acc, part, index) => {
    acc[part] = randomColors[index];
    return acc;
  }, {} as Record<string, string>);

  // Create bar series with dynamic colors
  const barSeries = malfCountChartList.map((item: { equipMalfPart: string; equipMalfCount: string }) => {
    const counts = (item.equipMalfCount || '').split(',').map((count: string) => {
      const num = parseInt(count.trim());
      return isNaN(num) || num < 0 || num > 1000 ? 0 : num;
    });
    return {
      name: item.equipMalfPart,
      type: 'bar',
      data: counts,
      barGap: '10%',
      itemStyle: {
        color: colorMap[item.equipMalfPart] || '#999',
      },
    };
  });

  // Check if all data is zero
  const pieAllZero = pieData.every((d: { value: number }) => d.value === 0) || pieParts.length === 0;
  const barAllZero = barSeries.every((s: { data: number[] }) => s.data.every((d: number) => d === 0)) || repairStatuses.length === 0;

  // Calculate y-axis max for bar chart
  const allCounts = barSeries.flatMap((series: { data: number[] }) => series.data).filter((num: number) => num > 0 && num < 1000);
  const yAxisMax = allCounts.length ? Math.ceil(Math.max(...allCounts) * 1.2) : 10;

  // Pie Chart (Left - Fault Part Distribution)
  const pieOption = {
    title: {
      text: pieAllZero ? `无数据 - ${malfRatioChart.chartTitle || '故障发生部件占比'}` : malfRatioChart.chartTitle || '故障发生部件占比',
      left: 'center',
      top: 10,
      textStyle: {
        fontSize: 18,
        fontWeight: 'bold',
        color: '#333',
      },
    },
    tooltip: {
      trigger: 'item',
      formatter: (params: any) => `${params.name}: ${params.value} (${Math.round(params.percent)}%)`,
    },
    legend: {
      orient: 'horizontal',
      left: 'center',
      top: 'bottom',
      data: pieParts,
      textStyle: { fontSize: 12 },
      itemGap: 10,
      itemWidth: 20,
      itemHeight: 14,
    },
    series: [
      {
        name: '故障占比',
        type: 'pie',
        radius: ['25%', '45%'], // Ring shape
        center: ['50%', '50%'],
        data: pieData,
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)',
          },
        },
        label: {
          show: true,
          formatter: (params: any) => `${params.name}: ${Math.round(params.percent)}%`,
        },
        itemStyle: {
          color: (params: any) => colorMap[params.data.name] || '#999',
        },
      },
    ],
    textStyle: { color: '#666' },
  };

  // Bar Chart (Right)
  const barOption = {
    title: {
      text: barAllZero ? `无数据 - ${chartTitle || '部件故障数统计'}` : chartTitle || '部件故障数统计',
      left: 'center',
      top: 10,
      textStyle: {
        fontSize: 18,
        fontWeight: 'bold',
        color: '#333',
      },
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
      formatter: (params: any) => {
        const status = params[0].name;
        const values = params.map((p: any) => `${p.seriesName}: ${p.value}`).join('<br>');
        return `${status}<br>${values}`;
      },
    },
    legend: {
      data: malfCountChartList.map((item: { equipMalfPart: string }) => item.equipMalfPart),
      top: 70,
      left: 'center',
      textStyle: { fontSize: 12 },
    },
    grid: {
      top: 120,
      bottom: 100,
      left: 80,
      right: 100,
      containLabel: true,
    },
    xAxis: {
      type: 'category',
      name: '维修状态',
      nameTextStyle: { fontSize: 12, padding: [10, 0, 0, 0] },
      data: repairStatuses,
      axisLabel: {
        interval: 0,
        rotate: repairStatuses.length > 3 ? 45 : 0,
        fontSize: 12,
        formatter: (value: string) => (value.length > 10 ? `${value.slice(0, 10)}...` : value),
      },
      axisLine: { lineStyle: { color: '#999' } },
    },
    yAxis: {
      type: 'value',
      name: '故障数',
      nameTextStyle: { fontSize: 12, padding: [0, 0, 10, 0] },
      max: yAxisMax,
      axisLabel: {
        formatter: '{value}', // Whole numbers
        fontSize: 12,
      },
      axisLine: { lineStyle: { color: '#999' } },
      splitLine: { lineStyle: { color: '#eee' } },
    },
    series: barSeries,
    textStyle: { color: '#666' },
  };

  // Apply chart options
  try {
    leftChart.setOption(pieOption, true);
    rightChart.setOption(barOption, true);
    console.log('Charts rendered successfully');
  } catch (error) {
    console.error('Chart rendering failed:', error);
    ElMessage.error('图表渲染失败，请查看控制台');
    leftChart.setOption({ title: { text: '图表渲染失败', left: 'center', top: 'center', textStyle: { fontSize: 24, fontWeight: 'bold', color: '#999' } } });
    rightChart.setOption({ title: { text: '图表渲染失败', left: 'center', top: 'center', textStyle: { fontSize: 24, fontWeight: 'bold', color: '#999' } } });
  }
};

const resizeCharts = () => {
  if (leftChart) {
    leftChart.resize();
  }
  if (rightChart) {
    rightChart.resize();
  }
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
  searchParams.value.ifUnpredicted = value;
  if (!value) {
    inputValue.value = '';
    searchParams.value.day = 0;
  } else {
    inputValue.value = '90';
    searchParams.value.day = 90;
  }
};

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

const _deprecated_handleNodeClick = async (nodeData: any) => {
  selectedUnit.value = nodeData.unitSimpleName;
  const unitPath = nodeData.unitPath ? nodeData.unitPath.toString() : '';
  if (!unitPath) {
    ElMessage.error('请选择有效的单位');
    return;
  }
  searchParams.value.unitPath = unitPath;

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

  if (!params.unitPath || !params.equipModelCode) {
    ElMessage.error('请选择单位/分队和装备型号');
    return;
  }
  const response = await repairpart(params);
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
</style>