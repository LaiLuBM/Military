<template><div>
  <el-row>
    <el-col :span="2" style="margin-top: 5px;margin-left: 200px;">规则编辑：</el-col>
    <el-col :span="2" style="margin-left: -60px;">
      <el-button type="primary" @click="Save">保存</el-button>
    </el-col>
    <el-col :span="2" style="margin-left: -65px;">
      <el-button @click="goBack">返回</el-button>
    </el-col>
  </el-row>
  <div class="wrap">
    <el-row style="margin-top: 20px;">
      <el-col :span="6" style="margin-right: 20px;">
        <el-form>
          <el-form-item label="规则名称：">
            <el-input v-model="formData.ruleName" placeholder="请输入规则名称" />
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="6" style="margin-right: 20px;">
        <el-form>
          <el-form-item label="建议类型：">
            <el-select style="width: 80%;" v-model="searchParams.suggestionType">
              <el-option v-for="item in SuggestionOptions" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="6">
        <el-form>
          <el-form-item label="适用装备：">
            <el-select multiple placeholder="请选择适用装备" style="width: 80%;" v-model="formData.applicableObject">
              <el-option-group v-for="group in equipmentModelOptions" :key="group.label" :label="group.label">
                <el-option v-for="option in group.options" :key="option.value" :label="option.label"
                  :value="option.value" />
              </el-option-group>
            </el-select>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="12">
        <el-form-item label="生效时间：" prop="effectiveDate">
          <el-date-picker type="date" placeholder="请选择日期" format="YYYY-MM-DD" value-format="YYYY-MM-DD"
            style="width: 45%;" v-model="formData.effectiveDate" />
          <el-time-picker placeholder="请选择时间" format="HH:mm:ss" value-format="HH:mm:ss" style="width: 45%;"
            v-model="formData.effectiveTime" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="失效时间：" prop="expirationDate">
          <el-date-picker type="date" placeholder="请选择日期" format="YYYY-MM-DD" value-format="YYYY-MM-DD"
            style="width: 50%;" v-model="formData.expirationDate" />
          <el-time-picker placeholder="请选择时间" format="HH:mm:ss" value-format="HH:mm:ss" style="width: 50%;"
            v-model="formData.expirationTime" />
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div>匹配以下条件:</div>
      </el-col>
    </el-row>
    <el-row v-for="(condition, index) in conditions" :key="index" style="margin-bottom: 15px;">
      <el-col :span="4">
        <el-select v-if="index > 0" placeholder="且" style="width: 30%;margin-left: 120px;"
          v-model="condition.logicalOperator">
          <el-option v-for="item in logicalOperatorOptions" :key="item.value" :label="item.label" :value="item.value" />
        </el-select>
      </el-col>
      <el-col :span="6">
        <el-select style="width: 80%;" v-model="condition.field">
          <el-option v-for="item in conditionOptions" :key="item.value" :label="item.label" :value="item.value" />
        </el-select>
      </el-col>
      <el-col :span="6">
        <el-select placeholder="等于" style="width: 80%;" v-model="condition.operator">
          <el-option v-for="item in operatorOptions" :key="item.value" :label="item.label" :value="item.value" />
        </el-select>
      </el-col>
      <el-col :span="6">
        <el-input placeholder="请输入具体数值" style="width: 80%;" v-model="condition.value" />
      </el-col>
      <el-col :span="2" style="margin-top: 5px;">
        <el-icon v-if="index === 0" class="clickable-icon" color="#409efc" @click="addCondition">
          <CirclePlus />
        </el-icon>
        <el-icon v-else class="clickable-icon" @click="removeCondition(index)">
          <Delete />
        </el-icon>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="4">
        <el-form>
          <el-form-item label="建议结论：">
            <el-select v-model="formData.suggestionConclusion">
              <el-option v-for="item in conclusionOptions" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="4" style="margin-left: 15px;">
        <el-form>
          <el-form-item label="影响范围：">
            <el-select v-model="formData.impactScope">
              <el-option v-for="item in impactOptions" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="16"></el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div>关联任务:</div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="16" style="margin-top: 15px;margin-left: 80px;">
        <el-input type="textarea" maxlength="200" v-model="formData.relatedTask" placeholder="请输入关联任务" show-word-limit
          style="border: 1px gray solid;" />
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div>详细建议:</div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="16" style="margin-top: 15px;margin-left: 80px;">
        <el-input type="textarea" maxlength="200" v-model="formData.detailedSuggestion" placeholder="请输入详细建议"
          show-word-limit style="border: 1px gray solid;" />
      </el-col>
    </el-row>
  </div>
</div></template>

<script lang="ts" setup>
import { ref, onMounted, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { CirclePlus, Delete } from '@element-plus/icons-vue';
import { RepairSaveApi, RepairUpdateApi } from '@/api/fault/repair-conditions-query';
import { equipmodelApi } from '@/api/fault/equipmodel';
import { ElMessage } from 'element-plus';

const route = useRoute();
const router = useRouter();
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;
const ruleId = ref<number | null>(null);
const formData = ref({
  ruleName: '',
  applicableObject: [] as string[],
  effectiveDate: '',
  effectiveTime: '',
  expirationDate: '',
  expirationTime: '',
  suggestionConclusion: '',
  impactScope: '',
  relatedTask: '',
  detailedSuggestion: '',
});

const goBack = () => {
  // addTab("使修维保建议规则", "/healthevaluate/singleevaluate", "Files");
  // setCurrentTab("使修维保建议规则", "/healthevaluate/singleevaluate");
  window.history.back()
};

const SuggestionOptions = ref([
  { label: '视情维修建议', value: '视情维修建议' },
  { label: '延寿使用建议', value: '延寿使用建议' },
  { label: '作战运用建议', value: '作战运用建议' },
]);


interface EquipmentModelOption {
  label: string;
  value: string;
}

interface EquipmentModelGroup {
  label: string;
  options: EquipmentModelOption[];
}
const equipmentModelOptions = ref<EquipmentModelGroup[]>([]);

const conditionOptions = ref([
  { label: '中修间隔期内发动机累计消耗摩托小时', value: 'motor_hours' },
  { label: '大修后大修间隔期内累计行驶里程', value: 'mileage_after_overhaul' },
  { label: 'wifi健康指数', value: 'wifi_health_index' },
]);

const operatorOptions = ref([
  { label: '等于', value: 'equals' },
  { label: '大于', value: 'greater' },
  { label: '小于', value: 'less' },
  { label: '大于等于', value: 'greater_equal' },
  { label: '小于等于', value: 'less_equal' },
]);

const conclusionOptions = ref([
  { label: '小修', value: '小修' },
  { label: '大修', value: '大修' },
]);

const impactOptions = ref([
  { label: 'wifi', value: 'wifi' },
]);

const conditions = ref([
  { logicalOperator: '', field: '', operator: 'equals', value: '' },
  { logicalOperator: 'and', field: '', operator: 'equals', value: '' },
  { logicalOperator: 'and', field: '', operator: 'equals', value: '' },
  { logicalOperator: 'and', field: '', operator: 'equals', value: '' },
]);

const operatorMap: { [key: string]: string } = {
  equals: '等于',
  greater: '大于',
  less: '小于',
  greater_equal: '大于等于',
  less_equal: '小于等于',
};

const fieldMap: { [key: string]: string } = {
  motor_hours: '中修间隔期内发动机累计消耗摩托小时',
  mileage_after_overhaul: '大修后大修间隔期内累计行驶里程',
  wifi_health_index: 'wifi健康指数',
};

const logicalOperatorOptions = ref([
  { label: '且', value: 'and' },
  { label: '或', value: 'or' },
]);

interface SearchType {
  suggestionType: string;
}
const searchParams = ref<SearchType>({
  suggestionType: '视情维修建议',
});

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

const storeSessionData = () => {
  if (formData.value.ruleName) {
    safeSessionStorage.set('regularEditRuleName', formData.value.ruleName);
  }
  if (formData.value.applicableObject.length > 0) {
    safeSessionStorage.set('regularEditApplicableObject', formData.value.applicableObject.join(','));
  }
  if (ruleId.value !== null) {
    safeSessionStorage.set('regularEditRuleId', ruleId.value.toString());
  }
  if (searchParams.value.suggestionType) {
    safeSessionStorage.set('regularEditSuggestionType', searchParams.value.suggestionType);
  }
};

const addCondition = () => {
  conditions.value.push({
    logicalOperator: 'and',
    field: '',
    operator: 'equals',
    value: '',
  });
};

const removeCondition = (index: number) => {
  if (conditions.value.length > 1 && index > 0) {
    conditions.value.splice(index, 1);
  } else {
    ElMessage.warning('至少保留一个条件！');
  }
};

const parseConditionContent = (conditionContent: string) => {
  console.log('Parsing conditionContent:', conditionContent);
  const conditionArray = conditionContent.split(' 且 ').map(cond => cond.trim());
  console.log('Split conditions:', conditionArray);

  const operatorMap: { [key: string]: string } = {
    '大于等于': 'greater_equal',
    '小于等于': 'less_equal',
    '大于': 'greater',
    '小于': 'less',
    '等于': 'equals',
  };

  const fieldMap: { [key: string]: string } = {
    '中修间隔期内发动机累计消耗摩托小时': 'motor_hours',
    '大修后大修间隔期内累计行驶里程': 'mileage_after_overhaul',
    'wifi健康指数': 'wifi_health_index',
  };

  const parsedConditions = conditionArray.map((cond, index) => {
    let field = '';
    let operator = '';
    let value = '';
    for (const op of Object.keys(operatorMap).sort((a, b) => b.length - a.length)) {
      if (cond.includes(op)) {
        const parts = cond.split(op).map(part => part.trim());
        if (parts.length === 2) {
          field = parts[0];
          operator = operatorMap[op];
          value = parts[1].replace('W公里', '0000');
          break;
        }
      }
    }
    field = fieldMap[field] || field;
    console.log('Parsed condition:', { logicalOperator: index > 0 ? 'and' : '', field, operator, value });
    const logicalOperator=index===0?'':(conditionContent.includes('或')?'or':'and');
    return {
      logicalOperator,
      field,
      operator,
      value,
    };
  });
  console.log('Parsed conditions:', parsedConditions);
  conditions.value = parsedConditions.length > 0 ? parsedConditions : [
    { logicalOperator: '', field: '', operator: 'equals', value: '' },
  ];
};

interface SaveParams {
  id: number;
  ruleName: string;
  applicableObject: string;
  suggestionType: string;
  effectiveTime: string | null;
  expiryTime: string | null;
  conditionContent: string;
  priority: string;
  impactScope: string;
  detailedSuggestion: string;
  relatedTask: string;
  isEnabled: boolean;
  [key: string]: any;
}

const validateConditions = () => {
  // 检查是否有至少一个有效条件
  const validConditions = conditions.value.filter(
    (cond) => cond.field && cond.operator && cond.value
  );
  if (validConditions.length === 0) {
    return { valid: false, message: "请至少添加一个完整的匹配条件" };
  }

  // 检查相同字段的条件是否逻辑矛盾
  const fieldGroups: { [key: string]: any[] } = {};
  validConditions.forEach((cond, index) => {
    if (!fieldGroups[cond.field]) {
      fieldGroups[cond.field] = [];
    }
    fieldGroups[cond.field].push({ ...cond, index });
  });

  for (const field in fieldGroups) {
    const group = fieldGroups[field];
    if (group.length > 1) {
      // 对相同字段的条件进行检查
      for (let i = 0; i < group.length; i++) {
        const cond1 = group[i];
        const value1 = Number(cond1.value); // 转换为数值
        if (isNaN(value1)) {
          return {
            valid: false,
            message: `条件值无效：${fieldMap[cond1.field] || cond1.field} 的值 "${cond1.value}" 不是有效数字`,
          };
        }
        for (let j = i + 1; j < group.length; j++) {
          const cond2 = group[j];
          const value2 = Number(cond2.value); // 转换为数值
          if (isNaN(value2)) {
            return {
              valid: false,
              message: `条件值无效：${fieldMap[cond2.field] || cond2.field} 的值 "${cond2.value}" 不是有效数字`,
            };
          }

          // 检查逻辑矛盾，例如 >= X 且 <= X
          if (
            (cond1.operator === "greater_equal" &&
              cond2.operator === "less_equal" &&
              value1 === value2) ||
            (cond1.operator === "less_equal" &&
              cond2.operator === "greater_equal" &&
              value1 === value2)
          ) {
            return {
              valid: false,
              message: `条件矛盾：${fieldMap[cond1.field] || cond1.field} ${operatorMap[cond1.operator]} ${value1} 且 ${operatorMap[cond2.operator]} ${value2}`,
            };
          }

          // 检查范围无效，例如 > X 且 < X 或 >= X 且 < X
          if (
            // > X 且 < X
            (cond1.operator === "greater" &&
              cond2.operator === "less" &&
              value1 >= value2) ||
            (cond1.operator === "less" &&
              cond2.operator === "greater" &&
              value1 <= value2) ||
            // >= X 且 < X
            (cond1.operator === "greater_equal" &&
              cond2.operator === "less" &&
              value1 >= value2) ||
            (cond1.operator === "less" &&
              cond2.operator === "greater_equal" &&
              value1 <= value2)
          ) {
            return {
              valid: false,
              message: `条件无效：${fieldMap[cond1.field] || cond1.field} ${operatorMap[cond1.operator]} ${value1} 且 ${operatorMap[cond2.operator]} ${value2}`,
            };
          }
        }
      }
    }
  }

  return { valid: true, message: "" };
};

const Save = async () => {
  const id = route.query.id ? Number(route.query.id) : null;
  // 验证条件
  const conditionValidation = validateConditions();
  if (!conditionValidation.valid) {
    ElMessage.error(conditionValidation.message);
    return;
  }

  if (id) {
    try {
      if (!id) {
        ElMessage.error('无法更新：缺少ID');
        return;
      }

      const formatDateTime = (date: string, time: string) => {
        if (!date || !time) return null;
        return `${date}T${time}`;
      };

      const generateConditionContent = () => {
        const operatorMap: { [key: string]: string } = {
          equals: '等于',
          greater: '大于',
          less: '小于',
          greater_equal: '大于等于',
          less_equal: '小于等于',
        };

        const fieldMap: { [key: string]: string } = {
          motor_hours: '中修间隔期内发动机累计消耗摩托小时',
          mileage_after_overhaul: '大修后大修间隔期内累计行驶里程',
          wifi_health_index: 'wifi健康指数',
        };

        return conditions.value
          .filter(cond => cond.field && cond.operator && cond.value)
          .map((cond, index) => {
            const logicalOperator = index > 0 && cond.logicalOperator ? (cond.logicalOperator==="or"?'或':'且'):''
            const field = fieldMap[cond.field] || cond.field;
            const operator = operatorMap[cond.operator] || cond.operator;
            return `${logicalOperator}${field} ${operator} ${cond.value}`;
          })
          .join('');
      };

      const Saveparams: SaveParams = {
        id: id,
        ruleName: formData.value.ruleName,
        applicableObject: formData.value.applicableObject.join(','),
        suggestionType: searchParams.value.suggestionType,
        effectiveTime: formatDateTime(formData.value.effectiveDate, formData.value.effectiveTime),
        expiryTime: formatDateTime(formData.value.expirationDate, formData.value.expirationTime),
        conditionContent: generateConditionContent(),
        priority: formData.value.suggestionConclusion,
        impactScope: formData.value.impactScope,
        detailedSuggestion: formData.value.detailedSuggestion,
        relatedTask: formData.value.relatedTask,
        isEnabled: true,
      };

      const requiredFields = [
        'ruleName',
        'applicableObject',
        'suggestionType',
        'effectiveTime',
        'expiryTime',
        'conditionContent',
        'priority',
        'impactScope',
        'detailedSuggestion',
        'relatedTask',
      ];
      for (const field of requiredFields) {
        if (!Saveparams[field] || (Array.isArray(Saveparams[field]) && Saveparams[field].length === 0)) {
          ElMessage.error(`请填写所有必填字段: ${field}`);
          return;
        }
      }

      const res = await RepairUpdateApi(Saveparams);
      if (res.code === 200) {
        ElMessage.success('数据更新成功');
        safeSessionStorage.set('dataUpdated', 'true');
        storeSessionData(); // 保存 suggestionType 到 sessionStorage
        goBack();
      } else {
        ElMessage.error('数据更新失败: ' + (res.message || '未知错误'));
      }
    } catch (error) {
      console.error('Error updating data:', error);
      ElMessage.error('数据更新失败，请稍后重试');
    }
  } else {
    try {
      const formatDateTime = (date: string, time: string) => {
        if (!date || !time) return null;
        return `${date}T${time}`;
      };

      const generateConditionContent = () => {


        return conditions.value
          .filter(cond => cond.field && cond.operator && cond.value)
          .map((cond, index) => {
            const logicalOperator = index > 0 && cond.logicalOperator ? ' 且 ' : '';
            const field = fieldMap[cond.field] || cond.field;
            const operator = operatorMap[cond.operator] || cond.operator;
            return `${logicalOperator}${field} ${operator} ${cond.value}`;
          })
          .join('');
      };

      const Saveparams = {
        ruleName: formData.value.ruleName,
        applicableObject: formData.value.applicableObject.join(','),
        suggestionType: searchParams.value.suggestionType,
        effectiveTime: formatDateTime(formData.value.effectiveDate, formData.value.effectiveTime),
        expiryTime: formatDateTime(formData.value.expirationDate, formData.value.expirationTime),
        conditionContent: generateConditionContent(),
        priority: formData.value.suggestionConclusion,
        impactScope: formData.value.impactScope,
        detailedSuggestion: formData.value.detailedSuggestion,
        relatedTask: formData.value.relatedTask,
        isEnabled: true,
      };

      type SaveParamsKeys = keyof typeof Saveparams;
      const requiredFields: SaveParamsKeys[] = [
        'ruleName',
        'applicableObject',
        'suggestionType',
        'effectiveTime',
        'expiryTime',
        'conditionContent',
        'priority',
        'impactScope',
        'detailedSuggestion',
        'relatedTask',
      ];

      for (const field of requiredFields) {
        if (!Saveparams[field] || (Array.isArray(Saveparams[field]) && Saveparams[field].length === 0)) {
          ElMessage.error(`请填写所有必填字段: ${field}`);
          return;
        }
      }

      const res = await RepairSaveApi(Saveparams);
      if (res.code === 200) {
        ElMessage.success('数据保存成功');
        safeSessionStorage.set('dataUpdated', 'true');
        storeSessionData(); // 保存 suggestionType 到 sessionStorage
        goBack();
      } else {
        ElMessage.error('数据保存失败: ' + (res.message || '未知错误'));
      }
    } catch (error) {
      console.error('Error saving data:', error);
      ElMessage.error('数据保存失败，请稍后重试');
    }
  }
};

const isEditMode = ref(false);

const loadEquipmentModels = async () => {
  const defaultUnitPath = '/11000/11001/';
  try {
    const response = await equipmodelApi(defaultUnitPath);
    console.log('装备统一编码数据:', response);

    if (response.code === 200 && response.data?.categories) {
      const groupedOptions = Object.entries(response.data.categories).map(([category, items]) => {
        if (!Array.isArray(items)) {
          console.error(`类别 ${category} 的数据不是数组:`, items);
          return { label: category, options: [] };
        }
        const options = items.map((item: any) => ({
          label: item.equipSimpleName,
          value: item.equipSimpleName,
        }));
        return {
          label: category,
          options,
        };
      });
      equipmentModelOptions.value = groupedOptions;
      console.log('生成的 equipmentModelOptions:', equipmentModelOptions.value);
    } else {
      console.error('数据格式错误或请求失败:', response.message);
      equipmentModelOptions.value = [];
      ElMessage.error('获取装备统一编码失败: ' + response.message);
    }
  } catch (error) {
    console.error('请求装备统一编码数据失败:', error);
    equipmentModelOptions.value = [];
    ElMessage.error('获取装备统一编码失败，请稍后重试');
  }
};

onMounted(() => {
  loadEquipmentModels()
  const {
    ruleName,
    applicableObject,
    suggestionType,
    effectiveTime,
    expiryTime,
    conditionContent,
    priority,
    impactScope,
    detailedSuggestion,
    relatedTask,
  } = route.query;

  isEditMode.value = !!ruleName;

  console.log('Received query:', route.query);

  if (ruleName) formData.value.ruleName = String(ruleName);
  if (suggestionType) searchParams.value.suggestionType = String(suggestionType);
  if (applicableObject) {
    const applicableObjectArray = String(applicableObject).split(',').map(item => item.trim());
    formData.value.applicableObject = applicableObjectArray;
  }
  if (effectiveTime) {
    const [date, time] = String(effectiveTime).split(' ');
    formData.value.effectiveDate = date;
    formData.value.effectiveTime = time;
  }
  if (expiryTime) {
    const [date, time] = String(expiryTime).split(' ');
    formData.value.expirationDate = date;
    formData.value.expirationTime = time;
  }
  if (priority) formData.value.suggestionConclusion = String(priority);
  if (impactScope) formData.value.impactScope = String(impactScope);
  if (detailedSuggestion) formData.value.detailedSuggestion = String(detailedSuggestion);
  if (relatedTask) formData.value.relatedTask = String(relatedTask);

  if (conditionContent) {
    parseConditionContent(String(conditionContent));
  }
  storeSessionData();
});

watch(() => route.query, (newQuery) => {
  const {
    ruleName,
    applicableObject,
    suggestionType,
    effectiveTime,
    expiryTime,
    conditionContent,
    priority,
    impactScope,
    detailedSuggestion,
    relatedTask,
  } = newQuery;

  isEditMode.value = !!ruleName;

  console.log('Query changed:', newQuery);

  if (ruleName) formData.value.ruleName = String(ruleName);
  if (suggestionType) searchParams.value.suggestionType = String(suggestionType);
  if (applicableObject) {
    const applicableObjectArray = String(applicableObject).split(',').map(item => item.trim());
    formData.value.applicableObject = applicableObjectArray;
  }
  if (effectiveTime) {
    const [date, time] = String(effectiveTime).split(' ');
    formData.value.effectiveDate = date;
    formData.value.effectiveTime = time;
  }
  if (expiryTime) {
    const [date, time] = String(expiryTime).split(' ');
    formData.value.expirationDate = date;
    formData.value.expirationTime = time;
  }
  if (priority) formData.value.suggestionConclusion = String(priority);
  if (impactScope) formData.value.impactScope = String(impactScope);
  if (detailedSuggestion) formData.value.detailedSuggestion = String(detailedSuggestion);
  if (relatedTask) formData.value.relatedTask = String(relatedTask);
  if (conditionContent) {
    parseConditionContent(String(conditionContent));
  }

  storeSessionData();
});
</script>

<style scoped>
.el-form-item {
  margin-bottom: 20px;
}

.wrap {
  width: 1200px;
  margin: auto;
}

.clickable-icon {
  cursor: pointer;
}

:deep(.el-select-group__title) {
  font-weight: bold;
  color: #409eff;
}

:deep(.el-select-group .el-select-dropdown__item) {
  padding-left: 30px;
}
</style>