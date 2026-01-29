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
            <h2 class="title-margin">精细化装备历史故障诊断</h2>
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
          <el-table-column prop="diagnoseId" label="诊断编码" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="diagnoseTime" label="诊断时间" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="diagnoseResult" label="诊断结果" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="exceptionInfo" label="异常信息" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="fixTiming" label="修理时机" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="modelAlgorithm" label="算法模型" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="usingSuggestion" label="使用建议" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column label="操作" class-name="shadow-label" :show-overflow-tooltip="true">
            <template #default="scope">
              <el-button type="warning" class="no-background"
                @click="goToDiagnoseDetail(equipCode, scope.row.diagnoseId)">
                诊断详情
              </el-button>
            </template>
          </el-table-column>
        </el-table>

        <el-pagination class="fr mt mb" v-model:current-page="pageInfo.current" v-model:page-size="pageInfo.size"
          :page-sizes="[10, 20, 30, 40]" layout="sizes, prev, pager, next, jumper" :total="totals"
          @size-change="handleSizeChange" @current-change="handleCurrentChange" background />
      </el-card>
    </el-col>
  </el-row>
  </div>
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { diagnoseHistoryApi } from '@/api/fault/diagnoseHistory';

interface TableItem {
  diagnoseId: number;         // 诊断ID
  userId: string;            // 用户ID
  equipCode: string;         // 装备统一编码
  diagnoseTime: string;      // 诊断时间
  diagnoseResult: string;    // 诊断结果
  fixTiming: string;         // 修理时机
  usingSuggestion: string;   // 使用建议
  exceptionInfo: string;     // 异常信息
  modelAlgorithm: string;    // 算法模型
  modelVersion: string;      // 模型版本
  equipDiagPartList: Array<{
    diagnoseDetailId: number; // 诊断详情ID
    subsysId: string;         // 子系统ID
    partId: number;           // 部件ID
    failureReason: string;    // 故障原因
  }>;
}

const router = useRouter();
const route = useRoute();

const equipCode = ref('');
const equipSimpleName = ref('');
const unitSimpleName = ref(''); // 改为 unitSimpleName，与显示一致
const team = ref('');

const tableData = ref<TableItem[]>([]);
const totals = ref<number>(0);
const pageInfo = reactive({
  current: 1,
  size: 10,
});

const goBack = () => {
  // router.push('/diagnose/Refineddiagnosis');
    window.history.back()

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
  }
};

const storeSessionData = () => {
  if (equipCode.value) {
    safeSessionStorage.set('generalfaultdiagnosisEquipCode', equipCode.value);
  }
  if (equipSimpleName.value) {
    safeSessionStorage.set('generalfaultdiagnosisEquipSimpleName', equipSimpleName.value);
  }
};

onMounted(async () => {
  const queryEquipCode = route.query.equipCode as string;
  const queryEquipSimpleName = route.query.equipSimpleName as string;

  if (queryEquipCode) {
    equipCode.value = queryEquipCode;
  } else {
    const storageEquipCode = safeSessionStorage.get('generalfaultdiagnosisEquipCode');
    equipCode.value = storageEquipCode || '';
  }

  if (queryEquipSimpleName) {
    equipSimpleName.value = queryEquipSimpleName;
  } else {
    const storageEquipSimpleName = safeSessionStorage.get('generalfaultdiagnosisEquipSimpleName');
    equipSimpleName.value = storageEquipSimpleName || '99A';
  }

  unitSimpleName.value = route.query.unitSimpleName as string || '中部战区';
  team.value = route.query.team as string || '';
  console.log('Query equipCode from route:', queryEquipCode);
  console.log('Storage equipCode:', safeSessionStorage.get('generalfaultdiagnosisEquipCode'));
  console.log('Final equipCode:', equipCode.value);
  console.log('Query equipSimpleName from route:', queryEquipSimpleName);
  console.log('Storage equipSimpleName:', safeSessionStorage.get('generalfaultdiagnosisEquipSimpleName'));
  console.log('Final equipSimpleName:', equipSimpleName.value);
  storeSessionData();
  await loadData();
});

const loadData = async () => {
  try {
    const params = {
      current: pageInfo.current,
      size: pageInfo.size,
      equipCode: equipCode.value,
      equipSimpleName: equipSimpleName.value,
      unitSimpleName: unitSimpleName.value
    };

    const response = await diagnoseHistoryApi(params);

    if (response.code === 200) {
      tableData.value = response.data.records.map((item: any) => ({
        diagnoseId: item.diagnoseId,
        diagnoseTime: item.diagnoseTime,
        diagnoseResult: item.diagnoseResult,
        exceptionInfo: item.exceptionInfo,
        fixTiming: item.fixTiming,
        modelAlgorithm: item.modelAlgorithm,
        usingSuggestion: item.usingSuggestion
      }));
      totals.value = response.data.total;
      console.log('加载的表格数据:', tableData.value);
    } else {
      console.error('后端返回的数据格式不正确：', response.data);
    }
  } catch (error) {
    console.error('获取历史故障预测数据失败：', error);
  }
};

const handleSizeChange = (size: number) => {
  pageInfo.size = size;
  pageInfo.current = 1;
  loadData();
};

const handleCurrentChange = (page: number) => {
  pageInfo.current = page;
  loadData();
};

const goToDiagnoseDetail = (equipCode: string, diagnoseId: number) => {
  const row = tableData.value.find((item: any) => item.diagnoseId === diagnoseId);
  const equipSimpleName = route.query.equipSimpleName as string || safeSessionStorage.get('generalfaultdiagnosisEquipSimpleName') || '未知型号';
  router.push(`/predict/diagnosedetail?equipCode=${equipCode}&diagnoseId=${diagnoseId}&equipSimpleName=${encodeURIComponent(equipSimpleName)}`);
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
