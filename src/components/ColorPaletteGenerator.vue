<template>
  <div class="palette-generator">
    <!-- Заголовок -->
    <header class="app-header">
      <h1>🎨 Генератор цветовых палитр</h1>
      <p>Создавайте гармоничные цветовые схемы для ваших проектов</p>
    </header>

    <!-- Основное содержимое -->
    <main class="main-content">
      <!-- Панель управления -->
      <div class="control-panel">
        <button @click="generateRandomPalette" class="generate-button">
          🎲 Случайная палитра
        </button>
        
        <div class="controls-group">
          <div class="control-item">
            <label for="color-count">Количество цветов:</label>
            <select 
              id="color-count" 
              v-model="colorCount"
              class="control-select"
              @change="handleColorCountChange"
            >
              <option value="3">3</option>
              <option value="5">5</option>
              <option value="7">7</option>
            </select>
          </div>
          
          <div class="control-item">
            <label for="color-format">Формат отображения:</label>
            <select 
              id="color-format" 
              v-model="colorFormat"
              class="control-select"
            >
              <option value="hex">HEX</option>
              <option value="rgb">RGB</option>
            </select>
          </div>
          
          <div class="control-item">
            <label class="toggle-switch">
              <input 
                type="checkbox" 
                v-model="darkMode"
                class="toggle-input"
              >
              <span class="toggle-slider"></span>
              {{ darkMode ? 'Тёмный фон' : 'Светлый фон' }}
            </label>
          </div>
        </div>
      </div>

      <!-- Отображение палитры с фиксированной высотой -->
      <div class="palette-container-wrapper">
        <div class="palette-container" ref="paletteContainer">
          <div 
            v-for="(color, index) in colors" 
            :key="index"
            class="color-card"
            :style="{
              backgroundColor: color.hex,
              flex: getColorFlex(index)
            }"
            @click="copyToClipboard(color)"
            @dblclick="toggleLock(color)"
          >
            <!-- Значок блокировки -->
            <div 
              v-if="color.locked" 
              class="lock-indicator"
              title="Цвет закреплён"
            >
              🔒
            </div>
            
            <!-- Цветовая информация -->
            <div class="color-info">
              <div class="color-value">
                {{ colorFormat === 'hex' ? color.hex : color.rgb }}
              </div>
              <div class="color-actions">
                <button 
                  @click.stop="copyToClipboard(color)"
                  class="copy-button"
                  :title="color.copied ? 'Скопировано!' : 'Копировать'"
                >
                  {{ color.copied ? '✓' : '📋' }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Уведомление -->
      <div 
        v-if="notification.show"
        class="notification"
        :class="{ 'notification-visible': notification.show }"
      >
        {{ notification.message }}
      </div>

      <!-- Предпросмотр в UI -->
      <div class="preview-section">
        <h3>Предпросмотр в интерфейсе</h3>
        <div 
          class="ui-preview"
          :style="{ backgroundColor: darkMode ? '#1a1a1a' : '#f5f5f5' }"
        >
          <!-- Мокап кнопки -->
          <button 
            class="mockup-button"
            :style="{ 
              backgroundColor: colors[0]?.hex || '#4CAF50',
              color: getContrastColor(colors[0]?.hex || '#4CAF50')
            }"
          >
            Пример кнопки
          </button>
          
          <!-- Мокап карточки -->
          <div 
            class="mockup-card"
            :style="{ 
              backgroundColor: colors[1]?.hex || '#FFFFFF',
              color: getContrastColor(colors[1]?.hex || '#FFFFFF')
            }"
          >
            <h4 class="mockup-title">Заголовок карточки</h4>
            <p class="mockup-text">Текст карточки с примером использования цветов из палитры.</p>
          </div>
          
          <!-- Мокап заголовка -->
          <h3 
            class="mockup-header"
            :style="{ color: colors[2]?.hex || '#333333' }"
          >
            Пример заголовка
          </h3>
        </div>
      </div>

      <!-- Информация о сохранении -->
      <div class="save-info">
        <p v-if="isAutoSaved" class="save-status saved">
          💾 Автоматически сохранено
        </p>
        <p v-else class="save-status">
          Изменения не сохранены
        </p>
        <button @click="resetPalette" class="reset-button">
          🔄 Сбросить палитру
        </button>
      </div>
    </main>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch, nextTick } from 'vue'

export default {
  name: 'ColorPaletteGenerator',
  
  setup() {
    // Реактивные данные
    const colors = ref([])
    const colorCount = ref(5)
    const colorFormat = ref('hex')
    const darkMode = ref(false)
    const notification = ref({
      show: false,
      message: ''
    })
    const paletteContainer = ref(null)

    // Вычисляемое свойство для пропорций цветов
    const getColorFlex = (index) => {
      // Для 3 цветов: 2 - 1 - 1 (первый цвет шире)
      // Для 5 цветов: 1 - 1 - 1 - 1 - 1 (все равны)
      // Для 7 цветов: 1 - 1 - 1 - 1 - 1 - 1 - 1 (все равны)
      
      if (colorCount.value === 3) {
        return index === 0 ? '2' : '1'
      } else {
        return '1'
      }
    }

    // Генерация случайного цвета в HEX формате
    const generateRandomColor = () => {
      return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0')
    }

    // Преобразование HEX в RGB
    const hexToRgb = (hex) => {
      const r = parseInt(hex.slice(1, 3), 16)
      const g = parseInt(hex.slice(3, 5), 16)
      const b = parseInt(hex.slice(5, 7), 16)
      return `rgb(${r}, ${g}, ${b})`
    }

    // Генерация гармоничных цветов
    const generateHarmoniousColors = (count) => {
      const baseHue = Math.floor(Math.random() * 360)
      const colorsArray = []

      for (let i = 0; i < count; i++) {
        // Генерация оттенков с небольшим смещением
        const hue = (baseHue + (i * (360 / count))) % 360
        const saturation = 70 + Math.random() * 20
        const lightness = 40 + Math.random() * 20
        
        // Преобразование HSL в HEX
        const hex = hslToHex(hue, saturation, lightness)
        
        colorsArray.push({
          hex,
          rgb: hexToRgb(hex),
          locked: false,
          copied: false
        })
      }

      return colorsArray
    }

    // Преобразование HSL в HEX
    const hslToHex = (h, s, l) => {
      h /= 360
      s /= 100
      l /= 100
      
      let r, g, b
      
      if (s === 0) {
        r = g = b = l
      } else {
        const hue2rgb = (p, q, t) => {
          if (t < 0) t += 1
          if (t > 1) t -= 1
          if (t < 1/6) return p + (q - p) * 6 * t
          if (t < 1/2) return q
          if (t < 2/3) return p + (q - p) * (2/3 - t) * 6
          return p
        }
        
        const q = l < 0.5 ? l * (1 + s) : l + s - l * s
        const p = 2 * l - q
        
        r = hue2rgb(p, q, h + 1/3)
        g = hue2rgb(p, q, h)
        b = hue2rgb(p, q, h - 1/3)
      }
      
      const toHex = (x) => {
        const hex = Math.round(x * 255).toString(16)
        return hex.length === 1 ? '0' + hex : hex
      }
      
      return `#${toHex(r)}${toHex(g)}${toHex(b)}`
    }

    // Генерация случайной палитры
    const generateRandomPalette = () => {
      const lockedColors = colors.value.filter(color => color.locked)
      const newColors = generateHarmoniousColors(parseInt(colorCount.value))
      
      // Сохраняем закреплённые цвета
      colors.value = newColors.map((newColor, index) => {
        if (index < lockedColors.length) {
          return { ...lockedColors[index] }
        }
        return newColor
      })
      
      saveToLocalStorage()
    }

    // Обработка изменения количества цветов
    const handleColorCountChange = () => {
      // Перегенерируем палитру с новым количеством цветов
      generateRandomPalette()
    }

    // Копирование в буфер обмена
    const copyToClipboard = async (color) => {
      const textToCopy = colorFormat.value === 'hex' ? color.hex : color.rgb
      
      try {
        await navigator.clipboard.writeText(textToCopy)
        
        // Показываем уведомление
        color.copied = true
        showNotification(`Скопировано: ${textToCopy}`)
        
        // Сбрасываем состояние через 2 секунды
        setTimeout(() => {
          color.copied = false
        }, 2000)
      } catch (err) {
        console.error('Ошибка копирования:', err)
        showNotification('Ошибка при копировании')
      }
    }

    // Закрепление/открепление цвета
    const toggleLock = (color) => {
      color.locked = !color.locked
      saveToLocalStorage()
    }

    // Показать уведомление
    const showNotification = (message) => {
      notification.value = {
        show: true,
        message
      }
      
      setTimeout(() => {
        notification.value.show = false
      }, 3000)
    }

    // Получение контрастного цвета
    const getContrastColor = (hexColor) => {
      if (!hexColor) return '#000000'
      
      // Преобразование HEX в RGB
      const r = parseInt(hexColor.slice(1, 3), 16)
      const g = parseInt(hexColor.slice(3, 5), 16)
      const b = parseInt(hexColor.slice(5, 7), 16)
      
      // Вычисление яркости
      const brightness = (r * 299 + g * 587 + b * 114) / 1000
      
      return brightness > 128 ? '#000000' : '#FFFFFF'
    }

    // Сохранение в localStorage
    const saveToLocalStorage = () => {
      try {
        const paletteData = {
          colors: colors.value,
          colorCount: colorCount.value,
          colorFormat: colorFormat.value,
          darkMode: darkMode.value
        }
        localStorage.setItem('colorPalette', JSON.stringify(paletteData))
      } catch (err) {
        console.error('Ошибка сохранения:', err)
      }
    }

    // Загрузка из localStorage
    const loadFromLocalStorage = () => {
      try {
        const saved = localStorage.getItem('colorPalette')
        if (saved) {
          const paletteData = JSON.parse(saved)
          colors.value = paletteData.colors || []
          colorCount.value = paletteData.colorCount || 5
          colorFormat.value = paletteData.colorFormat || 'hex'
          darkMode.value = paletteData.darkMode || false
        }
      } catch (err) {
        console.error('Ошибка загрузки:', err)
      }
      
      // Если нет сохранённых данных, генерируем новую палитру
      if (colors.value.length === 0) {
        generateRandomPalette()
      } else {
        // Если количество цветов не совпадает, перегенерируем
        if (colors.value.length !== parseInt(colorCount.value)) {
          generateRandomPalette()
        }
      }
    }

    // Сброс палитры
    const resetPalette = () => {
      if (confirm('Вы уверены, что хотите сбросить текущую палитру?')) {
        colors.value = []
        generateRandomPalette()
        showNotification('Палитра сброшена')
      }
    }

    // Автоматическое сохранение
    const isAutoSaved = computed(() => {
      try {
        const saved = localStorage.getItem('colorPalette')
        if (!saved) return false
        
        const currentState = JSON.stringify({
          colors: colors.value,
          colorCount: colorCount.value,
          colorFormat: colorFormat.value,
          darkMode: darkMode.value
        })
        
        return saved === currentState
      } catch {
        return false
      }
    })

    // Наблюдатели для автоматического сохранения
    watch(colors, () => {
      saveToLocalStorage()
    }, { deep: true })

    watch([colorCount, colorFormat, darkMode], () => {
      saveToLocalStorage()
    })

    // При монтировании компонента
    onMounted(() => {
      loadFromLocalStorage()
    })

    return {
      colors,
      colorCount,
      colorFormat,
      darkMode,
      notification,
      paletteContainer,
      generateRandomPalette,
      copyToClipboard,
      toggleLock,
      getContrastColor,
      resetPalette,
      isAutoSaved,
      getColorFlex,
      handleColorCountChange
    }
  }
}
</script>

<style scoped>
.palette-generator {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.app-header {
  text-align: center;
  margin-bottom: 40px;
  color: white;
}

.app-header h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.app-header p {
  font-size: 1.2rem;
  opacity: 0.9;
}

.main-content {
  max-width: 1200px;
  margin: 0 auto;
  background-color: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
  min-height: 80vh;
}

.control-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 30px;
}

.generate-button {
  padding: 15px 30px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.generate-button:hover {
  transform: translateY(-2px);
}

.controls-group {
  display: flex;
  gap: 30px;
  flex-wrap: wrap;
}

.control-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.control-item label {
  font-weight: 600;
  color: #333;
}

.control-select {
  padding: 10px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  min-width: 150px;
  transition: border-color 0.3s ease;
}

.control-select:focus {
  outline: none;
  border-color: #667eea;
}

.toggle-switch {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
}

.toggle-input {
  display: none;
}

.toggle-slider {
  position: relative;
  width: 50px;
  height: 26px;
  background-color: #ccc;
  border-radius: 34px;
  transition: .4s;
}

.toggle-slider:before {
  content: "";
  position: absolute;
  height: 18px;
  width: 18px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  border-radius: 50%;
  transition: .4s;
}

.toggle-input:checked + .toggle-slider {
  background-color: #667eea;
}

.toggle-input:checked + .toggle-slider:before {
  transform: translateX(24px);
}

.palette-container-wrapper {
  height: 200px; 
  width: 100% !important;
  max-width: 100% !important;
  margin-bottom: 40px;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
  background-color: #f5f5f5;
  min-width: 100% !important;
  max-width: 100% !important;
  resize: none;
  flex-shrink: 0; 
  flex-grow: 0; 
}

.palette-container {
  display: flex;
  height: 100%;
  width: 100%;
  flex-wrap: nowrap;

}

.color-card {
  position: relative;
  cursor: pointer;
  transition: all 0.3s ease;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 15px;
  min-width: 0; /* Предотвращает выход за границы */
}

.color-card:hover {
  transform: scale(1.05);
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.2);
  z-index: 1;
}

.lock-indicator {
  position: absolute;
  top: 15px;
  right: 15px;
  font-size: 1.2rem;
  opacity: 0.9;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.color-info {
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  padding: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  backdrop-filter: blur(5px);
}

.color-value {
  font-family: 'Monaco', 'Courier New', monospace;
  font-weight: bold;
  color: #333;
  font-size: 0.9rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.copy-button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.2rem;
  padding: 5px;
  border-radius: 5px;
  transition: background-color 0.3s ease;
  flex-shrink: 0;
}

.copy-button:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background-color: #4CAF50;
  color: white;
  padding: 15px 25px;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
  transform: translateX(150%);
  transition: transform 0.3s ease;
  z-index: 1000;
}

.notification-visible {
  transform: translateX(0);
}

.preview-section {
  margin-top: 40px;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 15px;
}

.preview-section h3 {
  margin-bottom: 20px;
  color: #333;
}

.ui-preview {
  padding: 30px;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  align-items: flex-start;
}

.mockup-button {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.mockup-button:hover {
  transform: translateY(-2px);
}

.mockup-card {
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  max-width: 300px;
}

.mockup-title {
  margin-bottom: 10px;
  font-size: 1.2rem;
}

.mockup-text {
  font-size: 0.9rem;
  line-height: 1.5;
}

.mockup-header {
  font-size: 1.5rem;
  margin: 0;
}

.save-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.save-status {
  color: #666;
}

.save-status.saved {
  color: #4CAF50;
  font-weight: bold;
}

.reset-button {
  padding: 10px 20px;
  background-color: #ff6b6b;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.reset-button:hover {
  background-color: #ff5252;
}

/* Адаптивность */
@media (max-width: 768px) {
  .palette-container-wrapper {
    height: 180px; /* Немного меньше на мобильных */
  }
  
  .controls-group {
    flex-direction: column;
    gap: 15px;
  }
  
  .control-select {
    min-width: 100%;
  }
  
  .color-value {
    font-size: 0.8rem;
  }
  
  .save-info {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }
  
  .app-header h1 {
    font-size: 2rem;
  }
}

/* Анимация появления цветов */
@keyframes colorAppear {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.color-card {
  animation: colorAppear 0.5s ease-out;
}

.color-card:nth-child(1) { animation-delay: 0.1s; }
.color-card:nth-child(2) { animation-delay: 0.2s; }
.color-card:nth-child(3) { animation-delay: 0.3s; }
.color-card:nth-child(4) { animation-delay: 0.4s; }
.color-card:nth-child(5) { animation-delay: 0.5s; }
.color-card:nth-child(6) { animation-delay: 0.6s; }
.color-card:nth-child(7) { animation-delay: 0.7s; }
</style>