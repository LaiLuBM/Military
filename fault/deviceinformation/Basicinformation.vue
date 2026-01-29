<template>
    <el-card>
        <el-row :gutter="20">
            <el-col :span="6">
                <el-form>
                    <!-- 单位/分队选择下拉框，使用树形结构展示单位层级 -->
                    <el-form-item label="单位/分队" prop="unitPath">
                                    <el-tree-select v-model="selectedUnit" :data="treeData" :props="treeProps" node-key="unitCode"
                                      placeholder="请选择单位（可搜索）" filterable clearable style="width: 100%" check-strictly
                                      @change="handleUnitSelect" />
                                  </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="6">
                <el-form>
                    <!-- 装备型号选择下拉框，支持分组展示 -->
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
            <el-col :span="6">
                <el-form>
                    <!-- 装备统一编码输入框，支持回车触发查询 -->
                    <el-form-item label="装备统一编码">
                        <el-input placeholder="装备统一编码" v-model="searchParams.equipCode" @keyup.enter="loadData">
                            <!-- 移除错误的el-option -->
                        </el-input>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="6">
                <!-- 查询按钮，触发数据加载 -->
                <el-button type="primary" @click="loadData">查询</el-button>
                <!-- <el-button type="primary" @click="synchron">全量数据同步</el-button> -->
            </el-col>
        </el-row>
        <el-row>

            <!-- <el-col :span="20"></el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item label="上次同步时间:">
                        <div style="margin-right: 10px;">{{ synchronizeTime }}</div>
                    </el-form-item>
                </el-form>
            </el-col> -->
        </el-row>
        <!-- 数据表格，展示装备信息列表 -->
        <el-table border class="mt" :data="dataList" v-loding="loading" :header-cell-style="headerCellStyle">
            <el-table-column type="selection" width="55" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="装备统一编码" prop="equipCode" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="装备型号" prop="equipModel.equipSimpleName" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="所属单位" prop="unit.unitFullname" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="所属分队" prop="unit.unitFullname" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="列装时间" prop="serviceDate" sortable align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="服役日期" prop="serviceLimit" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="大修间隔期内累计行驶里程(公里)" width="128px" prop="usageRecord.workMile" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="中修间隔期内发动机累计消耗摩托小时" width="142px" prop="equipRecord.vehicleMotor" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="当年摩托小时消耗限额(小时)" width="124px" prop="usageRecord.residueMile" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="出厂日期" prop="manufactureDate" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column label="操作" width="200px" align="center" :show-overflow-tooltip="true">
                <template #default="scope">
                    <!-- 查看详情按钮，点击跳转到装备详情页面 -->
                    <el-button type="primary" size="small" text
                        @click="handleDetail(scope.row.equipCode)">查看详情</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页组件，支持调整每页条数和页面跳转 -->
        <el-row style="margin-top: 15px;">
        </el-row>
        <el-pagination class="fr mt mb" v-model:current-page="pageInfo.current" v-model:page-size="pageInfo.size"
            :page-sizes="[10, 20, 30, 40]" layout="sizes, prev, pager, next, jumper" :total="totals"
            @size-change="handleSizeChange" @current-change="handleCurrentChange" background>
            <template #sizes="{ pageSizes }">
                <span v-for="size in pageSizes" :key="size" class="el-pagination__sizes">
                    {{ size }}条/页
                </span>
            </template>
        </el-pagination>
        <el-row style="margin-bottom: 15px;">
        </el-row>
    </el-card>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from "vue"
import { useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { useHttp } from "@/hooks/usersHttp";
// import { basicInfoApi } from "@/api/basicinfo"; // 被注释掉的 API 导入，可能用于获取基础信息
import { unitListApi } from '@/api/fault/unitlist';
import type { ElTree } from "element-plus";
import { equipmodelApi } from "@/api/fault/equipmodel";
import { allDataApi } from "@/api/fault/alldata";

// 定义查询参数接口
interface SearchType {
    equipModel: string;
    equipCode: string;
    equipModelCode: string;
    day: number;
    ifUnpredicted: boolean;
    isDetailed: boolean;
    unitPath: string;
    current: number;
    size: number;
}

const searchParams = ref<SearchType>({
    equipModel: '',
    equipCode: '',
    equipModelCode: '',
    day: 0,
    ifUnpredicted: false,
    unitPath: '',
    current: 1,
    size: 10,
    isDetailed: false
});

// 定义表头样式
const headerCellStyle = {
    backgroundColor: '#F8F8F8', // 表头背景颜色
    color: 'black', // 表头字体颜色
    fontWeight: 'bold' // 表头字体加粗
}
const selectedUnit = ref(''); // 选中的单位
const synchronizeTime = ref('') // 上次同步时间
// 使用自定义 hook 获取数据和分页状态
const {
    loadData,
    loading,
    dataList, // 更新总条数
    pageInfo,
    handleSizeChange,
    handleCurrentChange,
    totals
} = useHttp('/equipWithUsage', searchParams)

// 定义树形结构接口
interface Tree {
    unitCode: number;
    parentUnitCode: any;
    unitSimpleName: string;
    unitPath: number;
    childrenUnits?: Tree[]
}
const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref<Tree[]>([]);
const defaultProps = {
    children: 'childrenUnits',
    label: 'unitSimpleName',
}

const router = useRouter()
// const tabsStore = useTabsStore()
// const { addTab, setCurrentTab } = tabsStore
// 处理查看详情按钮的点击事件，跳转到详情页面
const handleDetail = (equipCode: string) => {
    console.log('跳转时的 equipCode:', equipCode); // 调试输出
    // addTab("基本信息详情", "/devcieinformation/detailinformation", "Warning")
    // setCurrentTab("基本信息详情", "/devcieinformation/detailinformation")
    router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode)
}

// 装备型号和编码下拉框选项（分组格式）
const equipmentTypeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);

// 处理树节点点击事件，更新装备型号选项
const _deprecated_handleNodeClick = async (nodeData: Tree) => {
    // Clear previous selections
    selectedUnit.value = nodeData.unitSimpleName;
    searchParams.value.equipModelCode = ''; // Clear the selected equipment model
    searchParams.value.equipCode = ''; // Clear the equipment code if needed

    console.log('点击的节点数据:', nodeData);
    const unitPath = nodeData.unitPath.toString();
    searchParams.value.unitPath = unitPath;

    try {
        const response = await equipmodelApi(unitPath);
        console.log('装备型号数据:', response);
        if (response.code === 200) {
            console.log('请求成功:', response.message);
            if (response.data && response.data.categories) {
                // Reset and update equipment type options
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
                equipmentCodeOptions.value = groupedOptions;
            } else {
                console.error('数据格式错误: categories 不存在或格式不正确');
                equipmentTypeOptions.value = [];
                equipmentCodeOptions.value = [];
            }
        } else {
            console.error('请求失败:', response.message);
            equipmentTypeOptions.value = [];
            equipmentCodeOptions.value = [];
        }
    } catch (error) {
        console.error('请求装备型号数据失败:', error);
        equipmentTypeOptions.value = [];
        equipmentCodeOptions.value = [];
    }

    // Optionally trigger data reload if you want
    // loadData();
};

// 监听单位选择变化，清空相关选项
watch(selectedUnit, (newVal) => {
    if (!newVal) {
        // Unit selection was cleared
        searchParams.value.unitPath = '';
        searchParams.value.equipModelCode = '';
        searchParams.value.equipCode = '';
        equipmentTypeOptions.value = [];
        equipmentCodeOptions.value = [];
    }
});

// 全量数据同步
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
// 页面加载时获取单位列表和同步时间
onMounted(async () => {
    try {
        const { data } = await unitListApi();
        treeData.value = data; // 将后端返回的数据赋值给 treeData
    } catch (error) {
        console.error('Failed to fetch city list:', error);
    }
    await synchron()
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
.sub-tree {
    ::v-deep .el-tree-node__content:hover {
        background-color: #e6f7ff !important;
    }

    ::v-deep .el-tree-node.is-current>.el-tree-node__content {
        background-color: #e6f7ff;
    }

    font-weight: normal;
}

// 调整分组下拉框的样式
:deep(.el-select-group__title) {
    font-weight: bold; // 分组标题加粗
    color: #409eff; // 分组标题颜色
}

:deep(.el-select-group .el-select-dropdown__item) {
    padding-left: 30px; // 子选项缩进
}
</style>