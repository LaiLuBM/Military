<template><div>
    <el-row :gutter="20">
        <el-col :span="1">
            <!-- 返回按钮 -->
            <el-button text icon="Back" style="font-size: 30px; padding: 0; margin-left: 15px;" @click="goBack" />
        </el-col>
        <el-col :span="8">
            <h2>单装使用消耗历史详情</h2>
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
        <div class="fl describe">单装使用消耗</div>
        <div style="clear: both;"></div>
    </div>
    <el-row style="margin-top: 15px;" v-for="(record, index) in usageRecords" :key="index">
        <el-col :span="6">
            <div class="date">{{ record.useStartDate }}</div>
        </el-col>
        <el-col :span="18" class="info">
            <el-row class="title">
                <el-col :span="8">
                    <div>动用开始时间：{{ record.useStartDate }}</div>
                </el-col>
                <el-col :span="8">
                    <div>动用结束时间：{{ record.useEndDate }}</div>
                </el-col>
                <el-col :span="8">
                    <div>任务性质：{{ record.taskNature }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>使用地点：{{ record.usePlace }}</div>
                </el-col>
                <el-col :span="8">
                    <div>剩余里程：{{ record.residueMile }}公里</div>
                </el-col>
                <el-col :span="8">
                    <div>大修间隔期内累计行驶里程：{{ record.workMile }}公里</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>发动机中修间隔期内发动机累计消耗摩托小时：{{ record.engineMotorHour }}</div>
                </el-col>
                <el-col :span="8">
                    <div>底盘中修间隔期内发动机累计消耗摩托小时：{{ record.chassisMotorHour }}</div>
                </el-col>
                <el-col :span="8">
                    <div>加电时间：{{ record.elecHour }}小时</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>架设次数：{{ record.erectTimes }}次</div>
                </el-col>
                <el-col :span="8">
                    <div>发射次数：{{ record.fireTimes }}次</div>
                </el-col>
                <el-col :span="8">
                    <div>发射实弹：{{ record.fireAmt }}次</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>其他：{{ record.elseDesc }}</div>
                </el-col>
                <el-col :span="8">
                    <div>单装使用记录ID：{{ record.usageRecordId }}</div>
                </el-col>
                <el-col :span="8">
                    <div>单装履历记录ID：{{ record.equipRecordId }}</div>
                </el-col>
            </el-row>
        </el-col>
    </el-row>
    <el-empty v-if="!usageRecords.length" description="暂无使用消耗记录" />
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

// 使用消耗记录数据接口
interface UsageRecord {
    useStartDate: string;
    useEndDate: string;
    taskNature: string;
    usePlace: string;
    residueMile: string;
    workMile: string;
    engineMotorHour: string;
    chassisMotorHour: string;
    elecHour: string;
    erectTimes: string;
    fireTimes: string;
    fireAmt: string;
    elseDesc: string;
    usageRecordId: string;
    equipRecordId: string;
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
const businessActivity = ref<string>(''); // 存储中文标签，例如“单装使用消耗”
const equipSimpleName = ref<string>('');
const unitFullname = ref<string>('');
const subUnitName = ref<string>('');
const serviceYear = ref<string>('');
const serviceGap = ref<string>('');
const manufactureDate = ref<string>('');
const usageRecords = ref<UsageRecord[]>([]);
const pageInfo = reactive({
    page: 1,
    pageSize: 3,
});
const totals = ref<number>(0);

// 路由和标签存储
const router = useRouter();
const route = useRoute();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;
// 返回到 Typeinformation
const goBack = () => {
    // addTab("业务活动信息", "/deviceinformation/typeinformation", "VideoCamera");
    // setCurrentTab("业务活动信息", "/deviceinformation/typeinformation");
    // router.push('/deviceinformation/typeinformation');
    window.history.back()

};
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
        safeSessionStorage.set('usageConsumptionEquipCode', equipCode.value);
    }
    if (businessActivity.value) {
        safeSessionStorage.set('usageConsumptionBusinessActivity', businessActivity.value);
    }
};

// 从 API 加载使用消耗记录
const loadData = async () => {
    try {
        const activityType = businessActivity.value || '单装使用消耗'; // 回退到“单装使用消耗”
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
            usageRecords.value = response.data.records.map((item: any) => ({
                useStartDate: item.useStartDate || '',
                useEndDate: item.useEndDate || '',
                taskNature: item.taskCharcterDictId || '', // 假设为ID，实际需映射为名称
                usePlace: item.usePlace || '',
                residueMile: item.residueMile ? `${item.residueMile} ` : '无',
                workMile: item.workMile ? `${item.workMile} ` : '无',
                engineMotorHour: item.engineMotorHour ? `${item.engineMotorHour} ` : '无',
                chassisMotorHour: item.chasisMotorHour ? `${item.chasisMotorHour} ` : '无', // 注意后端拼写为 chasis
                elecHour: item.elecHour ? `${item.elecHour} ` : '无',
                erectTimes: item.erectTimes || '0',
                fireTimes: item.fireTimes || '0',
                fireAmt: item.fireAmt || '0',
                elseDesc: item.elseDesc || '无',
                usageRecordId: item.usageRecordId || '',
                equipRecordId: item.equipRecordId || '',
            }));
            totals.value = response.data.total || 0;
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
            ElMessage.error('获取使用消耗记录失败: ' + response.message);
        }
    } catch (error) {
        ElMessage.error('获取使用消耗记录失败，请稍后重试');
        console.error('获取使用消耗记录错误:', error);
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



// 组件挂载时
onMounted(() => {
    // 从路由查询获取 equipCode 和 businessActivity
    const queryEquipCode = route.query.equipCode;
    const queryBusinessActivity = route.query.businessActivity;

    if (queryEquipCode) {
        equipCode.value = String(queryEquipCode);
    } else {
        equipCode.value = safeSessionStorage.get('usageConsumptionEquipCode');
    }

    if (queryBusinessActivity) {
        businessActivity.value = String(queryBusinessActivity);
    } else {
        businessActivity.value = safeSessionStorage.get('usageConsumptionBusinessActivity');
    }

    console.log('查询 equipCode:', queryEquipCode);
    console.log('存储 equipCode:', safeSessionStorage.get('usageConsumptionEquipCode'));
    console.log('初始 equipCode:', equipCode.value);
    console.log('查询 businessActivity:', queryBusinessActivity);
    console.log('存储 businessActivity:', safeSessionStorage.get('usageConsumptionBusinessActivity'));
    console.log('初始 businessActivity:', businessActivity.value);

    // 存储到 sessionStorage
    storeSessionData();

    // 加载使用消耗记录
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