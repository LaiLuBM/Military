
<template><div>
    <el-row :gutter="20">
        <el-col :span="1">
            <!-- 返回按钮 -->
            <el-button text icon="Back" style="font-size: 30px; padding: 0; margin-left: 15px;" @click="goBack" />
        </el-col>
        <el-col :span="8">
            <h2>单装修理记录详情</h2>
        </el-col>
    </el-row>
    <div class="title">
        <div class="bar fl"></div>
        <div class="fl describe">基础信息</div>
        <div style="clear: both;"></div>
    </div>
    <el-row class="title">
        <el-col :span="6">
            <div>装备统一编码：{{ equipCode }}</div>
        </el-col>
        <el-col :span="6">
            <div>装备统一编码：{{ equipSimpleName }}</div>
        </el-col>
        <el-col :span="6">
            <div>所属单位：{{ unitFullname }}</div>
        </el-col>
        <el-col :span="6">
            <div>所属分队：{{ unitFullname || '无' }}</div>
        </el-col>
    </el-row>
    <el-row class="title">
        <el-col :span="6">
            <div>列装时间：{{ serviceYear || '无' }}</div>
        </el-col>
        <el-col :span="6">
            <div>服役日期：{{ serviceGap || '无' }}</div>
        </el-col>
        <el-col :span="6">
            <div>出厂日期：{{ manufactureDate || '无' }}</div>
        </el-col>
    </el-row>
    <div class="title">
        <div class="bar fl"></div>
        <div class="fl describe">单装修理记录</div>
        <div style="clear: both;"></div>
    </div>
    <el-row style="margin-top: 15px;" v-for="(record, index) in repairRecords" :key="index" v-loading="loading">
        <el-col :span="6">
            <div class="date">{{ record.repairStartDate }}</div>
        </el-col>
        <el-col :span="18" class="info">
            <el-row class="title">
                <el-col :span="8">
                    <div>承修单位代码：{{ record.akeepUnitCode }}</div>
                </el-col>
                <el-col :span="8">
                    <div>承修单位负责人代码：{{ record.ainchargeStaffCode }}</div>
                </el-col>
                <el-col :span="8">
                    <div>修理内容：{{ record.repairContent }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>填写人代码：{{ record.fillStaffCode }}</div>
                </el-col>
                <el-col :span="8">
                    <div>检查人代码：{{ record.inspectStaffCode }}</div>
                </el-col>
                <el-col :span="8">
                    <div>部件ID：{{ record.partId }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>修理原因：{{ record.repairReason }}</div>
                </el-col>
                <el-col :span="8">
                    <div>修理状态：{{ record.repairStatus }}</div>
                </el-col>
                <el-col :span="8">
                    <div>部件名称：{{ record.partName }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>修理类别：{{ record.repairGradeDictId }}</div>
                </el-col>
                <el-col :span="8">
                    <div>修理开始时间：{{ record.repairStartDate }}</div>
                </el-col>
                <el-col :span="8">
                    <div>修理结束时间：{{ record.repairEndDate }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>累计使用情况：{{ record.usageSituation }}</div>
                </el-col>
                <el-col :span="8">
                    <div>备注：{{ record.remark }}</div>
                </el-col>
                <el-col :span="8">
                    <div>单装修理记录ID：{{ record.repairRecordId }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>单装履历记录ID：{{ record.equipRecordId }}</div>
                </el-col>
            </el-row>
        </el-col>
    </el-row>
    <el-empty v-if="!repairRecords.length && !loading" description="暂无修理记录" />
    <el-pagination class="fr mt mb" v-model:current-page="pageInfo.page" v-model:page-size="pageInfo.pageSize"
        :page-sizes="[3, 6, 9, 12]" layout="sizes, prev, pager, next, jumper" :total="totals"
        @size-change="handleSizeChange" @current-change="handleCurrentChange" background />
</div></template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { TypehistoryApi } from '@/api/fault/typeinfohistory'; // 调整为你的项目路径
import { ElMessage } from 'element-plus';

// 修理记录数据接口
interface RepairRecord {
    repairRecordId: string;
    equipRecordId: string;
    akeepUnitCode: string;
    ainchargeStaffCode: string;
    repairContent: string;
    fillStaffCode: string;
    inspectStaffCode: string;
    partId: string;
    partName: string;
    repairReason: string;
    repairStatus: string;
    repairGradeDictId: string;
    repairStartDate: string;
    repairEndDate: string;
    usageSituation: string;
    remark: string;
}

// 有效的业务活动类型（中文）
const validActivityTypes = [
    '单装使用消耗',
    '单装保养记录',
    '单装故障记录',
    '单装修理记录',
];

// 响应式引用
const equipCode = ref<string>('');
const businessActivity = ref<string>(''); // 存储中文标签，例如“单装修理记录”
const equipSimpleName = ref<string>('');
const unitFullname = ref<string>('');
const subUnitName = ref<string>('');
const serviceYear = ref<string>('');
const serviceGap = ref<string>('');
const manufactureDate = ref<string>('');
const repairRecords = ref<RepairRecord[]>([]);
const pageInfo = reactive({
    page: 1,
    pageSize: 3,
});
const totals = ref<number>(0);
const loading = ref<boolean>(false);

// 路由和标签存储
const router = useRouter();
const route = useRoute();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;

// SessionStorage 工具
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

// 存储 session 数据
const storeSessionData = () => {
    if (equipCode.value) {
        safeSessionStorage.set('repairRecordEquipCode', equipCode.value);
    }
    if (businessActivity.value) {
        safeSessionStorage.set('repairRecordBusinessActivity', businessActivity.value);
    }
};

// 从 API 加载修理记录
const loadData = async () => {
    loading.value = true;
    try {
        const activityType = businessActivity.value || '单装修理记录'; // 回退到“单装修理记录”
        if (!validActivityTypes.includes(activityType)) {
            ElMessage.error('无效的业务活动类型');
            return;
        }
        const params = {
            equipCode: equipCode.value,
            activityType, // 直接使用中文标签
            current: pageInfo.page,
            size: pageInfo.pageSize,
        };
        const response = await TypehistoryApi(params);
        if (response.code === 200) {
            repairRecords.value = response.data.records.map((item: any) => ({
                repairRecordId: item.repairRecordId || '',
                equipRecordId: item.equipRecordId || '',
                akeepUnitCode: item.akeepUnitCode || '',
                ainchargeStaffCode: item.ainchargeStaffCode || '',
                repairContent: item.repairContent || '',
                fillStaffCode: item.fillStaffCode || '',
                inspectStaffCode: item.inspectStaffCode || '',
                partId: item.partId || '无', // 后端未提供
                partName: item.partName || '未知', // 后端未提供
                repairReason: item.repairReason || '无', // 后端未提供
                repairStatus: item.repairStatus || '',
                repairGradeDictId: item.repairGradeDictId || '',
                repairStartDate: item.repairStartDate || '',
                repairEndDate: item.repairEndDate || '',
                usageSituation: item.usageSituation || '无',
                remark: item.remark || '无',
            }));
            totals.value = response.data.totalElements || 0;
            // 从第一条记录设置基础信息
            if (response.data.records.length > 0) {
                const firstRecord = response.data.records[0];
                equipSimpleName.value = firstRecord.equipSimpleName || '';
                unitFullname.value = firstRecord.unitFullname || '';
                subUnitName.value = firstRecord.subUnitName || '';
                serviceYear.value = firstRecord.serviceYear || '';
                serviceGap.value = firstRecord.serviceGap || '';
                manufactureDate.value = firstRecord.manufactureDate || '';
            }
        } else {
            ElMessage.error('获取修理记录失败: ' + response.message);
        }
    } catch (error) {
        ElMessage.error('获取修理记录失败，请稍后重试');
        console.error('获取修理记录错误:', error);
    } finally {
        loading.value = false;
    }
};

// 分页处理
const handleSizeChange = (size: number) => {
    pageInfo.pageSize = size;
    pageInfo.page = 1;
    loadData();
};

const handleCurrentChange = (page: number) => {
    pageInfo.page = page;
    loadData();
};

// 返回到 Typeinformation
const goBack = () => {
    // addTab("业务活动信息", "/deviceinformation/typeinformation", "VideoCamera");
    // setCurrentTab("业务活动信息", "/deviceinformation/typeinformation");
    // router.push('/deviceinformation/typeinformation');
    window.history.back()

};

// 组件挂载时
onMounted(() => {
    // 从路由查询获取 equipCode 和 businessActivity
    const queryEquipCode = route.query.equipCode;
    const queryBusinessActivity = route.query.businessActivity;

    if (queryEquipCode) {
        equipCode.value = String(queryEquipCode);
    } else {
        equipCode.value = safeSessionStorage.get('repairRecordEquipCode');
    }

    if (queryBusinessActivity) {
        businessActivity.value = String(queryBusinessActivity);
    } else {
        businessActivity.value = safeSessionStorage.get('repairRecordBusinessActivity');
    }

    console.log('查询 equipCode:', queryEquipCode);
    console.log('存储 equipCode:', safeSessionStorage.get('repairRecordEquipCode'));
    console.log('初始 equipCode:', equipCode.value);
    console.log('查询 businessActivity:', queryBusinessActivity);
    console.log('存储 businessActivity:', safeSessionStorage.get('repairRecordBusinessActivity'));
    console.log('初始 businessActivity:', businessActivity.value);

    // 存储到 sessionStorage
    storeSessionData();

    // 加载修理记录
    if (equipCode.value && businessActivity.value) {
        loadData();
    } else {
        ElMessage.warning('缺少装备统一编码或业务活动参数');
    }
});
</script>

<style lang="less" scoped>
.bar {
    width: 10px;
    height: 15px;
    background-color: #409eff;
}

.describe {
    color: #409eff;
    font-weight: bold;
    margin-left: 5px;
}

.title {
    line-height: 15px;
    margin-top: 15px;
    margin-bottom: 15px;
    margin-left: 30px;
}

.fl {
    float: left;
}

.date {
    color: purple;
    font-size: 20px;
    margin-top: 50px;
    margin-left: 30px;
}

.info {
    margin-top: 30px;
    border: gray 1px solid;
}

.fr {
    float: right;
}

.mt {
    margin-top: 20px;
}

.mb {
    margin-bottom: 20px;
}
</style>
