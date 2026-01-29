<!-- <template>
    <el-card>
        <el-tabs  v-model="activeTab" @tab-click="handleTabclick">
            <el-tab-pane label="故障预测数据总量" name="total-data">
                <total-data-content ref="totalDataRef"/>
            </el-tab-pane>
        </el-tabs>
    </el-card>
</template> -->

<template>
    <el-row :gutter="24" style="display: flex;">
        <!-- 左侧按钮区域 -->
        <el-col :span="4">
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='total-data' }"
                    @click="activeTab='total-data'">
                    故障预测数据总量
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='time-count' }"
                    @click="activeTab='time-count'">
                    故障发生时间数量
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='predict-trend' }"
                    @click="activeTab='predict-trend'">
                    故障概率发生趋势
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='system-trend' }"
                    @click="activeTab='system-trend'">
                    故障发生分系统趋势
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='part-ratio' }"
                    @click="activeTab='part-ratio'">
                    故障发生部件比例
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='occur-cause' }"
                    @click="activeTab='occur-cause'">
                    故障发生原因
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: activeTab==='occur-trend' }"
                    @click="activeTab='occur-trend'">
                    故障发生趋势
                </el-button>
            </div>
        </el-col>

        <!-- 右侧内容区域 -->
        <el-col :span="20">
            <el-card>
        <el-tabs  v-model="activeTab" @tab-click="handleTabclick">
            <el-tab-pane label="故障预测数据总量" name="total-data">
                <total-data-content v-if="activeTab==='total-data'" ref="totalDataRef"/>
            </el-tab-pane>
            <el-tab-pane label="故障发生时间数量" name="time-count">
                <time-count-content v-if="activeTab==='time-count'" ref="timeCountRef"/>
            </el-tab-pane>
            <el-tab-pane label="故障概率发生趋势" name="predict-trend">
                <predict-trend-content v-if="activeTab==='predict-trend'" ref="predictTrendRef"/>
            </el-tab-pane>
            <el-tab-pane label="故障发生分系统趋势" name="system-trend">
                <system-trend-content v-if="activeTab==='system-trend'" ref="systemTrendRef"/>
            </el-tab-pane>
            <el-tab-pane label="故障发生部件比例" name="part-ratio">
                <part-ratio-content v-if="activeTab==='part-ratio'" ref="partRatioRef"/>
            </el-tab-pane>
            <el-tab-pane label="故障发生原因" name="occur-cause">
                <occur-cause-content v-if="activeTab==='occur-cause'" ref="occurCauseRef"/>
            </el-tab-pane>
            <el-tab-pane label="故障发生趋势" name="occur-trend">
                <occur-trend-content v-if="activeTab==='occur-trend'" ref="occurTrendRef"/>
            </el-tab-pane>
        </el-tabs>
    </el-card>
        </el-col>
    </el-row>
</template>

<script setup lang="ts">
import {ref,onMounted,nextTick} from 'vue';
import TotalDataContent from '@/views/fault/analyse/timeana/TotalDataContent.vue'
import TimeCountContent from '@/views/fault/analyse/timeana/TimeCountContent.vue'
import PredictTrendContent from '@/views/fault/analyse/timeana/PredictTrendContent.vue'
import SystemTrendContent from '@/views/fault/analyse/timeana/SystemTrendContent.vue'
import PartRatioContent from '@/views/fault/analyse/timeana/PartRatioContent.vue'
import OccurCauseContent from '@/views/fault/analyse/timeana/OccurCauseContent.vue'
import OccurTrendContent from '@/views/fault/analyse/timeana/OccurTrendContent.vue'

const activeTab=ref('total-data')
const totalDataRef=ref<InstanceType<typeof TotalDataContent>>()
const handleTabclick=async()=>{
    await nextTick();
    if(activeTab.value==='total-data')totalDataRef.value?.handleQuery();
}
onMounted(async()=>{
    await nextTick()
    totalDataRef.value?.handleQuery()
})
</script>

<style lang="less" scoped>
.el-button {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 10px 20px;
    font-size: 18px;
    text-align: center;
    transition: all 0.3s;
}

/* 高亮样式：仅文字变蓝 */
.el-button.active {
    color: #409EFF;
    /* Element Plus primary blue */
}

/* 内容区域标题样式 */
.content-header {
    margin-bottom: 20px;
    padding: 10px 0;
    border-bottom: 1px solid #e6e6e6;
}

.content-header h2 {
    font-size: 24px;
    color: #303133;
    margin: 0;
}
</style>