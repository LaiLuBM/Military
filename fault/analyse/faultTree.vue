<template>
    <div>
        <el-card>
        <el-row :gutter="20">
            <el-col :span="6">
                <el-form>
                    <el-form-item label="故障树名称">
                        <el-input clearable placeholder="请输入故障树名称" v-model="searchParams.treeName" />
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="6">
                <el-form>
                    <el-form-item label="创建人">
                        <el-input clearable placeholder="请输入创建人" v-model="searchParams.creatorName"></el-input>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="6">
                <el-form>
                    <el-form-item label="创建时间">
                        <el-date-picker v-model="createTimeRange" type="daterange" range-separator="-"
                            start-placeholder="开始日期" end-placeholder="结束日期" @change="updateCreateTime" />
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="6">
                <el-form>
                    <el-form-item label="更新时间">
                        <el-date-picker v-model="updateTimeRange" type="daterange" range-separator="-"
                            start-placeholder="开始日期" end-placeholder="结束日期" @change="updateUpdateTime" />
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <el-row :gutter="20" style="margin-top: 10px;margin-left: 30px;">
            <el-col :span="4">
                <el-form>
                    <el-form-item label="状态">
                        <el-select v-model="searchParams.status" clearable>
                            <el-option label="未提交" value="未提交"></el-option>
                            <el-option label="待审核" value="待审核"></el-option>
                            <el-option label="已核准" value="已核准"></el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="10"></el-col>
            <el-col :span="4">
                <el-button type="primary" @click="loadData">查询</el-button>
                <el-button type="primary" @click="create">新增</el-button>
            </el-col>
        </el-row>
        <el-table border :header-cell-style="headerCellStyle" :data="dataList">
            <el-table-column align="center" label="故障树名称" prop="faultTree.treeName" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column align="center" label="创建时间" prop="faultTree.createTime" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column align="center" label="状态" prop="faultTree.status" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column align="center" label="创建人" prop="faultTree.creatorName" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column align="center" label="更新时间" prop="faultTree.updateTime" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column align="center" label="操作" :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-button text type="primary" @click="handleView(scope.row)">查看</el-button>
                    <el-button text type="danger" @click="handleDelete(scope.row)">删除</el-button>
                    <el-button text v-if="scope.row.faultTree.status === '未提交'" type="success"
                        @click="handleSubmit(scope.row)">提交审核</el-button>
                    <el-button text v-if="scope.row.faultTree.status === '已提交'" type="warning"
                        @click="handlePublish(scope.row)">发布</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination class="fr mt mb" v-model:current-page="pageInfo.current" v-model:page-size="pageInfo.size"
            :page-sizes="[10, 20, 30, 40]" layout="sizes, prev, pager, next, jumper" :total="totals"
            @size-change="handleSizeChange" @current-change="handleCurrentChange" background style="margin-top:10px">
            <template #sizes="{ pageSizes }">
                <span v-for="size in pageSizes" :key="size" class="el-pagination__sizes">
                    {{ size }}条/页
                </span>
            </template>
        </el-pagination>
        </el-card>
    </div>
</template>

<script setup lang="ts">
import { treeDeleteApi } from '@/api/fault/faultTree';
import { useHttp } from '@/hooks/userhttp';
import { ElMessage } from 'element-plus';
import { onActivated, onMounted, ref, getCurrentInstance } from 'vue';
import { useRouter, useRoute } from 'vue-router';

import { submitTreeApi } from '../../../api/fault/submitTree';
// import { useTabsStore } from '@/store/tabs';

// Reactive variables
const router = useRouter();
const route = useRoute();
const { proxy } = getCurrentInstance() as any;
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;

// 安全操作 sessionStorage
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

interface FaultTreeData {
    treeId: number | null;
    treeName: string;
    creatorName: string;
    description: string;
    solution: string;
    createTime: string;
    updateTime: string;
}

interface SearchType {
    createTimeEnd: string;
    createTimeStart: string;
    creatorName: string;
    page: number;
    pageSize: number;
    status: string;
    treeName: string;
    updateTimeEnd: string;
    updateTimeStart: string;
}

const searchParams = ref<SearchType>({
    createTimeEnd: '',
    createTimeStart: '',
    creatorName: '',
    page: 1,
    pageSize: 10,
    status: '',
    treeName: '',
    updateTimeEnd: '',
    updateTimeStart: ''
});

const createTimeRange = ref<string[]>([]);
const updateTimeRange = ref<string[]>([]);

// 更新创建时间范围
const updateCreateTime = (value: string[]) => {
    searchParams.value.createTimeStart = value[0] || '';
    searchParams.value.createTimeEnd = value[1] || '';
};

// 更新更新时间范围
const updateUpdateTime = (value: string[]) => {
    searchParams.value.updateTimeStart = value[0] || '';
    searchParams.value.updateTimeEnd = value[1] || '';
};

const {
    dataList,
    loadData,
    handleSizeChange,
    handleCurrentChange,
    pageInfo,
    totals,
} = useHttp('/fault-tree/list', searchParams);



const create = () => {
    // addTab("故障树列表新增", "/analyse/faulttree-add", "Warning");
    // setCurrentTab("故障树列表新增", "/analyse/faulttree-add");
    router.push('/predict/analyse/treeadd');
};

// 操作方法
const handleView = (row: any) => {
    const treeId = row.faultTree.treeId;
    const treeName = row.faultTree.treeName;
    // addTab("故障树列表编辑", "/analyse/faulttree-edit", "Warning");
    // setCurrentTab("故障树列表编辑", "/analyse/faulttree-edit");
    // router.push(`/analyse/treeEdit?treeId=${treeId}&treeName=${encodeURIComponent(treeName)}`);
     router.push({
        path: '/predict/analyse/treeEdit',
        query: { treeId: treeId,
                treeName:treeName
         }
    });
};

const handleDelete = async (row: any) => {
    const treeId = row.faultTree.treeId;
    proxy.$modal.confirm('是否确认删除故障树编号为"' + treeId + '"的数据项？').then(async () => {
        try {
            const response = await treeDeleteApi({ treeId });
            if (response.code === 200) {
                ElMessage.success('删除成功');
                loadData();
            } else {
                ElMessage.error('删除失败: ' + response.data.message);
            }
        } catch (error) {
            ElMessage.error('删除请求出错');
            console.error('Delete Error:', error);
        }
    }).catch(() => {});
};

const handleSubmit = async (row: any) => {
    try {
        const treeId = row.faultTree.treeId
        console.log(typeof (treeId))
        const res = await submitTreeApi(treeId);
        if (res.code === 200) {
            ElMessage.success("提交审核成功")
            loadData()
        }
        console.log('提交审核:', row.faultTree.treeId);
        // 实现提交审核逻辑
    } catch (error) {
        console.log(error)
    }

};

const handlePublish = (row: any) => {
    console.log('发布:', row);
    // 实现发布逻辑
};

// Table header style
const headerCellStyle = {
    backgroundColor: '#F8F8F8',
    color: 'black',
    fontWeight: 'bold'
};

// 初始化搜索参数并加载数据
const initSearchParams = () => {
    const dataUpdated = safeSessionStorage.get('dataUpdated');
    let faultTreeData: FaultTreeData | null = null;
    const faultTreeDataStr = safeSessionStorage.get('faultTreeData');
    if (faultTreeDataStr) {
        try {
            faultTreeData = JSON.parse(faultTreeDataStr) as FaultTreeData;
        } catch (e) {
            console.error('解析 faultTreeData 失败:', e);
        }
    }

    if (dataUpdated === 'true' || route.query.treeName || faultTreeData?.treeName) {
        searchParams.value.treeName = String(route.query.treeName || faultTreeData?.treeName || '');
        loadData();
        safeSessionStorage.set('dataUpdated', 'false');
    }
};

onMounted(() => {
    initSearchParams();
});

onActivated(() => {
    initSearchParams();
});
</script>