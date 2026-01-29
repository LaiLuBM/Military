<template>
    <div>
        <el-row :gutter="20">
            <el-col :span="1">
                <el-button text @click="goBack" icon="ArrowLeft"
                    style="font-size: 30px; padding: 0;margin: 10px 20px;margin:10px,20px" />
            </el-col>
             <el-col :span="11" class="tree-wrapper">

                <el-input clearable placeholder="搜索节点" v-model="searchQuery" :suffix-icon="Search" class="sear"
                    @input="filterTreeData" />
                    <div class="tree-container" style="position:relative">
                <el-tree :data="filteredTreeData" :props="treeProps" node-key="nodeId" @node-click="handleNodeClick"
                    highlight-current :default-expand-all="false" @node-contextmenu="handleRightClick"></el-tree>
                <div v-if="contextMenu.visible" :style="{ left: contextMenu.x + 'px', top: contextMenu.y + 'px' }"
                    class="context-menu">
                    <el-button @click="addNode" type="primary" >增加下一级节点</el-button>
                    <el-button @click="deleteNode" type="danger">删除</el-button>
                </div>
                </div>
            </el-col>
            <el-col :span="12">
                <div class="sub-tit"><strong>节点详情：</strong></div>
                <div class="sub-tit"><strong>基本信息：</strong></div>
                <el-row>
                    <el-col :span="16">
                        <el-form ref="formRef" :model="formData" :rules="rules" label-width="120px"
                            class="custom-border-form">
                            <!-- 故障树名称 -->
                            <el-form-item label="故障树名称:" prop="treeName" class="border-item">
                                <el-input v-model.trim="formData.treeName" placeholder="请输入故障树名称" clearable
                                    @blur="formData.treeName = formData.treeName?.trim()" />
                            </el-form-item>

                            <!-- 节点名称 -->
                            <el-form-item label="节点名称:" class="border-item">
                                <el-input v-model="formData.nodeName" placeholder="请输入节点名称" clearable />
                            </el-form-item>

                            <!-- 节点类型 -->
                            <el-form-item label="节点类型:" class="border-item">
                                <el-input v-model="formData.nodeType" placeholder="请输入节点类型" clearable />
                            </el-form-item>

                            <!-- 上级节点 -->
                            <el-form-item label="上级节点:" class="border-item">
                                <el-input v-model="formData.parentNode" placeholder="请输入上级节点" readonly />
                            </el-form-item>

                            <!-- 与上层节点关系 -->
                            <el-form-item label="与上层节点关系:" class="border-item">
                                <el-input v-model="formData.relationship" placeholder="请输入关系" clearable />
                            </el-form-item>

                            <!-- 创建时间 -->
                            <el-form-item label="创建时间:" class="border-item">
                                <el-input v-model="formData.createTime" readonly />
                            </el-form-item>

                            <!-- 更新时间 -->
                            <el-form-item label="更新时间:" class="border-item">
                                <el-input v-model="formData.updateTime" readonly />
                            </el-form-item>

                            <!-- 创建人 -->
                            <el-form-item label="创建人:" prop="creatorName" class="border-item">
                                <el-input v-model.trim="formData.creatorName" placeholder="请输入创建人" clearable
                                    @blur="formData.creatorName = formData.creatorName?.trim()" />
                            </el-form-item>
                        </el-form>
                    </el-col>
                    <el-col :span="6" style="margin-left: 10px;">
                        <div>
                        <el-button type="primary" @click="save">保存修改信息</el-button>
                        </div>
                        <!-- 只有状态是“未提交”时才显示提交审核按钮 -->
                        <el-button v-if="currentStatus === '未提交'" type="success" style="margin-top: 10px;"
                            @click="handleSubmit">
                            提交审核
                        </el-button>


                    </el-col>
                </el-row>
                <div class="sub-tit"><strong>故障描述：</strong></div>
                <el-input clearable type="textarea" v-model="formData.description" placeholder="请输入故障描述" show-word-limit
                    style="border: 1px gray solid;" />
                <div class="sub-tit"><strong>解决方案：</strong></div>
                <el-input clearable type="textarea" v-model="formData.solution" placeholder="请输入解决方案" show-word-limit
                    style="border: 1px gray solid;" />
            </el-col>
        </el-row>
        <el-dialog v-model="addNodeDialog" title="新增下一级节点" width="30%">
            <el-form :model="newNodeForm" label-width="100px" :rules="{
                nodeName: [{ required: true, message: '请输入节点名称', trigger: 'blur' }]
            }" ref="addNodeFormRef">
                <el-form-item label="节点名称" prop="nodeName">
                    <el-input v-model="newNodeForm.nodeName" placeholder="请输入节点名称" clearable />
                </el-form-item>
                <el-form-item label="父节点名称" v-if="contextMenu.currentNode">
                    <el-input :value="contextMenu.currentNode.nodeName" disabled />
                </el-form-item>
                <el-form-item label="所属故障树ID">
                    <el-input :value="formData.treeId" disabled />
                </el-form-item>
            </el-form>
            <template #footer>
                <el-button @click="addNodeDialog = false">取消</el-button>
                <el-button type="primary" @click="confirmAddNode">
                    确认
                </el-button>
            </template>
        </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { treeUpdateApi, treeCreateApi, nodeUpdateApi, nodeDeleteApi, nodeAddApi, treeDataApi } from '@/api/fault/faultTree';
import { ElMessage, ElMessageBox } from 'element-plus';
import { ref, onMounted, onUnmounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
// import { useTabsStore } from '@/store/tabs';
import { Search } from '@element-plus/icons-vue';
import { submitTreeApi } from '@/api/fault/submitTree';


// 表单数据
const formData = ref({
    nodeId: null as number | null,
    nodeName: '',
    treeId: '',
    nodeType: '',
    parentNode: '',
    relationship: '',
    description: '',
    solution: '',
    createTime: '',
    updateTime: '',
    creatorName: '',
    treeName: ''
});

// 定义节点类型
interface TreeNode {
    nodeId: number | null;
    nodeName: string;
    nodeType: string;
    parentNodeId?: number | null;
    relationship?: string;
    description?: string;
    solution?: string;
    createTime?: string;
    updateTime?: string;
    creatorName?: string;
    treeId?: string;
    treeName?: string;
    children?: TreeNode[];
}

// 初始表单数据（用于比较修改）
const initialFormData = ref({ ...formData.value });
const addNodeDialog = ref(false);
const newNodeForm = ref({
    nodeName: '',
    nodeType: ''
});

// Reactive variables
const router = useRouter();
const route = useRoute()
// const tabsStore = useTabsStore();
// const { addTab, setCurrentTab } = tabsStore;

// 树形结构数据
const treeData = ref<TreeNode[]>([]);
const filteredTreeData = ref<TreeNode[]>([]);
const searchQuery = ref('');
const treeId = ref<string | undefined>(undefined); // 显式声明 treeId
const treeName = ref<string | undefined>(undefined); // 显式声明 treeName

// 树形结构属性配置
const treeProps = {
    label: (node: any) => `${node.nodeName}(${node.nodeType})`,
    children: 'children'
};

// 安全操作 sessionStorage
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

const contextMenu = ref({
    visible: false,
    x: 0,
    y: 0,
    currentNode: null as TreeNode | null
});

// 右键点击事件
const handleRightClick = (event: MouseEvent, node: TreeNode) => {
    event.preventDefault();
    const treeContainer = document.querySelector('.tree-container')
    if(!treeContainer) return 
    const rect = treeContainer.getBoundingClientRect()
    const x =event.clientX-rect.left
    const y =event.clientY-rect.top
    contextMenu.value = {
        visible: true,
        x: x,
        y: y+10,
        currentNode: node
    };
    console.log(contextMenu.value.y)
};


// 关闭菜单
const closeMenu = () => {
    contextMenu.value.visible = false;
};

// 全局点击事件处理
const handleGlobalClick = (event: MouseEvent) => {
    const menuElement = document.querySelector('.context-menu');
    if (menuElement && !menuElement.contains(event.target as Node)) {
        closeMenu();
    }
};

// 菜单操作
const addNode = () => {
    if (!contextMenu.value.currentNode) {
        ElMessage.warning('请先选择父节点');
        return;
    }
    newNodeForm.value = { nodeName: '', nodeType: '' };
    addNodeDialog.value = true;
    closeMenu();
};

const confirmAddNode = async () => {
    if (!newNodeForm.value.nodeName) {
        ElMessage.warning('节点名称不能为空');
        return;
    }
    try {
        const parentNode = contextMenu.value.currentNode;
        const response = await nodeAddApi({
            nodeName: newNodeForm.value.nodeName,
            parentNodeId: parentNode?.nodeId || null, // 如果没有父节点则传null
            treeId: formData.value.treeId // 从当前表单获取treeId
        });

        if (response.code === 200) {
            ElMessage.success('节点添加成功');
            addNodeDialog.value = false;
            await fetchTreeData(); // 刷新树数据
            // 自动选中新添加的节点
            if (response.data?.nodeId) {
                const flatNodes = flattenNodes(treeData.value);
                const newNode = flatNodes.find(node => node.nodeId === response.data.nodeId);
                if (newNode) handleNodeClick(newNode);
            }
        } else {
            ElMessage.error(response.message || '添加节点失败');
        }
    } catch (error) {
        console.error('Add node error:', error);
        ElMessage.error('添加节点失败');
    }
};

const deleteNode = async () => {
    const nodeToDelete = contextMenu.value.currentNode;
    if (!nodeToDelete || nodeToDelete.nodeId === null || nodeToDelete.nodeId === undefined) {
        ElMessage.warning('无法删除：未选择有效节点或根节点');
        closeMenu();
        return;
    }
    ElMessageBox.confirm(`确定要删除节点 "${nodeToDelete.nodeName}" 吗？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(async () => {
        try {
            const response = await nodeDeleteApi({ nodeId: nodeToDelete.nodeId });
            if (response.code === 200) {
                ElMessage.success('节点删除成功');
                await fetchTreeData(); // 刷新树数据
                // 如果删除的是当前显示的节点，清空表单
                if (formData.value.nodeId === nodeToDelete.nodeId) {
                    formData.value = {
                        nodeId: null,
                        nodeName: '',
                        treeId: formData.value.treeId,
                        treeName: formData.value.treeName,
                        nodeType: '',
                        parentNode: '',
                        relationship: '',
                        description: '',
                        solution: '',
                        createTime: '',
                        updateTime: '',
                        creatorName: ''
                    };
                }
            } else {
                ElMessage.error(`节点删除失败: ${response.message || '未知错误'}`);
            }
        } catch (error) {
            ElMessage.error('删除节点失败，请稍后重试');
            console.error('Delete Node Error:', error);
        }
        closeMenu();
    }).catch(() => {
        ElMessage.info('已取消删除');
        closeMenu();
    });
};

// 获取父节点名称
const getParentNodeName = (nodeId: number | null, nodes: any[]): string => {
    if (!nodeId) return '';
    const parent = nodes.find(node => node.nodeId === nodeId);
    return parent ? parent.nodeName : '';
};

// 扁平化节点列表（用于查找父节点）
const flattenNodes = (nodes: any[]): any[] => {
    let flat: any[] = [];
    nodes.forEach(node => {
        flat.push(node);
        if (node.children && node.children.length) {
            flat = flat.concat(flattenNodes(node.children));
        }
    });
    return flat;
};

// 过滤树形数据
const filterTreeData = () => {
    const query = searchQuery.value.toLowerCase();
    if (!query) {
        filteredTreeData.value = [...treeData.value];
        return;
    }

    const filterNode = (node: TreeNode): TreeNode | null => {
        const matches = node.nodeName.toLowerCase().includes(query) || node.nodeType.toLowerCase().includes(query);
        let filteredChildren: TreeNode[] | null = null;
        if (node.children && node.children.length) {
            filteredChildren = node.children
                .map(filterNode)
                .filter((child): child is TreeNode => child !== null);
        }
        if (matches || (filteredChildren && filteredChildren.length)) {
            return {
                ...node,
                children: filteredChildren || undefined
            };
        }
        return null;
    };

    filteredTreeData.value = treeData.value.map(filterNode).filter((node): node is TreeNode => node !== null);
};


const formRef = ref<any>(null); // 新增：表单引用
const currentStatus = ref<string>('');


// 新增：表单校验规则
const rules = {
    creatorName: [
        { required: true, message: '创建人不能为空', trigger: 'blur' },
        {
            validator: (rule: any, value: string, callback: any) => {
                if (!value || value.trim() === '') {
                    callback(new Error('创建人不能为空或全是空格'));
                } else {
                    callback();
                }
            },
            trigger: 'blur'
        }
    ],
    // 你可以顺便把其他重要字段也加上校验，比如节点名称
    nodeName: [
        { required: true, message: '节点名称不能为空', trigger: 'blur' }
    ],
    treeName: [
        { required: true, message: '故障树名称不能为空', trigger: 'blur' },
        {
            validator: (rule: any, value: string, callback: any) => {
                if (!value || value.trim() === '') {
                    callback(new Error('故障树名称不能为空或全是空格'));
                } else {
                    callback();
                }
            },
            trigger: 'blur'
        }
    ]
};


const fetchTreeData = async () => {
    if (!treeId.value || !treeName.value) {
        try {
            const res = await treeCreateApi();
            if (res.code === 200) {
                const { faultTree, faultTreeStructure } = res.data;
                treeId.value = faultTree.treeId; // 更新 treeId
                treeName.value = faultTree.treeName; // 更新 treeName
                treeData.value = faultTreeStructure ? [faultTreeStructure as TreeNode] : [];
                filteredTreeData.value = [...treeData.value];
                if (faultTreeStructure) {
                    const flatNodes = flattenNodes([faultTreeStructure]);
                    formData.value = {
                        nodeId: faultTreeStructure.nodeId || null,
                        nodeName: faultTreeStructure.nodeName || '',
                        treeId: faultTree.treeId || '',
                        treeName: faultTree.treeName || '',
                        nodeType: faultTreeStructure.nodeType || '',
                        parentNode: getParentNodeName(faultTreeStructure.parentNodeId, flatNodes),
                        relationship: faultTreeStructure.relationship || '',
                        description: faultTreeStructure.description || faultTree.description || '',
                        solution: faultTreeStructure.solution || faultTree.solution || '',
                        createTime: faultTreeStructure.createTime || faultTree.createTime || '',
                        updateTime: faultTreeStructure.updateTime || faultTree.updateTime || '',
                        creatorName: faultTreeStructure.creatorName || faultTree.creatorName || ''
                    };
                    initialFormData.value = { ...formData.value }; // 更新初始数据
                } else {
                    formData.value = {
                        nodeId: null,
                        nodeName: faultTree.treeName || '',
                        treeId: faultTree.treeId || '',
                        treeName: faultTree.treeName || '',
                        nodeType: '',
                        parentNode: '',
                        relationship: '',
                        description: faultTree.description || '',
                        solution: faultTree.solution || '',
                        createTime: faultTree.createTime || '',
                        updateTime: faultTree.updateTime || '',
                        creatorName: faultTree.creatorName || ''
                    };
                    initialFormData.value = { ...formData.value }; // 更新初始数据
                }
            } else {
                ElMessage.error(`创建新树失败: ${res.message || '未知错误'}`);
            }
        } catch (error) {
            ElMessage.error('请求出错');
            console.error('Fetch Tree Data Error:', error);
        }
    } else {
        try {
            const response = await treeDataApi({ treeId: treeId.value });
            if (response.code === 200) {
                const { faultTree, faultTreeStructure } = response.data;
                treeData.value = faultTreeStructure ? [faultTreeStructure as TreeNode] : [];
                filteredTreeData.value = [...treeData.value];
                if (faultTreeStructure) {
                    const flatNodes = flattenNodes([faultTreeStructure as TreeNode]);
                    formData.value = {
                        nodeId: faultTreeStructure.nodeId || null,
                        nodeName: faultTreeStructure.nodeName || '',
                        treeId: faultTree.treeId || '',
                        treeName: faultTree.treeName || '',
                        nodeType: faultTreeStructure.nodeType || '',
                        parentNode: getParentNodeName(faultTreeStructure.parentNodeId, flatNodes),
                        relationship: faultTreeStructure.relationship || '',
                        description: faultTreeStructure.description || faultTree.description || '',
                        solution: faultTreeStructure.solution || faultTree.solution || '',
                        createTime: faultTreeStructure.createTime || faultTree.createTime || '',
                        updateTime: faultTreeStructure.updateTime || faultTree.updateTime || '',
                        creatorName: faultTreeStructure.creatorName || faultTree.creatorName || ''
                    };
                    initialFormData.value = { ...formData.value }; // 更新初始数据
                } else {
                    formData.value = {
                        nodeId: null,
                        nodeName: faultTree.treeName || '',
                        treeId: faultTree.treeId || '',
                        treeName: faultTree.treeName || '',
                        nodeType: '',
                        parentNode: '',
                        relationship: '',
                        description: faultTree.description || '',
                        solution: faultTree.solution || '',
                        createTime: faultTree.createTime || '',
                        updateTime: faultTree.updateTime || '',
                        creatorName: faultTree.creatorName || ''
                    };
                    initialFormData.value = { ...formData.value }; // 更新初始数据
                }
            } else {
                ElMessage.error(`获取数据失败: ${response.message || '未知错误'}`);
            }
        } catch (error) {
            ElMessage.error('请求出错');
            console.error('Fetch Tree Data Error:', error);
        }
    }
};

// 提交审核
const handleSubmit = async () => {
    if (!treeId.value) {
        ElMessage.error('缺少故障树ID，无法提交');
        return;
    }

    try {
        ElMessageBox.confirm('确定要提交当前故障树审核吗？提交后将无法修改！', '提交审核', {
            type: 'warning',
            confirmButtonText: '确定提交',
            cancelButtonText: '我再看看'
        }).then(async () => {
            const res = await submitTreeApi(treeId.value);
            if (res.code === 200) {
                ElMessage.success('提交审核成功');
                currentStatus.value = '待审核';  // 更新按钮状态
                // 可选：刷新数据或直接跳回列表
                // fetchTreeData();
            } else {
                ElMessage.error('提交失败：' + (res.message || '未知错误'));
            }
        });
    } catch (error) {
        console.error('提交审核出错:', error);
        ElMessage.error('提交审核请求失败');
    }
};

const save = async () => {
    // 关键：先校验表单
    if (!formRef.value) return;
    try {
        await formRef.value.validate();
    } catch (error) {
        ElMessage.error('请填写必填项 !');
        return;
    }
    const nodeId = formData.value.nodeId;
    const nodeFieldsChanged = [
        formData.value.nodeName !== initialFormData.value.nodeName,
        formData.value.nodeType !== initialFormData.value.nodeType,
        formData.value.relationship !== initialFormData.value.relationship,
        formData.value.description !== initialFormData.value.description,
        formData.value.solution !== initialFormData.value.solution
    ].some(change => change);

    const otherFieldsChanged = [
        formData.value.treeName !== initialFormData.value.treeName,
        formData.value.creatorName !== initialFormData.value.creatorName,
        formData.value.createTime !== initialFormData.value.createTime,
        formData.value.updateTime !== initialFormData.value.updateTime
    ].some(change => change);

    const editload = {
        creatorName: formData.value.creatorName,
        description: formData.value.description,
        solution: formData.value.solution,
        treeName: formData.value.treeName,
        treeId: formData.value.treeId,
    };

    if (!treeId.value) {
        try {
            const response = await treeUpdateApi(editload);
            if (response.code === 200) {
                ElMessage.success('数据创建成功');
                treeId.value = response.data.treeId;
                currentStatus.value = '未提交';  // 新建的默认是未提交
                treeName.value = response.data.treeName; // 更新 treeName
                safeSessionStorage.set('dataUpdated', 'true');
                safeSessionStorage.set('faultTreeData', JSON.stringify({
                    treeId: formData.value.treeId,
                    treeName: formData.value.treeName,
                    creatorName: formData.value.creatorName,
                    description: formData.value.description,
                    solution: formData.value.solution,
                    createTime: formData.value.createTime,
                    updateTime: formData.value.updateTime
                }));
                goBack();
            } else {
                ElMessage.error('数据创建失败: ' + (response.message || '未知错误'));
            }
        } catch (error) {
            console.error('Error creating fault tree:', error);
            ElMessage.error('创建失败，请稍后重试');
        }
    } else if (nodeId && nodeFieldsChanged && !otherFieldsChanged) {
        try {
            const response = await nodeUpdateApi({
                nodeId,
                nodeName: formData.value.nodeName,
                nodeType: formData.value.nodeType,
                relationship: formData.value.relationship,
                description: formData.value.description,
                solution: formData.value.solution
            });
            if (response.code === 200) {
                ElMessage.success('节点更新成功');
                initialFormData.value = { ...formData.value }; // 更新初始数据
                await fetchTreeData(); // 刷新树数据
            } else {
                ElMessage.error(`节点更新失败: ${response.message || '未知错误'}`);
            }
        } catch (error) {
            console.error('Error updating node:', error);
            ElMessage.error('节点更新失败，请稍后重试');
        }
    } else if (treeId.value && treeName.value && (nodeFieldsChanged || otherFieldsChanged)) {
        try {
            const response = await treeUpdateApi(editload);
            if (response.code === 200) {
                ElMessage.success('数据更新成功');
                safeSessionStorage.set('dataUpdated', 'true');
                safeSessionStorage.set('faultTreeData', JSON.stringify({
                    treeId: formData.value.treeId,
                    treeName: formData.value.treeName,
                    creatorName: formData.value.creatorName,
                    description: formData.value.description,
                    solution: formData.value.solution,
                    createTime: formData.value.createTime,
                    updateTime: formData.value.updateTime
                }));
                goBack();
            } else {
                ElMessage.error('数据更新失败: ' + (response.message || '未知错误'));
            }
        } catch (error) {
            console.error('Error saving fault tree:', error);
            ElMessage.error('保存失败，请稍后重试');
        }
    } else {
        ElMessage.warning('未检测到有效修改或缺少必要数据');
    }
};

// 返回到 faultTree 页面
const goBack = () => {
    // addTab("故障树节点管理", "/analyse/faulttree", "Warning");
    // setCurrentTab("故障树节点管理", "/analyse/faulttree");
    // router.push({
    //     path: '/predict/analyse/faulttree',
    //     query: { treeName: formData.value.treeName }
    // });
    window.history.back()
};

// 树节点点击事件
const handleNodeClick = (node: any) => {
    const flatNodes = flattenNodes(treeData.value);
    formData.value = {
        nodeId: node.nodeId || null,
        nodeName: node.nodeName || '',
        treeId: node.treeId || formData.value.treeId,
        treeName: node.treeName || formData.value.treeName,
        nodeType: node.nodeType || '',
        parentNode: getParentNodeName(node.parentNodeId, flatNodes),
        relationship: node.relationship || '',
        description: node.description || '',
        solution: node.solution || '',
        createTime: node.createTime || '',
        updateTime: node.updateTime || '',
        creatorName: node.creatorName || ''
    };
};

// 在组件挂载时获取数据
onMounted(() => {
    if (!route.query.treeId) {
        fetchTreeData();
    }
    document.addEventListener('click', handleGlobalClick);
});
onUnmounted(() => {
    document.removeEventListener('click', handleGlobalClick);
});
</script>

<style lang="less" scoped>
.sub-tit {
    margin: 10px 0;
}

.sear {
    margin-top: 100px;
    margin-bottom: 20px;
}

.custom-border-form {

    // 每项之间留点呼吸感，不再用负 margin
    .border-item {
        margin-bottom: 16px;

        // 核心：给输入框加边框 + 错误时红色边框
        :deep(.el-input__wrapper) {
            border: 1px solid #DCDFE6;
            box-shadow: none;
            transition: all 0.3s;
        }

        // 校验失败时：红色边框 + 红字提示
        &.is-error {
            :deep(.el-input__wrapper) {
                border-color: #F56C6C !important;
                background-color: #fef0f0;
            }
        }

        // 聚焦时好看一点
        :deep(.el-input__wrapper.is-focus) {
            border-color: #409EFF;
            box-shadow: 0 0 0 1px #409EFF inset;
        }
    }

    // 可选：让 label 更粗更黑
    :deep(.el-form-item__label) {
        font-weight: bold;
        color: #303133;
    }
}

.tree-container{
    position: relative;
    min-height: 400px;
}

.context-menu{
    position: absolute;

 .el-button{
     display:block;
     text-align: left;
     border: none;
     margin: 0;
 }   
}
</style>