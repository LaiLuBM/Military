<template>
    <div>
        <el-row style="margin-top: 10px;">
            <el-col :span="22">
                <h3>建制单位战斗力评估</h3>
            </el-col>
            <el-col :span="2">
                <el-button style="background-color: blueviolet; color: white;" size="small"
                    @click="handleSingleReassess">重新评估</el-button>
            </el-col>
        </el-row>
        <el-row style="margin-top: 15px;" :gutter="12">
            <el-col :span="6">
                <el-card>
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
                <el-table border :header-cell-style="headerCellStyle" :data="tableData" @select="handleSelect"
                    @selection-change="handleSelectionChange" ref="tableRef">
                    <!-- 单选列 -->
                    <el-table-column type="selection" width="35" :selectable="checkSelectable" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="建制单位" align="center" prop="unitFullname"  :show-overflow-tooltip="true"/>
                    <el-table-column label="战斗力基准值((100%健康时)" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <div v-for="(value, key) in row.fullCombatCapability" :key="key">
                                {{ key }}: {{ value }}
                            </div>
                        </template>
                    </el-table-column>
                    <el-table-column label="战斗力值" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <div v-for="(value, key) in row.estimateCombatCapability" :key="key">
                                {{ key }}: {{ value }}
                            </div>
                        </template>
                    </el-table-column>
                    <el-table-column label="战斗力保留百分比" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <div v-for="(value, key) in row.combatCapabilityPercentage" :key="key">
                                {{ key }}: {{ value }}
                            </div>
                        </template>
                    </el-table-column>
                    <el-table-column label="装备数量" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <div v-for="(value, key) in row.appreilAmount" :key="key">
                                {{ key }}: {{ value }}
                            </div>
                        </template>
                    </el-table-column>
                    <el-table-column label="评估时间" align="center" prop="assessmentDate"  :show-overflow-tooltip="true"/>
                    <el-table-column label="近3个月战斗力变化" align="center" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <el-button type="primary" text @click="viewTrend(row.unitFullname)">查看</el-button>
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
        <el-dialog v-model="trendDialogVisible" :title="`${rowunitFullname} - 近3个月战斗力变化趋势`" width="70%">
            <div ref="chartContainer" style="width: 100%; height: 400px;"></div>
        </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive, nextTick } from 'vue';
import { unitListApi } from '@/api/fault/unitlist';
import { ElMessage, ElTable } from 'element-plus';
import { UnitCombatCapabilityApi, UnitCombatCapabilityHistoryApi, UnitCombatCapabilityTrendApi } from '@/api/fault/equipCombatCapability';
import * as echarts from 'echarts';

// Define table data interface for unit-level data
interface TableDataItem {
    unitFullname: string;
    unitCCId: number;
    combatType: Record<string, any>;
    fullCombatCapability: Record<string, any>;
    estimateCombatCapability: Record<string, any>;
    combatCapabilityPercentage: Record<string, any>;
    appreilAmount: Record<string, any>;
    assessmentDate: string;
}

const treeData = ref([]);
const tableData = ref<TableDataItem[]>([]);
const selectedUnit = ref('');
const totals = ref<number>(0);
const pageInfo = reactive({
    page: 1,
    pageSize: 10,
});
const treeProps = {
    label: 'unitSimpleName',
    children: 'childrenUnits',
};
interface SearchType {
    unitPath: string;
    page: number;
    pageSize: number;
}
interface Tree {
    unitFullname: number;
    parentunitFullname: any;
    unitSimpleName: string;
    unitPath: number;
    childrenUnits?: Tree[];
}
const searchParams = ref<SearchType>({
    unitPath: '',
    page: 1,
    pageSize: 10,
});

// Chart-related state
const trendDialogVisible = ref(false);
const selectedUnitPath = ref('');
const rowunitFullname = ref('')
const chartContainer = ref<HTMLElement | null>(null);
let chartInstance: echarts.ECharts | null = null;

// 添加表格引用
const tableRef = ref<InstanceType<typeof ElTable>>();

// 选中的单位（单选，只保留一个）
const selectedUnits = ref<any[]>([]);

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
    selectedUnits.value = rows.length > 0 ? [rows[rows.length - 1]] : [];
    console.log('选中的单位:', selectedUnits.value);
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

const handleNodeClick = async (nodeData: Tree) => {
    selectedUnit.value = nodeData.unitSimpleName;
    const unitPath = nodeData.unitPath.toString();
    searchParams.value.unitPath = unitPath;
    searchParams.value.page = 1;
    pageInfo.page = 1;
    await handleQuery();
};

const handleQuery = async () => {
    if (!searchParams.value.unitPath) {
        ElMessage.warning('请先选择一个单位');
        return;
    }
    try {
        const res = await UnitCombatCapabilityHistoryApi(searchParams.value);
        console.log('API Response:', res);
        if (res.code === 200) {
            tableData.value = res.data.content.map((item: any) => ({
                unitFullname: item.unitFullname,
                unitCCId: item.unitCCId,
                combatType: item.unitFullCombatCapabitityMap || {},
                fullCombatCapability: item.unitFullCombatCapabitityMap || {},
                estimateCombatCapability: item.estimateCombatCapabilityMap || {},
                combatCapabilityPercentage: Object.fromEntries(
                    Object.entries(item.combatCapabilityPercentageMap || {}).map(([k, v]) => [k, `${v}%`])
                ),
                appreilAmount: item.appreilAmountMap || {},
                assessmentDate: formatDate(item.unitCCAssessmentDateTime),
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
    }
};

const handleSizeChange = (newSize: number) => {
    pageInfo.pageSize = newSize;
    searchParams.value.pageSize = newSize;
    searchParams.value.page = 1;
    handleQuery();
};

const handleCurrentChange = (newPage: number) => {
    pageInfo.page = newPage;
    searchParams.value.page = newPage;
    handleQuery();
};

const handleSingleReassess = async () => {
    if (selectedUnits.value.length === 0) {
        ElMessage.warning('请先选择一个单位');
        return;
    }

    const unitFullname = selectedUnits.value[0].unitFullname;
    const unitPath = searchParams.value.unitPath;
    console.log('Reassessing unitFullname:', unitFullname);
    try {
        const res = await UnitCombatCapabilityApi({ unitPath });
        if (res.code === 200) {
            const newData = res.data;
            const index = tableData.value.findIndex(item => item.unitFullname === unitFullname);
            if (index !== -1) {
                tableData.value[index] = {
                    ...tableData.value[index],
                    combatType: newData.unitFullCombatCapabitityMap || {},
                    fullCombatCapability: newData.unitFullCombatCapabitityMap || {},
                    estimateCombatCapability: newData.estimateCombatCapabilityMap || {},
                    combatCapabilityPercentage: Object.fromEntries(
                        Object.entries(newData.combatCapabilityPercentageMap || {}).map(([k, v]) => [k, `${v}%`])
                    ),
                    appreilAmount: newData.appreilAmountMap || {},
                    assessmentDate: formatDate(newData.unitCCAssessmentDateTime),
                };
                ElMessage.success(`单位 ${unitFullname} 重新评估成功`);
            } else {
                ElMessage.warning(`未找到单位 ${unitFullname}，请刷新数据`);
            }
        } else {
            ElMessage.error(res.message || '重新评估失败');
        }
    } catch (error) {
        ElMessage.error('重新评估请求失败，请稍后重试');
        console.error('Error reassessing combat capability:', error);
    }
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

const viewTrend = async (unitFullname: string) => {
    rowunitFullname.value = unitFullname
    selectedUnitPath.value = searchParams.value.unitPath; // Use the current unitPath
    try {
        const res = await UnitCombatCapabilityTrendApi({
            unitPath: selectedUnitPath.value,
        });

        if (res.code === 200) {
            const unitData = res.data;
            trendDialogVisible.value = true;

            await nextTick();
            renderChart(unitData);
        } else {
            ElMessage.warning('未找到该单位的趋势数据');
        }
    } catch (error) {
        ElMessage.error('获取趋势数据失败');
        console.error('Error fetching trend data:', error);
    }
};

const renderChart = (unitData: { timeArray: string[]; taskSeries: Record<string, number[]> }) => {
    if (!chartContainer.value) return;

    // Destroy old chart instance
    if (chartInstance) {
        chartInstance.dispose();
    }
    // Helper function to assign colors to tasks
    const getTaskColor = (task: string, opacity = 1) => {
        const colors: Record<string, string> = {
            '火力打击': `rgba(255, 99, 132, ${opacity})`, // Red
            '综合保障': `rgba(54, 162, 235, ${opacity})`, // Blue
            '地面突击': `rgba(75, 192, 192, ${opacity})`, // Teal
            '指挥控制': `rgba(255, 159, 64, ${opacity})`, // Orange
        };
        return colors[task] || `rgba(128, 128, 128, ${opacity})`; // Default gray
    };
    // Initialize chart
    chartInstance = echarts.init(chartContainer.value);

    // Prepare series data for each task
    const series = Object.entries(unitData.taskSeries).map(([task, values]) => ({
        name: task,
        type: 'line',
        smooth: false,
        symbol: 'circle',
        symbolSize: 8,
        data: values,
        itemStyle: {
            color: getTaskColor(task), // Assign distinct colors for tasks
        },
        lineStyle: {
            width: 3,
        },
        areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: getTaskColor(task, 0.5) },
                { offset: 1, color: getTaskColor(task, 0.1) },
            ]),
        },
    }));

    const option = {
        tooltip: {
            trigger: 'axis',
            formatter: (params: any) => {
                let result = `${params[0].axisValue}<br/>`;
                params.forEach((item: any) => {
                    result += `${item.seriesName}: ${item.value.toFixed(2)}<br/>`;
                });
                return result;
            },
        },
        legend: {
            data: Object.keys(unitData.taskSeries),
            top: '5%',
        },
        xAxis: {
            type: 'category',
            data: unitData.timeArray,
            axisLabel: {
                color: '#333',
            },
        },
        yAxis: {
            type: 'value',
            name: '战斗力值',
            axisLabel: {
                formatter: '{value}',
            },
        },
        series,
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true,
        },
    };

    chartInstance.setOption(option);

    // Resize chart on window resize
    window.addEventListener('resize', () => {
        chartInstance?.resize();
    });
};

onMounted(() => {
    fetchUnitList();
    if (chartInstance) {
        chartInstance.dispose();
    }
});

const headerCellStyle = {
    backgroundColor: '#F8F8F8',
    color: 'black',
    fontWeight: 'bold',
};
</script>

<style lang="less" scoped>
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

:deep(.el-table__header-wrapper .el-checkbox) {
    display: none;
}

.el-card{
    height: 100%;
}

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

</style>