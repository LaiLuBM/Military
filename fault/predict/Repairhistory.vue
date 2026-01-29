<template>
    <div>
    <el-row :gutter="20">
        <el-col :span="24">
            <el-card class="card-with-margin">
                <el-row>
                    <el-col :span="1">
                        <!-- 添加返回按钮 -->
                        <el-button text @click="goBack" icon="ArrowLeft"
                            style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
                    </el-col>
                    <el-col :span="20">
                        <h2 class="title-margin">送修预测历史记录</h2>
                    </el-col>
                </el-row>
            </el-card>

            <el-card>
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

                <el-table :data="tableData" style="width: 100%" class="mt" border :header-cell-style="headerCellStyle">
                    <el-table-column type="index" width="80" label="序号" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="predictTime" label="预测执行时间" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="repairType" label="修理类别" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="repairTime" label="修理时间" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column prop="modelAlgorithm" label="算法模型" class-name="shadow-label"  :show-overflow-tooltip="true"/>
                    <el-table-column label="操作" class-name="shadow-label" :show-overflow-tooltip="true">
                        <template #default="{ row }">
                            <el-button type="warning" class="no-background" @click="handleDetail(row)">
                                预测详情
                            </el-button>
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
    </div>
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';
import { repairMoreHistoryApi } from '@/api/fault/repairhistory';

interface TableItem {
    repairId: number;
    predictTime: string;
    repairType: string;
    repairTime: string;
    modelAlgorithm: string;
    equipCode: string;
}

const router = useRouter();
const route = useRoute();

const equipCode = ref('');
const equipSimpleName = ref('');
const unitSimpleName = ref('');
const tableData = ref<TableItem[]>([]);
const totals = ref<number>(0);
const pageInfo = reactive({
    page: 1,
    pageSize: 10,
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

const storeSessionData = () => {
    if (equipCode.value) {
        safeSessionStorage.set('repairHistoryEquipCode', equipCode.value);
    }
    if (equipSimpleName.value) {
        safeSessionStorage.set('repairHistoryEquipSimpleName', equipSimpleName.value);
    }
};

const goBack = () => {
    // router.push('/predict/timepredict');
    window.history.back()

};

const loadData = async () => {
    if (!equipCode.value) {
        ElMessage.warning('请先选择装备统一编码');
        return;
    }

    try {
        const params = {
            current: pageInfo.page,
            size: pageInfo.pageSize,
            equipCode: equipCode.value,
            equipSimpleName: equipSimpleName.value,
            unitSimpleName: unitSimpleName.value
        };
        const response = await repairMoreHistoryApi(params);

        if (response.code === 200) {
            tableData.value = response.data.records.map((item: any) => ({
                repairId: item.repairId,
                predictTime: item.predictTime || '未知',
                repairType: item.repairType || '未知',
                repairTime: item.repairTime || '未知',
                modelAlgorithm: item.modelAlgorithm || '未知',
                equipCode: item.equipCode || equipCode.value
            }));
            totals.value = response.data.total;
        } else {
            console.error('后端返回的数据格式不正确：', response.message);
            ElMessage.error(`加载数据失败: ${response.message}`);
        }
    } catch (error) {
        console.error('获取历史故障预测数据失败：', error);
        ElMessage.error('加载数据失败，请稍后重试');
    }
};

const handleSizeChange = (size: number) => {
    pageInfo.pageSize = size;
    loadData();
};

const handleCurrentChange = (page: number) => {
    pageInfo.page = page;
    loadData();
};

const handleDetail = (row: TableItem) => {

    router.push({
        path: '/predict/repairdetail',
        query: {
            equipCode: row.equipCode,
            equipSimpleName: equipSimpleName.value,
            repairId: row.repairId
        }
    });
};

onMounted(async () => {
    const queryEquipCode = route.query.equipCode as string;
    const queryEquipSimpleName = route.query.equipSimpleName ? decodeURIComponent(String(route.query.equipSimpleName)) : '';
    const queryunitSimpleName = route.query.unitSimpleName as string;

    if (queryEquipCode) {
        equipCode.value = queryEquipCode;
    } else {
        equipCode.value = safeSessionStorage.get('repairHistoryEquipCode') || '';
    }

    if (queryEquipSimpleName) {
        equipSimpleName.value = queryEquipSimpleName;
    } else {
        equipSimpleName.value = safeSessionStorage.get('repairHistoryEquipSimpleName') || '未知型号';
    }

    if (queryunitSimpleName) {
        unitSimpleName.value = queryunitSimpleName;
    } else {
        unitSimpleName.value = safeSessionStorage.get('repairHistoryunitSimpleName') || '未知单位';
    }

    storeSessionData();
    await loadData();
});

watch(() => safeSessionStorage.get('selectedEquipments'), (newEquipments) => {
    if (newEquipments) {
        const equipments = JSON.parse(newEquipments);
        if (equipments.length > 0) {
            if (!equipCode.value) {
                equipCode.value = equipments[0].equipCode || '';
            }
            if (!equipSimpleName.value) {
                equipSimpleName.value = equipments[0].equipSimpleName || '未知型号';
            }
            if (!unitSimpleName.value) {
                unitSimpleName.value = equipments[0].unitSimpleName || '未知单位';
            }
            storeSessionData();
            loadData();
        }
    }
}, { immediate: true });

// 设置表头样式
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

.mt {
    margin-top: 20px;
}

.mb {
    margin-bottom: 20px;
}

.fr {
    float: right;
}
</style>