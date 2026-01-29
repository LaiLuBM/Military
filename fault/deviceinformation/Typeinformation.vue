<template>
    <el-card>
        <el-row :gutter="20">
            <el-col :span="4">
                <el-form>
                    <el-form-item label="业务活动">
                        <el-select clearable v-model="businessActivity" placeholder="请选择">
                            <el-option v-for="item in businessActivityOptions" :key="item.value" :label="item.label"
                                :value="item.value" />
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item label="单位/分队" prop="unitPath">
                                    <el-tree-select v-model="selectedUnit" :data="treeData" :props="treeProps" node-key="unitCode"
                                      placeholder="请选择单位（可搜索）" filterable clearable style="width: 100%" check-strictly
                                      @change="handleUnitSelect" />
                                  </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item label="装备型号">
                        <el-select clearable placeholder="装备型号" v-model="searchParams.equipModelCode">
                            <el-option-group v-for="group in equipmentTypeOptions" :key="group.label"
                                :label="group.label">
                                <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                                    :value="option.value"></el-option>
                            </el-option-group>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item>
                        <div class="flex-container">
                            <el-form>
                                <el-form-item label="装备统一编码">
                                    <el-input v-model="searchParams.equipCode" style="width: 240px"
                                        placeholder="请输入装备统一编码" />
                                </el-form-item>
                            </el-form>
                            <el-button type="primary" class="ml" @click="handleQuery">查询</el-button>
                            <!-- <el-button type="primary" @click="synchron">全量数据同步</el-button> -->
                        </div>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <el-row>
            <!-- <el-col :span="20">

            </el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item label="上次同步时间:">
                        <div style="margin-right: 10px;">{{ synchronizeTime }}</div>
                    </el-form-item>
                </el-form>
            </el-col> -->
        </el-row>
        <el-row>
            <el-table border class="mt" :data="dataList" v-loding="loading" :header-cell-style="headerCellStyle">
                <el-table-column type="selection" width="55" align="center" :show-overflow-tooltip="true"></el-table-column>
                
                <!-- 公共字段 -->
                <el-table-column label="装备统一编码" prop="equipCode" sortable align="center" :show-overflow-tooltip="true"></el-table-column>
                <el-table-column label="装备型号" prop="equipSimpleName" align="center" :show-overflow-tooltip="true"></el-table-column>
                <el-table-column label="所属单位" prop="unitFullname" align="center" :show-overflow-tooltip="true"></el-table-column>
                <el-table-column label="所属分队" prop="unitFullname" align="center" :show-overflow-tooltip="true"></el-table-column>

                <!-- 单装使用消耗 (默认显示) -->
                <template v-if="!businessActivity || businessActivity === 'usageConsumption'">
                    <el-table-column label="最近行动开始时间" width="111px" prop="useStartDate" sortable align="center" :show-overflow-tooltip="true">
                        <template #default="scope">{{ scope.row.useStartDate || '无' }}</template>
                    </el-table-column>
                    <el-table-column label="使用地点" prop="usePlace" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="剩余里程(公里)" width="96px" prop="residueMile" sortable align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="本次大修间隔期内累计行驶里程(公里)" width="150px" prop="workMile" sortable align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="本次中修间隔期内发动机累计消耗摩托小时" width="180px" prop="engineMotorHour" sortable align="center" :show-overflow-tooltip="true"></el-table-column>
                </template>

                <!-- 单装保养记录 -->
                <template v-if="businessActivity === 'maintenanceRecord'">
                    <el-table-column label="保养类别" prop="content" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="最近保养日期" prop="maintDate" sortable align="center" :show-overflow-tooltip="true">
                        <template #default="scope">{{ scope.row.maintDate || '无' }}</template>
                    </el-table-column>
                    <el-table-column label="保养内容" prop="maintCategoryDictId" align="center" :show-overflow-tooltip="true"></el-table-column>
                </template>

                <!-- 单装故障记录 -->
                <template v-if="businessActivity === 'faultRecord'">
                    <el-table-column label="故障类型" prop="malfType" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="故障部位" prop="malfLocation" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="最近故障日期" prop="malfDate" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="处理结果" prop="handleResult" align="center" :show-overflow-tooltip="true"></el-table-column>
                </template>

                <!-- 单装修理记录 -->
                <template v-if="businessActivity === 'repairRecord'">
                    <el-table-column label="修理类别" prop="repairGradeDictId" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="最近修理日期" prop="repairStartDate" sortable align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="修理内容" prop="repairContent" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="修理原因" prop="repairContent" align="center" :show-overflow-tooltip="true"></el-table-column>
                    <el-table-column label="修理状态" prop="repairStatus" align="center" :show-overflow-tooltip="true"></el-table-column>
                </template>

                <!-- 操作列 -->
                <el-table-column label="操作" width="200px" align="center" :show-overflow-tooltip="true">
                    <template #default="scope">
                        <el-button type="primary" text :disabled="!scope.row.equipCode" @click="handleHistory(
                            scope.row.equipCode || '',
                        )">
                            历史详情
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-col :span="24">
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
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { useHttp } from '@/hooks/usersHttp';
import { unitListApi } from '@/api/fault/unitlist';
import { equipmodelApi } from '@/api/fault/equipmodel';
import { ElMessage, type ElTree } from 'element-plus';
import { allDataApi } from '@/api/fault/alldata';


// Type definitions
interface ColumnConfig {
    label: string;
    prop: string;
    sortable: boolean; // Standardized to ensure consistent header appearance
    align: string;
    render?: boolean;
    formatter?: (row: DataItem, column: ColumnConfig) => string;
}

interface DataItem {
    equipCode: string;
    equipRecordId: string;
    unitSimpleName?: string;
    useStartDate?: string;
    usePlace?: string;
    residueMile?: number;
    workMile?: number;
    engineMotorHour?: number;
    content?: string;
    maintDate?: string;
    maintCategoryDictId?: string;
    faultType?: string;
    faultDescription?: string;
    repairTime?: string;
    repairResult?: string;
    equipModel?: { equipSimpleName: string };
    [key: string]: any;
}

interface SearchType {
    activityType: string;
    equipCode: string;
    equipModelCode: string;
    unitPath: string;
    current: number;
    size: number;
}

interface Tree {
    unitCode: number;
    parentUnitCode: any;
    unitSimpleName: string;
    unitPath: number;
    childrenUnits?: Tree[];
}

const synchronizeTime = ref('')

// Column configurations with consistent sortable and align properties
const columnsConfig: Record<string, ColumnConfig[]> = {
    usageConsumption: [
        { label: '装备统一编码', prop: 'equipCode', sortable: true, align: 'center' },
        { label: '装备型号', prop: 'equipSimpleName', sortable: false, align: 'center' },
        { label: '所属单位', prop: 'unitFullname', sortable: false, align: 'center' },
        { label: '所属分队', prop: 'unitFullname', sortable: false, align: 'center', },
        {
            label: '最近行动开始时间',
            prop: 'useStartDate',
            sortable: true,
            align: 'center',
            formatter: (row) => row.useStartDate || '无',
        },
        { label: '使用地点', prop: 'usePlace', sortable: false, align: 'center' },
        {
            label: '剩余里程(公里)',
            prop: 'residueMile',
            sortable: true,
            align: 'center',
            // formatter: (row) => (row.residueMile ? `${row.residueMile} km` : '无'),
        },
        {
            label: '本次大修间隔期内累计行驶里程(公里)',
            prop: 'workMile',
            sortable: true,
            align: 'center',
            // formatter: (row) => (row.workMile ? `${row.workMile} km` : '无'),
        },
        {
            label: '本次中修间隔期内发动机累计消耗摩托小时',
            prop: 'engineMotorHour',
            sortable: true,
            align: 'center',
            // formatter: (row) => (row.engineMotorHour ? `${row.engineMotorHour} h` : '无'),
        },
        { label: '操作', prop: 'action', sortable: false, align: 'center', render: true },
    ],
    maintenanceRecord: [
        { label: '装备统一编码', prop: 'equipCode', sortable: true, align: 'center' },
        { label: '装备型号', prop: 'equipSimpleName', sortable: false, align: 'center' },
        { label: '所属单位', prop: 'unitFullname', sortable: false, align: 'center' },
        { label: '所属分队', prop: 'unitFullname', sortable: false, align: 'center', },
        { label: '保养类别', prop: 'content', sortable: false, align: 'center' },
        {
            label: '最近保养日期',
            prop: 'maintDate',
            sortable: true,
            align: 'center',
            formatter: (row) => row.maintDate || '无',
        },
        { label: '保养内容', prop: 'maintCategoryDictId', sortable: false, align: 'center' },
        { label: '操作', prop: 'action', sortable: false, align: 'center', render: true },
    ],
    faultRecord: [
        { label: '装备统一编码', prop: 'equipCode', sortable: true, align: 'center' },
        { label: '装备型号', prop: 'equipSimpleName', sortable: false, align: 'center' },
        { label: '所属单位', prop: 'unitFullname', sortable: false, align: 'center' },
        { label: '所属分队', prop: 'unitFullname', sortable: false, align: 'center', },
        { label: '故障类型', prop: 'malfType', sortable: false, align: 'center' },
        { label: '故障部位', prop: 'malfLocation', sortable: false, align: 'center' },
        { label: '最近故障日期', prop: 'malfDate', sortable: false, align: 'center' },
        { label: '处理结果', prop: 'handleResult', sortable: false, align: 'center' },
        { label: '操作', prop: 'action', sortable: false, align: 'center', render: true },
    ],
    repairRecord: [
        { label: '装备统一编码', prop: 'equipCode', sortable: true, align: 'center' },
        { label: '装备型号', prop: 'equipSimpleName', sortable: false, align: 'center' },
        { label: '所属单位', prop: 'unitFullname', sortable: false, align: 'center' },
        { label: '所属分队', prop: 'unitFullname', sortable: false, align: 'center', },
        { label: '修理类别', prop: 'repairGradeDictId', sortable: false, align: 'center' },
        {
            label: '最近修理日期',
            prop: 'repairStartDate',
            sortable: true,
            align: 'center',
            // formatter: (row) => row.repairTime || '无',
        },
        { label: '修理内容', prop: 'repairContent', sortable: false, align: 'center' },
        { label: '修理原因', prop: 'repairContent', sortable: false, align: 'center' },
        { label: '修理状态', prop: 'repairStatus', sortable: false, align: 'center' },
        { label: '操作', prop: 'action', sortable: false, align: 'center', render: true },
    ],
};

// 业务活动与历史详情页面路径的映射
const historyPageMap: Record<string, { title: string; path: string }> = {
    usageConsumption: {
        title: '单装使用消耗历史',
        path: '/deviceinformation/usageConsumption',
    },
    maintenanceRecord: {
        title: '单装保养记录历史',
        path: '/deviceinformation/maintenanceRecord',
    },
    faultRecord: {
        title: '单装故障记录历史',
        path: '/deviceinformation/faultRecord',
    },
    repairRecord: {
        title: '单装修理记录历史',
        path: '/deviceinformation/repairRecord',
    },
};

// Refs and reactive state
const currentColumns = ref<ColumnConfig[]>(columnsConfig.usageConsumption);
// const totals = ref<number>(0);
const pageInfo = reactive({
    current: 1,
    size: 10,
});
const searchParams = ref<SearchType>({
    activityType: '',
    equipCode: '',
    equipModelCode: '',
    unitPath: '',
    current: 1,
    size: 10,
});
const { dataList, loadData, handleSizeChange, handleCurrentChange, loading, totals
} = useHttp('/equipActivityRecord', searchParams);
const businessActivity = ref<string>('');
const businessActivityOptions = ref([
    { label: '单装使用消耗', value: 'usageConsumption' },
    { label: '单装保养记录', value: 'maintenanceRecord' },
    { label: '单装故障记录', value: 'faultRecord' },
    { label: '单装修理记录', value: 'repairRecord' },
]);
const selectedUnit = ref<string>('');
const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref<Tree[]>([]);
const defaultProps = {
    children: 'childrenUnits',
    label: 'unitSimpleName',
};
const equipmentTypeOptions = ref<{ label: string; options: { label: string; value: string }[] }[]>([]);
const router = useRouter();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;

// Table header style
const headerCellStyle = {
    backgroundColor: '#F8F8F8', // 表头背景颜色
    color: 'black', // 表头字体颜色
    fontWeight: 'bold' // 表头字体加粗
}

// Handle history navigation
const handleHistory = (equipCode: string) => {
    if (!businessActivity.value) {
        ElMessage.warning('请先选择业务活动');
        return;
    }
    const selectedOption = businessActivityOptions.value.find(
        (option) => option.value === businessActivity.value
    );
    if (!selectedOption) {
        ElMessage.error('无效的业务活动类型');
        return;
    }
    const activityLabel = selectedOption.label; // 使用中文标签，例如“单装故障记录”
    const pageInfo = historyPageMap[businessActivity.value] || historyPageMap.faultRecord; // 回退到故障记录页面
    // addTab(pageInfo.title, pageInfo.path, 'Warning');
    // setCurrentTab(pageInfo.title, pageInfo.path);
    router.push({
        path: pageInfo.path,
        query: {
            equipCode,
            businessActivity: activityLabel // 使用中文标签
        },
    });
};

const synchron = async () => {
    // try {
    //     const res = await allDataApi()
    //     if (res.code === 200) {
    //         synchronizeTime.value = res.data.synchronizeTime
    //     }
    // } catch (error) {
    //     console.error('同步数据失败:', error);
    // }
}

// Query handler
const handleQuery = async () => {
    if (!businessActivity.value) {
        ElMessage.warning('请选择业务活动后再查询');
        return;
    }
    try {
        loading.value = true;
        currentColumns.value = columnsConfig[businessActivity.value] || columnsConfig.usageConsumption;
        pageInfo.current = 1;
        dataList.value = [];
        totals.value = 0;
        await loadData();
    } catch (error) {
        ElMessage.error('查询失败，请稍后重试');
    } finally {
        loading.value = false;
    }
};

// Fetch unit list on mount
onMounted(async () => {
    try {
        const { data } = await unitListApi();
        treeData.value = data;
    } catch (error) {
        console.error('Failed to fetch unit list:', error);
    }
    await synchron()
});

// Handle unit selection
const _deprecated_handleNodeClick = async (nodeData: Tree) => {
    selectedUnit.value = nodeData.unitSimpleName;
    searchParams.value.equipModelCode = '';
    searchParams.value.equipCode = '';
    const unitPath = nodeData.unitPath.toString();
    searchParams.value.unitPath = unitPath;

    try {
        const response = await equipmodelApi(unitPath);
        if (response.code === 200) {
            if (response.data && response.data.categories) {
                const groupedOptions: { label: string; options: { label: string; value: string }[] }[] = [];
                for (const category in response.data.categories) {
                    if (Array.isArray(response.data.categories[category])) {
                        const categoryOptions = response.data.categories[category].map((item: any) => ({
                            label: item.equipSimpleName,
                            value: item.equipModelCode,
                        }));
                        groupedOptions.push({
                            label: category,
                            options: categoryOptions,
                        });
                    }
                }
                equipmentTypeOptions.value = groupedOptions;
            } else {
                console.error('数据格式错误: categories 不存在或格式不正确');
                equipmentTypeOptions.value = [];
            }
        } else {
            console.error('请求失败:', response.message);
            equipmentTypeOptions.value = [];
        }
    } catch (error) {
        console.error('请求装备型号数据失败:', error);
        equipmentTypeOptions.value = [];
    }
};

// Watch businessActivity
watch(businessActivity, (newVal) => {
    currentColumns.value = columnsConfig[newVal] || columnsConfig.usageConsumption;
    const selectedOption = businessActivityOptions.value.find((option) => option.value === newVal);
    searchParams.value.activityType = selectedOption ? selectedOption.label : '';
    dataList.value = [];
    totals.value = 0;
    pageInfo.current = 1;
    if (newVal) {
        handleQuery();
    }
});

// Watch selectedUnit
watch(selectedUnit, (newVal) => {
    if (!newVal) {
        searchParams.value.unitPath = '';
        searchParams.value.equipModelCode = '';
        searchParams.value.equipCode = '';
        searchParams.value.activityType = '';
        businessActivity.value = '';
        equipmentTypeOptions.value = [];
    }
});


// 自动添加的模糊查询相关逻辑
const treeProps = defaultProps;

const handleUnitSelect = (newValue: any) => {
  console.log('选中单位：', newValue)
  // 找到对应的节点
  // 使用 loose type 避免 Tree 类型未定义问题
  const selectedNode = findNodeByLabel(treeData.value, newValue)
  if (!selectedNode) return
  
  if (searchParams.value.equipModelCode !== undefined) searchParams.value.equipModelCode = ''
  if (searchParams.value.equipCode !== undefined) searchParams.value.equipCode = ''
  
  const unitPath = selectedNode.unitPath ? selectedNode.unitPath.toString() : ''
  // 兼容不同的字段名，有的可能是 unitCode
  if (searchParams.value.unitPath !== undefined) searchParams.value.unitPath = unitPath
  else if ((searchParams.value as any).unitCode !== undefined) (searchParams.value as any).unitCode = unitPath // Some files used unitCode

  // 加载装备型号
  loadEquipModels(unitPath)
}

const findNodeByLabel = (nodes: any, label: any) => {
  if (!nodes) return undefined
  for (const node of nodes) {
    if (node.unitSimpleName === label) return node
    if (node.childrenUnits) {
      const found = findNodeByLabel(node.childrenUnits, label)
      if (found) return found
    }
  }
  return undefined
}

// 加载装备型号
// 注意：这里假设了 API 方法名为 equipmodelApi，如果报错请检查 import
const loadEquipModels = async (unitPath: any) => {
  try {
    // 尝试调用 API，API名称在替换时会尝试自动修正
    const response = await equipmodelApi(unitPath)
    if (response.code === 200 && response.data?.categories) {
      const groupedOptions: any[] = []
      Object.entries(response.data.categories).forEach(([category, items]) => {
         if (!Array.isArray(items)) {
          // console.error(`类别 ${category} 的数据不是数组:`, items);
          return;
        }

        const options = items.map((item: any) => ({
          label: item.equipSimpleName,
          value: item.equipModelCode,
        }));

        groupedOptions.push({
          label: category,
          options,
        });
      })

      if (typeof equipmentTypeOptions !== 'undefined') {
        equipmentTypeOptions.value = groupedOptions;
        // console.log('生成的 equipmentTypeOptions:', equipmentTypeOptions.value);
      }
       // 同时也尝试更新 equipmentCodeOptions，部分文件可能用这个
      if (typeof equipmentCodeOptions !== 'undefined') {
          equipmentCodeOptions.value = groupedOptions;
      }
    }
  } catch (e) {
    console.error('加载装备型号失败', e)
  }
}

</script>

<style lang="less" scoped>
.ml {
    margin-left: 10px;
}

.mt {
    margin-top: 20px;
}

.mb {
    margin-bottom: 20px;
}

.el-table {
    width: 100%;



    // Ensure consistent cell styling
    .el-table-column {
        text-align: center;
    }

    .el-select {
        width: 100%;
    }
}

.sub-tree {
    ::v-deep .el-tree-node__content:hover {
        background-color: #e6f7ff !important;
    }

    ::v-deep .el-tree-node.is-current>.el-tree-node__content {
        background-color: #e6f7ff;
    }

    font-weight: normal;
}

.flex-container {
    display: flex;
    align-items: center;
    gap: 10px;
}

.switch-style {
    margin-right: 0;
}

.input-style {
    flex: auto;
}

.el-pagination__sizes {
    font-size: 14px;
    color: #606266;
}

:deep(.el-select-group__title) {
    font-weight: bold;
    color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px;
}
</style>
