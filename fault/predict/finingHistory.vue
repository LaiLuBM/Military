<template>
    <div>
    <el-card>
        <el-row :gutter="20">
            <el-col :span="24">
                <el-row>
                    <el-col :span="1">
                        <!-- 添加返回按钮 -->
                        <el-button text @click="goBack" icon="ArrowLeft"
                            style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
                    </el-col>
                    <el-col :span="20">
                        <h2 class="title-margin">装备精细化历史故障预测</h2>
                    </el-col>
                </el-row>

                <el-row style="margin-top: 15px;">
                    <el-col :span="6">
                        <div>所属单位：{{ unitSimpleName }}</div>
                    </el-col>
                    <el-col :span="6">
                        <div>装备型号：{{ equipSimpleName }}</div>
                    </el-col>
                    
                    <el-col :span="6">
                        <div>装备统一编码：{{ equipCode }}</div>
                    </el-col>
                    
                </el-row>
                <el-row style="margin-top: 15px;"></el-row>
                <el-table :data="tableData" border :header-cell-style="headerCellStyle">
                    <el-table-column type="index" width="80" label="序号" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="predictTime" label="预测执行时间" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="failureType" label="故障类型" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="failureProbability" label="月份对应故障概率" class-name="shadow-label" :show-overflow-tooltip="true">
                        <template #default="scope">
                            {{ formatPercentage(scope.row.failureProbability) }}
                        </template>
                    </el-table-column>
                    <el-table-column prop="failureReason" label="故障详情" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="modelAlgorithm" label="算法模型" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column label="操作" class-name="shadow-label" :show-overflow-tooltip="true">
                        <template #default="scope">
                            <el-button type="warning" @click="handleDetail(equipCode, scope.row.predictId)"
                                class="no-background">
                                故障预测结果详情
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <el-row style="margin-top: 15px;"></el-row>
                <el-pagination class="fr mt mb" v-model:current-page="pageInfo.current"
                    v-model:page-size="pageInfo.size" :page-sizes="[10, 20, 30, 40]"
                    layout="sizes, prev, pager, next, jumper" :total="totals" @size-change="handleSizeChange"
                    @current-change="handleCurrentChange" background>
                    <template #sizes="{ pageSizes }">
                        <span v-for="size in pageSizes" :key="size" class="el-pagination__sizes">
                            {{ size }}条/页
                        </span>
                    </template>
                </el-pagination>

            </el-col>
        </el-row>
    </el-card>
    </div>
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { listApi } from '@/api/fault/historicalpredict';

interface TableItem {
    predictTime: string;
    failureType: string;
    failureProbability: number;
    failureReason: string;
    modelAlgorithm: string;
    predictId?: number;
    failureLocation?: string;
    monthProbability?: string;
}

const router = useRouter();
const route = useRoute();

const equipCode = ref('');
const equipSimpleName = ref('');
const unitSimpleName = ref('')
const team = ref('');
const tableData = ref<TableItem[]>([]);
const totals = ref<number>(0);
const pageInfo = reactive({
    current: 1,
    size: 10,
});

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
    }
};
const formatPercentage = (value: number | null | undefined): string => {
    if (value == null || isNaN(value)) {
        return '-'; // 处理空值或无效值
    }
    return `${(value * 100).toFixed(0)}%`; // 转换为百分比，保留0位小数
};
const storeSessionData = () => {
    if (equipCode.value) {
        safeSessionStorage.set('finingHistoryEquipCode', equipCode.value);
    }
    if (equipSimpleName.value) {
        safeSessionStorage.set('finingHistoryEquipSimpleName', equipSimpleName.value);
    }
    if (unitSimpleName.value) {
        safeSessionStorage.set('generalHistoryunitSimpleName', unitSimpleName.value);
    }
};

const loadData = async () => {
    try {
        const params = {
            current: pageInfo.current,
            size: pageInfo.size,
            equipCode: equipCode.value,
            equipSimpleName: equipSimpleName.value,
            isDetailed: true
        };
        const response = await listApi(params);

        if (response.code == 200) {
            tableData.value = response.data.records.map((item: any) => ({
                predictTime: item.predictTime,
                failureType: item.failureType,
                failureProbability: parseFloat(item.failureProbability),
                failureReason: item.failureReason,
                modelAlgorithm: item.modelAlgorithm,
                predictId: item.predictId,
                failureLocation: item.failureLocation,
                monthProbability: item.monthProbability
            }));
            totals.value = response.data.total;
            console.log('Loaded tableData:', tableData.value);
        } else {
            console.error('后端返回的数据格式不正确：', response.data);
        }
    } catch (error) {
        console.error('获取历史故障预测数据失败：', error);
    }
};

onMounted(async () => {
    const queryEquipCode = route.query.equipCode as string;
    const queryEquipSimpleName = route.query.equipSimpleName as string;
    const queryunitSimpleName = route.query.unitSimpleName

    if (queryEquipCode) {
        equipCode.value = queryEquipCode;
    } else {
        const storageEquipCode = safeSessionStorage.get('finingHistoryEquipCode');
        equipCode.value = storageEquipCode || '';
    }

    if (queryEquipSimpleName) {
        equipSimpleName.value = queryEquipSimpleName;
    } else {
        const storageEquipSimpleName = safeSessionStorage.get('finingHistoryEquipSimpleName');
        equipSimpleName.value = storageEquipSimpleName || '99A';
    }
    if (queryunitSimpleName) {
        unitSimpleName.value = String(queryunitSimpleName);
    } else {
        const storageEquipSimpleName = safeSessionStorage.get('generalHistoryunitSimpleName');
        unitSimpleName.value = storageEquipSimpleName || '';
    }
    // unit.value = route.query.unit as string || '中部战区';
    team.value = route.query.team as string || '';
    console.log('Query equipCode from route:', queryEquipCode);
    console.log('Storage equipCode:', safeSessionStorage.get('finingHistoryEquipCode'));
    console.log('Final equipCode:', equipCode.value);
    console.log('Query equipSimpleName from route:', queryEquipSimpleName);
    console.log('Storage equipSimpleName:', safeSessionStorage.get('finingHistoryEquipSimpleName'));
    console.log('Final equipSimpleName:', equipSimpleName.value);
    storeSessionData();
    await loadData();
});

const handleSizeChange = (size: number) => {
    pageInfo.size = size;
    pageInfo.current = 1;
    loadData();
};

const handleCurrentChange = (page: number) => {
    pageInfo.current = page;
    loadData();
};

const handleDetail = (equipCode: string, predictId: number) => {
    const equipSimpleName = route.query.equipSimpleName as string || safeSessionStorage.get('finingHistoryEquipSimpleName') || '未知型号';
    router.push(`/predict/finingdetail?equipCode=${equipCode}&predictId=${predictId}&equipSimpleName=${encodeURIComponent(equipSimpleName)}`);
};

const goBack = () => {
    // router.push('/predict/finingfaultpredict');
    window.history.back()

};

const headerCellStyle = {
    backgroundColor: '#BBBBBB',
    color: 'black',
    fontWeight: 'bold'
};
</script>

<style scoped>
.no-background {
    background-color: transparent !important;
    border-color: transparent !important;
    color: rgba(53, 102, 236, 0.979) !important;
}

.title-margin {
    margin-bottom: 5px;
}

.shadow-label .cell {
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 5px;
}

.card-with-margin {
    margin-bottom: 20px;
}

.el-pagination__sizes {
    font-size: 14px;
    color: #606266;
}
</style>
```