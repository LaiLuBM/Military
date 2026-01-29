<template>
	<div>
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
						<el-form-item
							label="装备型号"
							v-model="searchParams.equipSimpleName"
						>
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
				<el-col :span="3">
					<el-form>
						<el-form-item label="核准状态">
							<el-select
								clearable
								placeholder="核准状态"
								v-model="searchParams.approvalStatus"
							>
								<el-option
									v-for="option in Options"
									:key="option.value"
									:label="option.label"
									:value="option.value"
								></el-option>
							</el-select>
						</el-form-item>
					</el-form>
				</el-col>
				<el-col :span="9">
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
								<el-button type="primary" class="ml" @click="handleDiagnose"
									>健康评估</el-button
								>
							</div>
						</el-form-item>
					</el-form>
				</el-col>
			</el-row>
			<el-row style="margin-top: 10px"> </el-row>
			<el-row>
				<el-table
					ref="tableRef"
					:data="dataList"
					:loading="loading"
					border
					:header-cell-style="headerCellStyle"
					@select="handleSelect"
					@selection-change="handleSelectionChange"
				>
					<el-table-column
						type="selection"
						width="35"
						:selectable="checkSelectable"
					 :show-overflow-tooltip="true"></el-table-column>
					<el-table-column label="装备统一编码" prop="equipCode" align="center" :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.equipCode || "无" }}
						</template>
					</el-table-column>
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
						label="装备型号"
						prop="equipSimpleName"
						align="center"
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.equipSimpleName || "无" }}
						</template>
					</el-table-column>
					<el-table-column
						label="中修间隔期内发动机累计消耗摩托小时"
						width="177px"
						prop="motorHour"
						align="center"
						sortable
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.motorHour || row.motorHour === 0 ? row.motorHour : "无" }}
						</template>
					</el-table-column>
					<el-table-column
						label="大修间隔期内累计行驶里程"
						width="134px"
						prop="workMile"
						align="center"
						sortable
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.workMile || row.workMile === 0 ? row.workMile : "无" }}
						</template>
					</el-table-column>
					<el-table-column
						label="评估时间"
						prop="latestAssessment.assessmentDate"
						align="center"
						sortable
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.latestAssessment?.assessmentDate || "无" }}
						</template>
					</el-table-column>
					<el-table-column
						label="整机健康指数"
						prop="latestAssessment.equipHealthScore"
						align="center"
						sortable
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{
								row.latestAssessment?.equipHealthScore ||
								row.latestAssessment?.equipHealthScore === 0
									? row.latestAssessment.equipHealthScore
									: "无"
							}}
						</template>
					</el-table-column>
					<el-table-column
						label="核准状态"
						prop="latestAssessment.approvalStatus"
						align="center"
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.latestAssessment?.approvalStatus || "无" }}
						</template>
					</el-table-column>
					<el-table-column
						label="核准时间"
						prop="latestAssessment.approvalSubmissionDate"
						align="center"
						sortable
					 :show-overflow-tooltip="true">
						<template #default="{ row }">
							{{ row.latestAssessment?.approvalSubmissionDate || "无" }}
						</template>
					</el-table-column>
					<el-table-column
						label="算法选择"
						prop="algorithm"
						align="center"
						width="120px"
					 :show-overflow-tooltip="true">
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
											>mlp：神经网络，拟合能力强，精度高，适合大数据；<br />
											PSO_SVR：粒子群优化支持向量回归，小样本泛化能力强；<br />
											DecisionTree：决策树，解释性强，速度快，容易过拟合；<br />
											         
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
					<el-table-column label="操作" align="center" width="280px" :show-overflow-tooltip="true">
						<template #default="scope">
							<div style="display: flex; align-items: center">
								<el-button
									type="primary"
									text
									size="small"
									@click="handleDetail(scope.row.equipCode)"
									>装备详细信息</el-button
								>
								<el-button
									type="warning"
									text
									size="small"
									@click="
										handleHistory(
											scope.row.equipCode,
											scope.row.latestAssessment.approvalStatus,
										)
									"
									>历史评估</el-button
								>
								<el-button
									type="primary"
									text
									size="small"
									@click="
										handleEquipInfo(
											scope.row.equipCode,
											scope.row.latestAssessment.approvalStatus,
											scope.row.latestAssessment.assessmentId,
										)
									"
								>
									评估详情</el-button
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
	</div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, onActivated, onUnmounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import { useHttp } from "@/hooks/healthhttp";
import { unitListApi } from "@/api/fault/unitlist";
import { Equipmodelhelth } from "@/api/fault/equipmodel";
import { assessment } from "@/api/fault/refinehealth";
import { ElMessage, ElTree, ElTable } from "element-plus";


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

// 定时器引用
const saveTimer = ref<NodeJS.Timeout | null>(null);

const selectedUnit = ref("");
const router = useRouter();
const route = useRoute();

const algorithmOptions = [
	{ label: "MLP", value: "mlp" },
	{ label: "PSO_SVR", value: "PSO_SVR" },
	{ label: "DecisionTree", value: "DecisionTree" },
];

const Options = [
	{ label: "已核准", value: "已核准" },
	{ label: "审核中", value: "审核中" },
	{ label: "待审核", value: "待审核" },
];

interface SearchType {
	equipCode: string;
	unitPath: string;
	equipModelCode: string;
	equipSimpleName: string;
	x: number;
	xFlag: number;
	approvalStatus: string;
	page: number;
	pageSize: number;
	isDetailed: number;
}

const searchParams = ref<SearchType>({
	approvalStatus: "",
	equipModelCode: "",
	equipSimpleName: "",
	equipCode: "",
	unitPath: "",
	xFlag: 0,
	x: 30,
	page: 1,
	pageSize: 10,
	isDetailed: 0,
});

const headerCellStyle = {
	backgroundColor: "#F8F8F8",
	color: "black",
	fontWeight: "bold",
};

const {
	dataList,
	loading,
	pageInfo,
	loadData,
	handleSizeChange,
	handleCurrentChange,
	totals,
} = useHttp("/equipHealthAssessments", searchParams);

interface Tree {
	unitCode: string;
	unitPath: string;
	parentUnitCode: any;
	unitSimpleName: string;
	childrenUnits?: Tree[];
}
const isRestoring=ref(false)
const treeRef = ref<InstanceType<typeof ElTree>>();
const treeData = ref<Tree[]>([]);
const defaultProps = {
	children: "childrenUnits",
	label: "unitSimpleName",
};

const tableRef = ref<InstanceType<typeof ElTable>>();

const checkSelectable = () => {
	return true;
};

const handleSelect = (_selection: any[], row: any) => {
	const table = tableRef.value;
	if (table) {
		table.clearSelection();
		table.toggleRowSelection(row, true);
	}
};

const handleSelectionChange = (rows: any[]) => {
	selectedEquipments.value = rows.length > 0 ? [rows[rows.length - 1]] : [];
	console.log("选中的装备:", selectedEquipments.value);
};

const equipmentTypeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);
const selectedEquipments = ref<any[]>([]);

// // 存储查询参数
// const saveSearchParams = () => {
// 	safeSessionStorage.set(
// 		"healthAssessmentSearchParams",
// 		JSON.stringify({
// 			...searchParams.value,
// 			selectedUnit: selectedUnit.value,
// 			isInputEnabled: isInputEnabled.value,
// 			inputValue: inputValue.value,
// 		}),
// 	);
// 	console.log({...searchParams.value})
// };

const saveSearchParams = () => {
	safeSessionStorage.set(
		"healthAssessmentSearchParams",
		JSON.stringify({
			...searchParams.value,
			selectedUnit: selectedUnit.value,
			isInputEnabled: isInputEnabled.value,
			inputValue: inputValue.value,
		}),
	);
};

// // 恢复查询参数
// const restoreSearchParams = () => {
// 	try {
// 		const saved = safeSessionStorage.get("healthAssessmentSearchParams");
// 		if (saved) {
// 			const params = JSON.parse(saved);
// 			console.log(111,params)
// 			// 恢复搜索参数
// 			Object.keys(params).forEach((key) => {
// 				if (
// 					key !== "selectedUnit" &&
// 					key !== "isInputEnabled" &&
// 					key !== "inputValue"
// 				) {
// 					if (key in searchParams.value) {
// 						searchParams.value[key] = params[key];
// 					}
// 				}
// 			});

// 			// 恢复UI状态
// 			selectedUnit.value = params.selectedUnit || "";
// 			isInputEnabled.value = params.isInputEnabled || false;
// 			inputValue.value = params.inputValue || "30";

// 			// 恢复装备型号选项（如果需要）
// 			if (params.unitPath) {
// 				restoreEquipmentOptions(params.unitPath);
// 			}

// 			return true;
// 		}
// 	} catch (error) {
// 		console.error("恢复查询参数失败:", error);
// 	}
// 	return false;
// };

const restoreEquipmentOptions = async (unitPath: string) => {
	try {
		const response = await Equipmodelhelth(unitPath);
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


const restoreSearchParams = async() => {
	try {
		const saved = safeSessionStorage.get("healthAssessmentSearchParams");
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
			searchParams.value.unitPath=params.unitPath || "";
			if(params.unitPath){
				await restoreEquipmentOptions(params.unitPath);
			}
			searchParams.value.equipModelCode=params.equipModelCode || ""
			isInputEnabled.value = params.isInputEnabled || false;
			inputValue.value = params.inputValue || "10";
			setTimeout(()=>{
				isRestoring.value=false
			})

			return true;
		}
	} catch (error) {
		console.error("恢复查询参数失败:", error);
	}
	return false;
};



// // 恢复装备型号选项
// const restoreEquipmentOptions = async (unitPath: string) => {
// 	try {
// 		const response = await Equipmodelhelth(unitPath);
// 		console.log("装备型号数据:", response);

// 	if (response.data && response.data.categories) {
// 				const groupedOptions: {
// 					label: string;
// 					options: { label: string; value: string }[];
// 				}[] = [];
// 				for (const category in response.data.categories) {
// 					if (Array.isArray(response.data.categories[category])) {
// 						const categoryOptions = response.data.categories[category].map(
// 							(item: any) => ({
// 								label: item.equipSimpleName || "",
// 								value: item.equipModelCode || "",
// 							}),
// 						);
// 						groupedOptions.push({
// 							label: category || "",
// 							options: categoryOptions,
// 						});
// 					}
// 				}
// 				equipmentTypeOptions.value = groupedOptions;
				
// 			} else {
// 			console.error("数据格式错误或请求失败:", response.message);
// 			equipmentTypeOptions.value = [];
// 		}
// 	} catch (error) {
// 		console.error("请求装备型号数据失败:", error);
// 		equipmentTypeOptions.value = [];
// 	}
// };

// const restoreEquipmentOptions = async (unitPath: string) => {
// 	try {
// 		const response = await Equipmodelhelth(unitPath);
// 		console.log("装备型号数据:", response);
// 		if (response.code === 200) {
// 			console.log("请求成功:", response.message);
// 			if (response.data && response.data.categories) {
// 				const groupedOptions: {
// 					label: string;
// 					options: { label: string; value: string }[];
// 				}[] = [];
// 				for (const category in response.data.categories) {
// 					if (Array.isArray(response.data.categories[category])) {
// 						const categoryOptions = response.data.categories[category].map(
// 							(item: any) => ({
// 								label: item.equipSimpleName || "",
// 								value: item.equipModelCode || "",
// 							}),
// 						);
// 						groupedOptions.push({
// 							label: category || "",
// 							options: categoryOptions,
// 						});
// 					}
// 				}
// 				equipmentTypeOptions.value = groupedOptions;
				
// 			} else {
// 				console.warn("categories 数据为空或格式不正确，使用空数组");
// 				equipmentTypeOptions.value = [];
			
// 			}
// 		} else {
// 			console.error("请求失败:", response.message);
// 			equipmentTypeOptions.value = [];
			
// 		}
// 	} catch (error) {
// 		console.error("请求装备型号数据失败:", error);
// 		equipmentTypeOptions.value = [];
		
// 	}
// };


// 修改查询函数，保存参数
const handleLoadData = async () => {
	await loadData();
	saveSearchParams(); // 保存查询参数
};

// const handleEquipModelChange = (value: string) => {
// 	console.log("选择的装备型号:", value);
// 	const foundOption = equipmentTypeOptions.value
// 		.flatMap((group) => group.options)
// 		.find((option) => option.value === value);
// 	if (foundOption) {
// 		searchParams.value.equipModelCode = value;
// 		searchParams.value.equipSimpleName = foundOption.label;
// 	} else {
// 		searchParams.value.equipModelCode = "";
// 		searchParams.value.equipSimpleName = "";
// 	}
// };
const _deprecated_handleNodeClick = async (nodeData: Tree) => {
	selectedUnit.value = nodeData.unitSimpleName;
	// 重置相关参数
	searchParams.value.unitPath = nodeData.unitPath;
	searchParams.value.equipCode = "";
	searchParams.value.equipModelCode = "";
	searchParams.value.equipSimpleName = "";
	// searchParams.value.equipModelCode = '';
	// searchParams.value.equipCode = '';
	// const unitPath = nodeData.unitPath.toString();
	// searchParams.value.unitPath = unitPath;
};
// const _deprecated_handleNodeClick = async (nodeData: Tree) => {
//   selectedUnit.value = nodeData.unitSimpleName;
//   searchParams.value.unitPath = nodeData.unitPath;
//   searchParams.value.equipCode = '';
//   searchParams.value.equipModelCode = '';
//   searchParams.value.equipSimpleName = '';

//   try {
//     const response = await Equipmodelhelth(nodeData.unitPath);
//     console.log('装备型号数据:', response);

//     if (response.code === 200 && response.data?.categories) {
//       const groupedOptions = Object.entries(response.data.categories).map(([category, items]) => {
//         if (!Array.isArray(items)) {
//           console.error(`类别 ${category} 的数据不是数组:`, items);
//           return { label: category, options: [] };
//         }

//         const options = items.map((item: any) => ({
//           label: item.equipSimpleName,
//           value: item.equipModelCode,
//         }));

//         return {
//           label: category,
//           options,
//         };
//       });

//       equipmentTypeOptions.value = groupedOptions;
//       console.log('生成的 equipmentTypeOptions:', equipmentTypeOptions.value);
//     } else {
//       console.error('数据格式错误或请求失败:', response.message);
//       equipmentTypeOptions.value = [];
//     }
//   } catch (error) {
//     console.error('请求装备型号数据失败:', error);
//     equipmentTypeOptions.value = [];
//     ElMessage.error('获取装备型号失败');
//   }
// };
const fetchEquipmentModels = async () => {
	const unitPath = searchParams.value.unitPath;
	if (!unitPath) {
		ElMessage.warning("请先选择单位/分队");
		return;
	}

	// 如果已经有数据了，可以选择不再重复查询（根据业务需求决定）
	if (equipmentTypeOptions.value.length > 0) return;

	try {
		const response = await Equipmodelhelth(unitPath);
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
const isInputEnabled = ref(false);
const inputValue = ref("30");

const updateSearchParamsDay = () => {
	const value = parseInt(inputValue.value) || 30;
	// 添加数值范围检查
	if (value > 999) {
		ElMessage.warning("输入的天数不能超过999天");
		inputValue.value = "999"; // 将输入值重置为最大值
		searchParams.value.x = 999;
		return;
	}
	searchParams.value.x = value;
};

const handleSwitchChange = (value: boolean) => {
	searchParams.value.xFlag = value ? 1 : 0;
	if (!value) {
		inputValue.value = "";
		searchParams.value.x = 0;
	} else {
		inputValue.value = "30";
		searchParams.value.x = 30;
	}
};

const handleEquipInfo = (
	equipCode: string,
	approvalStatus: string,
	assessmentId: string,
) => {
	router.push({
		path: "/healthevaluate/generalhealthdetail",
		query: { equipCode, approvalStatus, assessmentId },
	});
};

const handleDetail = (equipCode: string) => {
	router.push("/deviceinformation/Detailinformation?equipCode=" + equipCode);
};

const handleHistory = (equipCode: string, approvalStatus: string) => {
	router.push({
		path: "/healthevaluate/generalhealthhistory",
		query: { equipCode, approvalStatus },
	});
};

const handleDiagnose = async () => {
	if (selectedEquipments.value.length === 0) {
		ElMessage.warning("请先选择一个装备");
		return;
	}
	const equipCode = selectedEquipments.value[0].equipCode;
	const modelAlgorithm = selectedEquipments.value[0].algorithm;
	const isDetailed = false;
	const userId = 123;
	const params = { equipCode, modelAlgorithm, isDetailed, userId };
	const response = await assessment(params);
	const approvalStatus = response.data.approvalStatus;
	const assessmentId = response.data.assessmentId;
	router.push({
		path: "/healthevaluate/generalhealthdetail",
		query: { equipCode, approvalStatus, assessmentId },
	});
	await loadData();
};

// const loadUnitAndEquipData = async () => {
// 	try {
// 		// Fetch unit list
// 		const { data } = await unitListApi();
// 		treeData.value = data;

// 		// Process query parameters
// 		const equipModelCode = route.query.equipModelCode as string;
// 		const unitCode = route.query.unitCode as string;

// 		if (unitCode) {
// 			// Find the unit by unitCode in the treeData
// 			const findUnit = (nodes: Tree[], code: string): Tree | undefined => {
// 				for (const node of nodes) {
// 					if (node.unitCode === code) {
// 						return node;
// 					}
// 					if (node.childrenUnits) {
// 						const found = findUnit(node.childrenUnits, code);
// 						if (found) return found;
// 					}
// 				}
// 				return undefined;
// 			};
// 			const unit = findUnit(data, unitCode);
// 			if (unit) {
// 				selectedUnit.value = unit.unitSimpleName;
// 				searchParams.value.unitPath = unit.unitPath;
// 			} else {
// 				ElMessage.warning(`未找到单位编码 ${unitCode} 对应的单位名称`);
// 			}

// 			// Fetch equipment models for the selected unit
// 			if (unit?.unitPath) {
// 				const response = await Equipmodelhelth(unit.unitPath);
// 				if (response.code === 200 && response.data?.categories) {
// 					const groupedOptions = Object.entries(response.data.categories).map(
// 						([category, items]) => {
// 							if (!Array.isArray(items)) {
// 								return { label: category, options: [] };
// 							}
// 							return {
// 								label: category,
// 								options: items.map((item: any) => ({
// 									label: item.equipSimpleName,
// 									value: item.equipModelCode,
// 								})),
// 							};
// 						},
// 					);
// 					equipmentTypeOptions.value = groupedOptions;

// 					if (equipModelCode) {
// 						searchParams.value.equipModelCode = equipModelCode;
// 						const foundOption = groupedOptions
// 							.flatMap((group) => group.options)
// 							.find((option) => option.value === equipModelCode);
// 						if (foundOption) {
// 							searchParams.value.equipSimpleName = foundOption.label;
// 						} else {
// 							ElMessage.warning(
// 								`未找到装备型号编码 ${equipModelCode} 对应的名称`,
// 							);
// 						}
// 					}
// 				} else {
// 					console.error("Failed to fetch equipment models:", response.message);
// 					equipmentTypeOptions.value = [];
// 				}
// 			}

// 			// Trigger query if both parameters are provided
// 			if (equipModelCode && unitCode && unit?.unitPath) {
// 				await loadData();
// 			}
// 		}
// 	} catch (error) {
// 		console.error("Failed to load unit or equipment data:", error);
// 		ElMessage.error("加载单位或装备型号数据失败");
// 	}
// };

onMounted(async () => {

		try {
		const { data } = await unitListApi();
		treeData.value = data;
	} catch (error) {
		console.error("获取单位列表失败:", error);
	}
	// 先尝试恢复查询参数
	const hasRestored = restoreSearchParams();

	// await loadUnitAndEquipData();

	// 如果有恢复的参数，则重新加载数据
	if (hasRestored) {
		loadData();
	}else{
		loadData()
	}
});
watch(selectedUnit, (newVal) => {

if(newVal){
	searchParams.value.unitPath = newVal ? searchParams.value.unitPath : "";
}else{
	searchParams.value.equipModelCode = "";
	searchParams.value.equipCode = "";
}


	// 清空选项列表，强制下次展开时重新触发 fetchEquipmentModels
	equipmentTypeOptions.value = [];
	equipmentCodeOptions.value = [];
});
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

// 监听搜索参数变化并保存
watch(
	[
		() => searchParams.value.equipModelCode,
		() => searchParams.value.equipSimpleName,
		() => searchParams.value.equipCode,
		() => searchParams.value.approvalStatus,
		() => searchParams.value.x,
		() => searchParams.value.xFlag,
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

// 添加 activated 钩子
onActivated(() => {
	console.log("generalhealthassessments.vue activated");

	// 先尝试恢复查询参数
	const hasRestored = restoreSearchParams();

	// 如果有恢复的参数，则重新加载数据
	if (hasRestored) {
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
    const response = await Equipmodelhelth(unitPath)
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

:deep(.el-table__header-wrapper .el-checkbox) {
	display: none;
}
</style>