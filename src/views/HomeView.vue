<script setup>
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';
import { useWishEnergy } from '../composables/useWishEnergy';
import HeaderLogo from '../components/HeaderLogo.vue';
import WishInput from '../components/WishInput.vue';
import StepFlow from '../components/StepFlow.vue';
import WishResultCard from '../components/WishResultCard.vue';
import WishQuotaErrorCard from '../components/WishQuotaErrorCard.vue';
import AppFooter from '../components/AppFooter.vue';

// 路由
const router = useRouter();

// 状态管理
// 0: 输入愿望, 1: 审查愿望(3秒), 2: 正在构建现实/钻漏洞(API请求期间), 3: 展示结果, 4: 额度不足
const currentStep = ref(0); 
const wishText = ref('');
const isLoading = ref(false);
const error = ref(null);
const quotaErrorPrompt = ref('');

const { refundEnergy, decreaseStability, recoverStability } = useWishEnergy();

// 愿望实现结果数据
const wishResult = reactive({
  confirmed_wish: '',
  realization_scenario: '', // 存储LLM生成的反转剧情
});

/**
 * 处理愿望提交逻辑 - 加入了“恶意”延迟版
 */
async function handleWishSubmit(wish) {
  wishText.value = wish;
  // 1. 开始：进入【审查阶段】
  // 此时 StepFlow 显示第一步：正在扫描灵魂签署痕迹...
  currentStep.value = 1; 
  isLoading.value = true;
  error.value = null; // 清空之前的错误
  
  try {
    // 😈 关键点：强行制造 3 秒的心理压迫感
    // 让用户盯着进度条发慌
    await new Promise(resolve => setTimeout(resolve, 3000));

    // 2. 停顿结束：进入【寻找漏洞阶段】
    // 此时 StepFlow 显示第二步：正在检索因果律漏洞...
    currentStep.value = 2;

    // 3. 真正发起 API 请求（就在步骤 2 显示的时候）
    // 这里复用原本的 fetch 逻辑
    const response = await fetch('/api/validateWish', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ wish })
    });
    
    if (response.status === 402) {
      const data = await response.json();
      quotaErrorPrompt.value = data.prompt;
      refundEnergy(); // 额度不足不扣能量
      currentStep.value = 4;
      return;
    }

    if (!response.ok) {
      throw new Error(`因果连接中断: ${response.status}`);
    }

    const data = await response.json();

    // 4. 处理返回结果
    if (data.result && data.result.category === 'block') {
      refundEnergy(); // 审查未通过返还紫色能量
      decreaseStability(); // 审查未通过扣除 1 点灵魂稳定性
      router.push('/error'); // 违规处理
      return;
    }

    // 成功完成许愿，恢复灵魂稳定性
    recoverStability();

    // 拿到结果
    wishResult.confirmed_wish = data.result.confirmed_wish;
    wishResult.realization_scenario = data.result.scenario;
    
    // 为了防止 API 响应太快导致步骤 2 闪退，
    // 我们可以再额外增加一点点“构建现实”的时间（可选，这里加了1秒）
    await new Promise(resolve => setTimeout(resolve, 1000));

    // 5. 展示最终审判
    currentStep.value = 3; 

  } catch (err) {
    console.error(err);
    error.value = "系统在窥探命运时遭遇了不可抗力...请重试。";
    // 出错后回滚能量
    refundEnergy();
    // 出错后是否要重置回步骤0，看你喜好，这里暂时保留在当前界面方便看报错
    currentStep.value = 0;
  } finally {
    isLoading.value = false;
  }
}

/**
 * 重新开始许愿
 */
function handleRestart() {
  currentStep.value = 0;
  wishText.value = '';
  error.value = null;
  wishResult.confirmed_wish = '';
  wishResult.realization_scenario = '';
  quotaErrorPrompt.value = '';
}
</script>

<template>
  <div class="home-container">
    <HeaderLogo />
    
    <transition name="fade">
      <div v-if="error" class="error-message">
        {{ error }}
      </div>
    </transition>
    
    <transition name="fade" mode="out-in">
      <div v-if="currentStep === 0" class="input-section" key="input">
        <WishInput @submit="handleWishSubmit" />
      </div>
      
      <div v-else-if="currentStep < 3" class="processing-section" key="processing">
        <StepFlow :current-step="currentStep" />
      </div>
      
      <div v-else-if="currentStep === 3" class="result-section" key="result">
        <WishResultCard 
          :sign-data="wishResult" 
          @restart="handleRestart"
        />
      </div>

      <div v-else-if="currentStep === 4" class="result-section" key="quota-error">
        <WishQuotaErrorCard 
          :original-wish="wishText"
          :copy-text="quotaErrorPrompt"
          @restart="handleRestart"
        />
      </div>
    </transition>
    
    <AppFooter />
  </div>
</template>

<style scoped>
.home-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 错误提示条：加一点阴影和深色边框 */
.error-message {
  background-color: #fff5f5;
  color: #c0392b;
  padding: 12px 16px;
  border-radius: 4px;
  margin: 16px 0;
  border-left: 4px solid #c0392b;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  text-align: center;
  font-size: 0.9rem;
}

.input-section, .processing-section, .result-section {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding: 10px 0;
  margin-top: -10px;
  flex: 1; /* 让内容区占据剩余空间 */
}

/* 简单的淡入淡出过渡 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
