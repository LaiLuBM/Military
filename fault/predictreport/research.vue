<template>
	<div>
		<el-card>
			<el-row style="margin-top: 15px"> </el-row>
			<el-row :gutter="20" align="middle">
				<el-col :span="5">
					<el-form>
						<el-form-item label="单位/分队" prop="unitCode">
						                <el-tree-select v-model="selectedUnit" :data="treeData" :props="treeProps" node-key="unitCode"
						                  placeholder="请选择单位（可搜索）" filterable clearable style="width: 100%" check-strictly
						                  @change="handleUnitSelect" />
						              </el-form-item>
					</el-form>
				</el-col>
				<el-col :span="5">
					<el-form>
						<el-form-item label="装备型号" v-model="searchParams.equipModel">
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
				<el-col :span="5">
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
			</el-row>
			<el-row :gutter="20" align="middle">
				<el-col :span="10">
					<el-form>
						<el-form-item label="报告内容">
							<el-checkbox-group v-model="checkList">
								<el-checkbox label="故障预测" value="故障预测" />
								<el-checkbox label="故障诊断" value="故障诊断" />
								<el-checkbox label="送修预测" value="送修预测" />
							</el-checkbox-group>
						</el-form-item>
					</el-form>
				</el-col>
				<el-col :span="5">
					<el-form>
						<el-form-item label="报告状态">
							<el-select
								v-model="searchParams.reportStatus"
								placeholder="请选择"
							>
								<el-option
									v-for="item in options"
									:key="item.value"
									:label="item.label"
									:value="item.value"
								/>
							</el-select>
						</el-form-item>
					</el-form>
				</el-col>
				<el-col :span="5" style="text-align: right">
					<el-form>
						<el-form-item>
							<el-button type="primary" class="ml" @click="loadDataWithSave"
								>查询</el-button
							>
						</el-form-item>
					</el-form>
				</el-col>
			</el-row>
			<el-row style="margin-top: 10px"> </el-row>
			<el-row>
				<el-table
					:data="dataList"
					:loading="loading"
					border
					:header-cell-style="headerCellStyle"
				>
					<el-table-column
						label="装备统一编码"
						prop="equipCode"
						align="center"
						:show-overflow-tooltip="true"
					></el-table-column>
					<el-table-column
						label="装备型号"
						prop="equipSimpleName"
						align="center"
						:show-overflow-tooltip="true"
					></el-table-column>
					<el-table-column
						label="报告生成时间"
						prop="reportGenerationTime"
						align="center"
						:show-overflow-tooltip="true"
					></el-table-column>
					<el-table-column
						label="报告状态"
						prop="reportStatus"
						align="center"
						:show-overflow-tooltip="true"
					></el-table-column>
					<el-table-column
						label="报告包含内容"
						prop="reportContent"
						align="center"
						:show-overflow-tooltip="true"
					></el-table-column>
					<el-table-column label="操作" align="center" :show-overflow-tooltip="true">
						<template #default="scope">
							<el-button
								type="primary"
								text
								@click="
									handleDetail(
										scope.row.equipCode,
										scope.row.faultPredictionType,
										scope.row.diagnoseId,
										scope.row.repairId,
										scope.row.reportGenerationTime,
										scope.row.predictId,
										scope.row.predictIdDetail,
										scope.row.reportId,
									)
								"
								>详情</el-button
							>
							<el-button
								type="danger"
								text
								@click="reportdelate(scope.row.reportId)"
								>删除</el-button
							>
							<template v-if="scope.row.reportStatus === '待审核'">
								<el-button
									type="success"
									text
									@click="submit(scope.row.reportId)"
									>提交审核</el-button
								>
							</template>
							<template v-if="scope.row.reportStatus === '已核准'">
								<el-button
									type="success"
									text
									@click="publishreport(scope.row.reportId)"
									>发布报告</el-button
								>
							</template>
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
		</el-card>
	</div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, onActivated } from "vue";
import { ElMessage, ElTree } from "element-plus";
import { useHttp } from "@/hooks/reporthttp";
import { useRouter } from "vue-router";
import { unitListApi } from "@/api/fault/unitlist";
import { equipmodelApi } from "@/api/fault/equipmodel";
import { deleteReport, submitreview, publish } from "@/api/fault/report";

const selectedUnit = ref("");
const time = ref<[Date, Date] | []>([]);
const checkList = ref<string[]>([]);

watch(
	checkList,
	(newVal) => {
		searchParams.value.reportContent = newVal?.join(",");
	},
	{ deep: true },
);

watch(
	time,
	(newVal) => {
		if (newVal && newVal.length === 2) {
			// 手动构建 YYYY-MM-DD HH:mm:ss 格式
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

// 日期格式化函数
const formatDateToString = (date: Date): string => {
	const year = date.getFullYear();
	const month = String(date.getMonth() + 1).padStart(2, "0");
	const day = String(date.getDate()).padStart(2, "0");
	const hours = String(date.getHours()).padStart(2, "0");
	const minutes = String(date.getMinutes()).padStart(2, "0");
	const seconds = String(date.getSeconds()).padStart(2, "0");

	return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};

const options = [
	{ value: "待审核", label: "待审核" },
	{ value: "审核中", label: "审核中" },
	{ value: "已核准", label: "已核准" },
	{ value: "已发布", label: "已发布" },
];

const submit = async (reportId: number) => {
	try {
		const res = await submitreview(reportId);
		if (res.code === 200) {
			ElMessage.success("提交成功");
			loadDataWithSave();
		}
	} catch (error) {
		console.error(error);
	}
};

const publishreport = async (reportId: number) => {
	try {
		const res = await publish(reportId);
		if (res.code === 200) {
			ElMessage.success("提交成功");
			loadDataWithSave();
		}
	} catch (error) {
		console.error(error);
	}
};

interface SearchType {
	equipCode: string;
	equipModel: string;
	unitCode: string;
	unitPath: string;
	current: number;
	size: number;
	endTime: string;
	startTime: string;
	reportStatus: string;
	reportContent: string;
}

const searchParams = ref<SearchType>({
	equipModel: "",
	equipCode: "",
	unitCode: "",
	current: 1,
	size: 10,
	reportStatus: "",
	endTime: "",
	unitPath: "",
	startTime: "",
	reportContent: "",
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
	backgroundColor: "#F8F8F8", // 表头背景颜色
	color: "black", // 表头字体颜色
	fontWeight: "bold", // 表头字体加粗
};

const {
	dataList,
	loading,
	loadData: originalLoadData,
	pageInfo,
	handleSizeChange,
	handleCurrentChange,
	totals,
} = useHttp("/queryReports", searchParams);

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
			equipModel: searchParams.value.equipModel,
			unitCode: searchParams.value.unitCode,
			current: searchParams.value.current,
			size: searchParams.value.size,
			endTime: searchParams.value.endTime,
			startTime: searchParams.value.startTime,
			reportStatus: searchParams.value.reportStatus,
			reportContent: searchParams.value.reportContent,
		},
		time: time.value,
		checkList: checkList.value,
		pageInfo: {
			current: pageInfo.current,
			size: pageInfo.size,
		},
		equipmentTypeOptions: equipmentTypeOptions.value,
		dataList: dataList.value,
		totals: totals.value,
	};

	safeSessionStorage.set("researchQueryState", JSON.stringify(state));
	console.log("查询状态已保存到 sessionStorage");
};

// 从 sessionStorage 恢复查询状态
const restoreQueryState = () => {
	const saved = safeSessionStorage.get("researchQueryState");
	if (saved) {
		try {
			const state = JSON.parse(saved);
			console.log("从 sessionStorage 恢复查询状态:", state);

			// 恢复基本状态
			selectedUnit.value = state.selectedUnit || "";
			checkList.value = state.checkList || [];

			// 恢复时间选择器状态
			if (state.time && state.time.length === 2) {
				time.value = state.time.map((dateStr: string) => new Date(dateStr));
			} else {
				time.value = [];
			}

			// 恢复搜索参数
			if (state.searchParams) {
				searchParams.value.equipCode = state.searchParams.equipCode || "";
				searchParams.value.equipModel = state.searchParams.equipModel || "";
				searchParams.value.unitCode = state.searchParams.unitCode || "";
				searchParams.value.current = state.searchParams.current || 1;
				searchParams.value.size = state.searchParams.size || 10;
				searchParams.value.endTime = state.searchParams.endTime || "";
				searchParams.value.startTime = state.searchParams.startTime || "";
				searchParams.value.reportStatus = state.searchParams.reportStatus || "";
				searchParams.value.reportContent =
					state.searchParams.reportContent || "";
			}

			// 恢复分页信息
			if (state.pageInfo) {
				pageInfo.current = state.pageInfo.current || 1;
				pageInfo.size = state.pageInfo.size || 10;
			}

			// 恢复装备类型选项
			equipmentTypeOptions.value = state.equipmentTypeOptions || [];

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
	safeSessionStorage.remove("researchQueryState");
	console.log("sessionStorage 中的查询状态已清除");
};

// 包装原有的 loadData 函数，保存状态
const loadDataWithSave = async () => {
	// 添加校验逻辑
	if (checkList.value.length === 0) {
		ElMessage.warning("请至少勾选一项报告内容！");
		return;
	}
	if (searchParams.value.reportStatus === "") {
		ElMessage.warning("请选择报告状态！");
		return;
	}

	saveQueryState();
	await originalLoadData();
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
		if (searchParams.value.unitCode) {
			await loadEquipModelOptions();
		}

		// 如果有恢复的表格数据，跳过数据加载
		if (dataList.value.length === 0 && searchParams.value.unitCode) {
			// 如果没有恢复表格数据但查询条件有效，自动加载数据
			console.log("查询条件有效，自动加载表格数据");
			await loadDataWithSave();
		}
	} catch (error) {
		console.error("Failed to fetch city list:", error);
		ElMessage.error("获取单位列表失败");
	}
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
		// 需要从已选中的单位中获取 unitPath
		const selectedNode = findNodeByUnitCode(
			treeData.value,
			parseInt(searchParams.value.unitCode),
		);
		if (!selectedNode) {
			console.error("无法找到选中的单位节点");
			return;
		}

		const response = await equipmodelApi(selectedNode.unitPath.toString());
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
			equipmentTypeOptions.value = [];
		}
	} catch (error) {
		console.error("请求装备型号数据失败:", error);
		equipmentTypeOptions.value = [];
	}
};

// 辅助函数：在树形数据中查找节点
const findNodeByUnitCode = (nodes: Tree[], unitCode: number): Tree | null => {
	for (const node of nodes) {
		if (node.unitCode === unitCode) {
			return node;
		}
		if (node.childrenUnits && node.childrenUnits.length > 0) {
			const found = findNodeByUnitCode(node.childrenUnits, unitCode);
			if (found) {
				return found;
			}
		}
	}
	return null;
};

// 处理装备型号选择变化
const handleEquipModelChange = (value: string) => {
	console.log("选择的装备型号:", value);
	searchParams.value.equipModel = value;
	saveQueryState(); // 保存状态
};
const _deprecated_handleNodeClick = async (nodeData: Tree) => {
	selectedUnit.value = nodeData.unitSimpleName;
	// 重置相关参数
	searchParams.value.equipCode = "";
	const unitPath = nodeData.unitPath.toString();
	searchParams.value.unitPath = unitPath;
	//   selectedUnit.value = nodeData.unitSimpleName;
	//   searchParams.value.unitCode = nodeData.unitCode.toString();
	//   searchParams.value.equipCode = ''; // 清空装备统一编码条件
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
	searchParams.value.equipCode = "";
	// 清空选项列表，强制下次展开时重新触发 fetchEquipmentModels
	equipmentTypeOptions.value = [];
	equipmentCodeOptions.value = [];
});
// const _deprecated_handleNodeClick = async (nodeData: Tree) => {
//   selectedUnit.value = nodeData.unitSimpleName;
//   searchParams.value.unitCode = nodeData.unitCode.toString();
//   searchParams.value.equipCode = ''; // 清空装备统一编码条件

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
//       equipmentTypeOptions.value = [];
//     }
//   } catch (error) {
//     console.error('请求装备型号数据失败:', error);
//     equipmentTypeOptions.value = [];
//   }
// };

const reportdelate = async (reportId: number) => {
	try {
		const params = { reportId: reportId };
		const response = await deleteReport(params);
		if (response.code === 200) {
			ElMessage.success("报告删除成功");
			await loadDataWithSave();
		} else {
			ElMessage.error("删除失败: " + (response.message || "未知错误"));
		}
	} catch (error) {
		console.error("Error deleting task:", error);
		ElMessage.error("删除失败，请稍后重试");
	}
};

const router = useRouter();

const handleDetail = (
	equipCode: string | null,
	faultPredictionType: string | null,
	diagnoseId: number | null,
	repairId: number | null,
	reportGenerationTime: string | null,
	predictId: number | null,
	predictIdDetail: number | null,
	reportId: number | null,
) => {
	// 验证关键参数
	if (equipCode == null || equipCode === "") {
		ElMessage.error("装备统一编码不能为空");
		return;
	}

	// 将 null 转换为字符串 "null" 以明确传递
	const queryParams = {
		equipCode: equipCode ?? "null",
		faultPredictionType: faultPredictionType ?? "null",
		diagnoseId: diagnoseId != null ? diagnoseId.toString() : "null",
		repairId: repairId != null ? repairId.toString() : "null",
		predictId: predictId != null ? predictId.toString() : "null",
		reportGenerationTime: reportGenerationTime ?? "null",
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

// 监听搜索参数变化，保存状态
watch(
	() => searchParams.value,
	(newVal) => {
		saveQueryState();
	},
	{ deep: true },
);

// 监听时间选择器变化，保存状态
watch(
	() => time.value,
	(newVal) => {
		saveQueryState();
	},
	{ deep: true },
);

// 监听复选框变化，保存状态
watch(
	() => checkList.value,
	(newVal) => {
		saveQueryState();
	},
	{ deep: true },
);

// 监听表格数据变化，保存状态
watch(
	() => dataList.value,
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