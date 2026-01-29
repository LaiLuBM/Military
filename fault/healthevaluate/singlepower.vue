<template>
    <div>
        <el-card>
        <el-row style="margin-top: 10px;">
            <el-col :span="22">
                <h3>单装战斗力评估</h3>
            </el-col>
            <el-col :span="2">
                <el-button style="background-color: blueviolet; color: white;" size="small"
                    @click="handleSingleReassess">重新评估</el-button>
            </el-col>
        </el-row>
        <el-row style="margin-top: 15px;" :gutter="12">
            <el-col :span="6">
                <el-card>
                <div class="equip-selector">
                        <el-select clearable v-model="searchParams.equipModelCode" placeholder="装备型号"
                            @change="handleEquipModelChange">
                            <el-option-group v-for="group in equipmentTypeOptions" :key="group.label"
                                :label="group.label">
                                <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                                    :value="option.value"></el-option>
                            </el-option-group>
                        </el-select>
                    </div>
                <div class="left-panel">
                    <div class="tree-container">
                        <el-tree :data="treeData" :props="treeProps" :default-expand-all="false" class="sub-tree"
                            @node-click="handleNodeClick"></el-tree>
                    </div>
                    
                </div>
                </el-card>
            </el-col>
            <el-col :span="18">
                <el-card>
                <el-table border :header-cell-style="headerCellStyle" :data="tableData" :loading="tableLoading"
                    element-loading-text="拼命加载中" element-loading-spinner="el-icon-loading"
                    element-loading-background="rgba(0, 0, 0, 0.8)" @select="handleSelect"
                    @selection-change="handleSelectionChange" ref="tableRef">
                    <!-- 单选列 -->
                    <el-table-column type="selection" width="35" :selectable="checkSelectable" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="装备统一编码" align="center" prop="equipCode"  :show-overflow-tooltip="true"/>
                    <el-table-column label="装备型号" align="center" prop="equipSimpleName"  :show-overflow-tooltip="true"/>
                    <el-table-column label="作战用途" align="center" prop="task"  :show-overflow-tooltip="true"/>
                    <el-table-column label="战斗力基准值" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            {{ row.combatCapabilityStandard || '未提供' }}
                        </template>
                    </el-table-column>
                    <el-table-column label="战斗力值" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            {{ row.agedCombatCapability?.toFixed(2) || '未提供' }}
                        </template>
                    </el-table-column>
                    <el-table-column label="算法选择" prop="algorithm" align="center" width="120px" :show-overflow-tooltip="true">
                        <template #default="scope">
                            <el-select v-model="scope.row.algorithm" placeholder="选择算法" style="width: 100%"
                                placement="bottom">
                                <el-option v-for="option in algorithmOptions" :key="option.value" :label="option.label"
                                    :value="option.value"></el-option>
                            </el-select>
                        </template>
                    </el-table-column>
                    <el-table-column label="战斗力保留百分比" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            {{ row.combatCapabilityPercentage?.toFixed(2) || '未提供' }}%
                        </template>
                    </el-table-column>
                    <el-table-column label="健康评估时各分系统健康指数" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            {{ formatSubsysHealth(row.subsysHealthScoreMap || {}) }}
                        </template>
                    </el-table-column>
                    <el-table-column label="健康指数评估时间" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            {{ formatDate(row.ccAssessmentDate || row.assessmentDate) }}
                        </template>
                    </el-table-column>
                    <el-table-column label="近3个月战斗力变化" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <el-button type="primary" text @click="viewTrend(row.equipCode)">查看</el-button>
                        </template>
                    </el-table-column>
                </el-table>

                <el-pagination class="fr mt mb" v-model:current-page="pageInfo.page"
                    v-model:page-size="pageInfo.pageSize" :page-sizes="[10, 20, 30, 40]"
                    layout="sizes, prev, pager, next, jumper" :total="totals" @size-change="handleSizeChange"
                    @current-change="handleCurrentChange" background>
                    <template #sizes="{ pageSizes }">
                        <span v-for="size in pageSizes" :key="size" class="el-pagination__sizes">
                            {{ size }}条/页
                        </span>
                    </template>
                </el-pagination>
                </el-card>
            </el-col>
        </el-row>
        <el-dialog v-model="trendDialogVisible" :title="`${selectedEquipCode} - 近3个月战斗力变化趋势`" width="70%">
            <div ref="chartContainer" style="width: 100%; height: 400px;"></div>
        </el-dialog>
    </el-card>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive, nextTick, watch } from 'vue';
import { unitListApi } from '@/api/fault/unitlist';
import { ElMessage, ElTable } from 'element-plus';
import { CombatCapabilityApi, CombatCapabilityHistoryApi, CombatCapabilityTrendApi } from '@/api/fault/equipCombatCapability';
import * as echarts from 'echarts';
import { Equipmodelhelth } from '@/api/fault/equipmodel';

// 定义表格数据接口
interface EquipCombatCapability {
    equipCode: string;
    combatCapabilityStandard: number;
    taskId: string;
}

interface TableDataItem {
    ccId: number;
    equipCode: string;
    task: string;
    agedCombatCapability?: number;
    combatCapabilityPercentage?: number;
    ccAssessmentDate: string;
    assessmentDate?: string;
    equipSimpleName?: string;
    combatCapabilityStandard?: number;
    combatCapability?: number;
    subsysHealthScoreMap?: Record<string, number>;
    equipCombatCapability?: EquipCombatCapability;
}

const treeData = ref([]);
const tableData = ref<TableDataItem[]>([]);
const selectedUnit = ref('');
const totals = ref<number>(0);
const selectedEquipments = ref<any[]>([]);
const pageInfo = reactive({
    page: 1,
    pageSize: 10,
});
// 添加 loading 状态
const tableLoading = ref(false);
const algorithmOptions = [
    { label: "weight", value: "weight" },
    { label: "ridge", value: "ridge" },
]
const treeProps = {
    label: 'unitSimpleName',
    children: 'childrenUnits',
};
interface SearchType {
    equipModelCode: string;
    unitPath: string;
    page: number;
    pageSize: number;
}
interface Tree {
    unitCode: number;
    parentUnitCode: any;
    unitSimpleName: string;
    unitPath: string;
    childrenUnits?: Tree[];
}
const searchParams = ref<SearchType>({
    equipModelCode: '',
    unitPath: '',
    page: 1,
    pageSize: 10,
});
// 添加表格引用
const tableRef = ref<InstanceType<typeof ElTable>>();

// 检查是否可选（始终返回true）
const checkSelectable = () => {
    return true;
};

// 处理选择事件 - 实现单选逻辑
const handleSelect = (_selection: any[], row: any) => {
    const table = tableRef.value;
    if (table) {
        table.clearSelection();
        table.toggleRowSelection(row, true);
    }
};

// 处理选择变化
const handleSelectionChange = (rows: any[]) => {
    selectedEquipments.value = rows.length > 0 ? [rows[rows.length - 1]] : [];
    console.log('选中的装备:', selectedEquipments.value);
};
// 新增的状态变量
const trendDialogVisible = ref(false);
const selectedEquipCode = ref('');
const chartContainer = ref<HTMLElement | null>(null);
let chartInstance: echarts.ECharts | null = null;

// 修改 viewTrend 函数
const viewTrend = async (equipCode: string) => {
    selectedEquipCode.value = equipCode;
    try {
        const res = await CombatCapabilityTrendApi({
            equipCode,

        });

        if (res.code === 200) {
            const equipmentData = res.data
            trendDialogVisible.value = true;

            await nextTick();
            renderChart(equipmentData);
        } else {
            ElMessage.warning('未找到该装备的历史数据');
        }
    } catch (error) {
        ElMessage.error('获取趋势数据失败');
        console.error('Error fetching trend data:', error);
    }
};

// 新增渲染图表函数
const renderChart = (equipmentData: { "timeArray": [], "combatCapability": [] }) => {
    if (!chartContainer.value) return;

    // 销毁旧图表实例
    if (chartInstance) {
        chartInstance.dispose();
    }

    // 初始化图表
    chartInstance = echarts.init(chartContainer.value);

    // 只取最近3个月的数据（假设数组最后3个元素是最近3个月的数据）
    // const lastThreeMonthsData = pastSixMonthsCC.slice(-3);
    const months = equipmentData.timeArray
    const option = {
        tooltip: {
            trigger: 'axis',
            formatter: '{b}<br/>战斗力值: {c}'
        },
        xAxis: {
            type: 'category',
            data: months,
            axisLabel: {
                color: '#333'
            }
        },
        yAxis: {
            type: 'value',
            name: '战斗力值',
            axisLabel: {
                formatter: '{value}'
            },

        },
        series: [{
            data: equipmentData.combatCapability,
            type: 'line',
            smooth: false,
            symbol: 'circle',
            symbolSize: 8,
            itemStyle: {
                color: '#1890ff'
            },
            lineStyle: {
                width: 3
            },
            areaStyle: {
                color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                    {
                        offset: 0,
                        color: 'rgba(24, 144, 255, 0.5)'
                    },
                    {
                        offset: 1,
                        color: 'rgba(24, 144, 255, 0.1)'
                    }
                ])
            }
        }],
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        }
    };

    chartInstance.setOption(option);

    // 窗口大小变化时重新调整图表大小
    window.addEventListener('resize', function () {
        chartInstance?.resize();
    });
};


const fetchUnitList = async () => {
    try {
        const response = await unitListApi();
        if (response.code === 200) {
            treeData.value = response.data;
        } else {
            ElMessage.error(response.message || '无法加载单位列表');
        }
    } catch (error) {
        ElMessage.error('无法加载单位列表，请稍后重试');
        console.error('Error fetching unit list:', error);
    }
};
const equipmentTypeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);

const handleNodeClick = async (nodeData: Tree) => {
    selectedUnit.value = nodeData.unitSimpleName;
    searchParams.value.unitPath = nodeData.unitPath;
    searchParams.value.equipModelCode = '';
    searchParams.value.page = 1;
    pageInfo.page = 1;

    console.log(searchParams.value.unitPath);
    await handleQuery();
    try {
        const response = await Equipmodelhelth(nodeData.unitPath);
        console.log('装备统一编码数据:', response);

        if (response.code === 200 && response.data?.categories) {
            const groupedOptions = Object.entries(response.data.categories).map(([category, items]) => {
                if (!Array.isArray(items)) {
                    console.error(`类别 ${category} 的数据不是数组:`, items);
                    return { label: category, options: [] };
                }

                const options = items.map((item: any) => ({
                    label: item.equipSimpleName,
                    value: item.equipModelCode,
                }));

                return {
                    label: category,
                    options,
                };
            });

            equipmentTypeOptions.value = groupedOptions;
            console.log('生成的 equipmentTypeOptions:', equipmentTypeOptions.value);


        } else {
            console.error('数据格式错误或请求失败:', response.message);
            equipmentTypeOptions.value = [];
        }
    } catch (error) {
        console.error('请求装备统一编码数据失败:', error);
        equipmentTypeOptions.value = [];
        ElMessage.error('获取装备统一编码失败');
    }
};

const handleEquipModelChange = async (selectedModelCode: string) => {
    // 更新装备统一编码筛选条件
    searchParams.value.equipModelCode = selectedModelCode;
    searchParams.value.page = 1;
    pageInfo.page = 1;

    console.log('选中的装备统一编码:', selectedModelCode);
    console.log('当前查询参数:', searchParams.value);

    await handleQuery();
};



const handleQuery = async () => {
    tableLoading.value = true; // 开始加载
    try {
        const res = await CombatCapabilityHistoryApi(searchParams.value);
        console.log('API Response:', res); // 调试日志
        if (res.code === 200) {
            tableData.value = res.data.content.map((item: any) => ({
                ccId: item.equipCombatCapabilityHistory?.ccId || null,
                equipCode: item.equipCode,
                task: item.equipCombatCapabilityHistory?.task || '-',
                agedCombatCapability: item.equipCombatCapabilityHistory?.combatCapability || null,
                combatCapabilityPercentage: item.equipCombatCapabilityHistory?.combatCapabilityPercentage || null,
                ccAssessmentDate: item.equipCombatCapabilityHistory?.ccAssessmentDate || null,
                equipSimpleName: item.equipSimpleName,
                combatCapabilityStandard: item.combatCapabilityStandard,
                subsysHealthScoreMap: item.healthScoreMap || {}
            }));
            totals.value = res.data.totalElements || 0;
        } else {
            ElMessage.error(res.message || '查询失败');
            tableData.value = [];
            totals.value = 0;
        }
    } catch (error) {
        ElMessage.error('查询失败，请稍后重试');
        console.error('Error fetching combat capability:', error);
        tableData.value = [];
        totals.value = 0;
    } finally {
        tableLoading.value = false; // 结束加载
    }
};

const handleSizeChange = async (newSize: number) => {
    pageInfo.pageSize = newSize;
    searchParams.value.pageSize = newSize;
    searchParams.value.page = 1;
    await handleQuery();
};

const handleCurrentChange = async (newPage: number) => {
    pageInfo.page = newPage;
    searchParams.value.page = newPage;
    await handleQuery();
};

// 修改重新评估函数
const handleSingleReassess = async () => {
    if (selectedEquipments.value.length === 0) {
        ElMessage.warning('请先选择一个装备');
        return;
    }

    const equipCode = selectedEquipments.value[0].equipCode;
    const algorithm = selectedEquipments.value[0].algorithm || 'weight';

    console.log('Reassessing equipCode:', equipCode);
    try {
        const res = await CombatCapabilityApi({ equipCode, algorithm });
        if (res.code === 200) {
            const newData = res.data;
            const index = tableData.value.findIndex(item => item.equipCode === equipCode);
            if (index !== -1) {
                tableData.value[index] = {
                    ...tableData.value[index],
                    ...newData,
                    equipSimpleName: newData.equipSimpleName || tableData.value[index].equipSimpleName,
                    combatCapabilityStandard: newData.equipCombatCapability?.combatCapabilityStandard || tableData.value[index].combatCapabilityStandard,
                    combatCapability: newData.combatCapability,
                    subsysHealthScoreMap: newData.subsysHealthScoreMap,
                    ccAssessmentDate: newData.ccAssessmentDate || newData.assessmentDate,
                };
                ElMessage.success(`装备 ${equipCode} 重新评估成功`);
            } else {
                ElMessage.warning(`未找到装备 ${equipCode}，请刷新数据`);
            }
        } else {
            ElMessage.error(res.message || '重新评估失败');
        }
    } catch (error) {
        ElMessage.error('重新评估请求失败，请稍后重试');
        console.error('Error reassessing combat capability:', error);
    }
};

const formatSubsysHealth = (scoreMap: Record<string, number>) => {
    if (!scoreMap || Object.keys(scoreMap).length === 0) return '-';
    return Object.entries(scoreMap)
        .map(([key, value]) => `${key}: ${value}`)
        .join(', ');
};

const formatDate = (dateStr: string) => {
    if (!dateStr) return '-';
    return new Date(dateStr).toLocaleString('zh-CN', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
    });
};

onMounted(() => {
    fetchUnitList();
    if (chartInstance) {
        chartInstance.dispose();
    }
});

watch(tableData, (newDataList) => {
    if (newDataList && newDataList.length > 0) {
        newDataList.forEach((item: any) => {
            item.algorithm = 'weight';
        });
    }
}, { immediate: true });

const headerCellStyle = {
    backgroundColor: '#F8F8F8',
    color: 'black',
    fontWeight: 'bold'
};
</script>

<style lang="less" scoped>
.left-panel {
    display: flex;

    flex-direction: column;
    height: 100%;

    position: relative;
    margin-top: 10px;
    
}

.tree-container {
    flex: 1;
    overflow-y: auto;
    border: 1px solid #e4e7ed;
    border-radius: 4px;
    margin-bottom: 10px;
    max-height: 100%;
  
    /* 设置最大高度，超出可滚动 */
}

.equip-selector {
    flex-shrink: 0;
    /* 防止选择器被压缩 */
    width: 100%;
}

.sub-tree {
    ::v-deep .el-tree-node__content:hover {
        background-color: #dceefd !important;
        color: #409eff;
        font-weight: bold;
    }

    ::v-deep .el-tree-node.is-current>.el-tree-node__content {
        background-color: #dceefd;
        color: #409eff;
        font-weight: bold;
    }

    font-weight: normal;
        width: max-content;
        height: max-content;
    min-width: 100%;
}

.fr {
    float: right;
}

.mt {
    margin-top: 10px;
}

.mb {
    margin-bottom: 10px;
}

:deep(.el-select-group__title) {
    font-weight: bold;
    color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px;
}

:deep(.el-select .el-select__popper) {
    top: 100% !important;
    bottom: auto !important;
    margin-top: 5px !important;
}

:deep(.el-table__header-wrapper .el-checkbox) {
    display: none;
}
.el-card{
    height: 100%;
}
</style>