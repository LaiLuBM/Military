<template><div>
    <el-row :span="24" style="margin-bottom: 20px;" v-if="showTaskForm">
        自动评估设置：
        <el-button type="primary" @click="handleSaveTask">保存</el-button>
        <el-button type="primary" @click="handleBack">返回</el-button>
    </el-row>
    <el-row :span="24" v-if="showTaskForm">
        <el-col :span="6">
            <el-form-item label="任务名称：">
                <el-input v-model="taskName" placeholder="请输入任务名称" class="mb" style="width: 240px"
                    @input="logTaskName"></el-input>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-form-item label="时间设置：" style="display: flex; align-items: center;">
                每周
                <el-select v-model="selectedTime" placeholder="" style="width: 80px; margin: 0 4px;" clearable>
                    <el-option v-for="item in timeOptions" :key="item.value" :label="item.label" :value="item.value" />
                </el-select>
                <el-input v-model="assessTaskHour" placeholder="" class="mb"
                    style="width: 40px; margin: 0 4px;"></el-input>：
                <el-input v-model="assessTaskMinute" placeholder="" class="mb"
                    style="width: 40px; margin: 0 4px;"></el-input>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-form-item label="适用装备统一编码：">
                <el-select v-model="selectedEquipModel" placeholder="请选择装备统一编码" style="width: 240px" clearable multiple>
                    <el-option v-for="item in equipModelOptions" :key="item.equipModelCode"
                        :label="item.equipSimpleName" :value="item.equipModelCode" />
                </el-select>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-form-item label="适用装备范围：">
                <el-select v-model="selectedUnit" placeholder="请选择单位" style="width: 240px" clearable multiple>
                    <el-option v-for="item in selectedUnitOptions" :key="item.unitCode" :label="item.unitSimpleName"
                        :value="item.unitCode" />
                </el-select>
            </el-form-item>
        </el-col>
    </el-row>
    <el-row :span="24" v-if="showTaskForm">
        <el-col :span="6">
            <el-form-item label="生效时间：">
                <el-date-picker v-model="effectiveTime" type="datetime" placeholder="请选择生效时间"
                    format="YYYY-MM-DD HH:mm:ss" date-format="MMM DD, YYYY" time-format="HH:mm"
                    value-format="YYYY-MM-DD HH:mm:ss" />
                <!-- <el-input v-model="effectiveTime" placeholder="生效时间(yyyy-mm-dd hh:mm:ss)" class="mb"
                    style="width: 240px"></el-input> -->
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-form-item label="失效时间：">
                <el-date-picker v-model="expirationTime" type="datetime" placeholder="请选择失效时间"
                    format="YYYY-MM-DD HH:mm:ss" date-format="MMM DD, YYYY" time-format="HH:mm"
                    value-format="YYYY-MM-DD HH:mm:ss" />
                <!-- <el-input v-model="expirationTime" placeholder="失效时间(yyyy-mm-dd hh:mm:ss)" class="mb"
                    style="width: 240px"></el-input> -->
            </el-form-item>
        </el-col>
    </el-row>
    <el-row :span="24" style="margin-bottom: 20px;">
        自动评估任务查询：
    </el-row>
    <el-row :span="24">
        <el-col :span="6">
            <el-form-item label="适用装备统一编码：">
                <el-select v-model="queryEquipModel" placeholder="请选择装备统一编码" style="width: 240px" clearable multiple>
                    <el-option v-for="item in equipModelOptions" :key="item.equipModelCode"
                        :label="item.equipSimpleName" :value="item.equipModelCode" />
                </el-select>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-form-item label="适用装备范围：">
                <el-select v-model="queryUnit" placeholder="请选择单位" style="width: 240px" clearable multiple>
                    <el-option v-for="item in queryUnitOptions" :key="item.unitCode" :label="item.unitSimpleName"
                        :value="item.unitCode" />
                </el-select>
            </el-form-item>
        </el-col>
        <el-col :span="6">
            <el-button type="primary" @click="handleQuery">查询</el-button>
        </el-col>
    </el-row>
    <el-row :span="24" style="margin-bottom: 20px;">
        健康自动化评估任务列表：
        <el-button type="primary" @click="handleAddTask">新增</el-button>
    </el-row>
    <el-row :span="24">
        <el-table :data="tableData" border :header-cell-style="headerCellStyle" class="mt-el-table">
            <el-table-column type="index" width="80" label="序号" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="assessTaskName" label="任务名称" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="executionTime" label="任务执行时间" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="assessTaskEffectiveTime" label="任务生效时间" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="assessTaskInvalidTime" label="任务失效时间" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="equipSimpleName" label="适用装备统一编码" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="unitSimpleName" label="适用单位范围" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column label="是否启用" class-name="shadow-label" align="center" width="100" :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-switch v-model="scope.row.isEnabled" :active-value="1" :inactive-value="0"
                        @change="handleSwitchChange(scope.row)" />
                </template>
            </el-table-column>
            <el-table-column label="操作" class-name="operation-column shadow-label" align="center" width="300" :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-button type="primary" text @click="handleDeleteTask(scope.row.assessTaskId)">删除</el-button>
                    <el-button type="primary" text @click="handleEditTask(scope.row)">编辑</el-button>
                    <el-button type="primary" text @click="handleShowHistory(scope.row.assessTaskId)">历史运行情况</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-row>
    <el-row :span="24" style="margin-top: 20px;" v-if="showHistory">
        历史运行情况列表：
    </el-row>
    <el-row :span="24" v-if="showHistory">
        <el-table :data="historyTableData" border :header-cell-style="headerCellStyle" class="mt-el-table">
            <el-table-column type="index" width="80" label="序号" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="assessTaskName" label="任务名称" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="predictTime" label="任务执行时间" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column prop="assessTaskStatus" label="任务状态" class-name="shadow-label" align="center"  :show-overflow-tooltip="true"/>
            <el-table-column label="通用化健康评估总装备数/成功评估数/失败评估数" class-name="shadow-label" align="center" width="400" :show-overflow-tooltip="true">
                <template #default="{ row }">
                    {{ row.genalHealthNumber }}/{{ row.genalHealthSuccesNumber }}/{{ row.genalHealthFailuresNumber }}
                    <el-button type="primary" text @click="generalhandle(row.equipModelCode, row.unitCode)">
                        查看详情
                    </el-button>
                </template>
            </el-table-column>
            <el-table-column label="精细化健康评估总装备数/成功评估数/失败评估数" class-name="shadow-label" align="center" width="400" :show-overflow-tooltip="true">
                <template #default="{ row }">
                    {{ row.repairHealthNumber }}/{{ row.repairHealthSuccesNumber }}/{{ row.repairHealthFailuresNumber }}
                    <el-button type="primary" text @click="fininghandle(row.equipModelCode, row.unitCode)">
                        查看详情
                    </el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-row>
</div></template>

<script lang="ts" setup>
import { ref, onMounted, watch } from 'vue';
import { queryEquipModelAndUnit, taskList, queryTaskList, getNewAssessTaskName, addTask, deleteTask, updateTask, autoHealthAssessHistory, updateIsEnabled } from '@/api/fault/autohealth';
import { ElMessage } from 'element-plus';
import { useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';

// Reactive variables
const taskName = ref('');
const selectedTime = ref('');
const effectiveTime = ref('');
const expirationTime = ref('');
const assessTaskHour = ref('');
const assessTaskMinute = ref('');
const queryUnit = ref<string[]>([]);
const selectedEquipModel = ref<string[]>([]);
const queryEquipModel = ref<string[]>([]);
const selectedUnit = ref<string[]>([]);
const equipModelOptions = ref<{ equipModelCode: string; equipSimpleName: string }[]>([]);
const selectedUnitOptions = ref<{ unitCode: string; unitSimpleName: string }[]>([]);
const queryUnitOptions = ref<{ unitCode: string; unitSimpleName: string }[]>([]);
const apiData = ref<any[]>([]);
const tableData = ref<any[]>([]);
const showTaskForm = ref(false);
const isEditMode = ref(false);
const currentTaskId = ref<number | null>(null);
const showHistory = ref(false);
const historyTableData = ref<any[]>([]);
const currentHistoryTaskId = ref<number | null>(null);
const timeOptions = [
    { label: '一', value: '1' },
    { label: '二', value: '2' },
    { label: '三', value: '3' },
    { label: '四', value: '4' },
    { label: '五', value: '5' },
    { label: '六', value: '6' },
    { label: '日', value: '7' },
];

// Header style
const headerCellStyle = {
    backgroundColor: '#F8F8F8', // 表头背景颜色
    color: 'black', // 表头字体颜色
    fontWeight: 'bold' // 表头字体加粗
}
// Handle back button click to hide the task form
const handleBack = () => {
    showTaskForm.value = false;
};

// Log taskName on input change
const logTaskName = () => {
    console.log('taskName updated to:', taskName.value);
};

const handleSwitchChange = async (row: any) => {
    try {
        const params = {
            assessTaskId: row.assessTaskId,
            isEnabled: row.isEnabled, // 0 或 1
        };
        const response = await updateIsEnabled(params);
        if (response.code === 200) {
            ElMessage.success('任务状态更新成功');
            // 可选：刷新任务列表
            const taskResponse = await taskList();
            if (taskResponse.code === 200 && taskResponse.data) {
                tableData.value = processTaskData(taskResponse.data);
            } else {
                ElMessage.error('无法刷新任务列表');
            }
        } else {
            ElMessage.error('更新任务状态失败: ' + (response.message || '未知错误'));
            // 回滚状态
            row.isEnabled = row.isEnabled === 1 ? 0 : 1;
        }
    } catch (error) {
        console.error('Error updating task status:', error);
        ElMessage.error('更新任务状态失败，请稍后重试');
        // 回滚状态
        row.isEnabled = row.isEnabled === 1 ? 0 : 1;
    }
};
// Handle save task button click
const handleSaveTask = async () => {
    if (!taskName.value) {
        ElMessage.error('任务名称不能为空');
        return;
    }
    if (!selectedTime.value) {
        ElMessage.error('请选择每周执行时间');
        return;
    }
    if (!assessTaskHour.value || !assessTaskMinute.value) {
        ElMessage.error('请输入任务执行时间（小时和分钟）');
        return;
    }
    if (!effectiveTime.value) {
        ElMessage.error('生效时间不能为空');
        return;
    }
    if (!expirationTime.value) {
        ElMessage.error('失效时间不能为空');
        return;
    }
    if (!selectedEquipModel.value.length) {
        ElMessage.error('请选择适用装备统一编码');
        return;
    }
    if (!selectedUnit.value.length) {
        ElMessage.error('请选择适用装备范围');
        return;
    }

    const hour = parseInt(assessTaskHour.value);
    const minute = parseInt(assessTaskMinute.value);
    if (isNaN(hour) || hour < 0 || hour > 23) {
        ElMessage.error('小时必须在0-23之间');
        return;
    }
    if (isNaN(minute) || minute < 0 || minute > 59) {
        ElMessage.error('分钟必须在0-59之间');
        return;
    }

    const params = {
        assessTaskDay: parseInt(selectedTime.value),
        assessTaskEffectiveTime: effectiveTime.value,
        assessTaskHour: hour,
        assessTaskInvalidTime: expirationTime.value,
        assessTaskMinute: minute,
        assessTaskName: taskName.value,
        equipModelCode: selectedEquipModel.value.join(','), // 多选值用逗号连接
        unitCode: selectedUnit.value.join(','),
        isEnabled: 1,
    };

    try {
        let response;
        if (isEditMode.value && currentTaskId.value !== null) {
            response = await updateTask({ ...params, assessTaskId: currentTaskId.value });
        } else {
            response = await addTask(params);
        }
        console.log('Save response:', response);
        if (response.code === 200) {
            ElMessage.success(isEditMode.value ? '任务更新成功' : '任务保存成功');
            const taskResponse = await taskList();
            console.log('taskList response:', taskResponse);
            if (taskResponse.code === 200 && taskResponse.data) {
                tableData.value = processTaskData(taskResponse.data);
                console.log('tableData after save:', tableData.value); // 调试日志
            } else {
                ElMessage.error('无法刷新任务列表');
                tableData.value = [];
            }
            resetForm();
        } else {
            ElMessage.error(`${isEditMode.value ? '更新' : '保存'}失败: ${response.message || '未知错误'}`);
        }
    } catch (error) {
        console.error(`Error ${isEditMode.value ? 'updating' : 'saving'} task:`, error);
        ElMessage.error(`${isEditMode.value ? '更新' : '保存'}失败，请稍后重试`);
    }
};

const router = useRouter();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;

const generalhandle = (equipModelCode: string, unitCode: string,) => {
    // addTab("通用化健康评估", "/healthevaluate/generalhealthassessments", "Warning");
    // setCurrentTab("通用化健康评估", "/healthevaluate/generalhealthassessments");
    console.log('generalhandle', equipModelCode, unitCode);
    router.push({
        path: "/healthevaluate/generalhealth",
        query: {
            equipModelCode: equipModelCode,
            unitCode: unitCode
        }
    });
};
const fininghandle = (equipModelCode: string, unitCode: string,) => {
    // addTab("精细化健康评估", "/healthevaluate/refinedhealthassessments", "Warning");
    // setCurrentTab("精细化健康评估", "/healthevaluate/refinedhealthassessments");
    router.push({
        path: "/healthevaluate/fininghealth",
        query: {
            equipModelCode: equipModelCode,
            unitCode: unitCode
        }
    });
};

// Handle add task button click
const handleAddTask = async () => {
    try {
        const response = await getNewAssessTaskName();
        if (response.code === 200 && response.data) {
            taskName.value = response.data;
            console.log('New taskName set to:', taskName.value);
            ElMessage.success('成功获取新任务名称');
            selectedTime.value = '';
            assessTaskHour.value = '';
            assessTaskMinute.value = '';
            effectiveTime.value = '';
            expirationTime.value = '';
            selectedEquipModel.value = [];
            selectedUnit.value = [];
            isEditMode.value = false;
            currentTaskId.value = null;
            showTaskForm.value = true;
        } else {
            ElMessage.error('获取任务名称失败: ' + (response.message || '未知错误'));
        }
    } catch (error) {
        console.error('Error fetching new task name:', error);
        ElMessage.error('获取任务名称失败，请稍后重试');
    }
};

// Handle edit task button click
const handleEditTask = (task: any) => {
    // Set equipment model first
    selectedEquipModel.value = task.equipModelCode
        ? task.equipModelCode.split(',')
        : [];

    // Update unit options based on equipment model, but ensure all task units are included
    updateUnitOptions(task.equipModelCode, task.unitCode, selectedUnitOptions);

    // Set other form fields
    taskName.value = task.assessTaskName;
    selectedTime.value = task.assessTaskDay.toString();
    assessTaskHour.value = task.assessTaskHour.toString();
    assessTaskMinute.value = task.assessTaskMinute.toString();
    effectiveTime.value = task.assessTaskEffectiveTime;
    expirationTime.value = task.assessTaskInvalidTime;

    // Set unit codes (handle comma-separated values)
    const unitCodes = task.unitCode ? task.unitCode.split(',').filter((code: string) => code) : [];
    selectedUnit.value = unitCodes.length ? unitCodes : [];

    // Validate unit codes
    const validUnits = selectedUnitOptions.value.map((unit) => unit.unitCode);
    const invalidUnits = unitCodes.filter((code: string) => !validUnits.includes(code));
    if (invalidUnits.length) {
        console.warn('Invalid unit codes during edit:', invalidUnits);
        // Optionally, still show warning but allow editing
        ElMessage.warning(`部分单位代码无效: ${invalidUnits.join(', ')}，请检查是否需要更新单位选择`);
    }

    isEditMode.value = true;
    currentTaskId.value = task.assessTaskId;
    showTaskForm.value = true;
};

// Reset form fields and state
const resetForm = () => {
    taskName.value = '';
    selectedTime.value = '';
    assessTaskHour.value = '';
    assessTaskMinute.value = '';
    effectiveTime.value = '';
    expirationTime.value = '';
    selectedEquipModel.value = [];
    selectedUnit.value = [];
    showTaskForm.value = false;
    isEditMode.value = false;
    currentTaskId.value = null;
};

// Handle delete task button click
const handleDeleteTask = async (assessTaskId: number) => {
    try {
        const params = { assessTaskId };
        const response = await deleteTask(params);
        if (response.code === 200) {
            ElMessage.success('任务删除成功');
            const taskResponse = await taskList();
            if (taskResponse.code === 200 && taskResponse.data) {
                tableData.value = processTaskData(taskResponse.data);
            } else {
                ElMessage.error('无法刷新任务列表');
            }
        } else {
            ElMessage.error('删除失败: ' + (response.message || '未知错误'));
        }
    } catch (error) {
        console.error('Error deleting task:', error);
        ElMessage.error('删除失败，请稍后重试');
    }
};

// Process task data
const processTaskData = (tasks: any[]) => {
    return tasks.map((task: any) => {
        // 处理装备统一编码（多选，可能为逗号分隔字符串）
        const equipModelCodes = task.equipModelCode ? task.equipModelCode.split(',').filter((code: string) => code) : [];
        const equipSimpleNames = equipModelCodes
            .map((code: string) => {
                const model = apiData.value.find((item) => item.equipModelCode === code);
                return model ? model.equipSimpleName : code;
            })
            .join(', ');

        // 处理单位
        const unitCodes = task.unitCode ? task.unitCode.split(',').filter((code: string) => code) : [];
        const unitSimpleNames = unitCodes
            .map((code: string) => {
                const unit = apiData.value
                    .flatMap((item) => item.unitList)
                    .find((u: any) => u.unitCode === code);
                return unit ? unit.unitSimpleName : code;
            })
            .join(', ');

        // 处理执行时间
        const dayMap = ['每周一', '每周二', '每周三', '每周四', '每周五', '每周六', '每周日'];
        const day = task.assessTaskDay >= 1 && task.assessTaskDay <= 7 ? dayMap[task.assessTaskDay - 1] : '未知';
        const hour = task.assessTaskHour.toString().padStart(2, '0');
        const minute = task.assessTaskMinute.toString().padStart(2, '0');
        const executionTime = `${day} ${hour}:${minute}`;

        return {
            ...task,
            equipSimpleName: equipSimpleNames, // 添加 equipSimpleName
            unitSimpleName: unitSimpleNames,
            executionTime,
        };
    });
};

// Fetch equipment model and unit data on mount
onMounted(async () => {
    try {
        // 获取装备统一编码和单位数据
        const equipResponse = await queryEquipModelAndUnit();
        console.log('equipResponse:', equipResponse);
        if (equipResponse.code === 200 && equipResponse.data) {
            apiData.value = equipResponse.data;
            console.log('apiData:', apiData.value);
            equipModelOptions.value = equipResponse.data.map((item: any) => ({
                equipModelCode: item.equipModelCode,
                equipSimpleName: item.equipSimpleName,
            }));
            updateUnitOptions(null, null, selectedUnitOptions);
            console.log('selectedUnitOptions after init:', selectedUnitOptions.value);
            updateUnitOptions(null, null, queryUnitOptions);
        } else {
            ElMessage.error('无法加载装备统一编码和单位数据');
        }

        // 获取任务列表数据
        const taskResponse = await taskList();
        console.log('taskList response:', taskResponse);
        if (taskResponse.code === 200 && taskResponse.data) {
            tableData.value = processTaskData(taskResponse.data);
            console.log('tableData after init:', tableData.value);
        } else {
            ElMessage.error('无法加载任务列表');
            tableData.value = [];
        }
    } catch (error) {
        console.error('Error fetching data:', error);
        ElMessage.error('数据加载失败，请稍后重试');
    }
});
// Handle query button click
const handleQuery = async () => {
    try {
        const params: {
            equipModelCode?: string;
            unitCode?: string
        } = {};

        // 处理多选装备统一编码
        if (queryEquipModel.value.length) {
            params.equipModelCode = queryEquipModel.value.join(',');
        }

        // 处理多选单位
        if (queryUnit.value) {
            params.unitCode = queryUnit.value.join(',');
        }

        const response = await queryTaskList(params);
        if (response.code === 200 && response.data) {
            tableData.value = processTaskData(response.data);
        } else {
            ElMessage.warning('当前查询条件无结果');
            return;
        }
    } catch (error) {
        ElMessage.error('查询失败，请稍后重试');
        return;
    }
};

// Update unit options based on equipment model
const updateUnitOptions = (equipModelCode: string | null | string[], taskUnitCodes: string | null, optionsRef: any) => {
    optionsRef.value = [];
    const unitMap = new Map<string, { unitCode: string; unitSimpleName: string }>();

    // Add all units from apiData if no equipment model is selected or for broader compatibility
    apiData.value.forEach((item: any) => {
        if (item.unitList && item.unitList.length) {
            item.unitList.forEach((unit: any) => {
                unitMap.set(unit.unitCode, {
                    unitCode: unit.unitCode,
                    unitSimpleName: unit.unitSimpleName || '未知单位',
                });
            });
        }
    });

    // If editing, ensure task's unit codes are included in options
    if (taskUnitCodes) {
        const unitCodes = taskUnitCodes.split(',').filter((code: string) => code);
        unitCodes.forEach((code) => {
            if (!unitMap.has(code)) {
                // Add missing units to options (e.g., for units no longer tied to selected equipment models)
                unitMap.set(code, {
                    unitCode: code,
                    unitSimpleName: `单位 ${code} (历史数据)`, // Fallback name for unknown units
                });
            }
        });
    }

    // If equipment models are selected, filter units to include only those related to the models
    if (equipModelCode) {
        const modelCodes = Array.isArray(equipModelCode) ? equipModelCode : [equipModelCode];
        const filteredUnitMap = new Map<string, { unitCode: string; unitSimpleName: string }>();
        modelCodes.forEach((code) => {
            const model = apiData.value.find((item: any) => item.equipModelCode === code);
            if (model && model.unitList) {
                model.unitList.forEach((unit: any) => {
                    filteredUnitMap.set(unit.unitCode, {
                        unitCode: unit.unitCode,
                        unitSimpleName: unit.unitSimpleName || '未知单位',
                    });
                });
            }
        });
        // Merge filtered units with task units to ensure all are included
        unitMap.forEach((value, key) => {
            filteredUnitMap.set(key, value);
        });
        optionsRef.value = Array.from(filteredUnitMap.values());
    } else {
        optionsRef.value = Array.from(unitMap.values());
    }

    console.log('Updated optionsRef:', optionsRef.value);
};

const handleShowHistory = async (assessTaskId: number) => {
    try {
        currentHistoryTaskId.value = assessTaskId;
        const response = await autoHealthAssessHistory({ assessTaskId });

        if (response.code === 200 && response.data) {
            historyTableData.value = response.data;
            showHistory.value = true;
        } else {
            ElMessage.error('获取历史记录失败: ' + (response.message || '未知错误'));
            showHistory.value = false;
        }
    } catch (error) {
        console.error('Error fetching history:', error);
        ElMessage.error('获取历史记录失败，请稍后重试');
        showHistory.value = false;
    }
};

watch([queryEquipModel, queryUnit], ([newEquipModel, newUnit]) => {
    showHistory.value = false;
    historyTableData.value = [];

    // 当查询条件清空时（即两个数组都为空），加载 taskList 数据
    if (newEquipModel.length === 0 && newUnit.length === 0) {
        (async () => {
            try {
                const taskResponse = await taskList();
                if (taskResponse.code === 200 && taskResponse.data) {
                    tableData.value = processTaskData(taskResponse.data);
                    console.log('tableData after clearing query:', tableData.value);
                } else {
                    ElMessage.error('无法加载任务列表');
                    tableData.value = [];
                }
            } catch (error) {
                console.error('Error fetching task list:', error);
                ElMessage.error('加载任务列表失败，请稍后重试');
            }
        })();
    }
});

// Watch equipment model selections to update unit options
watch(selectedEquipModel, (newModel) => {
    console.log('selectedEquipModel changed:', newModel);
    updateUnitOptions(newModel, null, selectedUnitOptions); // Add null as second argument
    if (!isEditMode.value) {
        selectedUnit.value = [];
    }
});
watch(queryEquipModel, (newModel) => {
    updateUnitOptions(newModel, null, queryUnitOptions); // Add null as second argument
    queryUnit.value = [];
});
</script>

<style scoped>
.mt-el-table {
    margin-top: 20px;
}
</style>