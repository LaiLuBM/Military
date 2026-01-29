<template>
	<div>
		<el-row :gutter="20">
			<el-col :span="24">
				<el-card>
					<el-row style="margin-top: 15px"> </el-row>
					<el-row :gutter="20" align="middle">
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
						<el-col :span="4">
							<el-form>
								<el-form-item label="装备统一编码">
									<el-input
										placeholder="装备统一编码"
										v-model="searchParams.equipCode"
										@keyup.enter="handleLoadData"
									>
									</el-input>
								</el-form-item>
							</el-form>
						</el-col>
						<el-col :span="12">
							<el-form>
								<el-form-item>
									<div class="flex-container">
										<el-switch
											v-model="isInputEnabled"
											@change="handleSwitchChange"
											class="switch-style"
										></el-switch>
										<el-input
											v-model="inputValue"
											:disabled="!isInputEnabled"
											placeholder="输入数字即可"
											class="input-style"
											@input="updateSearchParamsDay"
										>
											<template #prepend>
												<span>最近</span>
											</template>
											<template #append>
												<span>天内没有执行预测的装备</span>
											</template>
										</el-input>
										<el-button type="primary" class="ml" @click="handleLoadData"
											>查询</el-button
										>
										<el-button
											type="primary"
											class="ml"
											:disabled="isPredictButtonDisabled"
											@click="handlePredict"
											>执行预测</el-button
										>
									</div>
								</el-form-item>
							</el-form>
						</el-col>
					</el-row>
					<el-row style="margin-top: 10px"> </el-row>
					<el-table
						:data="dataList"
						border
						v-loading="loading"
						:header-cell-style="headerCellStyle"
						@selection-change="handleSelectionChange"
					>
						<el-table-column
							align="center"
							type="selection"
							width="35"
						 :show-overflow-tooltip="true"></el-table-column>
						<el-table-column
							align="center"
							label="装备统一编码"
							prop="equipCode"
							sortable
						 :show-overflow-tooltip="true"></el-table-column>
						<el-table-column
							align="center"
							label="装备战斗编号"
							prop="fightCode"
							sortable
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.fightCode || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="装备型号"
							prop="equipModel.equipSimpleName"
							sortable
						 :show-overflow-tooltip="true"></el-table-column>
						<el-table-column
							align="center"
							label="列装时间"
							prop="serviceDate"
							sortable
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.serviceDate || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="中修间隔期内发动机累计消耗摩托小时"
							width="161px"
							prop="equipRecord.vehicleMotor"
							sortable
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.equipRecord?.vehicleMotor || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="大修间隔期内累计行驶里程(公里)"
							width="140px"
							prop="equipRecord.workMile"
							sortable
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{
									scope.row?.equipRecord?.workMile
										? formatfix2(scope.row?.equipRecord?.workMile)
										: "无"
								}}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="最近三个月预测次数"
							width="121px"
							prop="predictTimes"
							sortable
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.predictTimes || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="最近执行预测时间"
							prop="equipMalf.predictTime"
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.equipMalf?.predictTime || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="最近预测算法"
							prop="equipMalf.modelAlgorithm"
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.equipMalf?.modelAlgorithm || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="最近预测故障类型"
							prop="equipMalf.failureType"
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.equipMalf?.failureType || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="最近预测故障最可能月份"
							width="110px"
							prop="equipMalf.failureTiming"
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ scope.row?.equipMalf?.failureTiming || "无" }}
							</template>
						</el-table-column>
						<el-table-column
							align="center"
							label="月份对应故障概率"
							prop="equipMalf.failureProbability"
						 :show-overflow-tooltip="true">
							<template #default="scope">
								{{ formatPercentage(scope.row?.equipMalf?.failureProbability) }}
							</template>
						</el-table-column>
						<el-table-column align="center" label="算法选择" prop="algorithm" :show-overflow-tooltip="true">
							<template #header>
								<span
									style="
										display: flex;
										align-items: center;
										justify-content: center;
									"
								>
									<el-tooltip raw-content placement="top" effect="light">
										<template #content>
											<span style="text-align: right"
												>mlp：神经网络，拟合能力强，精度高，适合大数据；<br />PSO_SVR：粒子群优化支持向量回归，小样本泛化能力强；<br />DecisionTree：决策树，解释性强，速度快，容易过拟合；
											</span>
										</template>
										<div>
											算法选择<el-icon
												style="margin-left: 4px; color: #409eff; cursor: help"
												><question-filled
											/></el-icon>
										</div>
									</el-tooltip>
								</span>
							</template>
							<template #default="scope">
								<el-select
									v-model="scope.row.algorithm"
									placeholder="选择算法"
									style="width: 100%"
								>
									<el-option
										v-for="option in algorithmOptions"
										:key="option.value"
										:label="option.label"
										:value="option.value"
									></el-option>
								</el-select>
							</template>
						</el-table-column>
						<el-table-column align="center" label="操作" width="280px" :show-overflow-tooltip="true">
							<template #default="scope">
								<div style="display: flex; align-items: center">
									<el-button
										type="primary"
										text
										size="small"
										@click="handleEquipInfo(scope.row.equipCode)"
										>装备详细信息</el-button
									>
									<el-button
										type="warning"
										text
										size="small"
										@click="
											handleHistory(
												scope.row.equipCode,
												scope.row.equipModel.equipSimpleName,
												selectedUnit,
											)
										"
									>
										历史预测
									</el-button>
									<el-button
										type="primary"
										text
										size="small"
										@click="handleDetail(scope.row.equipCode)"
										>预测详情</el-button
									>
								</div>
							</template>
						</el-table-column>
					</el-table>
					<el-pagination
						class="fr mt mb"
						v-model:current-page="pageInfo.current"
						v-model:page-size="pageInfo.size"
						:page-sizes="[10, 20, 30, 40]"
						layout="sizes, prev, pager, next, jumper"
						:total="totals"
						@size-change="handleSizeChange"
						@current-change="handleCurrentChange"
						background
					>
						<template #sizes="{ pageSizes }">
							<span
								v-for="size in pageSizes"
								:key="size"
								class="el-pagination__sizes"
							>
								{{ size }}条/页
							</span>
						</template>
					</el-pagination>
				</el-card>
			</el-col>
		</el-row>
	</div>
</template>
<script lang="ts">
export default {
	name: "Generalfaultdiagnosis",
};
</script>
<script lang="ts" setup>
import { ref, onMounted, watch, onActivated, onUnmounted } from "vue";
import { unitListApi } from "@/api/fault/unitlist";
import { useRouter } from "vue-router";
import { ElMessage, ElTree } from "element-plus";
import { equipmodelApi } from "@/api/fault/equipmodel";
import { useHttp } from "@/hooks/usersHttp";
import { predictgeneralApi } from "@/api/fault/genenralpredict";
import { predictstartApi } from "@/api/fault/predict";

// 安全处理 sessionStorage
const safeSessionStorage = {
	set(key: string, value: string): void {
		try {
			sessionStorage.setItem(key, value);
		} catch (e) {
			console.error("SessionStorage存储失败:", e);
		}
	},
	get(key: string): string {
		try {
			return sessionStorage.getItem(key) || "";
		} catch (e) {
			console.error("SessionStorage读取失败:", e);
			return "";
		}
	},
};

// 添加按钮禁用状态
const isPredictButtonDisabled = ref(false);

// 定时器引用
const saveTimer = ref<NodeJS.Timeout | null>(null);

const selectedUnit = ref("");
interface SearchType {
	equipModel: string;
	equipCode: string;
	equipModelCode: string;
	day: number;
	ifUnpredicted: boolean;
	unitPath: string;
	current: number;
	size: number;
	isDetailed: boolean;
}

const searchParams = ref<SearchType>({
	equipModel: "",
	equipCode: "",
	equipModelCode: "",
	day: 10,
	ifUnpredicted: false,
	unitPath: "",
	current: 1,
	size: 10,
	isDetailed: false,
});
const {
	dataList,
	loading,
	pageInfo,
	loadData,
	handleSizeChange,
	handleCurrentChange,
	totals,
} = useHttp("/equip", searchParams);

const headerCellStyle = (row: any) => {
	const columnLabel = row.column.label;
	if (
		columnLabel === "最近三个月预测次数" ||
		columnLabel === "最近执行预测时间" ||
		columnLabel === "最近预测算法" ||
		columnLabel === "最近预测故障类型" ||
		columnLabel === "最近预测故障最可能月份" ||
		columnLabel === "月份对应故障概率"
	) {
		return {
			backgroundColor: "#F8F8F8",
			color: "black",
			fontWeight: "bold",
		};
	}
	return {
		backgroundColor: "#F8F8F8",
		color: "black",
		fontWeight: "bold",
	};
};

interface Tree {
	unitCode: number;
	parentUnitCode: any;
	unitSimpleName: string;
	unitPath: number;
	childrenUnits?: Tree[];
}
const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref<Tree[]>([]);

const defaultProps = {
	children: "childrenUnits",
	label: "unitSimpleName",
};

// 格式化故障概率为百分比
const formatPercentage = (value: number | null | undefined): string => {
	if (value == null || isNaN(value)) {
		return "-";
	}
	return `${(value * 100).toFixed(0)}%`;
};
const formatfix2 = (value: number | null | undefined) => {
	if (value == null || isNaN(value)) {
		return "-";
	}
	const numValue = typeof value === "string" ? parseFloat(value) : value;
	return numValue.toFixed(0);
};

// 存储查询参数
const saveSearchParams = () => {
	safeSessionStorage.set(
		"generalFaultSearchParams",
		JSON.stringify({
			...searchParams.value,
			selectedUnit: selectedUnit.value,
			isInputEnabled: isInputEnabled.value,
			inputValue: inputValue.value,
		}),
	);
};

// 恢复查询参数
const restoreSearchParams = () => {
	try {
		const saved = safeSessionStorage.get("generalFaultSearchParams");
		if (saved) {
			const params = JSON.parse(saved);

			// 恢复搜索参数
			Object.keys(params).forEach((key) => {
				if (
					key !== "selectedUnit" &&
					key !== "isInputEnabled" &&
					key !== "inputValue"
				) {
					if (key in searchParams.value) {
						searchParams.value[key] = params[key];
					}
				}
			});

			// 恢复UI状态
			selectedUnit.value = params.selectedUnit || "";
			isInputEnabled.value = params.isInputEnabled || false;
			inputValue.value = params.inputValue || "10";

			// 恢复装备型号选项（如果需要）
			if (params.unitPath) {
				restoreEquipmentOptions(params.unitPath);
			}

			return true;
		}
	} catch (error) {
		console.error("恢复查询参数失败:", error);
	}
	return false;
};

// 恢复装备型号选项
const restoreEquipmentOptions = async (unitPath: string) => {
	try {
		const response = await equipmodelApi(unitPath);
		console.log("装备型号数据:", response);
		if (response.code === 200) {
			console.log("请求成功:", response.message);
			if (response.data && response.data.categories) {
				const groupedOptions: {
					label: string;
					options: { label: string; value: string }[];
				}[] = [];
				for (const category in response.data.categories) {
					if (Array.isArray(response.data.categories[category])) {
						const categoryOptions = response.data.categories[category].map(
							(item: any) => ({
								label: item.equipSimpleName || "",
								value: item.equipModelCode || "",
							}),
						);
						groupedOptions.push({
							label: category || "",
							options: categoryOptions,
						});
					}
				}
				equipmentTypeOptions.value = groupedOptions;
				equipmentCodeOptions.value = groupedOptions;
			} else {
				console.warn("categories 数据为空或格式不正确，使用空数组");
				equipmentTypeOptions.value = [];
				equipmentCodeOptions.value = [];
			}
		} else {
			console.error("请求失败:", response.message);
			equipmentTypeOptions.value = [];
			equipmentCodeOptions.value = [];
		}
	} catch (error) {
		console.error("请求装备型号数据失败:", error);
		equipmentTypeOptions.value = [];
		equipmentCodeOptions.value = [];
	}
};

// 修改查询函数，保存参数
const handleLoadData = async () => {
	await loadData();
	saveSearchParams(); // 保存查询参数
};

onMounted(async () => {
	try {
		const { data } = await unitListApi();
		treeData.value = data;
	} catch (error) {
		console.error("获取单位列表失败:", error);
	}

	// 先尝试恢复查询参数
	const hasRestored = restoreSearchParams();

	// 如果有恢复的参数，则重新加载数据
	if (hasRestored) {
		loadData();
	} else {
		// 否则重新初始化
		console.log(1);
		loadData();
	}
});

const findUnitNodeByPath = (nodes: Tree[], path: string): Tree | undefined => {
	for (const node of nodes) {
		if (node.unitPath.toString() === path) {
			return node;
		}
		if (node.childrenUnits) {
			const found = findUnitNodeByPath(node.childrenUnits, path);
			if (found) return found;
		}
	}
	return undefined;
};

watch(
	dataList,
	(newDataList) => {
		if (newDataList && newDataList.length > 0) {
			newDataList.forEach((item: any) => {
				item.algorithm = "mlp";
			});
		}
	},
	{ immediate: true },
);

const isInputEnabled = ref(false);
const inputValue = ref("10");

const updateSearchParamsDay = () => {
	const value = parseInt(inputValue.value) || 10;
	// 添加数值范围检查
	if (value > 999) {
		ElMessage.warning("输入的天数不能超过999天");
		inputValue.value = "999"; // 将输入值重置为最大值
		searchParams.value.day = 999;
		return;
	}
	searchParams.value.day = value;
};

const handleSwitchChange = (value: boolean) => {
	searchParams.value.ifUnpredicted = value;
	if (!value) {
		inputValue.value = "";
		searchParams.value.day = 0;
	} else {
		inputValue.value = "10";
		searchParams.value.day = 10;
	}
};

const algorithmOptions = [
	{ label: "MLP", value: "mlp" },
	// { label: 'mlp', value: 'mlp' },
	{ label: "DecisionTree", value: "DecisionTree" },
	{ label: "PSO_SVR", value: "PSO_SVR" },
];

const router = useRouter();
const selectedEquipments = ref<any[]>([]);

const handleSelectionChange = (selection: any[]) => {
	selectedEquipments.value = selection;
	console.log("选中的装备:", selectedEquipments.value);
};

const handlePredict = async () => {
	if (selectedEquipments.value.length === 0) {
		ElMessage.warning("请先选择一个装备");
		return;
	}
	isPredictButtonDisabled.value = true;
	let sessionId = "";
	try {
		const startResponse = await predictstartApi();
		if (startResponse.code === 200 && startResponse.data) {
			sessionId = startResponse.data;
		} else {
			throw new Error("初始化预测失败: 无有效 sessionId");
		}
	} catch (error) {
		console.error("获取 sessionId 失败:", error);
		ElMessage.error("初始化预测失败，请稍后重试");
		return;
	}

	const equipCodes = selectedEquipments.value.map((item) => item.equipCode);
	const modelAlgorithms = selectedEquipments.value.map(
		(item) => item.algorithm || "mlp",
	);
	const userId = "12354";
	const isDetailed = false;

	try {
		const response = await predictgeneralApi(
			equipCodes,
			isDetailed,
			modelAlgorithms,
			sessionId,
			userId,
		);
		if (response.code === 200) {
			// Map response data to update selectedEquipments with predictId
			const updatedEquipments = response.data.map((prediction: any) => {
				const equip = selectedEquipments.value.find(
					(item) => item.equipCode === prediction.equipCode,
				);
				return {
					equipCode: prediction.equipCode,
					equipSimpleName: equip
						? equip.equipModel.equipSimpleName
						: "未知型号",
					algorithm: equip ? equip.algorithm || "mlp" : "mlp",
					predictId: prediction.predictId,
				};
			});

			// Store updated equipments in sessionStorage
			safeSessionStorage.set(
				"selectedEquipments",
				JSON.stringify(updatedEquipments),
			);
			safeSessionStorage.set("selectedEquipCodes", JSON.stringify(equipCodes));

			ElMessage.success("预测执行成功");
			isPredictButtonDisabled.value = false;

			// Navigate to detail page with the first equipment's equipCode and predictId
			if (response.data.length > 0) {
				const firstEquip = response.data[0];
				router.push(
					`/predict/generaldetail?equipCode=${firstEquip.equipCode}&predictId=${firstEquip.predictId}&sessionId=${sessionId}`,
				);
			}
		} else {
			ElMessage.error("批量预测执行失败: " + response.message);
		}
	} catch (error) {
		console.error("批量预测执行失败:", error);
		ElMessage.error("批量预测执行失败，请稍后重试");
		return;
	}

	// const checkProgress = async () => {
	//   if (!sessionId) {
	//     console.error('无法监控进度: sessionId 为空');
	//     ElMessage.error('无法监控进度: sessionId 无效');
	//     return;
	//   }

	//   try {
	//     const progressResponse = await predictProgressApi({ sessionId });
	//     if (progressResponse.code === 200) {
	//       console.log('预测进度:', progressResponse.data);
	//     } else {
	//       console.error('预测进度请求失败:', progressResponse.message);
	//     }
	//   } catch (error) {
	//     console.error('监控进度失败:', error);
	//     ElMessage.error('监控进度失败，请稍后重试');
	//   }
	// };
	// checkProgress();
};

const handleDetail = (equipCode: string) => {
	const row = dataList.value.find((item: any) => item.equipCode === equipCode);
	const predictId = row?.equipMalf?.predictId || null;
	if (row.equipMalf?.predictTime) {
		router.push(
			`/predict/generaldetail?equipCode=${equipCode}${
				predictId ? `&predictId=${predictId}` : ""
			}`,
		);
	} else {
		return ElMessage.warning("该装备暂无预测记录");
	}
};

const handleHistory = (
	equipCode: string,
	equipSimpleName: string,
	unitSimpleName: string,
) => {
	const row = dataList.value.find((item: any) => item.equipCode === equipCode);
	if (row?.equipMalf?.predictTime) {
		router.push({
			path: "/predict/generalhistory",
			query: { equipCode, equipSimpleName, unitSimpleName: selectedUnit.value },
		});
	} else {
		return ElMessage.warning("该装备暂无预测历史记录");
	}
};

const handleEquipInfo = (equipCode: string) => {
	router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode);
};

const equipmentTypeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);

const equipmentCodeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);

const _deprecated_handleNodeClick = async (nodeData: Tree) => {
	selectedUnit.value = nodeData.unitSimpleName;
	// 重置相关参数
	searchParams.value.equipModelCode = "";
	searchParams.value.equipCode = "";
	const unitPath = nodeData.unitPath.toString();
	searchParams.value.unitPath = unitPath;
};
const fetchEquipmentModels = async () => {
	const unitPath = searchParams.value.unitPath;
	if (!unitPath) {
		ElMessage.warning("请先选择单位/分队");
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
					const categoryOptions = response.data.categories[category].map(
						(item: any) => ({
							label: item.equipSimpleName || "",
							value: item.equipModelCode || "",
						}),
					);
					groupedOptions.push({
						label: category || "",
						options: categoryOptions,
					});
				}
			}
			equipmentTypeOptions.value = groupedOptions;
		} else {
			equipmentTypeOptions.value = [];
		}
	} catch (error) {
		console.error("获取型号失败:", error);
		equipmentTypeOptions.value = [];
	}
};

// watch(selectedUnit, (newVal) => {
//   if (!newVal) {
//     searchParams.value.unitPath = '';
//     searchParams.value.equipModelCode = '';
//     searchParams.value.equipCode = '';
//     equipmentTypeOptions.value = [];
//     equipmentCodeOptions.value = [];
//   }
// });
watch(selectedUnit, (newVal) => {
	searchParams.value.unitPath = newVal ? searchParams.value.unitPath : "";
	searchParams.value.equipModelCode = "";
	searchParams.value.equipCode = "";
	// 清空选项列表，强制下次展开时重新触发 fetchEquipmentModels
	equipmentTypeOptions.value = [];
	equipmentCodeOptions.value = [];
});
// 监听搜索参数变化并保存
watch(
	[
		() => searchParams.value.equipModelCode,
		() => searchParams.value.equipCode,
		() => searchParams.value.day,
		() => searchParams.value.ifUnpredicted,
		() => searchParams.value.unitPath,
		selectedUnit,
		isInputEnabled,
		inputValue,
	],
	() => {
		// 防抖保存，避免频繁存储
		if (saveTimer.value) {
			clearTimeout(saveTimer.value);
		}
		saveTimer.value = setTimeout(() => {
			saveSearchParams();
		}, 300);
	},
	{ deep: true },
);

onActivated(() => {
	console.log("Generalfaultdiagnosis.vue activated, refreshing data", 111111);

	// 先尝试恢复查询参数
	const hasRestored = restoreSearchParams();

	// 如果有恢复的参数，则重新加载数据
	if (hasRestored) {
		loadData();
	} else {
		// 否则重新初始化
		loadData();
	}
});

// 组件销毁时清理定时器
onUnmounted(() => {
	if (saveTimer.value) {
		clearTimeout(saveTimer.value);
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

	.el-table-column {
		text-align: center;
	}

	.el-select {
		width: 100%;
	}
}

.custom-header-cell {
	background-color: #f5f7fa;
	font-weight: bold;
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

:deep(.el-select-group__title) {
	font-weight: bold;
	color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
	padding-left: 30px;
}
</style>