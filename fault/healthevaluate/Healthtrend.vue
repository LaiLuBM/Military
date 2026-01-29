<template>
	<div>
		<el-row :gutter="20">
			<el-col :span="24">
				<el-card>
					<el-row :gutter="20" align="middle">
						<el-col :span="6">
							<el-form>
								<el-form-item label="单位/分队" prop="unitPath">
								                <el-tree-select v-model="selectedUnit" :data="treeData" :props="treeProps" node-key="unitCode"
								                  placeholder="请选择单位（可搜索）" filterable clearable style="width: 100%" check-strictly
								                  @change="handleUnitSelect" />
								              </el-form-item>
							</el-form>
						</el-col>
						<el-col :span="6">
							<el-form>
								<el-form-item label="装备型号">
									<el-select
										clearable
										placeholder="装备型号"
										v-model="searchParams.equipModelCode"
										@visible-change="(val) => val && fetchEquipmentModels()"
									>
										<el-option-group
											v-for="group in equipmentTypeOptions"
											:key="group.label"
											:label="group.label"
										>
											<el-option
												v-for="option in group.options"
												:key="option.value"
												:label="option.label"
												:value="option.value"
											></el-option>
										</el-option-group>
									</el-select>
								</el-form-item>
							</el-form>
						</el-col>
						<el-col :span="12">
							<el-form>
								<el-form-item>
									<div class="flex-container">
										<el-button type="primary" class="ml" @click="loadData"
											>统计</el-button
										>
									</div>
								</el-form-item>
							</el-form>
						</el-col>
					</el-row>
				</el-card>
			</el-col>
		</el-row>
		<el-row style="margin-top: 10px">
			<el-col :span="24">
				<el-card>
					<el-row>
						<el-col :span="8">
							<p style="font-size: 20px; font-weight: bold">总体统计结果</p>
						</el-col>
						<el-col :span="16"></el-col>
					</el-row>
					<el-row style="margin-top: 10px">
						<el-col :span="10">
							<div
								style="
									height: 80px;
									background-color: rgb(245, 247, 149);
									border-radius: 20px;
									width: 250px;
									padding-top: 20px;
								"
							>
								<div style="font-size: 18px; text-align: center">
									装备型号数:
								</div>
								<div style="font-size: 18px; text-align: center">
									<span
										style="font-size: 22px; font-weight: bold; color: orange"
										>{{ equipModelCount }}</span
									>种
								</div>
							</div>
						</el-col>
						<el-col :span="10">
							<div
								style="
									height: 80px;
									background-color: rgb(157, 247, 149);
									border-radius: 20px;
									width: 250px;
									padding-top: 20px;
								"
							>
								<div style="font-size: 18px; text-align: center">总装备数:</div>
								<div style="font-size: 18px; text-align: center">
									<span
										style="font-size: 22px; font-weight: bold; color: green"
										>{{ totalEquipCount }}</span
									>台
								</div>
							</div>
						</el-col>
						<el-col :span="4">
							<div
								style="
									height: 80px;
									background-color: rgb(156, 149, 247);
									border-radius: 20px;
									width: 250px;
									padding-top: 20px;
								"
							>
								<div style="font-size: 18px; text-align: center">
									精细化装备数:
								</div>
								<div style="font-size: 18px; text-align: center">
									<span
										style="font-size: 22px; font-weight: bold; color: blue"
										>{{ fineEquipCount }}</span
									>台
								</div>
							</div>
						</el-col>
					</el-row>
				</el-card>
			</el-col>
		</el-row>
		<el-row :gutter="2" style="margin-top: 5px">
			<el-col :span="12">
				<el-card style="height: 550px">
					<el-row>
						<el-button style="font-size: 24px; font-weight: bold" @click="" text
							>通用化健康评估</el-button
						>
						<p style="margin-left: 350px">
							最近评估时间: {{ lastGeneralAssessment }}
						</p>
					</el-row>
					<div style="margin-top: 20px; width: 100%">
						<el-radio-group
							v-model="radio1"
							size="large"
							style="width: 100%"
							@change="handleRadio1Change"
						>
							<el-radio-button
								label="各个健康等级的装备数量"
								value="general_total"
							/>
							<el-radio-button
								label="近6个月平均健康指数变化"
								value="general_six"
							/>
						</el-radio-group>
						<!-- 第一个图表容器 -->
						<div
							ref="chart1"
							style="width: 100%; height: 350px; margin-top: 20px"
						></div>
					</div>
				</el-card>
			</el-col>
			<el-col :span="12">
				<el-card style="height: 550px">
					<el-row>
						<el-button style="font-size: 24px; font-weight: bold" text
							>精细化健康评估</el-button
						>
						<p style="margin-left: 350px">
							最近评估时间: {{ lastFineAssessment }}
						</p>
					</el-row>
					<div style="margin-top: 20px; width: 100%" class="three">
						<el-radio-group
							v-model="radio2"
							size="large"
							style="width: 100%"
							@change="handleRadio2Change"
						>
							<el-radio-button
								label="各个健康等级的装备数量"
								value="fining_total"
							/>
							<el-radio-button label="各部件健康指数" value="part" />
							<el-radio-button
								label="近6个月平均健康指数变化"
								value="fining_six"
							/>
						</el-radio-group>
						<!-- 第二个图表容器 -->
						<div
							ref="chart2"
							style="width: 100%; height: 350px; margin-top: 20px"
						></div>
					</div>
				</el-card>
			</el-col>
			<el-col :span="12">
				<el-card>
					<el-row>
						<el-button style="font-size: 24px; font-weight: bold" @click="" text
							>关联分析</el-button
						>
						<p style="margin-left: 350px">
							最近评估时间: {{ lastFineAssessment }}
						</p>
					</el-row>
					<el-row style="margin-top: 100px">
						<el-col :span="20">结论：</el-col>
						<el-col :span="2">
							<el-button
								text
								type="primary"
								style="font-size: 18px"
								@click="goAna"
								>去分析>></el-button
							>
						</el-col>
						<el-col :span="24">
							<div style="height: 238px">
								<span>{{ description || "暂无统计结果" }}</span>
							</div>
						</el-col>
					</el-row>
				</el-card>
			</el-col>
			<el-col :span="12">
				<el-card>
					<el-row>
						<el-button style="font-size: 24px; font-weight: bold" @click="" text
							>战斗力评估</el-button
						>
						<p style="margin-left: 350px">
							最近评估时间: {{ lastFineAssessment }}
						</p>
					</el-row>
					<div style="margin-top: 20px; width: 100%">
						<div
							ref="chart3"
							style="width: 100%; height: 350px; margin-top: 20px"
						></div>
					</div>
				</el-card>
			</el-col>
		</el-row>
	</div>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref, watch } from 'vue';
import { useChart } from '@/hooks/useChart';
// import { HealthtrendApi } from '@/api/healthTrend';
import { equipmodelApi } from '@/api/fault/equipmodel';
import { ElMessage, type ElTree } from 'element-plus';
import { unitListApi } from '@/api/fault/unitlist';
import { useRouter } from 'vue-router';
import { useHttp } from '@/hooks/useHttp';
// import { get } from '@/utils/bgd-http';
import { HealthtrendApi } from '@/api/fault/healthTrend';

// Radio bindings
const radio1 = ref('general_total');
const radio2 = ref('fining_total');


const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref<Tree[]>([]);
const defaultProps = {
    children: 'childrenUnits',
    label: 'unitSimpleName',
};

// Statistic bindings
const equipModelCount = ref();
const totalEquipCount = ref();
const fineEquipCount = ref();
const lastGeneralAssessment = ref('2025-10-15');
const lastFineAssessment = ref('2025-10-15');
const selectedUnit = ref(''); // 默认选中“中部战区01军01旅”

// Chart references
const chart1 = ref<HTMLElement | null>(null);
const chart2 = ref<HTMLElement | null>(null);
const chart3 = ref<HTMLElement | null>(null);

interface SearchType {
    equipModelCode: string;
    unitPath: string;

}
// 树形数据
interface Tree {
    unitCode: number;
    parentUnitCode: any;
    unitSimpleName: string;
    unitPath: number;
    childrenUnits?: Tree[];
}

interface ResponseData {
    code: number;
    message: string;
    data: {
        analysisPage: {
            totalElements: number;
        };
        description: string;
    };
}

const searchParams = ref<SearchType>({
    equipModelCode: '',
    unitPath: ''
})
const description = ref<string>('');
const relate_searchParams = ref({
    equipModelCode: '',
    maxHealthIndex: 99,
    minHealthIndex: 0,
    page: 1,
    pageSize: 10
});

const router=useRouter()

const goAna=()=>{
    router.push("/healthevaluate/relatehealth")
}

// 使用计算属性同步 equipModelCode
const equipModelCode = computed({
    get: () => relate_searchParams.value.equipModelCode,
    set: (value) => {
        relate_searchParams.value.equipModelCode = value;
        searchParams.value.equipModelCode = value;
    }
});
const pageInfo = reactive({
    page: 1,
    pageSize: 10,
});

const { loading } = useHttp<ResponseData>('/relate-analysis', searchParams);
// 装备型号下拉框选项（分组格式）
const equipmentTypeOptions = ref<
    { label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);
// Hardcoded chart data
const chartData = reactive({
    generalHealthLevelChart: [
        { level: '暂无数据', count: '暂无数据' },
        
    ],
    generalHealthTrendChart: [
        // { month: '2025-05', index: '85' },
        // { month: '2025-06', index: '88' },
        // { month: '2025-07', index: '90' },
        // { month: '2025-08', index: '87' },
        // { month: '2025-09', index: '89' },
        // { month: '2025-10', index: '92' },
         { month: '暂无数据', index: '暂无数据' },
    ],
    finingHealthLevelChart: [
        // { level: '优', count: '25' },
        // { level: '良', count: '15' },
        // { level: '中', count: '8' },
        // { level: '差', count: '2' },
         { level: '暂无数据', count: '暂无数据' },
    ],
    finingHealthTrendChart: [
        // { month: '2025-05', index: '80' },
        // { month: '2025-06', index: '82' },
        // { month: '2025-07', index: '85' },
        // { month: '2025-08', index: '83' },
        // { month: '2025-09', index: '86' },
        // { month: '2025-10', index: '88' },
         { month: '暂无数据', index: '暂无数据' },
    ],
    partHealthChart: [
        // { part: '引擎', index: '90' },
        // { part: '底盘', index: '85' },
        // { part: '电路', index: '80' },
        // { part: '传感器', index: '88' },
        // { part: '控制器', index: '87' },
        // { part: '显示器', index: '86' },
         { part: '暂无数据', index: '暂无数据' },
    ],
});

//单位分队点击事件
// const _deprecated_handleNodeClick = async (nodeData: Tree) => {
//     selectedUnit.value = nodeData.unitSimpleName;
//     equipModelCode.value = ''; // 清空（通过计算属性同步清空两个参数）
//     searchParams.value.unitPath = nodeData.unitPath.toString();
//     try {
//         const response = await equipmodelApi(nodeData.unitPath.toString());
//         if (response.code === 200) {
//             const groupedOptions: { label: string; options: { label: string; value: string }[] }[] = [];
//             for (const category in response.data.categories) {
//                 if (Array.isArray(response.data.categories[category])) {
//                     const categoryOptions = response.data.categories[category].map((item: any) => ({
//                         label: item.equipSimpleName,
//                         value: item.equipModelCode,
//                     }));
//                     groupedOptions.push({
//                         label: category,
//                         options: categoryOptions,
//                     });
//                 }
//             }
//             equipmentTypeOptions.value = groupedOptions;
//         } else {
//             console.error('请求失败:', response.message);
//             equipmentTypeOptions.value = [];
//         }
//     } catch (error) {
//         console.error('请求装备型号数据失败:', error);
//         equipmentTypeOptions.value = [];
//     }
// };
const _deprecated_handleNodeClick = async (nodeData: Tree) => {
//   selectedUnit.value = nodeData.unitSimpleName;
//   // 重置相关参数
//   searchParams.value.equipModelCode = '';
//   searchParams.value.equipCode = '';
//   const unitPath = nodeData.unitPath.toString();
//   searchParams.value.unitPath = unitPath;
      selectedUnit.value = nodeData.unitSimpleName;
    equipModelCode.value = ''; // 清空（通过计算属性同步清空两个参数）
    searchParams.value.unitPath = nodeData.unitPath.toString();
  
};

const fetchEquipmentModels = async () => {
  const unitPath = searchParams.value.unitPath;
  if (!unitPath) {
    ElMessage.warning('请先选择单位/分队');
    return;
  }

  // 如果已经有数据了，可以选择不再重复查询（根据业务需求决定）
  if (equipmentTypeOptions.value.length > 0) return;

  try {
    const response = await equipmodelApi(unitPath);
    if (response.code === 200 && response.data?.categories) {
      const groupedOptions: any[] = [];
      for (const category in response.data.categories) {
        if (Array.isArray(response.data.categories[category])) {
          const categoryOptions = response.data.categories[category].map((item: any) => ({
            label: item.equipSimpleName || '',
            value: item.equipModelCode || '',
          }));
          groupedOptions.push({
            label: category || '',
            options: categoryOptions,
          });
        }
      }
      equipmentTypeOptions.value = groupedOptions;
    } else {
      equipmentTypeOptions.value = [];
    }
  } catch (error) {
    console.error('获取型号失败:', error);
    equipmentTypeOptions.value = [];
  }
};
watch(selectedUnit, (newVal) => {
  searchParams.value.unitPath = newVal ? searchParams.value.unitPath : '';
  searchParams.value.equipModelCode = '';
//   searchParams.value.equipCode = '';
  // 清空选项列表，强制下次展开时重新触发 fetchEquipmentModels
  equipmentTypeOptions.value = [];
  equipmentCodeOptions.value = [];
});
// Refresh charts (placeholder for static data)
const loadData = async () => {
    if (!selectedUnit.value && !equipModelCode.value) {
        ElMessage.warning('请选择单位/分队和设备型号后再进行统计');
        return;
    }
    if (selectedUnit.value && !equipModelCode.value) {
        ElMessage.warning('请选择设备型号后再进行统计');
        return;
    }
    try {
        loading.value = true;
        relate_searchParams.value.page = pageInfo.page;
        relate_searchParams.value.pageSize = pageInfo.pageSize;
        const relate_params = { ...relate_searchParams.value };
        const trend_params = { ...searchParams.value };

        const [response, response_relate] = await Promise.all([
            HealthtrendApi(trend_params),
            // get('/relate-analysis', relate_params)
        ]);

        if (response.code === 200) {
            // 更新统计数据
            equipModelCount.value = response.data.equipModelCount ?? 0;
            totalEquipCount.value = response.data.totalEquipCount ?? 0;
            fineEquipCount.value = response.data.detailedEquipCount ?? 0;
            lastGeneralAssessment.value = response.data.generalHealthChart?.mostRecentTime ?? '未知';
            lastFineAssessment.value = response.data.detailedHealthChart?.mostRecentTime ?? '未知';
            description.value = response_relate?.data.description || '';

            // 更新 generalHealthLevelChart 数据
            if (response.data.generalHealthChart?.equipHealthGrade && response.data.generalHealthChart?.equipHealthGradeCount) {
                const generalGrades = response.data.generalHealthChart.equipHealthGrade.split(', ').map((item: string) => item.trim());
                const generalCounts = response.data.generalHealthChart.equipHealthGradeCount.split(', ').map((item: string) => parseInt(item.trim()));
                 if(response.data.generalHealthChart.averageHealthScoreInSixMonths&&response.data.generalHealthChart.timeArray){
                    const timeArray=response.data.generalHealthChart.timeArray.split(', ').map((item: string) => item.trim());
                    const averageScores=response.data.generalHealthChart.averageHealthScoreInSixMonths.split(', ').map((item: string) => parseFloat(item.trim()));
                    chartData.generalHealthTrendChart=timeArray.map((month:string,index:number)=>({
                        month,
                        index:averageScores[index].toString()
                    }));
                }
                chartData.generalHealthLevelChart = generalGrades.map((level: any, index: any) => ({
                    level,
                    count: generalCounts[index].toString()
                }));
            } else {
                console.warn('generalHealthChart 数据不完整，设置为默认值');
                chartData.generalHealthLevelChart = [
                    { level: '优秀', count: '0' },
                    { level: '良好', count: '0' },
                    { level: '一般', count: '0' },
                    { level: '较差', count: '0' }
                ];
                ElMessage.warning('通用化健康数据不可用，使用默认值');
            }

            // 更新 finingHealthLevelChart 数据
            if (response.data.detailedHealthChart?.equipHealthGrade && response.data.detailedHealthChart?.equipHealthGradeCount) {
                const finingGrades = response.data.detailedHealthChart.equipHealthGrade.split(', ').map((item: string) => item.trim());
                const finingCounts = response.data.detailedHealthChart.equipHealthGradeCount.split(', ').map((item: string) => parseInt(item.trim()));
                chartData.finingHealthLevelChart = finingGrades.map((level: any, index: any) => ({
                    level,
                    count: finingCounts[index].toString()
                }));
            } else {
                console.warn('detailedHealthChart 数据不完整，设置为默认值');
                chartData.finingHealthLevelChart = [
                    { level: '优秀', count: '0' },
                    { level: '良好', count: '0' },
                    { level: '一般', count: '0' },
                    { level: '较差', count: '0' }
                ];
                ElMessage.warning('精细化健康数据不可用，使用默认值');
            }


            // 刷新 chart1 和 chart2
            await Promise.all([
                update1(getChartData1(radio1.value), true),
                update2(getChartData2(radio2.value), true)
            ]);
        } else {
            ElMessage.error('查询失败: ' + (response.message || response_relate.message));
        }
    } catch (error) {
        console.error('请求失败:', error);
        ElMessage.error('数据加载失败');
    } finally {
        loading.value = false;
    }
};

// Chart 1: 通用化健康评估 - 各个健康等级的装备数量 & 近6个月平均健康指数变化
const getChartData1 = (value: string) => {
    if (value === 'general_total' && chartData.generalHealthLevelChart?.length) {
        const levels = chartData.generalHealthLevelChart.map((item) => item.level);
        const counts = chartData.generalHealthLevelChart.map((item) => parseInt(item.count));
        return {
            tooltip: {
                trigger: 'axis',
                axisPointer: { type: 'shadow' }
            },
            grid: {
                left: '3%',
                right: '20%',
                containLabel: true
            },
            xAxis: {
                type: 'value',
                name: '装备数量',
                nameTextStyle: { fontSize: 14, padding: [0, 0, 10, 0] },
                min: 0,
                interval: 1, // 刻度间隔为 1
                axisLabel: {
                    formatter: '{value}' // 确保显示整数
                }
            },
            yAxis: {
                type: 'category',
                name: '健康等级',
                data: levels,
            },
            series: [
                {
                    name: '装备数量',
                    type: 'bar',
                    data: counts,
                    itemStyle: { color: '#2f7ed8' },
                    barWidth: '40%',
                    label: {
                        show: true,
                        position: 'right',
                        formatter: '{c}',
                        fontSize: 12
                    }
                }
            ]
        };
    } else if (value === 'general_six' && chartData.generalHealthTrendChart?.length) {
        const months = chartData.generalHealthTrendChart.map((item) => item.month);
        const indices = chartData.generalHealthTrendChart.map((item) => parseFloat(item.index));
        return {
            tooltip: {
                trigger: 'axis'
            },
            grid: {
                left: '3%',
                right: '8%',
                bottom: '3%',
                containLabel: true
            },
            xAxis: {
                type: 'category',
                boundaryGap: false,
                name: '月份',
                data: months,
            },
            yAxis: {
                type: 'value',
                name: '健康指数',
                min: 0,
                max: 100,
                interval: 20,
            },
            series: [
                {
                    name: '健康指数',
                    type: 'line',
                    smooth: true,
                    data: indices,
                    lineStyle: { color: '#2f7ed8' },
                    areaStyle: { color: '#2f7ed8', opacity: 0.12 },
                    symbol: 'circle',
                    symbolSize: 8
                }
            ]
        };
    }
    // Fallback if data is not available
    return {
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: [] },
        yAxis: { type: 'value' },
        series: []
    };
};

// Chart 2: 精细化健康评估 - 各个健康等级的装备数量, 各部件健康指数, 近6个月平均健康指数变化
const getChartData2 = (value: string) => {
    if (value === 'fining_total' && chartData.finingHealthLevelChart?.length) {
        const levels = chartData.finingHealthLevelChart.map((item) => item.level);
        const counts = chartData.finingHealthLevelChart.map((item) => parseInt(item.count));
        return {
            tooltip: {
                trigger: 'axis',
                axisPointer: { type: 'shadow' }
            },
            grid: {
                left: '3%',
                right: '20%',
                containLabel: true
            },
            xAxis: {
                type: 'value',
                name: '装备数量',
                nameTextStyle: { fontSize: 14, padding: [0, 0, 10, 0] },
                min: 0,
                interval: 1, // 刻度间隔为 1
                axisLabel: {
                    formatter: '{value}' // 确保显示整数
                }
            },
            yAxis: {
                type: 'category',
                name: '健康等级',
                data: levels,
            },
            series: [
                {
                    name: '装备数量',
                    type: 'bar',
                    data: counts,
                    itemStyle: { color: '#00c4b4' },
                    barWidth: '40%',
                    label: {
                        show: true,
                        position: 'right',
                        formatter: '{c}',
                        fontSize: 12
                    }
                }
            ]
        };
    } else if (value === 'fining_six' && chartData.finingHealthTrendChart?.length) {
        const months = chartData.finingHealthTrendChart.map((item) => item.month);
        const indices = chartData.finingHealthTrendChart.map((item) => parseFloat(item.index));
        return {
            tooltip: {
                trigger: 'axis'
            },
            grid: {
                left: '3%',
                right: '8%',
                bottom: '3%',
                containLabel: true
            },
            xAxis: {
                type: 'category',
                boundaryGap: false,
                name: '月份',
                data: months,
            },
            yAxis: {
                type: 'value',
                name: '健康指数',
                min: 0,
                max: 100,
                interval: 20,
            },
            series: [
                {
                    name: '健康指数',
                    type: 'line',
                    smooth: true,
                    data: indices,
                    lineStyle: { color: '#00c4b4' },
                    areaStyle: { color: '#00c4b4', opacity: 0.12 },
                    symbol: 'circle',
                    symbolSize: 8
                }
            ]
        };
    } else if (value === 'part' && chartData.partHealthChart?.length) {
        const parts = chartData.partHealthChart.map((item) => item.part);
        const indices = chartData.partHealthChart.map((item) => parseFloat(item.index));
        return {
            tooltip: {
                trigger: 'axis',
                axisPointer: { type: 'shadow' }
            },
            grid: {
                left: '3%',
                right: '8%',
                bottom: '3%',
                containLabel: true
            },
            xAxis: {
                type: 'category',
                name: '部件',
                data: parts,
                
            },
            yAxis: {
                type: 'value',
                name: '健康指数',
                min: 0,
                max: 100,
                interval: 20,
            },
            series: [
                {
                    name: '健康指数',
                    type: 'bar',
                    data: indices,
                    itemStyle: { color: '#ff851b' },
                    barWidth: '40%',
                    label: {
                        show: true,
                        position: 'top',
                        formatter: '{c}',
                        fontSize: 12
                    }
                }
            ]
        };
    }
    // Fallback if data is not available
    return {
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: [] },
        yAxis: { type: 'value' },
        series: []
    };
};


const setChartData3 = async () => {
    const chartOptions = reactive({
        tooltip: {
            trigger: 'axis'
        },
        legend: {
            data: ['指挥控制', '地面突击', '火力打击', '综合保障']
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        toolbox: {
            // feature: {
            //     saveAsImage: {}
            // }
        },
        xAxis: {
            type: 'category',
            boundaryGap: false,
            data: ['1.12', '1.18', '1.26', '2.3', '2.7', '3.6', '3.15']
        },
        yAxis: {
            type: 'value'
        },
        series: [
            {
                name: '指挥控制',
                type: 'line',
                data: [120, 132, 101, 134, 90, 230, 210],
                areaStyle: {}
            },
            {
                name: '地面突击',
                type: 'line',
                data: [220, 182, 191, 234, 290, 330, 310],
                areaStyle: {}
            },
            {
                name: '火力打击',
                type: 'line',
                data: [150, 232, 201, 154, 190, 330, 410],
                areaStyle: {}
            },
            {
                name: '综合保障',
                type: 'line',
                data: [320, 332, 301, 334, 390, 330, 320],
                areaStyle: {}
            },
        ]
    })
    return chartOptions;
}


// Initialize and update charts
const { update: update1 } = useChart(chart1, async () => getChartData1(radio1.value));
const { update: update2 } = useChart(chart2, async () => getChartData2(radio2.value));
useChart(chart3, setChartData3)

// Handle radio1 change
const handleRadio1Change = async (value: string) => {
    const options = getChartData1(value);
    await update1(options, true);
};

// Handle radio2 change
const handleRadio2Change = async (value: string) => {
    const options = getChartData2(value);
    await update2(options, true);
};

// Mounted hook
onMounted(async () => {
    try {
        const { data } = await unitListApi();
        treeData.value = data;

        // 查找“中部战区01军01旅”节点


    } catch (error) {
        console.error('Failed to fetch unit list:', error);
        ElMessage.error('加载单位数据失败');
    }
    // Initialize charts with static data
    await update1(getChartData1(radio1.value), true);
    await update2(getChartData2(radio2.value), true);
});
watch(
    () => relate_searchParams.value.equipModelCode,
    (newValue) => {
        searchParams.value.equipModelCode = newValue;
    }
);


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
.el-radio-button {
	width: 48%;

	:deep(.el-radio-button__inner) {
		width: 100%;
		padding: 12px 10px;
	}
}

.el-radio-button__inner {
	background-color: #ffffff;
	color: #606266;
}

.el-radio-button__orig-radio:checked + .el-radio-button__inner {
	background-color: #409eff !important;
	color: #ffffff !important;
	box-shadow: -1px 0 0 0 #409eff;
}

.el-radio-button__inner:hover {
	color: #409eff;
}

.three .el-radio-button {
	width: 33%;
}

.sub-tree {
	::v-deep .el-tree-node__content:hover {
		background-color: #e6f7ff !important;
	}

	::v-deep .el-tree-node.is-current > .el-tree-node__content {
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