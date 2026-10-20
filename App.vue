<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

// ==================== 1. ref 資料定義 (作業要求) ====================
const display = ref<string>('')
const activeKey = ref<string | null>(null) // 用於記錄當前按下的鍵盤按鍵（製作按鈕發亮特效）

// ==================== 2. computed 計算屬性 (作業要求) ====================
const advancedInfo = computed(() => {
  const num = parseFloat(display.value)
  if (isNaN(num) || !isFinite(num)) {
    return { hex: 'N/A', bin: 'N/A' }
  }
  
  const intVal = Math.floor(num)
  return {
    hex: intVal.toString(16).toUpperCase(),
    bin: intVal.toString(2)
  }
})

// ==================== 3. 計算機功能函式 ====================
const append = (char: string) => {
  display.value += char
}

const clear = () => {
  display.value = ''
}

const backspace = () => {
  display.value = display.value.slice(0, -1)
}

const mathFunc = (type: string) => {
  try {
    const num = parseFloat(display.value)
    if (isNaN(num)) return
    
    let result = 0
    switch (type) {
      case 'sqrt': result = Math.sqrt(num); break;
      case 'sqr':  result = Math.pow(num, 2); break;
      case 'sin':  result = Math.sin(num * Math.PI / 180); break;
      case 'cos':  result = Math.cos(num * Math.PI / 180); break;
      case 'tan':  result = Math.tan(num * Math.PI / 180); break;
      case 'log':  result = Math.log10(num); break;
      case 'pi':   display.value += Math.PI.toFixed(5); return;
    }
    display.value = String(parseFloat(result.toFixed(6)))
  } catch (error) {
    display.value = 'Error'
  }
}

const calculate = () => {
  try {
    if (!display.value) return
    // 將輸入的 × 轉為 *，÷ 轉為 / 後進行計算
    let expression = display.value.replace(/×/g, '*').replace(/÷/g, '/')
    const result = new Function(`return ${expression}`)()
    
    if (result === undefined || isNaN(result)) {
      display.value = 'Error'
    } else {
      display.value = String(parseFloat(result.toFixed(6)))
    }
  } catch (error) {
    display.value = 'Error'
  }
}

// ==================== 4. 鍵盤事件綁定邏輯 ====================
const handleKeyDown = (e: KeyboardEvent) => {
  const key = e.key

  // 1. 數字與基礎運算子 (對應畫面並轉換成顯示符號)
  if (/[0-9.]/.test(key)) {
    e.preventDefault()
    append(key)
    triggerActiveEffect(key)
  } else if (key === '+') {
    e.preventDefault()
    append('+')
    triggerActiveEffect('+')
  } else if (key === '-') {
    e.preventDefault()
    append('-')
    triggerActiveEffect('-')
  } else if (key === '*') {
    e.preventDefault()
    append('×')
    triggerActiveEffect('*')
  } else if (key === '/') {
    e.preventDefault()
    append('÷')
    triggerActiveEffect('/')
  } else if (key === '(' || key === ')') {
    e.preventDefault()
    append(key)
    triggerActiveEffect(key)
  }
  // 2. 功能鍵對應
  else if (key === 'Enter' || key === '=') {
    e.preventDefault()
    calculate()
    triggerActiveEffect('=')
  } else if (key === 'Backspace') {
    e.preventDefault()
    backspace()
    triggerActiveEffect('backspace')
  } else if (key === 'Escape') {
    e.preventDefault()
    clear()
    triggerActiveEffect('clear')
  }
}

// 觸發按鍵發亮效果
const triggerActiveEffect = (keyLabel: string) => {
  activeKey.value = keyLabel
  setTimeout(() => {
    activeKey.value = null
  }, 120) // 0.12 秒後移除發亮樣式
}

// 元件掛載時監聽全域鍵盤事件
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
})

// 元件卸載時移除監聽，避免記憶體洩漏
onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
})
</script>

<template>
  <div class="calculator-container">
    <h2>工程計算機 🧮</h2>

    <!-- 1. 計算機螢幕：使用 v-model 雙向綁定 (作業要求) -->
    <input 
      v-model="display" 
      type="text" 
      class="calc-screen" 
      placeholder="0"
    />

    <!-- 2. 進階進位顯示：使用 computed 即時計算結果 (作業要求) -->
    <div class="advanced-panel">
      <div><small>HEX (十六進位):</small> <span>{{ advancedInfo.hex }}</span></div>
      <div><small>BIN (二進位):</small> <span class="bin-text">{{ advancedInfo.bin }}</span></div>
    </div>

    <!-- 3. 按鍵配置群組 -->
    <div class="calc-buttons">
      <!-- 科學函數 -->
      <button @click="mathFunc('sin')" class="btn-sci">sin</button>
      <button @click="mathFunc('cos')" class="btn-sci">cos</button>
      <button @click="mathFunc('tan')" class="btn-sci">tan</button>
      <button @click="mathFunc('pi')" class="btn-sci">π</button>

      <button @click="mathFunc('sqrt')" class="btn-sci">√</button>
      <button @click="mathFunc('sqr')" class="btn-sci">x²</button>
      <button @click="mathFunc('log')" class="btn-sci">log</button>
      <button @click="backspace" class="btn-op" :class="{ 'kbd-active': activeKey === 'backspace' }">⌫</button>

      <!-- 控制鍵 -->
      <button @click="clear" class="btn-clear" :class="{ 'kbd-active': activeKey === 'clear' }">C</button>
      <button @click="append('(')" class="btn-op" :class="{ 'kbd-active': activeKey === '(' }">(</button>
      <button @click="append(')')" class="btn-op" :class="{ 'kbd-active': activeKey === ')' }">)</button>
      <button @click="append('÷')" class="btn-op" :class="{ 'kbd-active': activeKey === '/' }">÷</button>

      <!-- 數字與基礎運算 -->
      <button @click="append('7')" :class="{ 'kbd-active': activeKey === '7' }">7</button>
      <button @click="append('8')" :class="{ 'kbd-active': activeKey === '8' }">8</button>
      <button @click="append('9')" :class="{ 'kbd-active': activeKey === '9' }">9</button>
      <button @click="append('×')" class="btn-op" :class="{ 'kbd-active': activeKey === '*' }">×</button>

      <button @click="append('4')" :class="{ 'kbd-active': activeKey === '4' }">4</button>
      <button @click="append('5')" :class="{ 'kbd-active': activeKey === '5' }">5</button>
      <button @click="append('6')" :class="{ 'kbd-active': activeKey === '6' }">6</button>
      <button @click="append('-')" class="btn-op" :class="{ 'kbd-active': activeKey === '-' }">-</button>

      <button @click="append('1')" :class="{ 'kbd-active': activeKey === '1' }">1</button>
      <button @click="append('2')" :class="{ 'kbd-active': activeKey === '2' }">2</button>
      <button @click="append('3')" :class="{ 'kbd-active': activeKey === '3' }">3</button>
      <button @click="append('+')" class="btn-op" :class="{ 'kbd-active': activeKey === '+' }">+</button>

      <button @click="append('0')" :class="{ 'kbd-active': activeKey === '0' }">0</button>
      <button @click="append('.')" :class="{ 'kbd-active': activeKey === '.' }">.</button>
      <button @click="calculate" class="btn-equal" :class="{ 'kbd-active': activeKey === '=' }">=</button>
    </div>
  </div>
</template>

<style scoped>
.calculator-container {
  max-width: 360px;
  margin: 50px auto;
  padding: 20px;
  background-color: #2c3e50;
  border-radius: 16px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.3);
  font-family: sans-serif;
}
h2 {
  text-align: center;
  color: #42b883;
  margin-top: 0;
  margin-bottom: 15px;
}
.calc-screen {
  width: 100%;
  height: 60px;
  background-color: #34495e;
  border: none;
  border-radius: 8px;
  font-size: 1.8rem;
  color: #fff;
  text-align: right;
  padding: 0 15px;
  box-sizing: border-box;
  outline: none;
}
.advanced-panel {
  margin-top: 10px;
  margin-bottom: 15px;
  padding: 8px 12px;
  background-color: #1a252f;
  border-radius: 6px;
  font-size: 0.85rem;
  color: #95a5a6;
  text-align: left;
}
.advanced-panel div {
  margin: 4px 0;
  display: flex;
  justify-content: space-between;
}
.advanced-panel span {
  color: #42b883;
  font-weight: bold;
}
.bin-text {
  word-break: break-all;
  text-align: right;
  max-width: 200px;
}
.calc-buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}
button {
  height: 50px;
  font-size: 1.2rem;
  font-weight: bold;
  background-color: #bdc3c7;
  color: #2c3e50;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.1s, transform 0.05s;
}
button:hover {
  background-color: #ecf0f1;
}

/* 鍵盤按下時產生的虛擬發亮特效（加分亮點） */
button.kbd-active {
  background-color: #ffffff !important;
  transform: scale(0.95);
  box-shadow: 0 0 8px #42b883;
}

.btn-sci {
  background-color: #34495e;
  color: #42b883;
  font-size: 1rem;
}
.btn-sci:hover {
  background-color: #415b76;
}
.btn-sci.kbd-active {
  background-color: #42b883 !important;
  color: white !important;
}

.btn-op {
  background-color: #e67e22;
  color: white;
}
.btn-op:hover {
  background-color: #f39c12;
}
.btn-op.kbd-active {
  background-color: #f39c12 !important;
}

.btn-clear {
  background-color: #e74c3c;
  color: white;
}
.btn-clear:hover {
  background-color: #c0392b;
}
.btn-clear.kbd-active {
  background-color: #ff4d4d !important;
}

.btn-equal {
  background-color: #42b883;
  color: white;
  grid-column: span 2; /* 讓等號變寬方便盲打 */
}
.btn-equal:hover {
  background-color: #33a06f;
}
.btn-equal.kbd-active {
  background-color: #2cc585 !important;
}
</style>
