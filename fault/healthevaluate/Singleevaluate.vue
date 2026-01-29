<template><div>
    <el-card>
    <el-row :span="24" style="margin: 20px 0px 20px 0px;">
        使修维保建议规则查询：
    </el-row>
    <el-row :span="24">
        <el-col :span="6">
            <el-form-item label="建议类型：">
                <el-select style="width: 80%;" v-model="searchParams.suggestionType" clearable>
                    <el-option v-for="item in SuggestionOptions" :key="item.value" :label="item.label"
                        :value="item.value" />
                </el-select>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-form-item label="装备型号：">
                <el-select style="width: 80%;" v-model="searchParams.equipModel" clearable>
                    <el-option-group v-for="group in equipmentModelOptions" :key="group.label" :label="group.label">
                        <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                            :value="option.value" />
                    </el-option-group>
                </el-select>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-button type="primary" @click="loadData">查询</el-button>
        </el-col>
    </el-row>
    <el-row :span="24" style="margin-bottom: 20px;">
        规则列表：
        <el-button type="primary" @click="handle">新增</el-button>
    </el-row>
    <el-row :span="24">
        <el-table :data="tableData" class="mt-el-table" border :header-cell-style="headerCellStyle">
            <el-table-column type="index" width="80" label="序号" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="ruleName" label="规则名称" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="suggestionType" label="建议类型" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="effectiveTime" label="生效时间" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="expiryTime" label="失效时间" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column label="是否启用" class-name="shadow-label" align="center" width="100" :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-switch v-model="scope.row.isEnabled" :active-value="true" :inactive-value="false"
                        @change="handleSwitchChange(scope.row)" />
                </template>
            </el-table-column>
            <el-table-column prop="conditionContent" label="规则条件内容" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="priority" label="建议结论" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="impactScope" label="影响范围" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="relatedTask" label="关联任务" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="detailedSuggestion" label="详细建议" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="applicableObject" label="适用装备" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column label="规则编辑" class-name="operation-column shadow-label" align="center" width="300" :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-button type="primary" text @click="handleEdit(scope.row)">编辑</el-button>
                    <el-button type="primary" text @click="handleDelete(scope.row.ruleName)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination class="fr mt mb" v-model:current-page="pageInfo.current" v-model:page-size="pageInfo.size"
            :page-sizes="[10, 20, 30, 40]" layout="sizes, prev, pager, next, jumper" :total="totals"
            @size-change="handleSizeChange" @current-change="handleCurrentChange" background>
            <template #sizes="{ pageSizes }">
                <span v-for="size in pageSizes" :key="size" class="el-pagination__sizes">
                    {{ size }}条/页
                </span>
            </template>
        </el-pagination>
    </el-row>
</el-card></div></template>

<script lang="ts" setup>
import { ref, reactive, onMounted, watch, onActivated, getCurrentInstance } from 'vue';
import { useRouter, useRoute } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { RepairQueryApi } from '@/api/fault/repair-conditions-query';
import { RepairDeleteApi } from '@/api/fault/repair-conditions-query';
import { RepairUpdateApi } from '@/api/fault/repair-conditions-query';
import { equipmodelApi } from '@/api/fault/equipmodel';

import { ElMessage } from 'element-plus';
import { debounce } from 'lodash';

// Reactive variables
const router = useRouter();
const route = useRoute();
const { proxy } = getCurrentInstance() as any;
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;
const totals = ref<number>(0);
const pageInfo = reactive({
    current: 1,
    size: 10,
});
const tableData = ref<any[]>([]);

const SuggestionOptions = ref([
   
     { label: '单装保养建议', value: '单装保养建议' },
    
]);


interface EquipmentModelOption {
    label: string;
    value: string;
}

interface EquipmentModelGroup {
    label: string;
    options: EquipmentModelOption[];
}
const equipmentModelOptions = ref<EquipmentModelGroup[]>([]);


interface SearchType {
    equipModel: string;
    suggestionType: string;
    current: number;
    size: number;
}
const searchParams = ref<SearchType>({
    equipModel: '', // Default value for 装备型号
    suggestionType: '', // Default value for 建议类型
    current: 1,
    size: 10
});


const handleCurrentChange = (page: number) => {
    pageInfo.current = page;
    loadData();
};

const handleSizeChange = (size: number) => {
    pageInfo.size = size;
    pageInfo.current = 1;
    loadData();
};

const loadData = debounce(async () => {
    try {
        const res = await RepairQueryApi(searchParams.value);
        if (res.code === 200 && res.data) {
            tableData.value = res.data;
            totals.value = res.data.length; // Update total for pagination
            ElMessage.success('数据加载成功');
        } else {
            ElMessage.error('数据加载失败: ' + (res.message || '未知错误'));
            tableData.value = [];
            totals.value = 0;
        }
    } catch (error) {
        console.error('Error loading data:', error);
        ElMessage.error('数据加载失败，请稍后重试');
        tableData.value = [];
        totals.value = 0;
    }// 现有 loadData 逻辑
}, 300);

const handle = async () => {
    const query = {
        ruleName: '',
        applicableObject: searchParams.value.equipModel || '', // 默认从装备型号获取
        suggestionType: searchParams.value.suggestionType || '视情维修建议', // 默认建议类型
        effectiveTime: '',
        expiryTime: '',
        conditionContent: '',
        priority: '',
        impactScope: '',
        detailedSuggestion: '',
        relatedTask: '',
        isEnabled: 'true',
    };

    // addTab("使修维保建议规则编辑", "/healthevaluate/regularEdit", "Warning");
    // setCurrentTab("使修维保建议规则编辑", "/healthevaluate/regularEdit");
    router.push({
        path: '/healthevaluate/regularEdit',
        query: query,
    });
};

const formatDateTime = (dateStr: any) => {
    if (!dateStr) return null;
    return dateStr.replace(' ', 'T'); // Convert 'yyyy-MM-dd HH:mm:ss' to 'yyyy-MM-dd'T'HH:mm:ss'
};

const handleSwitchChange = async (row: any) => {
    try {
        const Editparams = {
            id: Number(row.id),
            ruleName: row.ruleName,
            applicableObject: row.applicableObject,
            suggestionType: row.suggestionType,
            effectiveTime: formatDateTime(row.effectiveTime), // Use formatDateTime
            expiryTime: formatDateTime(row.expiryTime), // Use formatDateTime
            conditionContent: row.conditionContent,
            priority: row.priority,
            impactScope: row.impactScope,
            detailedSuggestion: row.detailedSuggestion,
            relatedTask: row.relatedTask,
            isEnabled: row.isEnabled,
        };

        const response = await RepairUpdateApi(Editparams);
        if (response.code === 200) {
            ElMessage.success('状态更新成功');
            loadData();
        } else {
            ElMessage.error('状态更新失败: ' + (response.message || '未知错误'));
            row.isEnabled = !row.isEnabled; // Revert on failure
        }
    } catch (error) {
        console.error('Error updating status:', error);
        ElMessage.error('状态更新失败，请稍后重试');
        row.isEnabled = !row.isEnabled; // Revert on failure
    }
};

const handleEdit = async (row: any) => {
    try {
        // Ensure row.id exists and is a valid number
        if (!row.id) {
            ElMessage.error('无法编辑：缺少ID');
            return;
        }

        // Format effectiveTime and expiryTime to match the expected LocalDateTime format
        const formatDateTime = (dateStr: any) => {
            if (!dateStr) return null;
            // Assuming the input is 'yyyy-MM-dd HH:mm:ss' and we need 'yyyy-MM-dd'T'HH:mm:ss'
            return dateStr.replace(' ', 'T');
        };

        // Prepare the update payload with formatted date-time values
        const Editparams = {
            id: Number(row.id), // Ensure id is a number
            ruleName: row.ruleName,
            applicableObject: row.applicableObject,
            suggestionType: row.suggestionType,
            effectiveTime: formatDateTime(row.effectiveTime), // Format the date-time
            expiryTime: formatDateTime(row.expiryTime), // Format the date-time
            conditionContent: row.conditionContent,
            priority: row.priority,
            impactScope: row.impactScope,
            detailedSuggestion: row.detailedSuggestion,
            relatedTask: row.relatedTask,
            isEnabled: row.isEnabled
        };

        // Call the RepairUpdateApi to update the data
        const response = await RepairUpdateApi(Editparams);
        console.log("参数:", Editparams)
        if (response.code === 200) {
            ElMessage.success('数据更新成功');
            // Proceed with navigation after successful update
            // addTab("使修维保建议规则编辑", "/healthevaluate/regularEdit", "Warning");
            // setCurrentTab("使修维保建议规则编辑", "/healthevaluate/regularEdit");
            router.push({
                path: "/healthevaluate/regularEdit",
                query: {
                    id: row.id.toString(), // 添加 id
                    ruleName: row.ruleName,
                    applicableObject: row.applicableObject,
                    suggestionType: row.suggestionType,
                    effectiveTime: row.effectiveTime, // Keep original for query params
                    expiryTime: row.expiryTime, // Keep original for query params
                    conditionContent: row.conditionContent,
                    priority: row.priority,
                    impactScope: row.impactScope,
                    detailedSuggestion: row.detailedSuggestion,
                    relatedTask: row.relatedTask,
                    isEnabled: row.isEnabled.toString(),
                }
            });
        } else {
            ElMessage.error('数据更新失败: ' + (response.message || '未知错误'));
        }
    } catch (error) {
        console.error('Error updating data:', error);
        ElMessage.error('数据更新失败，请稍后重试');
    }
};
const handleDelete = async (ruleName: any) => {
    proxy.$modal.confirm('是否确认删除规则名称为"' + ruleName + '"的数据项？').then(async () => {
        try {
            // Replace with actual API call to delete the rule
            // const response = await deleteRuleApi({ id });
            // For demo, we'll simulate a successful deletion
            const deleteParam = { ruleName }
            const response = await RepairDeleteApi(deleteParam);
            if (response.code === 200) {
                ElMessage.success('删除成功');
                loadData(); // Refresh table data
            } else {
                ElMessage.error('删除失败: ' + (response.message || '未知错误'));
            }
        } catch (error) {
            console.error('Error deleting rule:', error);
            ElMessage.error('删除失败，请稍后重试');
        }
    }).catch(() => {});
};

const safeSessionStorage = {
    set(key: string, value: string): void {
        try {
            sessionStorage.setItem(key, value);
        } catch (e) {
            console.error('SessionStorage存储失败:', e);
        }
    },
    get(key: string): string {
        try {
            return sessionStorage.getItem(key) || '';
        } catch (e) {
            console.error('SessionStorage读取失败:', e);
            return '';
        }
    },
};

const initSearchParams = () => {
    const suggestionTypeFromQuery = route.query.suggestionType;
    const suggestionTypeFromStorage = safeSessionStorage.get('regularEditSuggestionType');
    if (suggestionTypeFromQuery) {
        searchParams.value.suggestionType = String(suggestionTypeFromQuery);
    } else if (suggestionTypeFromStorage) {
        searchParams.value.suggestionType = suggestionTypeFromStorage;
    }
};

// 加载装备型号数据
const loadEquipmentModels = async () => {
    const defaultUnitPath = '/11000/11001/';
    try {
        const response = await equipmodelApi(defaultUnitPath);
        console.log('装备型号数据:', response);

        if (response.code === 200 && response.data?.categories) {
            const groupedOptions = Object.entries(response.data.categories).map(([category, items]) => {
                if (!Array.isArray(items)) {
                    console.error(`类别 ${category} 的数据不是数组:`, items);
                    return { label: category, options: [] };
                }
                const options = items.map((item: any) => ({
                    label: item.equipSimpleName,
                    value: item.equipSimpleName,
                }));
                return {
                    label: category,
                    options,
                };
            });
            equipmentModelOptions.value = groupedOptions;
            console.log('生成的 equipmentModelOptions:', equipmentModelOptions.value);
        } else {
            console.error('数据格式错误或请求失败:', response.message);
            equipmentModelOptions.value = [];
            ElMessage.error('获取装备型号失败: ' + response.message);
        }
    } catch (error) {
        console.error('请求装备型号数据失败:', error);
        equipmentModelOptions.value = [];
        ElMessage.error('获取装备型号失败，请稍后重试');
    }
};

onMounted(async () => {
    loadEquipmentModels();
    initSearchParams();
    const dataUpdated = safeSessionStorage.get('dataUpdated');
    if (dataUpdated === 'true' || searchParams.value.suggestionType) {
        await loadData();
        safeSessionStorage.set('dataUpdated', 'false');
    } else {
        await loadData();
    }
});

onActivated(async () => {
    initSearchParams();
    const dataUpdated = safeSessionStorage.get('dataUpdated');
    if (dataUpdated === 'true' || searchParams.value.suggestionType) {
        await loadData();
        safeSessionStorage.set('dataUpdated', 'false');
    }
});
// 可选：优化 watch 逻辑，使用响应式变量
const dataUpdated = ref(safeSessionStorage.get('dataUpdated'));
watch(dataUpdated, async (newValue) => {
    if (newValue === 'true') {
        await loadData();
        safeSessionStorage.set('dataUpdated', 'false');
        dataUpdated.value = 'false';
    }
});
// 监听 sessionStorage 变化（通过轮询或事件）
const checkSessionStorage = () => {
    dataUpdated.value = safeSessionStorage.get('dataUpdated');
};
watch(() => safeSessionStorage.get('dataUpdated'), checkSessionStorage);
// Header style
const headerCellStyle = {
    backgroundColor: '#F8F8F8', // 表头背景颜色
    color: 'black', // 表头字体颜色
    fontWeight: 'bold' // 表头字体加粗
}
</script>

<style scoped>
.mt {
    margin-top: 20px;
}

.mb {
    margin-bottom: 20px;
}

.fr {
    float: right;
}

.mt-el-table {
    margin-top: 20px;
}

:deep(.el-select-group__title) {
    font-weight: bold;
    color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px;
}
</style>