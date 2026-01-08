<script setup>
import { ref } from 'vue';

const props = defineProps({
  originalWish: {
    type: String,
    required: true
  },
  copyText: {
    type: String,
    required: true
  }
});

const emit = defineEmits(['restart']);

const isCopied = ref(false);

async function handleCopy() {
  try {
    await navigator.clipboard.writeText(props.copyText);
    isCopied.value = true;
    setTimeout(() => {
      isCopied.value = false;
    }, 2000);
  } catch (err) {
    console.error('Failed to copy: ', err);
  }
}
</script>

<template>
  <div class="quota-error-card">
    <div class="card-header">
      <span class="status-badge">因果律超载</span>
      <h3 class="wish-title">「{{ originalWish }}」</h3>
    </div>

    <div class="card-body">
      <div class="error-content">
        <p class="main-msg">由于实现了大量的愿望，许愿器(暂时)额度不够了，需要等管理员补充。</p>
        <p class="sub-msg">在许愿器功能恢复之前，你可以点击下方复制按钮。然后把内容粘贴给任意一个你喜欢的AI助手（Deepseek/Kimi/豆包/智谱...任何你能用上的AI助手）。</p>
      </div>
    </div>

    <div class="card-footer">
      <button @click="handleCopy" class="action-button copy-btn" :class="{ 'success': isCopied }">
        <span v-if="!isCopied">复制引导指令</span>
        <span v-else>已复制到剪切板</span>
      </button>
      <button @click="emit('restart')" class="action-button restart-btn">
        重新许愿
      </button>
    </div>
  </div>
</template>

<style scoped>
.quota-error-card {
  background-color: #fff;
  border: 2px solid #2c3e50;
  border-radius: 8px;
  padding: 30px;
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
  box-shadow: 10px 10px 0px #2c3e50;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.card-header {
  text-align: center;
  border-bottom: 1px solid #eee;
  padding-bottom: 15px;
}

.status-badge {
  background-color: #e74c3c;
  color: white;
  font-size: 0.7rem;
  padding: 2px 8px;
  border-radius: 10px;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 10px;
  display: inline-block;
}

.wish-title {
  font-size: 1.2rem;
  color: #2c3e50;
  margin: 0;
  font-style: italic;
}

.error-content {
  color: #34495e;
  line-height: 1.6;
}

.main-msg {
  font-weight: bold;
  margin-bottom: 10px;
  color: #c0392b;
}

.sub-msg {
  font-size: 0.9rem;
  color: #7f8c8d;
}

.card-footer {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 10px;
}

.action-button {
  padding: 12px;
  border-radius: 4px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
  border: 2px solid #2c3e50;
  background: white;
  color: #2c3e50;
}

.copy-btn {
  background-color: #2c3e50;
  color: white;
}

.copy-btn:hover {
  background-color: #000;
}

.copy-btn.success {
  background-color: #27ae60;
  border-color: #27ae60;
}

.restart-btn:hover {
  background-color: #f8f9fa;
}

@media (max-width: 480px) {
  .quota-error-card {
    padding: 20px;
  }
}
</style>
