<template>
    <div>
        <el-card>
        <div class="container">
            <div class="label">算法选择:</div>
            <div style="width: 100px;margin-left: -80px;">
                <el-select v-model="searchParams.modelAlgorithm">
                    <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
                </el-select>
            </div>
            <div class="label">装备型号:</div>
            <div class="select-wrapper">
                <el-select clearable v-model="searchParams.equipModelCode" placeholder="请选择装备型号" style="width: 100%">
                    <el-option-group v-for="group in equipmentModelOptions" :key="group.label" :label="group.label">
                        <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                            :value="option.value" />
                    </el-option-group>
                </el-select>
            </div>
            <div class="label">健康指数:</div>
            <div class="input-range">
                <el-row :gutter="10">
                    <el-col :span="6">
                        <el-input v-model="searchParams.minHealthIndex" @input="updateMinHealthIndex" type="number" :min="0" oninput="if(value>99)value=99;if(value<0)value=0" />
                    </el-col>
                    <el-col :span="4" class="dash">——</el-col>
                    <el-col :span="6">
                        <el-input v-model="searchParams.maxHealthIndex" @input="updateMaxHealthIndex" type="number"
                            :max="99" oninput="if(value>99)value=99;if(value<0)value=0" />
                    </el-col>
                </el-row>
            </div>
            <div class="button-wrapper">
                <el-button type="primary" @click="handleQuery">计算关联指数</el-button>
            </div>
        </div>
        <div class="container" style="border: none;">
            <div class="description" v-if="formattedDescription" v-html="formattedDescription" />
        </div>
        <div class="container" style="border: none;">
            <el-table style="margin-top: 15px;" :data="tableData" v-loading="loading" border
                :header-cell-style="headerCellStyle">
                <el-table-column label="因素类型" prop="factorType" sortable align="center"  :show-overflow-tooltip="true"/>
                <el-table-column label="因素内容" prop="factorContent" sortable align="center"  :show-overflow-tooltip="true"/>
                <el-table-column label="置信度" prop="support" sortable align="center"  :show-overflow-tooltip="true"/>
                <el-table-column label="支持度" prop="confidence" sortable align="center"  :show-overflow-tooltip="true"/>
                <el-table-column label="提升度" prop="lift" sortable align="center"  :show-overflow-tooltip="true"/>
            </el-table>
        </div>
        <el-pagination class="fr mt mb" v-model:current-page="pageInfo.page" v-model:page-size="pageInfo.size"
            :page-sizes="[10, 20, 30, 40]" layout="sizes, prev, pager, next, jumper" :total="totals"
            @size-change="handleSizeChange" @current-change="handleCurrentChange" background />
            </el-card>
    </div>
</template>

<script setup lang="ts">
import { reactive, ref, computed, onMounted } from 'vue';
import { useHttp } from '@/hooks/usersHttp';
import { get } from '@/utils/bgd-http';
import { ElMessage } from 'element-plus';
import { equipmodelApi } from '@/api/fault/equipmodel';

interface TableRow {
    type: string;
    content: string;
    support: number;
    confidence: number;
    lift: number;
}

interface ResponseData {
    code: number;
    message: string;
    data: {
        analysisPage: {
            content: TableRow[];
            totalElements: number;
        };
        description: string;
    };
}

interface SearchType {
    equipModelCode: string;
    maxHealthIndex: number;
    minHealthIndex: number;
    modelAlgorithm: string;
    page: number;
    size: number;
}

interface EquipmentModelOption {
    label: string;
    value: string;
}

interface EquipmentModelGroup {
    label: string;
    options: EquipmentModelOption[];
}

const searchParams = ref<SearchType>({
    equipModelCode: '',
    maxHealthIndex: 99,
    minHealthIndex: 0,
    modelAlgorithm: "Aprior",
    page: 1,
    size: 10,
});

const options = [
    {
        value: 'Aprior',
        label: 'Aprior',
    },
    {
        value: 'Spearmanr',
        label: 'Spearmanr',
    },

]

const description = ref<string>('');
const tableData = ref<TableRow[]>([]);
const pageInfo = reactive({
    page: 1,
    size: 10,
});
const totals = ref<number>(0);
const equipmentModelOptions = ref<EquipmentModelGroup[]>([]);

const headerCellStyle = {
    backgroundColor: '#F8F8F8',
    color: 'black',
    fontWeight: 'bold',
};

// Computed property to format the description with underlined numbers
const formattedDescription = computed(() => {
    if (!description.value) return '';
    return description.value.replace(
        /(\d+-\d+)/g,
        '<span style="text-decoration: underline;">$1</span>'
    );
});

const { loading } = useHttp<ResponseData>('/relate-analysis', searchParams);

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
                    value: item.equipModelCode,
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

const handleQuery = async () => {
    if (!searchParams.value.equipModelCode) {
        ElMessage.warning('请选择装备型号后再查询');
        return;
    }
    if (searchParams.value.maxHealthIndex > 100) {
        ElMessage.warning('最大健康指数不超过100');
        return;
    }
    if (searchParams.value.minHealthIndex < 0) {
        ElMessage.warning('最小健康指数不少于0');
        return;
    }
    try {
        loading.value = true;

        // 重点：后端只认 page 和 size，不要传 current
        const params = {
            equipModelCode: searchParams.value.equipModelCode,
            modelAlgorithm: searchParams.value.modelAlgorithm || undefined,
            minHealthIndex: searchParams.value.minHealthIndex,
            maxHealthIndex: searchParams.value.maxHealthIndex,
            page: pageInfo.page,     // 后端认 page
            size: pageInfo.size,        // 后端认 size
        };

        console.log('实际请求参数:', params);  // 关键！一定要看这个日志

        const response = await get('/relate-analysis', params);

        if (response.code === 200) {
            tableData.value = response.data.analysisPage.records || [];
            totals.value = response.data.analysisPage.total || 0;
            description.value = response.data.description || '';
        } else {
            tableData.value = [];
            totals.value = 0;
            description.value = '';
            ElMessage.error(`查询失败: ${response.message}`);
        }
    } catch (error) {
        console.error('查询失败:', error);
        ElMessage.error('查询失败，请稍后重试');
    } finally {
        loading.value = false;
    }
};

const handleSizeChange = (newSize: number) => {
    pageInfo.size = newSize;
    pageInfo.page = 1;
    handleQuery();
};


const handleCurrentChange = (newPage: number) => {
    pageInfo.page = newPage;
    handleQuery();
};

const updateMinHealthIndex = (value: string) => {
    const parsedValue = parseInt(value) || 0;
    if (parsedValue > searchParams.value.maxHealthIndex) {
        searchParams.value.minHealthIndex = searchParams.value.maxHealthIndex;
    } else {
        searchParams.value.minHealthIndex = parsedValue;
    }
};
const updateMaxHealthIndex = (value: string) => {
    let parsedValue = parseInt(value) || 99;

    // 强制限制最大值不超过99
    if (parsedValue > 99) {
        parsedValue = 99;
    }

    // 强制限制最小值不低于0
    if (parsedValue < 0) {
        parsedValue = 0;
    }

    if (parsedValue < searchParams.value.minHealthIndex) {
        searchParams.value.maxHealthIndex = searchParams.value.minHealthIndex;
    } else {
        searchParams.value.maxHealthIndex = parsedValue;
    }
};

onMounted(() => {
    loadEquipmentModels();
});
</script>

<style lang="less" scoped>
.container {
    width: 1500px;
    margin: 20px auto;
    padding: 15px;
    border: 1px solid gray;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.label {
    font-size: 14px;
    color: #333;
    font-weight: 500;
    width: 80px;
    text-align: right;
    margin-right: 25px;
}

.select-wrapper {
    width: 200px;
    margin-left: -80px;
}

.input-range {
    width: 300px;
    display: flex;
    align-items: center;
    margin-left: -80px;
}

.input-range .dash {
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 14px;
    color: #666;
}

.button-wrapper {
    width: 150px;
}

.el-button {
    width: 100%;
    font-size: 14px;
}

.description {
    font-size: 16px;
    color: #333;
    padding: 10px;
    text-align: left;
}

:deep(.el-select-group__title) {
    font-weight: bold;
    color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px;
}
</style>
