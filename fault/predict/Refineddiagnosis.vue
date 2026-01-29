```vue
<template>
	<el-card>
		<el-row style="margin-top: 15px"> </el-row>
		<el-row :gutter="20">
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
							v-model="searchParams.equipCode"
							style="width: 240px"
							placeholder="请输入装备统一编码"
						/>
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
									<span>天内没有执行诊断的装备</span>
								</template>
							</el-input>
							<el-button type="primary" class="ml" @click="handleLoadData"
								>查询</el-button
							>
							<el-button
								type="primary"
								class="ml"
								@click="handleDiagnose"
								:disabled="isPredictButtonDisabled"
								>执行诊断</el-button
							>
						</div>
					</el-form-item>
				</el-form>
			</el-col>
		</el-row>
		<el-row style="margin-top: 30px"> </el-row>
		<el-row>
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
					width="173px"
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
					width="150px"
					prop="equipRecord.workMile"
					sortable
				 :show-overflow-tooltip="true">
					<template #default="scope">
						{{ scope.row?.equipRecord?.workMile || "无" }}
					</template>
				</el-table-column>
				<el-table-column
					align="center"
					label="最近三个月诊断次数"
					width="115px"
					prop="diagnoseTimes"
					sortable
				 :show-overflow-tooltip="true">
					<template #default="scope">
						{{ scope.row?.diagnoseTimes || "无" }}
					</template>
				</el-table-column>
				<el-table-column
					align="center"
					label="最近执行诊断时间"
					prop="equipDiag.diagnoseTime"
				 :show-overflow-tooltip="true">
					<template #default="scope">
						{{ scope.row?.equipDiag?.diagnoseTime || "无" }}
					</template>
				</el-table-column>
				<el-table-column
					align="center"
					label="最近诊断算法"
					prop="equipDiag.modelAlgorithm"
				 :show-overflow-tooltip="true">
					<template #default="scope">
						{{ scope.row?.equipDiag?.modelAlgorithm || "无" }}
					</template>
				</el-table-column>
				<el-table-column
					align="center"
					label="最近诊断故障类型"
					prop="equipMalf.failureType"
				 :show-overflow-tooltip="true">
					<template #default="scope">
						{{ scope.row?.equipMalf?.failureType || "无" }}
					</template>
				</el-table-column>
				<el-table-column
					align="center"
					label="最近诊断故障最可能月份"
					width="112px"
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
							算法选择
							<el-tooltip raw-content placement="top" effect="light">
								<template #content>
									<div style="text-align: left; line-height: 1.5">
										deepforest：级联多层随机森林，层级特征提取，超参数少、训练快，对小样本友好；<br />
										transformer：注意力网络，计算复杂度高，并行能力强，适合大量数据；<br />
									</div>
								</template>
								<el-icon style="margin-left: 4px; color: #409eff; cursor: help">
									<QuestionFilled />
								</el-icon>
							</el-tooltip>
						</span>
					</template>
					<template #default="scope">
						<el-select
							clearable
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
				<el-table-column align="center" label="操作" width="300px" :show-overflow-tooltip="true">
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
										scope.row.unit.unitSimpleName,
									)
								"
							>
								历史诊断
							</el-button>
							<el-button
								type="primary"
								text
								size="small"
								@click="
									handleDetail(
										scope.row.equipCode,
										scope.row.equipDiag?.diagnoseId,
									)
								"
								>诊断详情</el-button
							>
						</div>
					</template>
				</el-table-column>
			</el-table>
			<el-col :span="24">
				<el-pagination
					class="fr mt mb"
					v-model:current-page="pageInfo.page"
					v-model:page-size="pageInfo.pageSize"
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
			</el-col>
		</el-row>
	</el-card>
</template>

<script lang="ts" setup>
import { ref, onMounted, reactive, watch, onActivated, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import { useHttp } from "@/hooks/usersHttp";
import { unitListApi } from "@/api/fault/unitlist";
import { FiningequipmodelApi } from "@/api/fault/finingequipmodel";
import { diagnoseApi } from "@/api/fault/diagnose";
import { ElMessage, ElTree } from "element-plus";
import { predictstartApi, predictProgressApi } from "@/api/fault/predict";

const selectedUnit = ref("");
const isPredictButtonDisabled = ref(false);

const handleHistory = (
	equipCode: string,
	equipSimpleName: string,
	unitSimpleName: string,
) => {
	const row = dataList.value.find((item: any) => item.equipCode === equipCode);
	if (row.equipDiag?.diagnoseTime) {
		router.push({
			path: "/predict/diagnosehistory",
			query: { equipCode, equipSimpleName, unitSimpleName },
		});
	} else {
		return ElMessage.warning("该装备暂无诊断历史记录");
	}
};

const totals = ref<number>(0);
const pageInfo = reactive({
	page: 1,
	pageSize: 10,
});
const selectedEquipments = ref<any[]>([]);
const handleSelectionChange = (selection: any[]) => {
	selectedEquipments.value = selection;
	console.log("选中的装备:", selectedEquipments.value);
};

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

const formatPercentage = (value: number | null | undefined): string => {
	if (value == null || isNaN(value)) {
		return "-";
	}
	return `${(value * 100).toFixed(0)}%`;
};
const clearDiagnosisCache = () => {
	try {
		// 清空所有相关的 sessionStorage 数据
		sessionStorage.removeItem("refinedDiagnosisEquipments");
		sessionStorage.removeItem("refinedDiagnosisEquipCodes");
		sessionStorage.removeItem("diagnoseDetailEquipments");

		console.log("诊断缓存已清空");
	} catch (error) {
		console.error("清空诊断缓存失败:", error);
	}
};
const handleDiagnose = async () => {
	if (selectedEquipments.value.length === 0) {
		ElMessage.warning("请先选择一个装备");
		return;
	}
	clearDiagnosisCache();
	isPredictButtonDisabled.value = true;
	let sessionId = "";
	try {
		const startResponse = await predictstartApi();
		if (startResponse.code === 200 && startResponse.data) {
			sessionId = startResponse.data;
		} else {
			throw new Error("初始化诊断失败: 无有效 sessionId");
		}
	} catch (error) {
		console.error("获取 sessionId 失败:", error);
		ElMessage.error("初始化诊断失败，请稍后重试");
		isPredictButtonDisabled.value = false;
		return;
	}

	const equipCodes = selectedEquipments.value.map((item) => item.equipCode);
	const modelAlgorithms = selectedEquipments.value.map(
		(item) => item.algorithm || "transformer",
	);
	const userId = "12354";
	const isDetailed = true;

	try {
		const response = await diagnoseApi(
			equipCodes,
			modelAlgorithms,
			sessionId,
			userId,
		);
		if (response.code === 200) {
			const updatedEquipments = response.data.map((record: any) => {
				const equip = selectedEquipments.value.find(
					(item) => item.equipCode === record.equipCode,
				);
				return {
					equipCode: record.equipCode,
					equipSimpleName: equip
						? equip.equipModel.equipSimpleName
						: "未知型号",
					algorithm: equip ? equip.algorithm || "transformer" : "transformer",
					diagnoseId: record.diagnoseId || null,
				};
			});

			safeSessionStorage.set(
				"refinedDiagnosisEquipments",
				JSON.stringify(updatedEquipments),
			);
			safeSessionStorage.set(
				"refinedDiagnosisEquipCodes",
				JSON.stringify(equipCodes),
			);

			ElMessage.success("诊断执行成功");
			isPredictButtonDisabled.value = false;

			if (response.data.length > 0) {
				const firstEquip = response.data[0];
				router.push({
					path: "/predict/diagnosedetail",
					query: {
						equipCode: firstEquip.equipCode,
						diagnoseId: firstEquip.diagnoseId,
						sessionId: sessionId,
						clearCache: "true", // 明确标记需要清理模式
					},
				});
			}
		} else {
			ElMessage.error("批量诊断执行失败: " + response.message);
			isPredictButtonDisabled.value = false;
		}
	} catch (error) {
		console.error("批量诊断执行失败:", error);
		ElMessage.error("批量诊断执行失败，请稍后重试");
		isPredictButtonDisabled.value = false;
		return;
	}
};

const handleEquipInfo = (equipCode: string) => {
	router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode);
};

const handleDetail = (equipCode: string, diagnoseId: number | null) => {
	const row = dataList.value.find((item: any) => item.equipCode === equipCode);
	if (row.equipDiag?.diagnoseTime) {
		router.push(
			`/predict/diagnosedetail?equipCode=${equipCode}${
				diagnoseId ? `&diagnoseId=${diagnoseId}` : ""
			}`,
		);
	} else {
		return ElMessage.warning("该装备暂无诊断记录");
	}
};

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
	day: 0,
	ifUnpredicted: false,
	unitPath: "",
	current: 1,
	size: 10,
	isDetailed: true,
});

const { dataList, loading, loadData, handleSizeChange, handleCurrentChange } =
	useHttp("/equip", searchParams);

const headerCellStyle = {
	backgroundColor: "#F8F8F8",
	color: "black",
	fontWeight: "bold",
};

const isInputEnabled = ref(false);
const inputValue = ref("10");

const updateSearchParamsDay = () => {
	const value = parseInt(inputValue.value) || 10;
	searchParams.value.day = value;
	// 添加数值范围检查
	if (value > 999) {
		ElMessage.warning("输入的天数不能超过999天");
		inputValue.value = "999"; // 将输入值重置为最大值
		searchParams.value.day = 999;
		return;
	}
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

watch(
	dataList,
	(newDataList) => {
		if (newDataList && newDataList.length > 0) {
			newDataList.forEach((item: any) => {
				item.algorithm = "transformer";
			});
		}
	},
	{ immediate: true },
);

const algorithmOptions = [
	{ label: "deepforest", value: "deepforest" },
	{ label: "transformer", value: "transformer" },
];

const router = useRouter();
// 存储查询参数
const saveSearchParams = () => {
	safeSessionStorage.set(
		"finingFaultDiagSearchParams",
		JSON.stringify({
			...searchParams.value,
			selectedUnit: selectedUnit.value,
			isInputEnabled: isInputEnabled.value,
			inputValue: inputValue.value,
		}),
	);
		console.log({...searchParams.value})
};

// 修改查询函数，保存参数
const handleLoadData = async () => {
	await loadData();
	saveSearchParams(); // 保存查询参数
};

// 恢复查询参数
const restoreSearchParams = () => {
	try {
		const saved = safeSessionStorage.get("finingFaultDiagSearchParams");
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
		const response = await FiningequipmodelApi(unitPath);
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
			} else {
				console.warn("categories 数据为空或格式不正确，使用空数组");
				equipmentTypeOptions.value = [];
			}
		} else {
			console.error("请求失败:", response.message);
			equipmentTypeOptions.value = [];
		}
	} catch (error) {
		console.error("请求装备型号数据失败:", error);
		equipmentTypeOptions.value = [];
	}
};

onMounted(async () => {
	try {
		const { data } = await unitListApi();
		treeData.value = data;
	} catch (error) {
		console.error("Failed to fetch city list:", error);
	}
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
		const response = await FiningequipmodelApi(unitPath);
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
watch(selectedUnit, (newVal) => {
	searchParams.value.unitPath = newVal ? searchParams.value.unitPath : "";
	searchParams.value.equipModelCode = "";
	searchParams.value.equipCode = "";
	// 清空选项列表，强制下次展开时重新触发 fetchEquipmentModels
	equipmentTypeOptions.value = [];
	equipmentCodeOptions.value = [];
});
// 定时器引用
const saveTimer = ref<NodeJS.Timeout | null>(null);
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
	console.log("finingfaultpredict.vue activated, refreshing data");

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
    const response = await FiningequipmodelApi(unitPath)
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
