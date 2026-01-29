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
      <el-col :span="14">
        <el-form>
          <el-form-item label="隶属单位">
            <el-select v-model="selectedCompareUnits" placeholder="请先选择单位/分队" style="width: 650px" multiple clearable
              :multiple-limit="5" @visible-change="handleUnitSelectVisibleChange" :popper-append-to-body="false">
              <el-option v-if="!selectedUnit" :value="null" disabled>
                <div style="color: #999; text-align: center;">请先选择单位/分队</div>
              </el-option>
              <el-option v-else :value="selectedUnit2" style="height: auto; padding: 0">
                <el-tree ref="unitTreeRef" :data="unitTreeData" :props="defaultTreeProps" node-key="unitCode"
                  :default-expand-all="false" @node-click="handleUnitTreeNodeClick" class="sub-tree"></el-tree>
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="6">
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
        <div ref="leftChartInstance" style="width: 40%; height: 100%;"></div>
        <div ref="rightChartInstance" style="width: 60%; height: 100%;"></div>
      </div>
    </el-row>
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
import { malfunitpart } from '@/api/fault/unitanalyse';
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
const selectedUnit2 = ref<string>('');
const selectedCompareUnits = ref<string[]>([]);
const selectedUnitPaths = ref<string[]>([]);
const localChartData = ref(null);

const searchParams = ref({
  equipModel: '',
  equipCode: '',
  equipModelCode: '',
  day: 90,
  ifUnpredicted: false,
  unitPath: '',
  unit: '',
  current: 1,
  size: 10,
  isDetailed: false,
});

const unitTreeRef = ref<InstanceType<typeof ElTree>>();
const unitTreeData = ref<any[]>([]);
const treeData = ref([]);
const defaultTreeProps = {
  children: 'childrenUnits',
  label: 'unitSimpleName',
};

const isInputEnabled = ref(false);
const inputValue = ref('90');
const equipmentTypeOptions = ref<
  { label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
  { label: string; options: { label: string; value: string }[] }[]
>([]);

const chartDescription = ref<string>('');
const leftChartInstance = ref<HTMLElement | null>(null);
const rightChartInstance = ref<HTMLElement | null>(null);

let leftChart: echarts.ECharts | null = null;
let rightChart: echarts.ECharts | null = null;

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
  try {
    if (!leftChartInstance.value || !rightChartInstance.value) {
      ElMessage.error('图表容器未找到');
      return;
    }
    leftChart = echarts.init(leftChartInstance.value);
    rightChart = echarts.init(rightChartInstance.value);
  } catch (error) {
    ElMessage.error('图表初始化失败');
    return;
  }

  const malfRatioChart = data?.malfRatioChart;
  const malfCountChartList = data?.malfCountChartList || [];
  const description = data?.description || '';

  chartDescription.value = description === '。' ? '暂无故障概率。' : (description || '');

  // Pie Chart (malfRatioChart)
  const reasons = malfRatioChart.equipMalfPart.includes(',')
    ? malfRatioChart.equipMalfPart.split(',')
    : [malfRatioChart.equipMalfPart];
  const counts = malfRatioChart.equipMalfCount.includes(',')
    ? malfRatioChart.equipMalfCount.split(',').map((val: string) => {
      const num = Number(val);
      return isNaN(num) || num < 0 || num > 1000 ? 0 : num;
    })
    : [Number(malfRatioChart.equipMalfCount)].map((num) =>
      isNaN(num) || num < 0 || num > 1000 ? 0 : num
    );
  const pieData = reasons.map((reason: string, index: number) => ({
    name: reason,
    value: counts[index],
  }));

  // Line Chart (malfCountChartList)
  const units = malfCountChartList[0]?.equipMalfUnit.split(',') || [];
  const lineSeries = malfCountChartList.map((item: any) => {
    const counts = item.equipMalfCount.split(',').map((val: string) => {
      const num = Number(val);
      return isNaN(num) || num < 0 || num > 1000 ? 0 : num;
    });
    return {
      name: item.equipMalfPart, // Use equipMalfPart for legend
      type: 'line',
      data: counts,
    };
  });

  const generateRandomColors = (count: number) => {
    const colors = [];
    for (let i = 0; i < count; i++) {
      // 生成鲜艳的颜色（避免太浅的颜色）
      const hue = Math.floor(Math.random() * 360);
      const saturation = 70 + Math.floor(Math.random() * 30); // 70-100%
      const lightness = 40 + Math.floor(Math.random() * 30); // 40-70%
      colors.push(`hsl(${hue}, ${saturation}%, ${lightness}%)`);
    }
    return colors;
  };

  const pieOption = {
    color: generateRandomColors(reasons.length),
    title: {
      text: malfRatioChart.chartTitle,
      left: 'center',
      top: 10,
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold',
      },
    },
    tooltip: {
      trigger: 'item',
      formatter: '{a} <br/>{b}: {c} ({d}%)',
    },
    legend: {
      orient: 'horizontal',
      left: 'center',
      top: 'bottom',
      data: reasons,
      textStyle: {
        fontSize: 12,
      },
      itemGap: 10,
      itemWidth: 20,
      itemHeight: 14,
    },
    series: [
      {
        name: '故障占比',
        type: 'pie',
        radius: '50%', // Full pie chart, no inner radius
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
          formatter: '{b}: {d}%',
        },
      },
    ],
  };

  const lineOption = {
    color: generateRandomColors(malfCountChartList.length),
    title: {
      text: data.chartTitle,
      left: 'center',
      top: 10,
      textStyle: {
        fontSize: 16,
        fontWeight: 'bold',
      },
    },
    tooltip: {
      trigger: 'axis',
    },
    legend: {
      data: malfCountChartList.map((item: any) => item.equipMalfPart),
      top: 60,
      left: 'center',
    },
    grid: {
      left: '10%',
      right: '10%',
      top: '20%',
      bottom: '15%',
    },
    xAxis: {
      type: 'category',
      name: '隶属单位',
      nameGap: 30,
      data: units,
      axisLabel: {
        interval: 0,
        rotate: 0, // Rotate labels for better readability
      },
    },
    yAxis: {
      type: 'value',
      name: '故障数',
      nameGap: 50,
    },
    series: lineSeries,
  };

  try {
    leftChart.setOption(pieOption, true);
    rightChart.setOption(lineOption, true);
    console.log('Charts rendered successfully');
  } catch (error) {
    ElMessage.error('图表渲染失败，请查看控制台');
    console.error(error);
    leftChart.setOption({ title: { text: '图表渲染失败' } });
    rightChart.setOption({ title: { text: '图表渲染失败' } });
  }
};

const unitOptions = ref<{ unitCode: string; unitSimpleName: string; unitPath: string }[]>([]);
const handleUnitTreeNodeClick = async (nodeData: any) => {
  const { unitCode, unitSimpleName } = nodeData;
  const index = selectedCompareUnits.value.indexOf(unitSimpleName);
  if (index > -1) {
    selectedCompareUnits.value.splice(index, 1);
    selectedUnitPaths.value.splice(index, 1);
    unitOptions.value = unitOptions.value.filter((unit) => unit.unitCode !== unitSimpleName);
  } else if (selectedCompareUnits.value.length < 6) {
    selectedCompareUnits.value.push(unitSimpleName);
    selectedUnitPaths.value.push(unitCode);
  } else {
    ElMessage.warning('最多只能选择6个单位进行对比');
    return;
  }
};

const handleUnitSelectVisibleChange = (visible: boolean) => {
  if (visible) {
    if (!selectedCompareUnits.value.length) {
      unitTreeRef.value?.setCurrentKey();
    }
  }
};

// Find children of a unit by unitCode in treeData
const findUnitChildren = (unitCode: string, data: any[]): any[] => {
  for (const unit of data) {
    if (unit.unitCode === unitCode) {
      return unit.childrenUnits || [];
    }
    if (unit.childrenUnits && unit.childrenUnits.length > 0) {
      const found = findUnitChildren(unitCode, unit.childrenUnits);
      if (found.length > 0) return found;
    }
  }
  return [];
};

onMounted(async () => {
  try {
    const { data } = await unitListApi();
    unitTreeData.value = JSON.parse(JSON.stringify(data));
    treeData.value = JSON.parse(JSON.stringify(data));
    await nextTick();
    renderChart(localChartData.value);
  } catch (error) {
    console.error('初始化失败:', error);
  }
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

// 修改handleNodeClick函数中的equipmodelApi调用
const _deprecated_handleNodeClick = async (nodeData: any) => {
  selectedUnit.value = nodeData.unitSimpleName;
  const unitPath = nodeData.unitPath ? nodeData.unitPath.toString() : '';
  if (!unitPath) {
    ElMessage.error('请选择有效的单位');
    return;
  }
  searchParams.value.unitPath = unitPath;

  // 更新 unitTreeData 为当前选中单位的子集
  const children = findUnitChildren(nodeData.unitCode, treeData.value);
  unitTreeData.value = children.length > 0 ? JSON.parse(JSON.stringify(children)) : [];

  // 清除之前的比较单位选择
  selectedCompareUnits.value = [];
  selectedUnitPaths.value = [];

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
    isDetailed: searchParams.value.isDetailed,
    unitList: selectedCompareUnits.value.join(','), // Use unitCode instead of unitName
  };

  if (!params.unitList || !params.equipModelCode) {
    ElMessage.error('请选择单位/分队和装备型号');
    return;
  }
  const response = await malfunitpart(params);
  if (response.code === 200) {
    ElMessage.success('查询成功');
    localChartData.value = response.data;
  } else {
    ElMessage.error(response.message || '查询失败');
  }
};

onUnmounted(() => {
  disposeCharts();
});


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

.chart-description {
  text-align: left;
  font-size: 14px;
  color: #3968ce;
  padding: 10px;
  background-color: #bcd5fc;
  border-radius: 4px;
}

.sub-tree {
  ::v-deep .el-tree-node__content:hover {
    background-color: #e6f7ff !important;
  }

  ::v-deep .el-tree-node.is-current>.el-tree-node__content {
    background-color: #e6f7ff;
  }

  font-weight: normal;
}
</style>