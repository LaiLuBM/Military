<template>
    <el-row :gutter="24" style="display: flex;">
        <!-- 左侧按钮区域 -->
        <el-col :span="4">
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/total-data') }"
                    @click="$router.push('/maintainanalyse/total-data')">
                    故障预测数据总量
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/time-count') }"
                    @click="$router.push('/maintainanalyse/time-count')">
                    故障发生时间数量
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/predict-trend') }"
                    @click="$router.push('/maintainanalyse/predict-trend')">
                    故障概率发生趋势
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/system-trend') }"
                    @click="$router.push('/maintainanalyse/system-trend')">
                    故障发生分系统趋势
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/part-ratio') }"
                    @click="$router.push('/maintainanalyse/part-ratio')">
                    故障发生部件比例
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/occur-cause') }"
                    @click="$router.push('/maintainanalyse/occur-cause')">
                    故障发生原因
                </el-button>
            </div>
            <div>
                <el-button style="width: 100%; height: 105px; font-size: 18px; margin-bottom: 5px;"
                    :class="{ active: isActive('/maintainanalyse/occur-trend') }"
                    @click="$router.push('/maintainanalyse/occur-trend')">
                    故障发生趋势
                </el-button>
            </div>
        </el-col>

        <!-- 右侧内容区域 -->
        <el-col :span="20">
            <keep-alive>
                <router-view />
            </keep-alive>
        </el-col>
    </el-row>
</template>

<script lang="ts" setup>
import { useRoute, useRouter } from 'vue-router';
import { onMounted } from 'vue';

// 获取当前路由和路由器
const route = useRoute();
const router = useRouter();

// 路由到标题的映射
const routeTitles: Record<string, string> = {
    '/maintainanalyse/total-data': '故障预测数据总量',
    '/maintainanalyse/time-count': '故障发生时间数量',
    '/maintainanalyse/predict-trend': '故障概率发生趋势',
    '/maintainanalyse/system-trend': '故障发生分系统趋势',
    '/maintainanalyse/part-ratio': '故障发生部件比例',
    '/maintainanalyse/occur-cause': '故障发生原因',
    '/maintainanalyse/occur-trend': '故障发生趋势',
};

// 判断按钮是否为当前路由
const isActive = (path: string) => {
    return route.path === path;
};

// 默认选中第一个按钮
onMounted(() => {
    if (!routeTitles[route.path]) {
        router.push('/maintainanalyse/total-data');
    }
});
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