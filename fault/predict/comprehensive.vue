<template>
    <div>
    <el-row :gutter="20">
        <el-col :span="24">
            <el-card>
                <el-row :gutter="20" align="middle">
                    <el-col :span="6">
                        <el-form>
                            <el-form-item label="单位/分队" prop="unitPath">
                                            <el-tree-select v-model="selectedUnit" :data="treeData" :props="treeProps" node-key="unitCode"
                                              placeholder="请选择单位（可搜索）" filterable clearable style="width: 100%" check-strictly
                                              @change="handleUnitSelect" />
                                          </el-form-item>
                        </el-form>
                    </el-col>
                    <el-col :span="6">
                        <el-form>
                            <el-form-item label="装备型号">
                                <el-select clearable placeholder="装备型号" v-model="searchParams.equipModelCode">
                                    <el-option-group v-for="group in equipmentTypeOptions" :key="group.label"
                                        :label="group.label">
                                        <el-option v-for="option in group.options" :key="option.value"
                                            :label="option.label" :value="option.value"></el-option>
                                    </el-option-group>
                                </el-select>
                            </el-form-item>
                        </el-form>
                    </el-col>
                    <el-col :span="12">
                        <el-form>
                            <el-form-item>
                                <div class="flex-container">
                                    <el-input v-model="inputValue" placeholder="输入数字即可" class="input-style"
                                        @input="updateSearchParamsDay">
                                        <template #prepend>
                                            <span>最近</span>
                                        </template>
                                        <template #append>
                                            <span>天内的预测和诊断</span>
                                        </template>
                                    </el-input>
                                    <el-button type="primary" class="ml" @click="loadData">统计</el-button>
                                </div>
                            </el-form-item>
                        </el-form>
                    </el-col>
                </el-row>
            </el-card>
        </el-col>
    </el-row>
    <el-row style="margin-top: 10px;">
        <el-col :span="24">
            <el-card>
                <el-row>
                    <el-col :span="8">
                        <p style="font-size: 20px;font-weight: bold;">总体统计结果</p>
                    </el-col>
                    <el-col :span="16"></el-col>
                </el-row>
                <el-row style="margin-top: 10px;">
                    <el-col :span="10">
                        <div
                            style="height: 80px;background-color:rgb(245, 247, 149);border-radius: 20px;width: 250px;padding-top: 20px;">
                            <div style="font-size: 18px;text-align: center;">装备型号数:</div>
                            <div style="font-size: 18px;text-align: center;"><span
                                    style="font-size: 22px;font-weight: bold;color: orange;">{{ modelNum }}</span>种
                            </div>
                        </div>
                    </el-col>
                    <el-col :span="10">
                        <div
                            style="height: 80px;background-color: rgb(157, 247, 149);border-radius: 20px;width: 250px;padding-top: 20px;">
                            <div style="font-size: 18px;text-align: center;">总装备数:</div>
                            <div style="font-size: 18px;text-align: center;"><span
                                    style="font-size: 22px;font-weight: bold;color: green;">{{ equipNum }}</span>台
                            </div>
                        </div>
                    </el-col>
                    <el-col :span="4">
                        <div
                            style="height: 80px;background-color: rgb(156, 149, 247);border-radius: 20px;width: 250px;padding-top: 20px;">
                            <div style="font-size: 18px;text-align: center;">精细化装备数:</div>
                            <div style="font-size: 18px;text-align: center;"><span
                                    style="font-size: 22px;font-weight: bold;color: blue;">{{ equipDetailNum
                                    }}</span>台
                            </div>
                        </div>
                    </el-col>
                </el-row>
            </el-card>
        </el-col>
    </el-row>

    <!-- 2x3 网格布局，每个图表使用 el-card 包裹 -->
    <el-row style="margin-top: 10px;" :gutter="20">
        <!-- 第一行 -->
        <el-col :span="8">
            <el-card>
                <el-button style="font-size: 24px; font-weight: bold;" @click="handleGeneral" text>通用化故障预测</el-button>
                <p style="font-size: 16px; color: #666;margin-left: 160px;margin-top: 20px;">已预测/未预测: <span
                        style="color: #408de6;border-bottom: #408de6 1px solid;">{{ equipMalfCount }}/{{
                            disequipMalfCount
                        }}</span>
                </p>

                <div class="chart-container" ref="chartRef1"></div>
            </el-card>
        </el-col>
        <el-col :span="8">
            <el-card>
                <el-button style="font-size: 24px; font-weight: bold;" @click="handleDetail" text>精细化故障预测</el-button>
                <p style="font-size: 16px; color: #666;margin-left: 160px;margin-top: 20px;">已预测/未预测: <span
                        style="color: #408de6;border-bottom: #408de6 1px solid;">{{ equipMalfDetailCount }}/{{
                            disequipMalfDetailCount }}</span>
                </p>
                <div class="chart-container" ref="chart2"></div>
            </el-card>
        </el-col>
        <el-col :span="8">
            <el-card style="height: 425px;">
                <el-button style="font-size: 24px; font-weight: bold;margin-bottom: 40px;" @click="handleType"
                    text>型号故障预测</el-button>
                <div class="chart-container" ref="chart3"></div>
            </el-card>
        </el-col>
    </el-row>
    <div style="width: 1200px;margin: auto;">
        <el-row style="margin-top: 20px;" :gutter="20">
            <!-- 第二行 -->
            <el-col :span="12">
                <el-card>
                    <el-row>
                        <el-col :span="6">
                            <el-button style="font-size: 24px; font-weight: bold;" @click="handleDiag"
                                text>精细化诊断</el-button>
                        </el-col>
                        <el-col :span="18"></el-col>
                        <el-col :span="8">
                            <p style="font-size: 14px; color: #666;margin-top: 10px;">已诊断:<span
                                    style="color: #408de6;border-bottom: #408de6 1px solid;">
                                    {{ equipDiagCount }}</span></p>
                            <el-table border :data="tableData" style="margin-top: 20px;">
                                <el-table-column label="部件名称" prop="name" :show-overflow-tooltip="true"></el-table-column>
                                <el-table-column label="故障数" prop="number" :show-overflow-tooltip="true"></el-table-column>
                            </el-table>
                        </el-col>
                        <el-col :span="16">
                            <div class="chart-container" ref="chart4" style="height: 340px;"></div>
                        </el-col>
                    </el-row>
                </el-card>
            </el-col>
            <el-col :span="12">
                <el-card style="height: 410px;">
                    <el-button style="font-size: 24px; font-weight: bold;" @click="handleRepair" text>修理时机预测</el-button>
                    <p style="font-size: 14px; color: #666;margin-left: 160px;">已预测/未预测: <span
                            style="color: #408de6;border-bottom: #408de6 1px solid;">{{ equipRepairCount }}/{{
                                disequipRepairCount }}</span></p>
                    <p style="font-size: 14px; color: #666;">大修装备数:<span
                            style="color: #408de6;border-bottom: #408de6 1px solid;margin-right: 80px;">{{
                                bigEquipRepairCount
                            }}</span>中修装备数:<span
                            style="color: #408de6;border-bottom: #408de6 1px solid;margin-right: 80px;">{{
                                mediumEquipRepairCount }}</span>
                        小修装备数:<span style="color: #408de6;border-bottom: #408de6 1px solid;">{{ smallEquipRepairCount
                            }}</span>
                    </p>
                    <div class="chart-container" ref="chart5"></div>
                </el-card>
            </el-col>
            <!-- <el-col :span="8">
            <el-card>
                <el-row>
                    <el-col :span="6">
                        <el-button style="font-size: 24px; font-weight: bold;" text>交互式诊断</el-button>
                    </el-col>
                    <el-col :span="18"></el-col>
                    <el-col :span="8">
                        <p style="font-size: 14px; color: #666;">已诊断: 200/50</p>
                        <el-table border :data="tableData" style="margin-top: 20px;">
                            <el-table-column label="部件名称" prop="name" :show-overflow-tooltip="true"></el-table-column>
                            <el-table-column label="故障数" prop="number" :show-overflow-tooltip="true"></el-table-column>
                        </el-table>
                    </el-col>
                    <el-col :span="16">
                        <div class="chart-container" ref="chart6" style="height: 340px;"></div>
                    </el-col>
                </el-row>
            </el-card>
        </el-col> -->
        </el-row>
    </div>
    </div>

</template>

<script lang="ts" setup>
import { ref, onMounted, reactive, watch } from 'vue';
import { unitListApi } from '@/api/fault/unitlist';
import { ElMessage, ElTree } from 'element-plus';
import { equipmodelApi } from '@/api/fault/equipmodel';
import { useChart } from '@/hooks/useChart';
import { comprehensiveInfoApi } from '@/api/fault/comprehensiveinfo';
import { useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';

const selectedUnit = ref('中部战区01军01旅'); // 默认选中“中部战区01军01旅”

// 搜索参数
interface SearchType {
    day: number;
    equipModelCode?: string;
    unitPath: string;
}

const searchParams = ref<SearchType>({
    day: 999,
    equipModelCode: "",
    unitPath: '',
});

const tableData = ref()

// 统计数据变量
const modelNum = ref('');
const equipNum = ref('');
const equipDetailNum = ref('');
const equipMalfCount = ref('');
const disequipMalfCount = ref('');
const equipMalfDetailCount = ref('')
const disequipMalfDetailCount = ref('')
const chartData = ref<any>(null);
const detailchartData = ref<any>(null);
const repairchartData = ref<any>(null);
const diagnosechartData = ref<any>(null);
const modelchartData = ref<any>(null);
const equipRepairCount = ref('')
const disequipRepairCount = ref('')
const bigEquipRepairCount = ref('')
const mediumEquipRepairCount = ref('')
const smallEquipRepairCount = ref('')
const equipDiagCount = ref('')

const chartRef1 = ref<HTMLElement | null>(null);
const chart2 = ref<HTMLElement | null>(null);
const chart3 = ref<HTMLElement | null>(null);
const chart4 = ref<HTMLElement | null>(null);
const chart5 = ref<HTMLElement | null>(null);
const chart6 = ref<HTMLElement | null>(null);

// 树形数据
interface Tree {
    unitCode: number;
    parentUnitCode: any;
    unitSimpleName: string;
    unitPath: number;
    childrenUnits?: Tree[];
}

const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref<Tree[]>([]);
const defaultProps = {
    children: 'childrenUnits',
    label: 'unitSimpleName',
};

const router = useRouter();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;
// 跳转到通用故障诊断页面
const handleGeneral = () => {
    // addTab("通用化故障预测", "/predict/generalfaultdiagnosis", "Warning");
    // setCurrentTab("通用化故障预测", "/predict/generalfaultdiagnosis");
    router.push({
        path: '/4/4.2',
        query: {
            unitPath: searchParams.value.unitPath,
            equipModelCode: searchParams.value.equipModelCode,
            day: searchParams.value.day.toString()
        }
    });
};
const handleDetail = () => {
    // addTab("精细化故障预测", "/predict/finingfaultpredict", "VideoCamera");
    // setCurrentTab("精细化故障预测", "/predict/finingfaultpredict");
    router.push({
        path: '/4/4.3',
        query: {
            unitPath: searchParams.value.unitPath,
            equipModelCode: searchParams.value.equipModelCode,
            day: searchParams.value.day.toString()
        }
    });
};

const handleType = () => {
    // addTab("型号故障预测", "/predict/typefaultpredict", "Warning");
    // setCurrentTab("型号故障预测", "/predict/typefaultpredict");
    router.push({
        path: '/predict/typefaultpredict',
        query: {
            unitPath: searchParams.value.unitPath,
            equipModelCode: searchParams.value.equipModelCode,
            day: searchParams.value.day.toString()
        }
    });
};
const handleDiag = () => {
    // addTab("精细化诊断", "/predict/refineddiagnosis", "Warning");
    // setCurrentTab("精细化诊断", "/predict/refineddiagnosis");
    router.push("/predict/finingfaultpredict");
    router.push({
        path: '/predict/refineddiagnosis',
        query: {
            unitPath: searchParams.value.unitPath,
            equipModelCode: searchParams.value.equipModelCode,
            day: searchParams.value.day.toString()
        }
    });
};
const handleRepair = () => {
    // addTab("修理时机预测", "/predict/timepredict", "Warning");
    // setCurrentTab("修理时机预测", "/predict/timepredict");
    router.push({
        path: '/predict/timepredict',
        query: {
            unitPath: searchParams.value.unitPath,
            equipModelCode: searchParams.value.equipModelCode,
            day: searchParams.value.day.toString()
        }
    });
};

// 递归查找“中部战区01军01旅”的节点
const findUnitNode = (nodes: Tree[], targetName: string): Tree | undefined => {
    for (const node of nodes) {
        if (node.unitSimpleName === targetName) {
            return node;
        }
        if (node.childrenUnits) {
            const found = findUnitNode(node.childrenUnits, targetName);
            if (found) return found;
        }
    }
    return undefined;
};

const loadData = async () => {
    try {
        const params = { ...searchParams.value };
        if (params.equipModelCode === '') {
            delete params.equipModelCode;
        }
        console.log('发送的参数:', params);
        const response = await comprehensiveInfoApi(params);
        console.log('接口返回的数据:', response);
        if (response.code === 200) {
            modelNum.value = response.data.modelNum || 0;
            equipNum.value = response.data.equipNum || 0;
            equipDetailNum.value = response.data.equipDetailNum || 0;
            equipMalfCount.value = response.data.equipMalfChart.equipMalfCount || 0;
            disequipMalfCount.value = response.data.equipMalfChart.disequipMalfCount || 0;
            equipMalfDetailCount.value = response.data.equipMalfDetailChart.equipMalfDetailCount || 0;
            disequipMalfDetailCount.value = response.data.equipMalfDetailChart.disequipMalfDetailCount || 0;
            chartData.value = response.data.equipMalfChart;
            detailchartData.value = response.data.equipMalfDetailChart;
            repairchartData.value = response.data.equipRepairChart;
            equipRepairCount.value = response.data.equipRepairChart.equipRepairCount || 0;
            disequipRepairCount.value = response.data.equipRepairChart.disequipRepairCount || 0;
            bigEquipRepairCount.value = response.data.equipRepairChart.bigEquipRepairCount || 0;
            mediumEquipRepairCount.value = response.data.equipRepairChart.mediumEquipRepairCount || 0;
            smallEquipRepairCount.value = response.data.equipRepairChart.smallEquipRepairCount || 0;
            diagnosechartData.value = response.data.equipDiagChart;
            const partCountTopFour = response.data.equipDiagChart.partCountTopFour;
            tableData.value = Object.entries(partCountTopFour).map(([name, number]) => ({
                name,
                number
            }));
            equipDiagCount.value = response.data.equipDiagChart.equipDiagCount || 0;
            modelchartData.value = response.data.equipModelMalfChart;
        } else {
            ElMessage.error('查询失败: ' + response.message);
        }
    } catch (error) {
        console.error('请求失败:', error);

    }
};

// 开关和输入框逻辑
const inputValue = ref('999'); // 默认值 999 天

// 更新 searchParams.day
const updateSearchParamsDay = () => {
  const value = parseInt(inputValue.value) || 10;
  // 添加数值范围检查
  if (value > 999) {
    ElMessage.warning('输入的天数不能超过999天');
    inputValue.value = '999'; // 将输入值重置为最大值
    searchParams.value.day = 999;
    return;
  }
  searchParams.value.day = value;
};

// 装备型号下拉框选项（分组格式）
const equipmentTypeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);

// 如果 equipmentCodeOptions 也需要分组
const equipmentCodeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);
const _deprecated_handleNodeClick = async (nodeData: Tree) => {
    // Clear previous selections
    selectedUnit.value = nodeData.unitSimpleName;
    searchParams.value.equipModelCode = ''; // Clear the selected equipment model

    console.log('点击的节点数据:', nodeData);
    const unitPath = nodeData.unitPath.toString();
    searchParams.value.unitPath = unitPath;

    try {
        const response = await equipmodelApi(unitPath);
        console.log('装备型号数据:', response);
        if (response.code === 200) {
            console.log('请求成功:', response.message);
            if (response.data && response.data.categories) {
                // Reset and update equipment type options
                const groupedOptions: { label: string; options: { label: string; value: string }[] }[] = [];
                for (const category in response.data.categories) {
                    if (Array.isArray(response.data.categories[category])) {
                        const categoryOptions = response.data.categories[category].map((item: any) => ({
                            label: item.equipSimpleName,
                            value: item.equipModelCode,
                        }));
                        groupedOptions.push({
                            label: category,
                            options: categoryOptions,
                        });
                    }
                }

                equipmentTypeOptions.value = groupedOptions;
                equipmentCodeOptions.value = groupedOptions;
            } else {
                console.error('数据格式错误: categories 不存在或格式不正确');
                equipmentTypeOptions.value = [];
                equipmentCodeOptions.value = [];
            }
        } else {
            console.error('请求失败:', response.message);
            equipmentTypeOptions.value = [];
            equipmentCodeOptions.value = [];
        }
    } catch (error) {
        console.error('请求装备型号数据失败:', error);
        equipmentTypeOptions.value = [];
        equipmentCodeOptions.value = [];
    }

    // Optionally trigger data reload if you want
    // loadData();
};

watch(selectedUnit, (newVal) => {
    if (!newVal) {
        // Unit selection was cleared
        searchParams.value.unitPath = '';
        searchParams.value.equipModelCode = '';
        equipmentTypeOptions.value = [];
        equipmentCodeOptions.value = [];
    }
});

// 获取树形数据并设置默认值
onMounted(async () => {
    try {
        const { data } = await unitListApi();
        treeData.value = data;

        // 查找“中部战区01军01旅”节点
        const centralUnit = findUnitNode(treeData.value, '中部战区01军01旅');
        if (centralUnit) {
            selectedUnit.value = centralUnit.unitSimpleName;
            searchParams.value.unitPath = centralUnit.unitPath.toString();
            console.log('默认选中“中部战区01军01旅”:', centralUnit);
            // 可选：加载装备型号（如果需要显示下拉框选项）
            await handleNodeClick(centralUnit);
        } else {
            console.warn('未找到“中部战区01军01旅”节点');
            ElMessage.warning('未找到“中部战区01军01旅”，请检查数据');
        }
    } catch (error) {
        console.error('Failed to fetch unit list:', error);
        ElMessage.error('加载单位数据失败');
    }
});

// 默认图表配置（当数据未加载时使用）
const defaultChartOptions = reactive({
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'cross',
            label: {
                backgroundColor: '#6a7985'
            }
        }
    },
    legend: {
        data: ['可能故障的装备数', '可能故障装备的的故障率']
    },
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis: [
        {
            type: 'category',
            boundaryGap: false,
            data: ['暂无数据']
        }
    ],
    yAxis: [
        {
            type: 'value',
            name: '故障数量',
            position: 'left',
            interval: 1, // 设置间隔为1
            axisLabel: {
                formatter: '{value}' // 强制显示整数
            }
        },
        {
            type: 'value',
            name: '故障率 (%)',
            position: 'right',
            min: 0,
            max: 100,
            interval: 20,
            axisLabel: {
                formatter: '{value} '
            }
        }
    ],
    series: [
        {
            name: '可能故障的装备数',
            type: 'line',
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        },
        {
            name: '可能故障装备的的故障率',
            type: 'line',
            yAxisIndex: 1,
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        }
    ]
});

// 默认图表配置（当数据未加载时使用）
const detaildefaultChartOptions = reactive({
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'cross',
            label: {
                backgroundColor: '#6a7985'
            }
        }
    },
    legend: {
        data: ['可能故障的装备数', '可能故障装备的的故障率']
    },
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis: [
        {
            type: 'category',
            boundaryGap: false,
            data: ['暂无数据']
        }
    ],
    yAxis: [
        {
            type: 'value',
            name: '故障数量',
            position: 'left',
            interval: 1, // 设置间隔为1
            axisLabel: {
                formatter: '{value}' // 强制显示整数
            }
        },
        {
            type: 'value',
            name: '故障率 (%)',
            position: 'right',
            min: 0,
            max: 100,
            interval: 20,
            axisLabel: {
                formatter: '{value} '
            }
        }
    ],
    series: [
        {
            name: '可能故障的装备数',
            type: 'line',
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        },
        {
            name: '可能故障装备的的故障率',
            type: 'line',
            yAxisIndex: 1,
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        }
    ]
});

const repairdefaultChartOptions = reactive({
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'cross',
            label: {
                backgroundColor: '#6a7985'
            }
        }
    },
    legend: {
        data: ['计划送修装备数']
    },
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    grid: {
        left: '3%',
        right: '7%',
        bottom: '10%',
        containLabel: true
    },
    xAxis: [
        {
            type: 'category',
            boundaryGap: false,
            data: ['暂无数据']
        }
    ],
    yAxis: [
        {
            type: 'value',
            name: '故障数量',
            position: 'left',
            interval: 1, // 设置间隔为1
            axisLabel: {
                formatter: '{value}' // 强制显示整数
            }
        },
        {
            type: 'value',
            name: '故障率 (%)',
            position: 'right',
            min: 0,
            max: 100,
            interval: 20,
            axisLabel: {
                formatter: '{value} '
            }
        }
    ],
    series: [
        {
            name: '计划送修装备数',
            type: 'line',
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        },
    ]
});

const diagnosedefaultChartOptions = reactive({
    title: {
        text: '故障分布',
        left: 'center',
        top: '5%',
        textStyle: {
            fontSize: 16,
            fontWeight: 'bold'
        }
    },
    tooltip: {
        trigger: 'item'
    },
    legend: {
        orient: 'horizontal',
        top: '15%',
        // bottom:'15%',
        left: 'center',
        itemGap: 5,
        textStyle: {
            fontSize: 12
        }
    },
    series: [
        {
            type: 'pie',
            radius: '50%',
            center: ['50%', '65%'],
            avoidLabelOverlap: true,
            itemStyle: {
                borderRadius: 10,
                borderColor: '#fff',
                borderWidth: 2
            },
            emphasis: {
                label: {
                    show: true,
                    fontSize: 20,
                    fontWeight: 'bold'
                }
            },
            data: [
                { value: 1048, name: '部件1' },
                { value: 735, name: '部件2' },
                { value: 580, name: '部件3' },
                { value: 484, name: '部件4' },
                { value: 300, name: '其他' }
            ]
        }
    ]

});
const modeldefaultChartOptions = reactive({
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'cross',
            label: {
                backgroundColor: '#6a7985'
            }
        }
    },
    legend: {
        data: ['可能故障的装备数', '可能故障装备的的故障率']
    },
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis: [
        {
            type: 'category',
            boundaryGap: false,
            data: ['暂无数据']
        }
    ],
    yAxis: [
        {
            type: 'value',
            name: '故障数量',
            position: 'left',
            interval: 1, // 设置间隔为1
            axisLabel: {
                formatter: '{value}' // 强制显示整数
            }
        },
        {
            type: 'value',
            name: '故障率 (%)',
            position: 'right',
            min: 0,
            max: 100,
            interval: 20,
            axisLabel: {
                formatter: '{value} '
            }
        }
    ],
    series: [
        {
            name: '可能故障的装备数',
            type: 'line',
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        },
        {
            name: '可能故障装备的的故障率',
            type: 'line',
            yAxisIndex: 1,
            stack: 'Total',
            areaStyle: {},
            emphasis: {
                focus: 'series'
            },
            data: [0]
        }
    ]
});

const setChartData1 = async () => {
    // 如果 chartData 为空，返回默认配置
    if (!chartData.value) {
        return defaultChartOptions;
    }
    const { equipMalfDate, equipMalfStatis, equipMalfRate } = chartData.value;
    const dateArray = equipMalfDate.split(','); // 拆分为数组
    const statisArray = equipMalfStatis.split(',').map(Number);
    const rateArray = equipMalfRate.split(',').map((value: string) => Number(value) * 100); // 转换为百分比

    const chartOptions = reactive({
        tooltip: {
            trigger: 'axis',
            axisPointer: {
                type: 'cross',
                label: {
                    backgroundColor: '#6a7985'
                }
            }
        },
        legend: {
            data: ['可能故障的装备数', '可能故障装备的的故障率'],
        },
        toolbox: {
            feature: {
                saveAsImage: {}
            }
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        xAxis: [
            {
                type: 'category',
                boundaryGap: false,
                data: dateArray, // 使用拆分后的日期数组
                axisLabel: {
                    interval: 0, // 强制显示所有标签
                    rotate: 0, // 旋转 45 度以避免重叠
                    textStyle: {
                        fontSize: 12 // 减小字体大小
                    }
                }
            }
        ],
        yAxis: [
            {
                type: 'value',
                name: '故障数量',
                position: 'left',
                interval: 1, // 设置间隔为1
                axisLabel: {
                    formatter: '{value}' // 强制显示整数
                }
            },
            {
                type: 'value',
                name: '故障率 (%)',
                position: 'right',
                min: 0,
                max: 100,
                interval: 20,
                axisLabel: {
                    formatter: '{value} '
                }
            }
        ],
        series: [
            {
                name: '可能故障的装备数',
                type: 'line',
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: statisArray // 使用后端返回的故障统计数据
            },
            {
                name: '可能故障装备的的故障率',
                type: 'line',
                yAxisIndex: 1,
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: rateArray // 使用转换后的百分比数据
            }
        ]
    });

    console.log('chartRef1 options:', chartOptions);
    return chartOptions;
};
// 使用 useChart 并获取 update 方法
const { update: updateChart1 } = useChart(chartRef1, setChartData1);
// 监听 chartData 变化，更新图表
watch(chartData, async () => {
    const options = await setChartData1(); // 获取最新的图表配置
    updateChart1(options); // 更新图表
});

const setChartData2 = async () => {
    // 如果 chartData 为空，返回默认配置
    if (!detailchartData.value) {
        return detaildefaultChartOptions;
    }
    const { equipMalfDetailDate, equipMalfDetailStatis, equipMalfDetailRate } = detailchartData.value;
    const dateArray = equipMalfDetailDate.split(','); // 拆分为数组
    const statisArray = equipMalfDetailStatis.split(',').map(Number);
    const rateArray = equipMalfDetailRate.split(',').map((value: string) => Number(value) * 100); // 转换为百分比

    const chartOptions = reactive({
        tooltip: {
            trigger: 'axis',
            axisPointer: {
                type: 'cross',
                label: {
                    backgroundColor: '#6a7985'
                }
            }
        },
        legend: {
            data: ['可能故障的装备数', '可能故障装备的的故障率'],
        },
        toolbox: {
            feature: {
                saveAsImage: {}
            }
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        xAxis: [
            {
                type: 'category',
                boundaryGap: false,
                data: dateArray, // 使用拆分后的日期数组
                axisLabel: {
                    interval: 0, // 强制显示所有标签
                    rotate: 0, // 旋转 45 度以避免重叠
                    textStyle: {
                        fontSize: 12 // 减小字体大小
                    }
                }
            }
        ],
        yAxis: [
            {
                type: 'value',
                name: '故障数量',
                position: 'left',
                interval: 1, // 设置间隔为1
                axisLabel: {
                    formatter: '{value}' // 强制显示整数
                }
            },
            {
                type: 'value',
                name: '故障率 (%)',
                position: 'right',
                min: 0,
                max: 100,
                interval: 20,
                axisLabel: {
                    formatter: '{value} '
                }
            }
        ],
        series: [
            {
                name: '可能故障的装备数',
                type: 'line',
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: statisArray // 使用后端返回的故障统计数据
            },
            {
                name: '可能故障装备的的故障率',
                type: 'line',
                yAxisIndex: 1,
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: rateArray // 使用转换后的百分比数据
            }
        ]
    });

    console.log('chart2 options:', chartOptions);
    return chartOptions;
};
// 使用 useChart 并获取 update 方法
const { update: updateChart2 } = useChart(chart2, setChartData2);
// 监听 chartData 变化，更新图表
watch(detailchartData, async () => {
    const options = await setChartData2(); // 获取最新的图表配置
    updateChart2(options); // 更新图表
});

const setChartData5 = async () => {
    // 如果 chartData 为空，返回默认配置
    if (!repairchartData.value) {
        return repairdefaultChartOptions;
    }
    const { equipRepairDate, equipRepairStatis, equipRepairRate } = repairchartData.value;
    const dateArray = equipRepairDate.split(','); // 拆分为数组
    const statisArray = equipRepairStatis.split(',').map(Number);
    const rateArray = equipRepairRate.split(',').map((value: string) => Number(value) * 100); // 转换为百分比
    const maxRate = Math.max(...rateArray, 10); // Ensure at least 10% for visibility
    const chartOptions = reactive({
        tooltip: {
            trigger: 'axis',
            axisPointer: {
                type: 'cross',
                label: {
                    backgroundColor: '#6a7985'
                }
            }
        },
        legend: {
            data: ['计划送修装备数'],
        },
        toolbox: {
            feature: {
                saveAsImage: {}
            }
        },
        grid: {
            left: '3%',
            right: '7%',
            bottom: '10%',
            containLabel: true
        },
        xAxis: [
            {
                type: 'category',
                boundaryGap: false,
                data: dateArray, // 使用拆分后的日期数组
                axisLabel: {
                    interval: 0, // 强制显示所有标签
                    rotate: 0, // 旋转 45 度以避免重叠
                    textStyle: {
                        fontSize: 12 // 减小字体大小
                    }
                }
            }
        ],
        yAxis: [
            {
                type: 'value',
                name: '故障数量',
                position: 'left',
                interval: 1, // 设置间隔为1
                axisLabel: {
                    formatter: '{value}' // 强制显示整数
                }
            },
            {
                type: 'value',
                name: '故障率 (%)',
                position: 'right',
                min: 0, // Set minimum to 0
                max: Math.ceil(maxRate / 10) * 10, // Round up to nearest 10

                interval: 5, // Set interval to 5% for clarity
                axisLabel: {
                    formatter: '{value} ' // Format as percentage
                },
                data: rateArray,
            }
        ],
        series: [
            {
                name: '计划送修装备数',
                type: 'line',
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: statisArray // 使用后端返回的故障统计数据
            },
        ]
    });
    console.log('chart5 options:', chartOptions);
    return chartOptions;
};
// 使用 useChart 并获取 update 方法
const { update: updateChart5 } = useChart(chart5, setChartData5);
// 监听 chartData 变化，更新图表
watch(repairchartData, async () => {
    const options = await setChartData5(); // 获取最新的图表配置
    updateChart5(options); // 更新图表
});

const setChartData4 = async () => {
    // 如果 equipchartData 为空，返回默认配置
    if (!diagnosechartData.value || !diagnosechartData.value.partCountMap) {
        return diagnosedefaultChartOptions;
    }

    const partCountMap = diagnosechartData.value.partCountMap;

    // 将 partCountMap 转换为饼图数据格式
    const pieData = Object.entries(partCountMap).map(([name, value]) => ({
        name: name === '' ? '其他' : name, // 处理空键名
        value: value as number,
    }));

    const chartOptions = reactive({
        title: {
            text: '故障分布',
            left: 'center',
            top: '5%',
            textStyle: {
                fontSize: 16,
                fontWeight: 'bold'
            }
        },
        tooltip: {
            trigger: 'item'
        },
        legend: {
            orient: 'horizontal',
            top: '10%',
            bottom:'10%',
            left: 'center',
            itemGap: 5,
            textStyle: {
                fontSize: 12
            }
        },
        series: [
            {
                type: 'pie',
                radius: '50%',
                center: ['50%', '70%'],
                avoidLabelOverlap: true,
                itemStyle: {
                    borderRadius: 10,
                    borderColor: '#fff',
                    borderWidth: 2
                },
                emphasis: {
                    label: {
                        show: true,
                        fontSize: 20,
                        fontWeight: 'bold'
                    }
                },
                data: pieData // 使用转换后的数据
            }
        ]
    });

    console.log('chart4 options:', chartOptions);
    return chartOptions;
};

// 使用 useChart 并获取 update 方法
const { update: updateChart4 } = useChart(chart4, setChartData4);

// 监听 equipchartData 变化，更新图表
watch(diagnosechartData, async (newData) => {
    console.log('diagnosedechartData 更新:', newData);
    const options = await setChartData4(); // 获取最新的图表配置
    updateChart4(options); // 更新图表
});

const setChartData3 = async () => {
    // 如果 chartData 为空，返回默认配置
    if (!modelchartData.value) {
        return modeldefaultChartOptions;
    }
    const { equipModelMalfDate, equipModelMalfStatis, equipModelMalfRate } = modelchartData.value;
    const dateArray = equipModelMalfDate.split(','); // 拆分为数组
    const statisArray = equipModelMalfStatis.split(',').map(Number);
    const rateArray = equipModelMalfRate.split(',').map((value: string) => Number(value) * 100); // 转换为百分比

    const chartOptions = reactive({
        tooltip: {
            trigger: 'axis',
            axisPointer: {
                type: 'cross',
                label: {
                    backgroundColor: '#6a7985'
                }
            }
        },
        legend: {
            data: ['可能故障的装备数', '可能故障装备的的故障率'],
        },
        toolbox: {
            feature: {
                saveAsImage: {}
            }
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        xAxis: [
            {
                type: 'category',
                boundaryGap: false,
                data: dateArray, // 使用拆分后的日期数组
                axisLabel: {
                    interval: 0, // 强制显示所有标签
                    rotate: 0, // 旋转 45 度以避免重叠
                    textStyle: {
                        fontSize: 12 // 减小字体大小
                    }
                }
            }
        ],
        yAxis: [
            {
                type: 'value',
                name: '故障数量',
                position: 'left',
                interval: 1, // 设置间隔为1
                axisLabel: {
                    formatter: '{value}' // 强制显示整数
                }
            },
            {
                type: 'value',
                name: '故障率 (%)',
                position: 'right',
                axisLabel: {
                    formatter: '{value} '
                }
            }
        ],
        series: [
            {
                name: '可能故障的装备数',
                type: 'line',
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: statisArray // 使用后端返回的故障统计数据
            },
            {
                name: '可能故障装备的的故障率',
                type: 'line',
                yAxisIndex: 1,
                areaStyle: {},
                emphasis: {
                    focus: 'series'
                },
                data: rateArray // 使用转换后的百分比数据
            }
        ]
    });
    console.log('chartRef3 options:', chartOptions);
    return chartOptions;
};
// 使用 useChart 并获取 update 方法
const { update: updateChart3 } = useChart(chart3, setChartData3);
// 监听 chartData 变化，更新图表
watch(modelchartData, async () => {
    const options = await setChartData3(); // 获取最新的图表配置
    updateChart3(options); // 更新图表
});

const setChartData6 = async () => {
    const chartOptions = reactive({
        title: {
            text: '故障分布',
            left: 'center',
            top: '5%',
            textStyle: {
                fontSize: 16,
                fontWeight: 'bold'
            }
        },
        tooltip: {
            trigger: 'item'
        },
        legend: {
            orient: 'horizontal',
            top: '15%',
            left: 'center',
            itemGap: 5,
            textStyle: {
                fontSize: 12
            }
        },
        series: [
            {
                type: 'pie',
                radius: '50%',
                center: ['50%', '65%'],
                avoidLabelOverlap: true,
                itemStyle: {
                    borderRadius: 10,
                    borderColor: '#fff',
                    borderWidth: 2
                },
                emphasis: {
                    label: {
                        show: true,
                        fontSize: 20,
                        fontWeight: 'bold'
                    }
                },
                data: [
                    { value: 1048, name: '部件1' },
                    { value: 735, name: '部件2' },
                    { value: 580, name: '部件3' },
                    { value: 484, name: '部件4' },
                    { value: 300, name: '部件5' }
                ]
            }
        ]
    });
    console.log('chart6 options:', chartOptions);
    return chartOptions;
};
useChart(chart6, setChartData6);


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
.ml {
    margin-left: 10px;
}

.mt {
    margin-top: 20px;
}

.mb {
    margin-bottom: 20px;
}

.el-table {
    width: 100%;

    .el-table-column {
        text-align: center;
    }

    .el-select {
        width: 100%;
    }
}

.custom-header-cell {
    background-color: #f5f7fa;
    font-weight: bold;
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

.flex-container {
    display: flex;
    align-items: center;
    gap: 10px;
}

.switch-style {
    margin-right: 0;
}

.input-style {
    flex: auto;
}

.a {
    color: #408de6;
}

/* 图表容器样式 */
.chart-container {
    width: 100%;
    height: 300px;
    display: flex;
    justify-content: center;
    align-items: center;
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