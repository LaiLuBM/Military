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
        <el-form :inline="true">
          <el-form-item label="列装时间">
            <el-input v-model="begininput" style="width: 120px" placeholder="最小年限(≥0)" />
            <span style="margin: 0 10px;">年—</span>
            <el-input v-model="endinput" style="width: 120px" placeholder="最大年限(≤20)" />
            <span style="margin: 0 10px;">年</span>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="8">
        <el-form>
          <el-form-item label="装备型号">
            <el-select placeholder="装备型号" style="width: 450px" v-model="searchParams.equipModelCode">
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
import { failurereasonstats } from '@/api/fault/timeanalyse';
import { ElMessage, ElTree } from 'element-plus';
import { equipmodelApi } from '@/api/fault/equip';

// Chart instances
const leftChartInstance = ref<HTMLElement | null>(null);
const rightChartInstance = ref<HTMLElement | null>(null);

// ECharts instances
let leftChart: echarts.ECharts | null = null;
let rightChart: echarts.ECharts | null = null;
// 输入框变化处理
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
const begininput = ref('');
const endinput = ref('');
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
  unit: '',
  unitData: '',
  day: 90,
  isAllRecords: false,
  serviceYearEnd: 20,
  serviceYearStart: 0,
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
const chartDescription = ref<string>('');

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
watch([begininput, endinput], ([newBegin, newEnd]) => {
  searchParams.value.serviceYearStart = newBegin ? parseInt(newBegin) || 0 : 0;
  searchParams.value.serviceYearEnd = newEnd ? parseInt(newEnd) || 0 : 20;
});
const renderChart = async (data: any) => {
  await nextTick();

  // Dispose existing charts
  disposeCharts();

  // Check if chart DOMs are available
  if (!leftChartInstance.value || !rightChartInstance.value) {
    console.error('Chart DOM not found');
    ElMessage.error('图表容器未找到');
    return;
  }

  // Initialize ECharts instances
  try {
    leftChart = echarts.init(leftChartInstance.value);
    rightChart = echarts.init(rightChartInstance.value);
  } catch (error) {
    console.error('Failed to initialize charts:', error);
    ElMessage.error('图表初始化失败');
    return;
  }

  const equipSimpleName = data.stats.equipSimpleName || '未知型号';
  const pieData = data.stats.reasonProportions || {};
  const lineData = data.stats.topThreeReasonFaultsByYear || {};
  const pieTitle = data.description.pietitle || '故障原因占比';
  const lineTitle = data.description.linetitle || '故障原因的故障数统计';
  const description = data.description.description || '';

  // Prepare pie chart data (left chart)
  const pieChartData = Object.keys(pieData).map((key) => ({
    name: key,
    value: pieData[key], // Already in percentage
  }));

  if (!pieChartData.length) {
    leftChart.setOption({
      title: {
        text: '',
        left: 'center',
        top: 'center',
        textStyle: { fontSize: 16, fontWeight: 'bold', color: '#999' },
      },
    });
    chartDescription.value = '暂无故障数据';
    return;
  }

  // Prepare line chart data (right chart)
  const allYears = [...new Set(
    Object.values(lineData)
      .flatMap((reason: any) => Object.keys(reason))
      .map(year => parseInt(year))
      .filter(year => !isNaN(year))
  )].sort((a, b) => a - b);

  if (!allYears.length) {
    rightChart.setOption({
      title: {
        text: '',
        left: 'center',
        top: 'center',
        textStyle: { fontSize: 16, fontWeight: 'bold', color: '#999' },
      },
    });
    chartDescription.value = '暂无故障数据';
    return;
  }

  const reasons = Object.keys(lineData);

  // Function to generate random colors
  const getRandomColor = () => {
    const letters = '0123456789ABCDEF';
    let color = '#';
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  };

  // Generate colors for all reasons
  const reasonColors: Record<string, string> = {};
  reasons.forEach(reason => {
    reasonColors[reason] = getRandomColor();
  });

  const lineSeries = reasons.map((reason) => {
    const values = allYears.map((year) => {
      const value = lineData[reason][year] || 0;
      return value;
    });
    return {
      name: reason,
      type: 'line',
      data: values,
      symbol: 'circle',
      symbolSize: 6,
      itemStyle: {
        color: reasonColors[reason], // Use the generated color
      },
    };
  });

  // Pie Chart (Left, Ring-Shaped)
  const pieOption = {
    title: {
      text: pieTitle,
      left: 'center',
      top: 10,
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold',
      },
    },
    tooltip: {
      trigger: 'item',
      formatter: '{b}: {c}% ({d}%)',
    },
    legend: {
      orient: 'horizontal',
      left: 'center',
      top: 'bottom',
      data: reasons,
      textStyle: { fontSize: 12 },
      itemGap: 10,
      itemWidth: 20,
      itemHeight: 14,
    },
    series: [
      {
        name: equipSimpleName,
        type: 'pie',
        radius: ['25%', '45%'], // Ring shape
        center: ['50%', '50%'],
        data: pieChartData.map(item => ({
          ...item,
          itemStyle: {
            color: reasonColors[item.name] || '#999' // Use the same color as line chart
          }
        })),
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)',
          },
        },
        label: {
          show: false, // Disable labels on pie slices
        },
      },
    ],
  };

  // Line Chart (Right)
  const lineOption = {
    title: {
      text: lineTitle,
      left: 'center',
      top: 10,
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold',
      },
    },
    tooltip: {
      trigger: 'axis',
      formatter: (params: any) => {
        const year = params[0].name;
        let result = `列装时间: ${year}年<br>`;
        params.forEach((item: any) => {
          if (item.value !== null && item.value !== undefined) {
            result += `${item.seriesName}: ${item.value}次<br>`;
          }
        });
        return result;
      },
    },
    legend: {
      data: reasons,
      top: 40,
      left: 'center',
      textStyle: { fontSize: 12 },
    },
    grid: {
      left: '8%',
      right: '8%',
      top: 80,
      bottom: '10%',
      containLabel: true,
    },
    xAxis: {
      type: 'category',
      name: '列装时间',
      data: allYears,
      axisTick: { alignWithLabel: true },
      axisLabel: { interval: 0 },
    },
    yAxis: {
      type: 'value',
      name: '故障数',
      min: 0,
      max: (value: any) => Math.max(5, Math.ceil(value.max / 1) * 1), // Dynamic max
      interval: 1,
      axisLabel: { formatter: '{value}' },
    },
    series: lineSeries,
  };

  try {
    leftChart.setOption(pieOption, true);
    rightChart.setOption(lineOption, true);
    chartDescription.value = description || '显示各故障原因在不同列装时间下的故障数及占比';
  } catch (error) {
    console.error('Chart rendering failed:', error);
    ElMessage.error('图表渲染失败');
    leftChart.setOption({
      title: { text: '图表渲染失败', left: 'center', top: 'center', textStyle: { fontSize: 16 } },
    });
    rightChart.setOption({
      title: { text: '图表渲染失败', left: 'center', top: 'center', textStyle: { fontSize: 16 } },
    });
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
  searchParams.value.isAllRecords = value;
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

// 修改handleNodeClick函数中的equipmodelApi调用
const _deprecated_handleNodeClick = async (nodeData: any) => {
  selectedUnit.value = nodeData.unitSimpleName;
  const unitPath = nodeData.unitPath ? nodeData.unitPath.toString() : '';
  if (!unitPath) {
    ElMessage.error('请选择有效的单位');
    return;
  }
  searchParams.value.unitPath = unitPath;
  searchParams.value.unitData = nodeData.unitCode;
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
  const begin = searchParams.value.serviceYearStart;
  const end = searchParams.value.serviceYearEnd;
  if (begin > 0 && end > 0 && end < begin) {
    ElMessage.warning('最大年限不能小于最小年限');
    return;
  }
  if (begin < 0) {
    ElMessage.warning('最小年限不能小于0');
    return;
  }
  if (end > 20) {
    ElMessage.warning('最大年限不能大于20');
    return;
  }
  const params = {
    days: searchParams.value.day,
    equipModel: searchParams.value.equipModelCode,
    isDetailed: searchParams.value.isDetailed,
    isAllRecords: searchParams.value.isAllRecords,
    serviceYearStart: searchParams.value.serviceYearStart,
    serviceYearEnd: searchParams.value.serviceYearEnd,
    unit: searchParams.value.unitData,
  };

  if (!params.unit || !params.equipModel) {
    ElMessage.error('请选择单位/分队和装备型号');
    return;
  }
  try {
    const response = await failurereasonstats(params);
    if (response.code === 200) {
      ElMessage.success('查询成功');
      localChartData.value = response.data;
    } else {
      ElMessage.error(response.message || '查询失败');
      localChartData.value = null;
    }
  } catch (error) {
    console.error('Query error:', error);
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

/* Ensure chart container is visible */
.chart-container {
  width: 100%;
  height: 500px;
  min-height: 500px;
  visibility: visible !important;
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