<template>
  <div>
  <el-row :gutter="20">
    <el-col :span="24">
      <el-card>
        <el-row>
          <el-col :span="1">
            <!-- 添加返回按钮 -->
            <el-button text @click="goBack" icon="ArrowLeft"
              style="font-size: 30px; margin-top: 18px; margin-left: 10px; padding: 0;" />
          </el-col>
          <el-col :span="15">
            <h2 class="title-margin">装备型号历史故障预测</h2>
          </el-col>
        </el-row>
        <el-row style="margin-top: 15px;">
          <el-col :span="1">
          </el-col>
          <el-col :span="6">
            <div>装备型号：{{ equipSimpleName }}</div>
          </el-col>
          <el-col :span="6">
            <div>所属单位：{{ unitName }}</div>
          </el-col>
          <el-col :span="6">
            <div>所属分队：{{ unitName }}</div>
          </el-col>
          <el-col :span="3">
          </el-col>
        </el-row>
        <el-row style="margin-top: 15px;"></el-row>
        <el-table :data="tableData" style="width: 100%" class="mt" border :header-cell-style="headerCellStyle">
          <el-table-column type="index" width="80" label="序号" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="predictTime" label="预测执行时间" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="equipNumber" label="装备总数 " class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="showString" label="未来三月故障分布" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column prop="modelAlgorithm" label="算法模型" class-name="shadow-label"  :show-overflow-tooltip="true"/>
          <el-table-column label="操作" class-name="shadow-label" :show-overflow-tooltip="true">
            <template #default="scope">
              <el-button type="warning" @click="handleDetail(scope.row)" class="no-background">
                预测详情
              </el-button>
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
    </el-col>
  </el-row>
</div></template>

<script lang="ts" setup>
import { ref, reactive, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { typeHistory } from '@/api/fault/type';

interface TableItem {
  predictTime: string;
  equipNumber: number;
  showString: string;
  modelAlgorithm: string;
  timeStamp: string;
}
// 设置表头样式
const headerCellStyle = {
  backgroundColor: '#BBBBBB', // 表头背景颜色
  color: 'black', // 表头字体颜色
  fontWeight: 'bold' // 表头字体加粗
};
const router = useRouter();

const tableData = ref<TableItem[]>([]);

const unitCode = ref('');
const equipModelCode = ref('');
const unitName = ref('');
const equipSimpleName = ref('');

const totals = ref<number>(0);
const pageInfo = reactive({
  page: 1,
  pageSize: 10,
});

const goBack = () => {
  window.history.back();
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
  if (equipModelCode.value) {
    safeSessionStorage.set('equipModelCode', equipModelCode.value);
  }
  if (unitCode.value) {
    safeSessionStorage.set('unitCode', unitCode.value);
  }
  if (equipSimpleName.value) {
    safeSessionStorage.set('equipSimpleName', equipSimpleName.value);
  }
  if (unitName.value) {
    safeSessionStorage.set('unitName', unitName.value);
  }
};

const loadData = async () => {
  try {
    const params = {
      current: pageInfo.page,
      equipModelCode: equipModelCode.value,
      unitCode: unitCode.value,
      size: pageInfo.pageSize,
    };
    const response = await typeHistory(params);
    totals.value = response.data.total; // 使用 totalElements 表示总数
    if (response.data) {
      tableData.value = response.data.records;
    } else {
      console.error('后端返回的数据格式不正确：', response.data);
    }
  } catch (error) {
    console.error('获取历史故障预测数据失败：', error);
  }
};

onMounted(() => {
  // Retrieve from route query or session storage
  const query = router.currentRoute.value.query;
  equipModelCode.value = String(query.equipModelCode || safeSessionStorage.get('equipModelCode') || '');
  unitCode.value = String(query.unitCode || safeSessionStorage.get('unitCode') || '');
  equipSimpleName.value = String(query.equipSimpleName || safeSessionStorage.get('equipSimpleName') || '');
  unitName.value = String(query.unitName || safeSessionStorage.get('unitName') || '');

  console.log('Loaded values:', { equipModelCode: equipModelCode.value, unitCode: unitCode.value, equipSimpleName: equipSimpleName.value, unitName: unitName.value });
  storeSessionData();
  loadData();
});

const handleSizeChange = (size: number) => {
  pageInfo.pageSize = size;   // 更新每页大小
  pageInfo.page = 1;          // 更改每页大小后重置到第一页
  loadData();                 // 重新加载数据
};

const handleCurrentChange = (page: number) => {
  pageInfo.page = page;       // 更新当前页码
  loadData();                 // 重新加载数据
};

const handleDetail = (row: TableItem) => {
  const selectedTimeStamp = row.timeStamp;
  console.log('Selected timeStamp:', selectedTimeStamp);
  // addTab("型号预测结果详情", "/predict/Typedetail", "Warning");
  // setCurrentTab("型号预测结果详情", "/predict/Typedetail");
  router.push(`/predict/typedetail?timeStamp=${selectedTimeStamp}`);
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