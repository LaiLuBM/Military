<template>
    <div>
    <el-row :gutter="20">
        <el-col :span="1">
            <!-- 返回按钮 -->
            <el-button text icon="Back" style="font-size: 30px; padding: 0; margin-left: 15px;" @click="goBack" />
        </el-col>
        <el-col :span="8">
            <h2>单装故障记录详情</h2>
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
            <div>装备型号：{{ equipSimpleName }}</div>
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
        <div class="fl describe">单装故障记录</div>
        <div style="clear: both;"></div>
    </div>
    <el-row style="margin-top: 15px;" v-for="(record, index) in faultRecords" :key="index">
        <el-col :span="6">
            <div class="date">{{ record.faultDate }}</div>
        </el-col>
        <el-col :span="18" class="info">
            <el-row class="title">
                <el-col :span="8">
                    <div>部件编码：{{ record.componentCode }}</div>
                </el-col>
                <el-col :span="8">
                    <div>故障部位：{{ record.faultLocation }}</div>
                </el-col>
                <el-col :span="8">
                    <div>单装故障记录ID: {{ record.recordId }}</div>
                </el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>故障日期：{{ record.faultDate }}</div>
                </el-col>
                <el-col :span="8">
                    <div>故障类型: {{ record.malfType }}</div>
                </el-col>
                <el-col :span="8"></el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div class="info-item">
                        <span class="info-label">是否排除故障</span>
                    </div>
                    <el-radio-group class="radio-group" v-model="record.isFixed" :disabled="true">
                        <el-radio label="是">是</el-radio>
                        <el-radio label="否">否</el-radio>
                    </el-radio-group>
                </el-col>
                <el-col :span="8">
                    <div>填写人代码: {{ record.fillerCode }}</div>
                </el-col>
                <el-col :span="8"></el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>负责人代码: {{ record.responsibleCode }}</div>
                </el-col>
                <el-col :span="8">
                    <div>故障原因: {{ record.faultReason }}</div>
                </el-col>
                <el-col :span="8"></el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>处理结果: {{ record.handleResult }}</div>
                </el-col>
                <el-col :span="8">
                    <div>故障现象: {{ record.faultPhenomenon }}</div>
                </el-col>
                <el-col :span="8"></el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>备注: {{ record.remarks }}</div>
                </el-col>
                <el-col :span="8">
                    <div>单装履历记录ID: {{ record.equiprecordId }}</div>
                </el-col>
                <el-col :span="8"></el-col>
            </el-row>
            <el-row class="title">
                <el-col :span="8">
                    <div>上次数据同步时间：{{ record.synchronizeTime }}</div>
                </el-col>
            </el-row>
        </el-col>
    </el-row>
    <el-empty v-if="!faultRecords.length" description="暂无故障记录" />
    <el-pagination class="fr mt mb" v-model:current-page="pageInfo.page" v-model:page-size="pageInfo.pageSize"
        :page-sizes="[3, 6, 9, 12]" layout="sizes, prev, pager, next, jumper" :total="totals"
        @size-change="handleSizeChange" @current-change="handleCurrentChange" background />
        </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue';
import { useRoute, useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { TypehistoryApi } from '@/api/fault/typeinfohistory'; // 调整为你的项目路径
import { ElMessage } from 'element-plus';

// 故障记录数据接口
interface FaultRecord {
    faultDate: string;
    componentCode: string;
    faultLocation: string;
    recordId: string;
    equiprecordId: string;
    malfType: string;
    isFixed: string;
    fillerCode: string;
    responsibleCode: string;
    faultReason: string;
    handleResult: string;
    faultPhenomenon: string;
    remarks: string;
    synchronizeTime: string;
}
const pageInfo = reactive({
    page: 1,
    pageSize: 3,
});
const totals = ref<number>(0);
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

// 有效的业务活动类型（中文）
const validActivityTypes = [
    '单装使用消耗',
    '单装保养记录',
    '单装故障记录',
    '单装修理记录',
];

// 响应式引用
const equipCode = ref<string>('');
const businessActivity = ref<string>(''); // 存储中文标签，例如“单装故障记录”
const equipSimpleName = ref<string>('');
const unitFullname = ref<string>('');
// const subUnitName = ref<string>('');
const serviceYear = ref<string>('');
const serviceGap = ref<string>('');
const manufactureDate = ref<string>('');

const faultRecords = ref<FaultRecord[]>([]);

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
        safeSessionStorage.set('faultRecordEquipCode', equipCode.value);
    }
    if (businessActivity.value) {
        safeSessionStorage.set('faultRecordBusinessActivity', businessActivity.value);
    }
};

// 从 API 加载故障记录
const loadData = async () => {
    try {
        const activityType = businessActivity.value || '单装故障记录'; // 回退到“单装故障记录”
        if (!validActivityTypes.includes(activityType)) {
            ElMessage.error('无效的业务活动类型');
            return;
        }
        const params = {
            equipCode: equipCode.value,
            activityType, // 直接使用中文标签
            current: 1,
            size: 10,
        };
        const response = await TypehistoryApi(params);
        if (response.code === 200) {
            faultRecords.value = response.data.records.map((item: any) => ({
                faultDate: item.malfDate || '',
                componentCode: item.partId || '',
                faultLocation: item.malfLocation || '',
                recordId: item.malfRecordId || '',
                malfType: item.malfType || '',
                isFixed: item.rectified ? '是' : '否',
                fillerCode: item.fillStaffCode || '',
                responsibleCode: item.inchargeStaffCode || '',
                faultReason: item.reason || '',
                handleResult: item.handleResult || '',
                faultPhenomenon: item.phenomenon || '',
                remarks: '无', // API 未提供备注
                synchronizeTime: item.synchronizeTime, // API 未提供上次同步时间
                equiprecordId: item.equipRecordId
            }));
            totals.value = response.data.totalElements || 0;
            // 从第一条记录设置基础信息
            if (response.data.records.length > 0) {
                const firstRecord = response.data.records[0];
                equipSimpleName.value = firstRecord.equipSimpleName || '';
                unitFullname.value = firstRecord.unitFullname || '';
                serviceYear.value = firstRecord.serviceYear || '';
                serviceGap.value = firstRecord.serviceGap || '';
                manufactureDate.value = firstRecord.manufactureDate || '';
            }
        } else {
            ElMessage.error('获取故障记录失败: ' + response.message);
        }
    } catch (error) {
        ElMessage.error('获取故障记录失败，请稍后重试');
        console.error('获取故障记录错误:', error);
    }
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
        equipCode.value = safeSessionStorage.get('faultRecordEquipCode');
    }

    if (queryBusinessActivity) {
        businessActivity.value = String(queryBusinessActivity);
    } else {
        businessActivity.value = safeSessionStorage.get('faultRecordBusinessActivity');
    }

    console.log('查询 equipCode:', queryEquipCode);
    console.log('存储 equipCode:', safeSessionStorage.get('faultRecordEquipCode'));
    console.log('初始 equipCode:', equipCode.value);
    console.log('查询 businessActivity:', queryBusinessActivity);
    console.log('存储 businessActivity:', safeSessionStorage.get('faultRecordBusinessActivity'));
    console.log('初始 businessActivity:', businessActivity.value);

    // 存储到 sessionStorage
    storeSessionData();

    // 加载故障记录
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

/* Target disabled el-radio-group and its radio buttons */
:deep(.el-radio-group .el-radio.is-disabled) {

    /* Style for the radio button itself */
    .el-radio__input .el-radio__inner {

        /* Blue border */
        background-color: #E6F7FF !important;
        /* Light blue background */
    }

    /* Style for the checked radio button */
    &.is-checked .el-radio__inner {
        background-color: #409EFF !important;
        /* Blue fill for checked */

    }

    /* Style for the label text */
    .el-radio__label {
        color: black !important;
        /* Blue text */
    }
}
</style>