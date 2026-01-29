<template>
    <el-card>
        <el-row :gutter="20" align="middle">
            <el-col :span="4">
                <el-form>
                    <el-form-item label="装备型号">
                        <el-select clearable ref="equipSelectRef" v-model="selectedEquipModelName" placeholder="请选择装备型号"
                            style="width: 100%" :popper-append-to-body="false" @clear="handleEquipClear">
                            <el-option :value="selectedEquipModelName" style="height: auto; padding: 0">
                                <el-tree ref="equipTreeRef" :data="equipModelOptions" :props="equipTreeProps"
                                    node-key="value" :default-expand-all="false" @node-click="handleEquipNodeClick"
                                    class="sub-tree"></el-tree>
                            </el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="1"></el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item label="单位/分队">
                        <el-select clearable ref="unitSelectRef" v-model="selectedUnitName" placeholder="请选择单位/分队"
                            style="width: 100%" :popper-append-to-body="false" @clear="handleUnitClear"
                            :loading="isLoadingUnits">
                            <el-option :value="selectedUnitName" style="height: auto; padding: 0">
                                <el-tree ref="unitTreeRef" :data="unitOptions" :props="unitTreeProps" node-key="value"
                                    :default-expand-all="false" @node-click="handleUnitNodeClick"
                                    class="sub-tree"></el-tree>
                            </el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="11" style="position: relative; left: 0px; top: -9px">
                <div class="flex-container">
                    <el-button type="primary" @click="handleLoadData" :loading="isLoading">查询</el-button>
                    <el-button type="primary" @click="execute">执行预测</el-button>
                </div>
            </el-col>
        </el-row>
        <el-table ref="tableRef" :data="tableData" style="width: 100%" :header-cell-style="headerCellStyle" border
            @select="handleSelect" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55" :selectable="checkSelectable" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="所在单位/分队" prop="unitName" align="center" sortable :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="装备型号" prop="equipSimpleName" align="center" sortable :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="装备总数" prop="equipNumber" align="center" sortable :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="未来三月故障分布" prop="showString" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="最近一次预测执行时间" prop="predictTime" align="center" sortable :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="预测算法" prop="algorithm" align="center" :show-overflow-tooltip="true">
                <template #header>
                    <span style="display: flex; align-items: center; justify-content: center;">
                        算法选择
                        <el-tooltip raw-content placement="top" effect="light">
                            <template #content>
                                <div style="text-align: left; line-height: 1.5;">
                                    LSTM：长短时记忆网络，中等复杂度，适合中等规模数据；<br>
                                    SAR：季节性自回归，捕捉季节周期，中等复杂度，适合带季节性的中等规模序列；<br>
                                    SAE：堆叠自编码器，深度特征提取，拟合力强，适合大规模高维序列；
                                </div>
                            </template>
                            <el-icon style="margin-left: 4px; color: #409eff; cursor: help;">
                                <QuestionFilled />
                            </el-icon>
                        </el-tooltip>
                    </span>
                </template>
                <template #default="scope">
                    <el-select v-model="scope.row.algorithm" placeholder="选择算法" style="width: 100%"
                        @change="(value: any) => handleAlgorithmChange(scope.row, value)">
                        <el-option v-for="option in algorithmOptions" :key="option.value" :label="option.label"
                            :value="option.value"></el-option>
                    </el-select>
                </template>
            </el-table-column>
            <el-table-column label="操作" align="center" width="300px" :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-button type="warning" text @click="handleDetail(scope.row)">预测详情</el-button>
                    <el-divider direction="vertical" />
                    <el-button type="primary" text @click="handleHistory(scope.row)">更多历史预测</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination class="fr mt mb" v-model:current-page="pageInfo.page" v-model:page-size="pageInfo.pageSize"
            :page-sizes="[10, 20, 30, 40]" layout="sizes, prev, pager, next, jumper" :total="totals"
            @size-change="handleSizeChange" @current-change="handleCurrentChange" background>
            <template #sizes="{ pageSizes }">
                <span v-for="size in pageSizes" :key="size" class="el-pagination__sizes">
                    {{ size }}条/页
                </span>
            </template>
        </el-pagination>
    </el-card>
</template>

<script lang="ts" setup>
import { ref, onMounted, reactive } from 'vue';
import { useRouter } from 'vue-router';
import { unit, types, predict, equipModel, queryEquipModelAndUnit } from '@/api/fault/type';
import { ElMessage, type ElTable, type ElTree } from 'element-plus';

const router = useRouter();
const selectedUnitCode = ref<string>(''); // 存储选中的单位/分队编码
const selectedUnitName = ref<string>(''); // 存储选中的单位/分队名称（用于显示）
const selectedEquipModelCode = ref<string>(''); // 存储选中的装备型号编码
const selectedEquipModelName = ref<string>(''); // 存储选中的装备型号名称（用于显示）
const selectedRows = ref<TableItem[]>([]); // 用于存储勾选的行数据
const isLoadingUnits = ref(false); // Loading state for unit dropdown
const isClearingEquip = ref(false); // Flag to prevent loadData during clear
const isLoading = ref(false); // 查询按钮loading状态

const headerCellStyle = {
    backgroundColor: '#F8F8F8', // 表头背景颜色
    color: 'black', // 表头字体颜色
    fontWeight: 'bold' // 表头字体加粗
};

const totals = ref<number>(0);
const pageInfo = reactive({
    page: 1,
    pageSize: 10,
});

// 保存查询状态到 sessionStorage
const saveQueryState = () => {
    const state = {
        selectedEquipModelCode: selectedEquipModelCode.value,
        selectedEquipModelName: selectedEquipModelName.value,
        selectedUnitCode: selectedUnitCode.value,
        selectedUnitName: selectedUnitName.value,
        pageInfo: {
            page: pageInfo.page,
            pageSize: pageInfo.pageSize
        },
        // 保存表格数据以确保回退时能看到数据
        tableData: tableData.value,
        totals: totals.value
    };
    sessionStorage.setItem('faultPredictQuery', JSON.stringify(state));
    console.log('查询状态已保存到 sessionStorage');
};

// 从 sessionStorage 恢复查询状态
const restoreQueryState = () => {
    const saved = sessionStorage.getItem('faultPredictQuery');
    if (saved) {
        try {
            const state = JSON.parse(saved);
            console.log('从 sessionStorage 恢复查询状态:', state);

            selectedEquipModelCode.value = state.selectedEquipModelCode || '';
            selectedEquipModelName.value = state.selectedEquipModelName || '';
            selectedUnitCode.value = state.selectedUnitCode || '';
            selectedUnitName.value = state.selectedUnitName || '';

            if (state.pageInfo) {
                pageInfo.page = state.pageInfo.page || 1;
                pageInfo.pageSize = state.pageInfo.pageSize || 10;
            }

            // 恢复表格数据
            if (state.tableData && state.tableData.length > 0) {
                tableData.value = state.tableData;
                totals.value = state.totals || 0;
                console.log('表格数据已从 sessionStorage 恢复');
            }
        } catch (error) {
            console.error('恢复查询状态失败:', error);
            clearQueryState(); // 如果解析失败，清除错误的状态
        }
    }
};

// 清除保存的查询状态
const clearQueryState = () => {
    sessionStorage.removeItem('faultPredictQuery');
    console.log('sessionStorage 中的查询状态已清除');
};

const handleSizeChange = (size: number) => {
    pageInfo.pageSize = size;
    saveQueryState(); // 保存状态
    loadData();
};

const handleCurrentChange = (page: number) => {
    pageInfo.page = page;
    saveQueryState(); // 保存状态
    loadData();
};

const algorithmOptions = [
    { label: 'LSTM', value: 'LSTM' },
    { label: 'SVR', value: 'SVR' },
    { label: 'SAE', value: 'SAE' },
];

interface TableItem {
    unitCode: string;
    equipSimpleName: string;
    equipNumber: string;
    algorithm: string;
    equipModelCode: string;
    showString: string;
    predictTime: string;
    timeStamp: string;
    unitName: string;
}

const tableData = ref<TableItem[]>([]);
const tableRef = ref<InstanceType<typeof ElTable>>();

// 单位/分队树形结构
const unitTreeRef = ref<InstanceType<typeof ElTree>>();
const unitOptions = ref<{ label: string; value: string; children?: any[] }[]>([]);
const unitTreeProps = {
    label: 'label',
    children: 'children',
};

// 装备型号树形结构
const equipTreeRef = ref<InstanceType<typeof ElTree>>();
const equipModelOptions = ref<{ label: string; value: string; children?: any[] }[]>([]);
const equipTreeProps = {
    label: 'label',
    children: 'children',
};

// 递归处理单位数据，构建树形结构
const processUnitData = (units: any[]): any[] => {
    return units.map(unit => ({
        label: unit.unitSimpleName,
        value: unit.unitCode,
        children: unit.childrenUnits && unit.childrenUnits.length > 0
            ? processUnitData(unit.childrenUnits)
            : []
    }));
};

// 处理装备型号数据，构建两级树形结构
const processEquipModelData = (equipModels: any[]): any[] => {
    return equipModels.map(model => ({
        label: model.equipCategory,
        value: model.equipCategory,
        children: model.childrenEquipModels && model.childrenEquipModels.length > 0
            ? model.childrenEquipModels.map((child: any) => ({
                label: child.equipSimpleName,
                value: child.equipModelCode,
                children: []
            }))
            : []
    }));
};

// 加载单位/分队树形数据
const loadUnitTree = async () => {
    try {
        const response = await unit();
        if (response.code === 200 && response.data.length > 0) {
            unitOptions.value = processUnitData(response.data);
        } else {
            console.error('请求单位数据失败:', response.message);
            unitOptions.value = [];
            ElMessage.warning('无法加载单位数据');
        }
    } catch (error) {
        console.error('请求单位数据失败:', error);
        unitOptions.value = [];
    }
};

// 加载装备型号树形数据
const loadEquipModelTree = async () => {
    try {
        const response = await equipModel();
        if (response.code === 200 && response.data.length > 0) {
            equipModelOptions.value = processEquipModelData(response.data);
        } else {
            console.error('请求装备型号数据失败:', response.message);
            equipModelOptions.value = [];
            ElMessage.warning('无法加载装备型号数据');
        }
    } catch (error) {
        console.error('请求装备型号数据失败:', error);
        equipModelOptions.value = [];
        ElMessage.error('加载装备型号数据失败，请稍后重试');
    }
};

// 根据选中的装备型号加载单位数据
const loadUnitsByEquipModel = async (equipModelCode: string) => {
    isLoadingUnits.value = true;
    const params ={
        equipModelCode:equipModelCode
    }
    try {
        const response = await queryEquipModelAndUnit(params);
        console.log('queryEquipModelAndUnit response:', response);

        if (response.code === 200 && response.data && response.data.length > 0) {
            const selectedModel = response.data.find(
                (item: any) => item.equipModelCode === equipModelCode
            );
            console.log("selectedModel",selectedModel);
            if (selectedModel && selectedModel.unitList && selectedModel.unitList.length > 0) {
                console.log("xiangying")
                loadUnitTree();
                unitOptions.value = processUnitData(selectedModel.childrenUnits);
                console.log('unitOptions:', unitOptions.value);
            } else {
                unitOptions.value = [];
                ElMessage.warning('该装备型号下无关联单位');
            }
        } else {
            console.error('请求单位数据失败:', response.message);
            unitOptions.value = [];
            ElMessage.warning('无法加载单位数据');
        }
    } catch (error) {
        console.error('请求单位数据失败:', error);
        unitOptions.value = [];
    } finally {
        isLoadingUnits.value = false;
    }
};

// 处理单位/分队树节点点击
const handleUnitNodeClick = (nodeData: any) => {
    selectedUnitCode.value = nodeData.value;
    selectedUnitName.value = nodeData.label;
    console.log('选中的单位/分队:', {
        unitCode: selectedUnitCode.value,
        unitName: selectedUnitName.value,
    });
    // 保存状态
    saveQueryState();
};

// 处理装备型号树节点点击
const handleEquipNodeClick = (nodeData: any) => {
    if (nodeData.value && !nodeData.children.length) {
        selectedEquipModelCode.value = nodeData.value;
        selectedEquipModelName.value = nodeData.label;
        console.log('选中的装备型号:', {
            equipModelCode: selectedEquipModelCode.value,
            equipSimpleName: selectedEquipModelName.value,
        });

        // 保存状态
        saveQueryState();

        // Clear unit selection
        selectedUnitCode.value = '';
        selectedUnitName.value = '';
        unitTreeRef.value?.setCurrentKey(); // Clear unit tree selection
        // 保存清空单位的状态
        saveQueryState();

        // Load associated units
        loadUnitsByEquipModel(nodeData.value);
    }
};

// 处理装备型号下拉框清空
const handleEquipClear = () => {
    console.log('装备型号下拉框清空');
    isClearingEquip.value = true;
    selectedEquipModelCode.value = '';
    selectedEquipModelName.value = '';
    selectedUnitCode.value = '';
    selectedUnitName.value = '';
    unitOptions.value = [];
    unitTreeRef.value?.setCurrentKey();
    equipTreeRef.value?.setCurrentKey();
    tableData.value = [];
    totals.value = 0;

    // 清除保存的状态
    clearQueryState();

    isClearingEquip.value = false;
};

// 处理单位/分队下拉框清空
const handleUnitClear = () => {
    console.log('单位/分队下拉框清空');
    selectedUnitCode.value = '';
    selectedUnitName.value = '';
    unitTreeRef.value?.setCurrentKey();

    // 保存状态
    saveQueryState();
};

// 查询按钮点击事件
const handleLoadData = () => {
    saveQueryState(); // 保存查询状态
    loadData();
};

// 加载表格数据
const loadData = async () => {
    if (isClearingEquip.value) {
        console.log('正在清空装备型号，跳过 loadData');
        return;
    }

    if (!selectedEquipModelCode.value) {
        console.log('无选中的装备型号，表格数据清空');
        tableData.value = [];
        totals.value = 0;
        ElMessage.warning('请至少选择装备型号');

        // 保存状态
        saveQueryState();
        return;
    }

    isLoading.value = true;
    try {
        const params: any = {
            current: pageInfo.page,
            size: pageInfo.pageSize,
            equipModelCode: selectedEquipModelCode.value,
        };
        if (selectedUnitCode.value) {
            params.unitCode = selectedUnitCode.value;
        }

        console.log('加载表格数据，参数:', params);
        const response = await types(params);
        if (response.data) {
            tableData.value = response.data.records.map((item: any) => ({
                ...item,
                algorithm: item.algorithm || 'LSTM',
            }));
            totals.value = response.data.total;
            console.log('表格数据加载成功:', tableData.value);

            // 保存最新的表格数据到 sessionStorage
            saveQueryState();
        } else {
            console.error('后端返回的数据格式不正确：', response.data);
            tableData.value = [];
            totals.value = 0;
            ElMessage.error('获取数据失败：后端返回数据格式不正确');

            // 保存状态
            saveQueryState();
        }
    } catch (error) {
        console.error('获取历史故障预测数据失败：', error);
        tableData.value = [];
        totals.value = 0;
        ElMessage.error('获取历史故障预测数据失败，请稍后重试');
    } finally {
        isLoading.value = false;
    }
};

// 处理算法选择的变化
const handleAlgorithmChange = (row: TableItem, algorithm: string) => {
    row.algorithm = algorithm;
    console.log('Selected Algorithm:', algorithm, 'for Row:', row);

    // 保存状态（因为表格数据中的算法选择被修改了）
    saveQueryState();
};

// 检查行是否可选
const checkSelectable = () => {
    return true;
};

// 处理单选逻辑
const handleSelect = (_selection: TableItem[], row: TableItem) => {
    const table = tableRef.value;
    if (table) {
        table.clearSelection();
        table.toggleRowSelection(row, true);
    }
};

// 处理勾选事件
const handleSelectionChange = (rows: TableItem[]) => {
    selectedRows.value = rows.length > 0 ? [rows[rows.length - 1]] : [];
    console.log('Selected Rows:', selectedRows.value);
};

// 执行预测
const execute = async () => {
    try {
        if (selectedRows.value.length === 0) {
            ElMessage.warning('请先选择一行数据');
            return;
        }

        const equipModelCode = selectedRows.value[0].equipModelCode;
        const selectedAlgorithm = selectedRows.value[0].algorithm;
        const unitCode = selectedRows.value[0].unitCode;
        const userId = '001';

        const response = await predict({
            equipModelCode: equipModelCode,
            modelAlgorithm: selectedAlgorithm,
            userId: userId,
            unitCode: unitCode,
        });

        if (response && response.data) {
            const timeStamp = response.data[0].timeStamp;
            router.push({
                path: '/predict/typedetail',
                query: {
                    timeStamp: timeStamp,
                    equipModelCode: equipModelCode,
                }
            });
        loadData();
        } else {
            ElMessage.error('预测执行失败：后端返回数据格式不正确');
        }
    } catch (error) {
        console.error('Execute prediction failed:', error);
        ElMessage.error('执行预测失败，请稍后重试');
    }
};

// 处理预测详情跳转
const handleDetail = (row: TableItem) => {
    if (!row) {
        console.error('No row data provided');
        ElMessage.error('无法查看详情：未提供行数据');
        return;
    }

    if (!row.equipModelCode || !row.timeStamp) {
        console.error('Missing equipModelCode or timeStamp', row);
        ElMessage.warning('当前无最近预测结果，无法查看详情！');
        return;
    }

    if (row.showString === '无最近预测结果') {
        ElMessage.warning('当前无最近预测结果，无法查看详情！');
        return;
    }

    router.push({
        path: '/predict/typedetail',
        query: {
            equipModelCode: row.equipModelCode,
            timeStamp: row.timeStamp,
        },
    });
};

// 处理更多历史预测跳转
const handleHistory = (row: TableItem) => {
    if (!row) {
        console.error('No row data provided');
        return;
    }
    router.push({
        path: '/predict/typehistory',
        query: {
            equipModelCode: row.equipModelCode,
            unitCode: row.unitCode,
            unitName: row.unitName,
            equipSimpleName: row.equipSimpleName,
        },
    });
};

onMounted(() => {
    // 1. 先恢复查询状态
    restoreQueryState();

    // 2. 加载装备型号树形数据
    loadEquipModelTree();

    // 
    loadUnitTree();
    // 3. 如果已有选中的装备型号，则加载对应的单位数据
    if (selectedEquipModelCode.value) {
        console.log('恢复装备型号，加载对应单位数据:', selectedEquipModelCode.value);
        loadUnitsByEquipModel(selectedEquipModelCode.value);

        // 如果有已保存的表格数据，则不重新加载（提高回退体验）
        if (tableData.value.length === 0) {
            // 4. 如果恢复的查询条件有效，则自动加载表格数据
            console.log('查询条件有效，自动加载表格数据');
            loadData();
        }
    }
});
</script>

<style lang="less" scoped>
.no-header-select :deep(.el-table__header-wrapper .el-checkbox) {
    display: none;
}

.message {
    text-align: center;
    font-size: 16px;
    color: red;
}

.text-red {
    color: red;
}

.mt {
    margin-top: 20px;
}

.mb {
    margin-bottom: 20px;
}

.fr {
    float: right;
}

.sub-tree {
    ::v-deep .el-tree-node__content:hover {
        background-color: #e6f7ff !important;
    }

    ::v-deep .el-tree-node.is-current>.el-tree-node__content {
        background-color: #e6f7ff;
    }

    ::v-deep.el-tree__father {
        font-weight: bold;
        color: #409eff;
    }

    font-weight: normal;
}

.flex-container {
    display: flex;
    align-items: center;
    gap: 10px;
}

// 隐藏表头的复选框
:deep(.el-table__header-wrapper .el-checkbox) {
    display: none;
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px;
}

:deep(.el-select-group__title) {
    font-weight: bold;
    color: #409eff;
}
</style>