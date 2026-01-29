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
            <el-col :span="20">
              <h2 class="title-margin">精细化装备历史健康评估</h2>
            </el-col>
          </el-row>
          <el-row style="margin-top: 15px;">
            <el-col :span="2">
            </el-col>
            <el-col :span="6">
              <div>装备统一编码：{{ equipCode }}</div>
            </el-col>
            <el-col :span="6">
              <div>装备型号：{{ equipSimpleName }}</div>
            </el-col>
            <el-col :span="6">
              <div>所属单位/分队：{{ unitFullname }}</div>
            </el-col>
            <el-col :span="2">
            </el-col>
          </el-row>
          <el-row style="margin-top: 15px;">
          </el-row>
          <el-table :data="tableData" border :header-cell-style="headerCellStyle" class="mt">
            <el-table-column type="index" width="80" label="序号" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.motorHourAssessed" label="预测时中修间隔期内发动机累计消耗摩托小时数" class-name="shadow-label"
              align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.equipHealthScore" label="整机健康指数" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="subsysNameAndScore" label="分系统健康指数" class-name="shadow-label" align="center" :show-overflow-tooltip="true">
              <template #default="scope">
                <span>{{ formatSubsystemScores(scope.row.subsysNameAndScore) }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="result.equipHealthGrade" label="健康等级" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.assessmentMethod" label="评估方式" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.suggestionGeneratedStatus" label="是否已生成健康建议" class-name="shadow-label"
              align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.algorithm" label="算法模型" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.approvalStatus" label="核准状态" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="result.approvalSubmissionDate" label="提交核准时间" class-name="shadow-label"
              align="center"  :show-overflow-tooltip="true"/>
            <el-table-column label="操作" class-name="operation-column shadow-label" align="center" width="250" :show-overflow-tooltip="true">
              <template #default="scope">
                <span>
                  <el-button type="primary" text
                    @click="handleEquipInfo(scope.row.result.equipCode, scope.row.result.approvalStatus, scope.row.result.assessmentId)">
                    评估详情</el-button>
                </span>
                <span v-if="scope.row.result.suggestionGeneratedStatus === '未生成'">
                  <el-button type="primary" text
                    @click="generateHealthSuggestion(scope.row.result.assessmentId)">生成健康建议</el-button>
                </span>
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
  </div>
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
import { useRoute } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { equipHealthAssessmentHistory, advicegenerate } from '@/api/fault/refinehealth';
import { ElMessage } from 'element-plus'; // Import ElMessage for notifications

interface TableItem {
  serialNumber: number;
  result: {
    assessmentId: string;
    equipCode: string;
    assessmentDate: string;
    vehicleMotorAssessed: number;
    assessmentMethod: string;
    equipHealthScore: number;
    equipHealthGrade: string;
    suggestionGeneratedStatus: string;
    algorithm: string;
    approvalStatus: string;
    approvalSubmissionDate: string | null;
    subsysList: {
      subsysAssessmentId: string;
      subsysId: string;
      subsysHealthScore: string;
      subsysHealthGrade: string;
      subsysName: string;
      partList: {
        partAssessmentId: string;
        partId: string;
        partHealthScore: string;
        partHealthGrade: string;
        partName: string | null;
      }[];
    }[];
  };
  subsysNameAndScore: string;
}

const router = useRouter();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;

const tableData = ref<TableItem[]>([]);
const totals = ref<number>(0);
const equipCode = ref('');
const equipSimpleName = ref('');
const unitFullname = ref('');
const goBack = () => {
  // addTab("精细化健康评估", "/healthevaluate/refinedhealthassessments", "Warning")
  // setCurrentTab("精细化健康评估", "/healthevaluate/refinedhealthassessments")
  router.push('/healthevaluate/fininghealth')
}
const pageInfo = reactive({
  page: 1,
  pageSize: 10,
});

// 设置表头样式
const headerCellStyle = () => {
  return {
    backgroundColor: '#E8E8E8', // 默认表头背景颜色
    color: 'black', // 默认字体颜色
    fontWeight: 'bold', // 默认字体加粗
  };
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

const storeSessionData = () => {
  if (equipCode.value) {
    safeSessionStorage.set('equipCode', equipCode.value);
  }
};
const route = useRoute();
onMounted(async () => {
  const queryEquipCode = route.query.equipCode;
  if (queryEquipCode) {
    equipCode.value = String(queryEquipCode);
  } else {
    const storageEquipCode = safeSessionStorage.get('generalfaultdetailEquipCode');
    equipCode.value = storageEquipCode || '';
  }
  storeSessionData();
  loadData();
});

watch(() => route.query.equipCode, (newEquipCode) => {
  console.log('Watched equipCode change:', newEquipCode);
  if (newEquipCode) {
    equipCode.value = String(newEquipCode);
    console.log('Route updated equipCode:', equipCode.value);
    storeSessionData();
    loadData();
  }
});
const formatSubsystemScores = (subsysNameAndScore: string) => {
  if (!subsysNameAndScore || subsysNameAndScore === '{}') return '-';
  return subsysNameAndScore.slice(1, -1);
};

const loadData = async () => {
  try {
    const params = {
      equipCode: equipCode.value,
      page: pageInfo.page,
      pageSize: pageInfo.pageSize,
    };
    const response = await equipHealthAssessmentHistory(params);

    if (response.data && response.data.content && response.data.content.length > 0) {
      // 直接使用后端返回的 content 数组
      tableData.value = response.data.content;

      // 从第一个数据项中获取装备信息
      const firstItem = response.data.content[0];
      equipSimpleName.value = firstItem.equipSimpleName || '';
      unitFullname.value = firstItem.unitFullname || '';
      totals.value = response.data.totalElements || 0;

      console.log('后端返回的历史故障预测数据：', response.data);
      console.log('表格数据：', tableData.value);
      storeSessionData();
    } else {
      console.error('后端返回的数据格式不正确：', response.data);
      tableData.value = [];
      totals.value = 0;
      equipSimpleName.value = '';
      unitFullname.value = '';
    }
  } catch (error) {
    console.error('获取历史故障预测数据失败：', error);
    tableData.value = [];
    totals.value = 0;
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

const handleEquipInfo = (equipCode: string, approvalStatus: string, assessmentId: string) => {
  // addTab("精细化健康评估结果详情", "/healthevaluate/refinedetail", "Warning")
  // setCurrentTab("精细化健康评估结果详情", "/healthevaluate/refinedetail")
  console.log(equipCode, approvalStatus, assessmentId)
  router.push({
    path: "/healthevaluate/fininghealthdetail",
    query: { equipCode, approvalStatus, assessmentId }
  })
}

const generateHealthSuggestion = async (assessmentId: number) => {
  try {
    const response = await advicegenerate({ assessmentId });
    if (response.data) {
      ElMessage.success('健康建议生成成功');
      // Reload table data to reflect updated suggestionGeneratedStatus
      await loadData();
    } else {
      ElMessage.error('健康建议生成失败：' + (response.data?.message || '未知错误'));
    }
  } catch (error) {
    console.error('生成健康建议失败：', error);
    ElMessage.error('生成健康建议失败，请稍后重试');
  }
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

.operation-column .cell {
  background-color: #E6F0FA !important;
}
</style>