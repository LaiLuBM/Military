<template>
	<div>
		<el-card>
			<el-row style="margin-top: 15px"></el-row>
			<el-row :gutter="20" align="middle">
				<el-col :span="4">
					<el-form>
						<el-form-item label="单位/分队" prop="unitCode">
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
							v-model="searchParams.equipModelCode"
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
				<el-col :span="5">
					<el-form>
						<el-form-item label="装备统一编码">
							<el-input
								placeholder="装备统一编码"
								v-model="searchParams.equipCode"
								@keyup.enter="loadDataWithSave"
							>
							</el-input>
						</el-form-item>
					</el-form>
				</el-col>
				<el-col :span="7">
					<el-form>
						<el-form-item label="日期范围">
							<div class="block">
								<el-date-picker
									v-model="time"
									type="datetimerange"
									start-placeholder="开始时间"
									end-placeholder="结束时间"
									format="YYYY-MM-DD HH:mm:ss"
								/>
							</div>
						</el-form-item>
					</el-form>
				</el-col>
				<el-col :span="4">
					<el-form>
						<el-form-item>
							<el-button type="primary" class="ml" @click="loadDataWithSave"
								>查询</el-button
							>
							<el-button type="primary" class="ml" @click="handleDetail"
								>生成报告</el-button
							>
						</el-form-item>
					</el-form>
				</el-col>
			</el-row>
			<el-row style="margin-top: 10px"></el-row>
			<el-row>
				<el-table
					:data="dataList"
					:loading="loading"
					border
					:header-cell-style="headerCellStyle"
					@selection-change="handleSelectionChange"
				>
					<el-table-column type="selection" width="55" :show-overflow-tooltip="true"></el-table-column>
					<el-table-column
						label="装备统一编码"
						prop="equip.equipCode"
						align="center"
					 :show-overflow-tooltip="true"></el-table-column>
					<el-table-column
						label="装备型号"
						prop="equip.equipModel.equipSimpleName"
						align="center"
					 :show-overflow-tooltip="true"></el-table-column>
					<el-table-column
						label="故障预测类型"
						prop="reportManagement.faultPredictionType"
						align="center"
					 :show-overflow-tooltip="true"></el-table-column>
					<el-table-column label="预测日期" align="center" :show-overflow-tooltip="true">
						<template #default="scope">
							{{
								scope.row?.reportManagement?.faultPredictionType?.includes(
									"通用化",
								)
									? scope.row?.equip.equipMalf?.predictTime
									: scope.row?.equip.equipMalfDetail?.predictTime || "无"
							}}
						</template>
					</el-table-column>
					<el-table-column
						label="是否已生成报告"
						prop="reportManagement.reportGenerated"
						align="center"
					 :show-overflow-tooltip="true"></el-table-column>
					<el-table-column label="操作" align="center" width="400" :show-overflow-tooltip="true">
						<template #default="scope">
							<el-button
								type="primary"
								text
								@click="showPredictionDetailDialog(scope.row)"
								>预测详情</el-button
							>
							<el-button type="warning" text @click="DiagnoseReport(scope.row)"
								>添加诊断报告</el-button
							>
							<el-button type="danger" text @click="RepairReport(scope.row)"
								>添加送修建议报告</el-button
							>
						</template>
					</el-table-column>
				</el-table>
				<el-col :span="24">
					<el-pagination
						class="fr mt mb"
						v-model:current-page="pageInfo.current"
						v-model:page-size="pageInfo.size"
						:page-sizes="[10, 20, 30, 40]"
						layout="sizes, prev, pager, next, jumper"
						:total="totals"
						@size-change="handleSizeChangeWithSave"
						@current-change="handleCurrentChangeWithSave"
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
			<el-dialog
				title="预测详情"
				v-model="predictionDialogVisible"
				width="30%"
				center
				:before-close="handlePredictionDialogClose"
				draggable
			>
				<el-row :gutter="20" class="dialog-content" justify="center">
					<el-col :span="24" style="text-align: center">
						<el-button
							v-if="
								currentRow?.reportManagement?.faultPredictionType.includes(
									'通用化',
								)
							"
							type="primary"
							@click="viewGeneralPredictionDetail"
							>通用化预测详情</el-button
						>
						<el-button
							v-if="
								currentRow?.reportManagement?.faultPredictionType.includes(
									'精细化',
								)
							"
							type="primary"
							@click="viewDetailedPredictionDetail"
							>精细化预测详情</el-button
						>
					</el-col>
				</el-row>
				<template #footer>
					<span class="dialog-footer">
						<el-button @click="predictionDialogVisible = false">关闭</el-button>
					</span>
				</template>
			</el-dialog>
			<el-dialog
				title="精细化诊断"
				v-model="dialogVisible"
				width="50%"
				center
				:before-close="handleDialogClose"
				draggable
			>
				<el-row :gutter="20" class="dialog-content">
					<el-col :gutter="24">
						<el-table
							:data="diagnosisHistory"
							border
							:header-cell-style="headerCellStyle"
							class="diagnosis-table"
							@selection-change="handleDiagnosisSelectionChange"
						>
							<el-table-column type="selection" width="55"  :show-overflow-tooltip="true"/>
							<el-table-column
								label="装备统一编码"
								prop="equipCode"
								align="center"
								width="180"
							 :show-overflow-tooltip="true"></el-table-column>
							<el-table-column
								label="装备型号"
								prop="equipSimpleName"
								align="center"
								width="180"
							 :show-overflow-tooltip="true"></el-table-column>
							<el-table-column
								label="诊断日期"
								prop="diagnoseTime"
								align="center"
								width="250"
							 :show-overflow-tooltip="true"></el-table-column>
							<el-table-column label="操作" align="center" :show-overflow-tooltip="true">
								<template #default="scope">
									<el-button
										type="primary"
										text
										@click="viewDiagnosisDetail(scope.row)"
										>诊断详情</el-button
									>
								</template>
							</el-table-column>
						</el-table>
					</el-col>
					<el-col :span="24" class="pagination-container">
						<el-pagination
							class="fr"
							v-model:current-page="dialogParams.current"
							v-model:page-size="dialogParams.size"
							:page-sizes="[5, 10, 15, 20]"
							layout="sizes, prev, pager, next, jumper"
							:total="dialogTotals"
							@size-change="handleDialogSizeChange"
							@current-change="handleDialogCurrentChange"
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
				<template #footer>
					<span class="dialog-footer">
						<el-button @click="dialogVisible = false">取消</el-button>
						<el-button type="primary" @click="submitDiagnoseReport"
							>确定</el-button
						>
					</span>
				</template>
			</el-dialog>
			<el-dialog
				title="送修预测"
				v-model="repairDialogVisible"
				width="50%"
				center
				:before-close="handleRepairDialogClose"
				draggable
			>
				<el-row :gutter="20" class="dialog-content">
					<el-col :span="24">
						<el-table
							:data="repairHistory"
							border
							:header-cell-style="headerCellStyle"
							class="diagnosis-table"
							@selection-change="handleRepairSelectionChange"
						>
							<el-table-column type="selection" width="55"  :show-overflow-tooltip="true"/>
							<el-table-column
								label="装备统一编码"
								prop="equipCode"
								align="center"
								width="180"
							 :show-overflow-tooltip="true"></el-table-column>
							<el-table-column
								label="装备型号"
								prop="equipSimpleName"
								align="center"
								width="180"
							 :show-overflow-tooltip="true"></el-table-column>
							<el-table-column
								label="诊断日期"
								prop="predictTime"
								align="center"
								width="250"
							 :show-overflow-tooltip="true"></el-table-column>
							<el-table-column label="操作" align="center" :show-overflow-tooltip="true">
								<template #default="scope">
									<el-button
										type="primary"
										text
										@click="viewRepairDetail(scope.row)"
										>预测详情</el-button
									>
								</template>
							</el-table-column>
						</el-table>
					</el-col>
					<el-col :span="24" class="pagination-container">
						<el-pagination
							class="fr"
							v-model:current-page="repairDialogParams.current"
							v-model:page-size="repairDialogParams.size"
							:page-sizes="[5, 10, 15, 20]"
							layout="sizes, prev, pager, next, jumper"
							:total="repairTotals"
							@size-change="handleRepairDialogSizeChange"
							@current-change="handleRepairDialogCurrentChange"
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
				<template #footer>
					<span class="dialog-footer">
						<el-button @click="repairDialogVisible = false">取消</el-button>
						<el-button type="primary" @click="submitRepairReport"
							>确定</el-button
						>
					</span>
				</template>
			</el-dialog>
		</el-card>
	</div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, reactive, onActivated } from "vue";
import { ElMessage, ElTree } from "element-plus";
import { useRouter } from "vue-router";
import { useHttp } from "@/hooks/report";
import { unitListApi } from "@/api/fault/unitlist";
import { equipmodelApi } from "@/api/fault/equipmodel";
import {
	addDiagnoseReport,
	addRepairReport,
	morediaghistory,
	morerepairhistory,
	generateReport,
} from "@/api/fault/report";
import { report } from "process";

const selectedUnit = ref("");
const time = ref<[Date, Date] | []>([]);

watch(
	time,
	(newVal) => {
		if (newVal && newVal.length === 2) {
			searchParams.value.startTime = newVal[0]
				? formatDateToString(newVal[0])
				: "";
			searchParams.value.endTime = newVal[1]
				? formatDateToString(newVal[1])
				: "";
			console.log("开始时间:", searchParams.value.startTime);
			console.log("结束时间:", searchParams.value.endTime);
		} else {
			searchParams.value.startTime = "";
			searchParams.value.endTime = "";
		}
	},
	{ deep: true },
);

const formatDateToString = (date: Date): string => {
	const year = date.getFullYear();
	const month = String(date.getMonth() + 1).padStart(2, "0");
	const day = String(date.getDate()).padStart(2, "0");
	const hours = String(date.getHours()).padStart(2, "0");
	const minutes = String(date.getMinutes()).padStart(2, "0");
	const seconds = String(date.getSeconds()).padStart(2, "0");
	return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};

interface SearchType {
	equipCode: string;
	equipModelCode: string;
	unitCode: string;
	current: number;
	size: number;
	endTime: string;
	startTime: string;
	unitPath: string;
}

const searchParams = ref<SearchType>({
	equipModelCode: "",
	equipCode: "",
	unitCode: "",
	current: 1,
	size: 10,
	endTime: "",
	startTime: "",
	unitPath: "",
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

const headerCellStyle = {
	backgroundColor: "#F8F8F8",
	color: "black",
	fontWeight: "bold",
};

const {
	dataList,
	loading,
	loadData,
	pageInfo,
	handleSizeChange,
	handleCurrentChange,
	totals,
} = useHttp("/equipmentPredictionList", searchParams);

// 安全 sessionStorage 操作
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
	remove(key: string): void {
		try {
			sessionStorage.removeItem(key);
		} catch (e) {
			console.error("SessionStorage删除失败:", e);
		}
	},
};

// 保存查询状态到 sessionStorage
const saveQueryState = () => {
	const state = {
		selectedUnit: selectedUnit.value,
		searchParams: {
			equipCode: searchParams.value.equipCode,
			equipModelCode: searchParams.value.equipModelCode,
			unitCode: searchParams.value.unitCode,
			current: searchParams.value.current,
			size: searchParams.value.size,
			endTime: searchParams.value.endTime,
			startTime: searchParams.value.startTime,
			unitPath: searchParams.value.unitPath,
		},
		time: time.value,
		pageInfo: {
			current: pageInfo.current,
			size: pageInfo.size,
		},
		equipmentTypeOptions: equipmentTypeOptions.value,
		selectedRows: selectedRows.value,
		dataList: dataList.value,
		totals: totals.value,
	};

	safeSessionStorage.set("generateReportQueryState", JSON.stringify(state));
	console.log("生成报告查询状态已保存到 sessionStorage");
};

// 从 sessionStorage 恢复查询状态
const restoreQueryState = () => {
	const saved = safeSessionStorage.get("generateReportQueryState");
	if (saved) {
		try {
			const state = JSON.parse(saved);
			console.log("从 sessionStorage 恢复生成报告查询状态:", state);

			// 恢复基本状态
			selectedUnit.value = state.selectedUnit || "";

			// 恢复时间选择器状态
			if (state.time && state.time.length === 2) {
				time.value = state.time.map((dateStr: string) => new Date(dateStr));
			} else {
				time.value = [];
			}

			// 恢复搜索参数
			if (state.searchParams) {
				searchParams.value.equipCode = state.searchParams.equipCode || "";
				searchParams.value.equipModelCode =
					state.searchParams.equipModelCode || "";
				searchParams.value.unitCode = state.searchParams.unitCode || "";
				searchParams.value.current = state.searchParams.current || 1;
				searchParams.value.size = state.searchParams.size || 10;
				searchParams.value.endTime = state.searchParams.endTime || "";
				searchParams.value.startTime = state.searchParams.startTime || "";
				searchParams.value.unitPath = state.searchParams.unitPath || "";
			}

			// 恢复分页信息
			if (state.pageInfo) {
				pageInfo.current = state.pageInfo.current || 1;
				pageInfo.size = state.pageInfo.size || 10;
			}

			// 恢复装备类型选项
			equipmentTypeOptions.value = state.equipmentTypeOptions || [];

			// 恢复选中的行
			selectedRows.value = state.selectedRows || [];

			// 恢复表格数据和总数
			if (state.dataList && state.dataList.length > 0) {
				dataList.value = state.dataList;
				totals.value = state.totals || 0;
				console.log("表格数据已从 sessionStorage 恢复");
			}
		} catch (error) {
			console.error("恢复查询状态失败:", error);
			clearQueryState();
		}
	}
};

// 清除保存的查询状态
const clearQueryState = () => {
	safeSessionStorage.remove("generateReportQueryState");
	console.log("sessionStorage 中的查询状态已清除");
};

// 包装原有的 loadData 函数，保存状态
const loadDataWithSave = async () => {
	saveQueryState();
	await loadData();
};

// 包装分页大小改变函数
const handleSizeChangeWithSave = (size: number) => {
	handleSizeChange(size);
	saveQueryState();
};

// 包装当前页改变函数
const handleCurrentChangeWithSave = (page: number) => {
	handleCurrentChange(page);
	saveQueryState();
};

onMounted(async () => {
	try {
		// 先恢复查询状态
		restoreQueryState();

		// 加载单位树形数据
		const { data } = await unitListApi();
		treeData.value = data;

		// 如果已恢复单位选择，加载对应的装备型号数据
		if (searchParams.value.unitPath) {
			await loadEquipModelOptions();
		}

		// 如果有恢复的表格数据，跳过数据加载
		if (dataList.value.length === 0 && searchParams.value.unitPath) {
			// 如果没有恢复表格数据但查询条件有效，自动加载数据
			console.log("查询条件有效，自动加载表格数据");
			await loadData();
		}
	} catch (error) {
		console.error("Failed to fetch unit list:", error);
		ElMessage.error("获取单位列表失败");
	}

	watch(
		dataList,
		() => {
			if (!dataList.value) return;
			dataList.value.forEach((item: any) => {
				if (!item.reportManagement) {
					item.reportManagement = { faultPredictionType: [] };
				} else if (!item.reportManagement.faultPredictionType) {
					item.reportManagement.faultPredictionType = [];
				} else if (
					typeof item.reportManagement.faultPredictionType === "string"
				) {
					item.reportManagement.faultPredictionType =
						item.reportManagement.faultPredictionType
							.split(",")
							.map((t: string) => t.trim());
				}
			});

			// 保存数据到 sessionStorage
			saveQueryState();
		},
		{ deep: true, immediate: true },
	);
});

const equipmentTypeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);
const equipmentCodeOptions = ref<
	{ label: string; options: { label: string; value: string }[] }[]
>([]);

// 加载装备型号选项
const loadEquipModelOptions = async () => {
	try {
		const response = await equipmodelApi(
			searchParams.value.unitPath.toString(),
		);
		console.log("装备型号数据:", response);

		if (response.code === 200 && response.data?.categories) {
			const groupedOptions = Object.entries(response.data.categories).map(
				([category, items]) => {
					if (!Array.isArray(items)) {
						console.error(`类别 ${category} 的数据不是数组:`, items);
						return { label: category, options: [] };
					}

					const options = items.map((item: any) => ({
						label: item.equipSimpleName,
						value: item.equipModelCode,
					}));

					return {
						label: category,
						options,
					};
				},
			);

			equipmentTypeOptions.value = groupedOptions;
			console.log("生成的 equipmentTypeOptions:", equipmentTypeOptions.value);

			// 保存装备型号选项状态
			saveQueryState();
		} else {
			console.error("数据格式错误或请求失败:", response.message);
			ElMessage.error("操作失败");
			equipmentTypeOptions.value = [];
		}
	} catch (error) {
		console.error("请求装备型号数据失败:", error);
		ElMessage.error("请求装备型号数据失败，请稍后重试");
		equipmentTypeOptions.value = [];
	}
};

const handleEquipModelChange = (value: string) => {
	console.log("选择的装备型号:", value);
	searchParams.value.equipModelCode = value;
	saveQueryState(); // 保存状态
};

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
watch(selectedUnit, (newVal) => {
	searchParams.value.unitPath = newVal ? searchParams.value.unitPath : "";
	searchParams.value.equipModelCode = "";
	searchParams.value.equipCode = "";
	// 清空选项列表，强制下次展开时重新触发 fetchEquipmentModels
	equipmentTypeOptions.value = [];
	equipmentCodeOptions.value = [];
});
// const _deprecated_handleNodeClick = async (nodeData: Tree) => {
//   selectedUnit.value = nodeData.unitSimpleName;
//   searchParams.value.unitPath = nodeData.unitPath.toString();
//   searchParams.value.unitCode = nodeData.unitCode.toString();
//   searchParams.value.equipCode = '';

//   // 保存状态
//   saveQueryState();

//   try {
//     const response = await equipmodelApi(nodeData.unitPath.toString());
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

//       // 保存装备型号选项状态
//       saveQueryState();
//     } else {
//       console.error('数据格式错误或请求失败:', response.message);
//       ElMessage.error('操作失败');
//       equipmentTypeOptions.value = [];
//     }
//   } catch (error) {
//     console.error('请求装备型号数据失败:', error);
//     ElMessage.error('请求装备型号数据失败，请稍后重试');
//     equipmentTypeOptions.value = [];
//   }
// };

const router = useRouter();

const selectedRows = ref<any[]>([]);
const handleSelectionChange = (selection: any[]) => {
	selectedRows.value = selection;
	console.log("Selected rows:", selectedRows.value);

	// 保存选中状态
	saveQueryState();
};

const handleDetail = async () => {
	if (selectedRows.value.length === 0) {
		ElMessage.warning("请至少勾选一行数据");
		return;
	}
	const selectedRow = selectedRows.value[0];
	console.log();
	const equipCode = selectedRow.equip.equipCode;
	const predictId = selectedRow.reportManagement?.predictId;
	const reportId = selectedRow.reportManagement?.reportId;
	const diagnoseId = selectedRow.reportManagement?.diagnoseId;
	const repairId = selectedRow.reportManagement?.repairId;
	const faultPredictionType =
		selectedRow.reportManagement?.faultPredictionType || [];
	const reportGenerationTime =
		selectedRow.reportManagement?.reportGenerationTime;
	const predictIdDetail = selectedRow.reportManagement?.predictIdDetail;
	if (!equipCode || !reportId) {
		ElMessage.error("选中的行缺少必要数据(装备统一编码或报告ID)");
		return;
	}
	const params = {
		equipCode: equipCode,
		reportId: reportId,
	};
	console.log("receive params:", params);
	const response = await generateReport(params);
	if (response.data) {
		console.log("receive response:", response.data);
	} else {
		ElMessage.error("未获取到故障报告数据");
	}
	const queryParams = {
		equipCode: equipCode ?? "null",
		faultPredictionType: Array.isArray(faultPredictionType)
			? faultPredictionType.join(",")
			: faultPredictionType || "null",
		diagnoseId: diagnoseId != null ? diagnoseId.toString() : "null",
		repairId: repairId != null ? repairId.toString() : "null",
		reportGenerationTime: reportGenerationTime ?? "null",
		predictId: predictId != null ? predictId.toString() : "null",
		predictIdDetail:
			predictIdDetail != null ? predictIdDetail.toString() : "null",
		reportId: reportId != null ? reportId.toString() : "null",
	};

	console.log("handleDetail triggered with parameters:", queryParams);

	router.push({
		path: "/predictreport/reportdetail",
		query: queryParams,
	});
};

const predictionDialogVisible = ref(false);
const currentRow = ref<any>(null);

const showPredictionDetailDialog = (row: any) => {
	currentRow.value = row;
	if (!currentRow.value.reportManagement) {
		currentRow.value.reportManagement = { faultPredictionType: [] };
	}
	if (
		typeof currentRow.value.reportManagement.faultPredictionType === "string"
	) {
		currentRow.value.reportManagement.faultPredictionType =
			currentRow.value.reportManagement.faultPredictionType
				.split(",")
				.map((t: string) => t.trim());
	} else if (
		!Array.isArray(currentRow.value.reportManagement.faultPredictionType)
	) {
		currentRow.value.reportManagement.faultPredictionType = [];
	}
	predictionDialogVisible.value = true;
};

const handlePredictionDialogClose = () => {
	predictionDialogVisible.value = false;
	currentRow.value = null;
};

const viewGeneralPredictionDetail = () => {
	console.log("查看通用化预测详情:", currentRow.value);
	const equipCode = currentRow.value.equip.equipCode;
	const predictId = currentRow.value.equip.equipMalf.predictId;
	const predictTime = currentRow.value.equip.equipMalf.predictTime;
	if (predictTime) {
		router.push(
			`/predict/generaldetail?equipCode=${equipCode}${
				predictId ? `&predictId=${predictId}` : ""
			}`,
		);
	} else {
		return ElMessage.warning("该装备暂无预测记录");
	}
};

const viewDetailedPredictionDetail = () => {
	console.log("查看精细化预测详情:", currentRow.value);
	const equipCode = currentRow.value.equip.equipCode;
	const predictId = currentRow.value.equip.equipMalfDetail.predictId;
	const predictTime = currentRow.value.equip.equipMalfDetail.predictTime;
	if (predictTime) {
		router.push(
			`/predict/finingdetail?equipCode=${equipCode}${
				predictId ? `&predictId=${predictId}` : ""
			}`,
		);
	} else {
		return ElMessage.warning("该装备暂无预测记录");
	}
};

const dialogVisible = ref(false);
const diagnosisForm = reactive({
	equipCode: "",
	diagnoseId: "",
	reportId: "",
});
const diagnosisHistory = ref<any[]>([]);
const dialogParams = ref({
	equipCode: "",
	current: 1,
	size: 5,
});
const dialogTotals = ref(0);

const loadDiagnosisHistory = async (row: any) => {
	try {
		console.log("Sending moreDiagHistory params:", dialogParams.value);
		const response = await morediaghistory(dialogParams.value);
		console.log("moreDiagHistory response:", response);

		if (response.code === 200) {
			const equipSimpleName = row.equip?.equipModel?.equipSimpleName || "";
			diagnosisHistory.value = (response.data.records || []).map(
				(item: any) => {
					if (!item.equipCode || !item.diagnoseId) {
						console.warn("Invalid diagnosis record:", item);
					}
					return {
						...item,
						equipSimpleName: item.equipSimpleName || equipSimpleName,
					};
				},
			);
			dialogTotals.value = response.data.totalElements || 0;
		} else {
			ElMessage.warning(`获取诊断历史失败: ${response.message || "未知错误"}`);
			diagnosisHistory.value = [];
			dialogTotals.value = 0;
		}
	} catch (error: any) {
		console.error("Error fetching diagnosis history:", error);
		ElMessage.error("获取诊断失败，请稍后重试");
		diagnosisHistory.value = [];
		dialogTotals.value = 0;
	}
};

const DiagnoseReport = async (row: any) => {
	console.log("DiagnoseReport called with row:", row);
	try {
		if (!row.equip?.equipCode) {
			ElMessage.error("装备统一编码缺失，无法获取诊断历史");
			return;
		}

		currentPredictionData.value = row;
		diagnosisForm.equipCode = row.equip.equipCode || "";
		diagnosisForm.diagnoseId = "";
		diagnosisForm.reportId = row.reportManagement?.reportId || "";
		dialogParams.value.equipCode = row.equip.equipCode;
		dialogParams.value.current = 1;
		dialogParams.value.size = 5;

		if (!diagnosisForm.reportId) {
			console.warn("reportId is missing in row:", row);
			ElMessage.warning("报告ID缺失，将尝试加载诊断历史但可能无法提交");
		}

		await loadDiagnosisHistory(row);
		dialogVisible.value = true;
	} catch (error) {
		console.error("DiagnoseReport error:", error);
		ElMessage.warning("打开诊断历史对话框失败");
		diagnosisHistory.value = [];
		dialogTotals.value = 0;
		dialogVisible.value = true;
	}
};

const handleDialogSizeChange = async (size: number) => {
	dialogParams.value.size = size;
	dialogParams.value.current = 1;
	await loadDiagnosisHistory(currentPredictionData.value);
};

const handleDialogCurrentChange = async (current: number) => {
	dialogParams.value.current = current;
	await loadDiagnosisHistory(currentPredictionData.value);
};

const handleDiagnosisSelectionChange = (selection: any[]) => {
	if (selection.length > 0) {
		const selectedRow = selection[0];
		console.log("Selected diagnosis row:", selectedRow);
		diagnosisForm.equipCode = selectedRow.equipCode || "";
		diagnosisForm.diagnoseId = selectedRow.diagnoseId || "";
		if (!selectedRow.equipCode || !selectedRow.diagnoseId) {
			ElMessage.warning("选中的诊断记录缺少 equipCode 或 diagnoseId");
		}
	} else {
		diagnosisForm.equipCode = "";
		diagnosisForm.diagnoseId = "";
		console.log("No diagnosis row selected");
	}
};

const submitDiagnoseReport = async () => {
	try {
		console.log("Submitting diagnose report with:", diagnosisForm);
		if (!diagnosisForm.equipCode) {
			ElMessage.error("请确保选择一条诊断记录（缺少装备统一编码）");
			return;
		}
		if (!diagnosisForm.diagnoseId) {
			ElMessage.error("请确保选择一条诊断记录（缺少诊断ID）");
			return;
		}
		if (!diagnosisForm.reportId) {
			ElMessage.error("请确保选择一条诊断记录（缺少报告ID）");
			return;
		}

		const payload = {
			diagnoseId: diagnosisForm.diagnoseId,
			equipCode: diagnosisForm.equipCode,
			reportId: diagnosisForm.reportId,
		};
		console.log("Sending addDiagnoseReport payload:", payload);
		const response = await addDiagnoseReport(payload);
		if (response.code === 200) {
			ElMessage.success("诊断报告添加成功");
			dialogVisible.value = false;
			loadDataWithSave(); // 使用包装的函数重新加载数据
		} else {
			ElMessage.error(`添加诊断报告失败: ${response.message || "未知错误"}`);
		}
	} catch (error) {
		console.error("添加诊断报告请求失败:", error);
		ElMessage.error("添加诊断报告失败，请稍后重试");
	}
};

const repairDialogVisible = ref(false);
const repairForm = reactive({
	equipCode: "",
	repairId: "",
	reportId: "",
});
const repairHistory = ref<any[]>([]);
const repairDialogParams = ref({
	equipCode: "",
	current: 1,
	size: 5,
});
const repairTotals = ref(0);

const loadRepairHistory = async (row: any) => {
	try {
		console.log("Sending morerepairhistory params:", repairDialogParams.value);
		const response = await morerepairhistory(repairDialogParams.value);
		console.log("Repair history response:", response);

		if (response.code === 200) {
			const equipSimpleName = row.equip?.equipModel?.equipSimpleName || "";
			repairHistory.value = (response.data?.records || []).map((item: any) => ({
				...item,
				equipSimpleName: item.equipSimpleName || equipSimpleName,
			}));
			repairTotals.value = response.data.totalElements || 0;
		} else {
			ElMessage.warning(`获取送修历史失败: ${response.message || "未知错误"}`);
			repairHistory.value = [];
			repairTotals.value = 0;
		}
	} catch (error: any) {
		console.error("Error fetching repair history:", error);
		repairHistory.value = [];
		repairTotals.value = 0;
	}
};

const currentPredictionData = ref<any>(null);

const RepairReport = async (row: any) => {
	console.log("RepairReport called with row:", row);
	try {
		if (!row.equip?.equipCode) {
			ElMessage.error("装备统一编码缺失，无法获取送修历史");
			return;
		}

		currentPredictionData.value = row;
		repairForm.equipCode = row.equip.equipCode || "";
		repairForm.repairId = "";
		repairForm.reportId = row.reportManagement?.reportId || "";
		repairDialogParams.value.equipCode = row.equip.equipCode;
		repairDialogParams.value.current = 1;
		repairDialogParams.value.size = 5;

		await loadRepairHistory(row);
		repairDialogVisible.value = true;
	} catch (error) {
		console.error("RepairReport error:", error);
		ElMessage.warning("打开送修历史对话框失败");
		repairHistory.value = [];
		repairTotals.value = 0;
		repairDialogVisible.value = true;
	}
};

const handleRepairDialogSizeChange = async (size: number) => {
	repairDialogParams.value.size = size;
	repairDialogParams.value.current = 1;
	await loadRepairHistory(currentPredictionData.value);
};

const handleRepairDialogCurrentChange = async (current: number) => {
	repairDialogParams.value.current = current;
	await loadRepairHistory(currentPredictionData.value);
};

const handleRepairSelectionChange = (selection: any[]) => {
	if (selection.length > 0) {
		const selectedRow = selection[0];
		repairForm.equipCode = selectedRow.equipCode || "";
		repairForm.repairId = selectedRow.repairId || "";
	} else {
		repairForm.equipCode = "";
		repairForm.repairId = "";
	}
};

const submitRepairReport = async () => {
	try {
		if (!repairForm.equipCode || !repairForm.repairId || !repairForm.reportId) {
			ElMessage.error("请确保选择一条送修记录");
			return;
		}

		const payload = {
			repairId: repairForm.repairId,
			equipCode: repairForm.equipCode,
			reportId: repairForm.reportId,
		};
		console.log("Sending addDiagnoseReport payload for repair:", payload);
		const response = await addRepairReport(payload);
		if (response.code === 200) {
			ElMessage.success("送修报告添加成功");
			repairDialogVisible.value = false;
			loadDataWithSave(); // 使用包装的函数重新加载数据
		} else {
			ElMessage.error(`添加送修报告失败: ${response.message || "未知错误"}`);
		}
	} catch (error) {
		console.error("添加送修报告请求失败:", error);
		ElMessage.error("添加送修报告失败，请稍后重试");
	}
};

const handleDialogClose = () => {
	dialogVisible.value = false;
	diagnosisHistory.value = [];
	dialogParams.value.equipCode = "";
	dialogTotals.value = 0;
};

const handleRepairDialogClose = () => {
	repairDialogVisible.value = false;
	repairHistory.value = [];
	repairDialogParams.value.equipCode = "";
	repairTotals.value = 0;
};

const viewDiagnosisDetail = (row: any) => {
	const equipCode = row.equipCode;
	const diagnoseId = row.diagnoseId;
	console.log("查看诊断详情:", row);
	router.push(
		`/predict/diagnosedetail?equipCode=${equipCode}${
			diagnoseId ? `&diagnoseId=${diagnoseId}` : ""
		}`,
	);
};

const viewRepairDetail = (row: any) => {
	console.log("查看送修详情:", row);
	const repairId = row.repairId;
	const equipCode = row.equipCode;
	router.push(
		`/predict/Repairdetail?equipCode=${equipCode}${
			repairId ? `&repairId=${repairId}` : ""
		}`,
	);
};

// 监听搜索参数变化，保存状态
watch(
	() => searchParams.value,
	(newVal) => {
		saveQueryState();
	},
	{ deep: true },
);

// 监听选中行的变化，保存状态
watch(
	selectedRows,
	(newVal) => {
		saveQueryState();
	},
	{ deep: true },
);

// 在组件激活时恢复状态
onActivated(() => {
	restoreQueryState();
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
.dialog-content {
	padding: 20px;
}

.diagnosis-table {
	margin-bottom: 20px;
}

.pagination-container {
	display: flex;
	justify-content: flex-end;
}

.dialog-content button {
	margin: 0 10px;
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
</style>