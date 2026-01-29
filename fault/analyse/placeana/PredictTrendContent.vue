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
              <el-switch v-model="isInputEnabled" @change="handleSwitchChange" class="-switch-style"></el-switch>
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
        <el-form>
          <el-form-item label="部署地域">
            <el-select v-model="region" placeholder="请选择" style="width: 600px" clearable multiple :multiple-limit="6">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="8">
        <el-form>
          <el-form-item label="装备型号">
            <el-select placeholder="装备型号" style="width: 450px" v-model="searchParams.equipModelCode" multiple
              :multiple-limit="5" clearable>
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
      <div ref="chartInstance" style="width: 100%; height: 550px;"></div>
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
import { ref, onMounted, watch, nextTick } from 'vue';
import * as echarts from 'echarts';
import { unitListApi } from '@/api/fault/unitlist';
import { malfregionlist, malfregionrate } from '@/api/fault/placeanalyse';
import { ElMessage, ElTree } from 'element-plus';
import { equipmodelApi } from '@/api/fault/equip';

// Explicitly type the chart variable as ECharts
let chart: echarts.ECharts | null = null;

const types = ref('');
const typeoptions = [
  { value: 'false', label: '通用化故障预测' },
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
  equipModelCode: [] as string[], // 支持多选
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
const region = ref<string[]>([]);
const options = ref<{ value: string; label: string }[]>([]);
const equipmentTypeOptions = ref<
  { label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
  { label: string; options: { label: string; value: string }[] }[]
>([]);
const chartDescription = ref<string>('');
const chartInstance = ref<HTMLElement | null>(null);
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
const renderChart = (data: any) => {
  if (chart) {
    chart.dispose();
  }
  if (!chartInstance.value) {
    console.error('Chart DOM not found');
    return;
  }
  chart = echarts.init(chartInstance.value);

  if (data && data.rateCountChartList && data.rateCountChartList.length) {
    const { chartTitle, rateCountChartList, description } = data;

    // Update chart description
    chartDescription.value = description || '';

    // Get unique regions for xAxis
    const allRegions = rateCountChartList
      .map((item: any) =>
        typeof item.equipMalfRegion === 'string'
          ? item.equipMalfRegion.split(',')
          : item.equipMalfRegion || []
      )
      .flat();
    const uniqueRegions = [...new Set(allRegions)];

    // Prepare series data for each equipSimpleName
    const series = rateCountChartList.map((item: any) => {
      const regions = typeof item.equipMalfRegion === 'string'
        ? item.equipMalfRegion.split(',')
        : item.equipMalfRegion || [];
      const rates = typeof item.equipMalfRate === 'string'
        ? item.equipMalfRate.split(',').map((rate: string) => parseFloat((Number(rate) * 100).toFixed(2)))
        : (item.equipMalfRate || []).map((rate: number) => parseFloat((rate * 100).toFixed(2)));
      // Map rates to uniqueRegions, fill missing regions with 0
      const data = uniqueRegions.map((region) => {
        const index = regions.indexOf(region);
        return index !== -1 && index < rates.length ? rates[index] : 0;
      });

      return {
        name: item.equipSimpleName,
        type: 'line',
        data,
        // smooth: true, // Smooth line for better appearance
        symbol: 'circle', // Show points on the line
        symbolSize: 8,
      };
    });

    chart.setOption({
      title: {
        text: chartTitle || '故障概率趋势',
        left: 'center',
        top: 10,
        textStyle: {
          fontSize: 20,
          fontWeight: 'bold',
        },
      },
      legend: {
        data: rateCountChartList.map((item: any) => item.equipSimpleName),
        top: 70,
        left: 'center',
      },
      tooltip: {
        trigger: 'axis',

      },
      xAxis: {
        type: 'category',
        name: '部署地域',
        data: uniqueRegions,
        axisLabel: {
          rotate: 0, // Rotate labels to avoid overlap
          interval: 0, // Show all labels
          width: 80, // 设置标签宽度
        },
      },
      yAxis: {
        type: 'value',
        name: '故障发生概率(%)',
        min: 0,
        max: 100,
        interval: 20,
        axisLabel: {
          formatter: '{value}',
        },
      },
      series,
      grid: {
        top: 120, // Increase space for legend
        bottom: 100, // Extra space for rotated labels
        left: 80,
        right: 100,
      },
    });
  } else {
    chartDescription.value = '';
  }
};

onMounted(async () => {
  const { data } = await unitListApi();
  treeData.value = data;
  await nextTick();
  renderChart(localChartData.value); // Initial render (empty chart if no data)
});

watch(localChartData, (newData) => {
  renderChart(newData); // Update chart when query data changes
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
          searchParams.value.equipModelCode = [];
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
    searchParams.value.equipModelCode = [];
  }
  const response2 = await malfregionlist(unitPath);
  if (response2 && response2.code === 200) {
    options.value = Array.isArray(response2.data)
      ? response2.data.map((item: string) => ({ label: item, value: item }))
      : [];
  }
};
const handleQuery = async () => {
  const params = {
    day: searchParams.value.day,
    equipModelCodeList: searchParams.value.equipModelCode.join(','),
    ifPredicted: isInputEnabled.value,
    isDetailed: searchParams.value.isDetailed,
    regionList: region.value.join(','),
    unitPath: searchParams.value.unitPath,
  };

  if (!params.unitPath || !params.equipModelCodeList) {
    ElMessage.warning('请选择单位/分队和装备型号');
    return;
  }
  if (!params.regionList) {
    ElMessage.warning('请选择部署地域');
    return;
  }
  const response = await malfregionrate(params);
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