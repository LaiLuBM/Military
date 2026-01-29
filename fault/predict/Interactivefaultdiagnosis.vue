<template>
    <div class="chat-container">
        <div class="chat-history">
            <div v-for="(item, index) in history" :key="index" class="chat-item">
                <div v-if="item.isDivider" class="divider">
                    <hr />
                    <span>--- 新会话 ---</span>
                    <hr />
                    <br />
                </div>
                <div v-else class="question">
                    <el-card class="question-card">
                        <p>{{ item.question }}</p>
                    </el-card>
                </div>
                <div v-if="!item.isDivider" class="answer">
                   
                    <el-card class="answer-card">
                        <p v-if="item.answer">{{ item.answer }}</p>
                        <div v-else class="loading-text">正在进行深度思考...</div>
                    </el-card>
                </div>
            </div>
        </div>
        <div class="input-container">
            <el-input v-model="inputQuestion" placeholder="请输入您的问题" type="textarea" :rows="2"
                @keyup.enter.prevent="sendQuestion"></el-input>
            <el-button type="primary" @click="sendQuestion" :disabled="!inputQuestion.trim() || isLoading"
                :loading="isLoading">发送</el-button>
            <el-button type="primary" @click="endSession">结束当前对话</el-button>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue';
import { ElMessage } from 'element-plus';
import { askQuestion } from '@/api/fault/chat';

interface ChatItem {
    question?: string;
    answer?: string | null;
    isDivider?: boolean;
}

export default defineComponent({
    name: 'ChatApp',
    setup() {
        const inputQuestion = ref<string>('');
        const history = ref<ChatItem[]>([]);
        const isLoading = ref<boolean>(false);

        // Load history from sessionStorage on mount
        onMounted(() => {
            const savedHistory = sessionStorage.getItem('chatHistory');
            if (savedHistory) {
                try {
                    history.value = JSON.parse(savedHistory);
                } catch (e) {
                    console.error('Failed to parse sessionStorage chatHistory:', e);
                    sessionStorage.removeItem('chatHistory'); // Clear invalid data
                }
            }
        });

        const sendQuestion = async () => {
            if (!inputQuestion.value.trim()) {
                ElMessage.warning('请输入问题');
                return;
            }

            const question = inputQuestion.value.trim();
            // Explicitly set history to '无' for the first request or after a divider
            const historyToSend = history.value.length === 0 || history.value[history.value.length - 1].isDivider
                ? '无'
                : history.value[history.value.length - 1].answer || '无';

            history.value.push({
                question,
                answer: null,
            });

            // Save to sessionStorage after adding new question
            sessionStorage.setItem('chatHistory', JSON.stringify(history.value));

            try {
                isLoading.value = true;
                const response = await askQuestion({
                    history: historyToSend,
                    question,
                });

                const lastIndex = history.value.length - 1;
                history.value[lastIndex].answer = response.data.answer || '收到回答';

                // Update sessionStorage with the new answer
                sessionStorage.setItem('chatHistory', JSON.stringify(history.value));
            } catch (error) {
                ElMessage.error('请求失败，请稍后重试');
                console.log('Error sending question:', error);
                const lastIndex = history.value.length - 1;
                history.value[lastIndex].answer = '请求失败';
                // Update sessionStorage on error
                sessionStorage.setItem('chatHistory', JSON.stringify(history.value));
            } finally {
                isLoading.value = false;
            }

            inputQuestion.value = '';
        };

        const endSession = () => {
            // Save current history to a different key for persistence
            const savedHistory = sessionStorage.getItem('chatHistory');
            if (savedHistory) {
                const archivedHistories = sessionStorage.getItem('archivedChatHistories') || '[]';
                const archived = JSON.parse(archivedHistories);
                archived.push(JSON.parse(savedHistory));
                sessionStorage.setItem('archivedChatHistories', JSON.stringify(archived));
            }

            // Add divider and clear current history
            history.value.push({ isDivider: true });
            sessionStorage.setItem('chatHistory', JSON.stringify(history.value));
            ElMessage.success('当前对话已结束');
        };

        return {
            inputQuestion,
            history,
            sendQuestion,
            isLoading,
            endSession,
        };
    },
});
</script>

<style scoped>
.chat-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    padding: 20px;
    box-sizing: border-box;
    background-color: #ffffff;
}

.chat-history {
    flex: 1;
    overflow-y: auto;
    padding-bottom: 20px;
}

.chat-item {
    margin-bottom: 20px;
}

.question {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 10px;
}

.answer {
    display: flex;
    justify-content: flex-start;
}

.question-card {
    max-width: 60%;
    background-color: #409EFF;
    border-radius: 20px;
    margin-right: 30px;
}

.answer-card {
    max-width: 60%;
    background-color: #ffffff;
    border-radius: 20px;
    margin-left: 30px;
}

.input-container {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    width: 80%;
    max-width: 800px;
    display: flex;
    gap: 10px;
    align-items: flex-end;
}

.el-input {
    flex: 1;
}

.el-button {
    width: 100px;
    height: 52px;
}

.divider {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    margin: 20px 0;
    color: #999;
}

.divider hr {
    flex: 1;
    border: none;
    border-top: 1px solid #ddd;
}

.divider span {
    font: size 14px;
}
</style>