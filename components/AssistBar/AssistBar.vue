<template>
  <div>
    <div id="accessibility-toolbar" ref="toolbar">
      <button @click="increaseTextSize">增大字体</button>
      <button @click="decreaseTextSize">缩小字体</button>
      <button @click="increaseSpeechRate">加快朗读语速</button>
      <button @click="decreaseSpeechRate">减慢朗读语速</button>
      <button @click="toggleTextReader">{{ textReaderButtonText }}</button>
      <button @click="toggleCrosshair">{{ crosshairButtonText }}</button>
      <button @click="toggleCaption">{{ captionButtonText }}</button>
      <button @click="toggleContrast">切换高对比度</button>
      <button @click="resetSettings">重置配置</button>
    </div>
    <div v-if="caption && isCaptionActive" id="caption-display">{{ caption }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentFontSize: parseFloat(window.getComputedStyle(document.documentElement).fontSize),
      speechRate: 1,
      isHighContrast: 0, // 0: 关闭, 1: 黑底白字, 2: 蓝底黄字
      isCrosshairActive: false,
      isCaptionActive: false,
      isTextReaderActive: false,
      caption: '',
      originalSettings: {
        fontSize: parseFloat(window.getComputedStyle(document.documentElement).fontSize),
        speechRate: 1,
        isHighContrast: 0,
        isCrosshairActive: false,
        isCaptionActive: false,
        isTextReaderActive: false,
      }
    };
  },
  computed: {
    textReaderButtonText() {
      return this.isTextReaderActive ? '关闭文本朗读' : '开启文本朗读';
    },
    crosshairButtonText() {
      return this.isCrosshairActive ? '隐藏十字辅助线' : '显示十字辅助线';
    },
    captionButtonText() {
      return this.isCaptionActive ? '隐藏字幕' : '显示字幕';
    }
  },
  mounted() {
    // 调整页面顶部空间
    document.body.style.paddingTop = `${this.$refs.toolbar.offsetHeight}px`;
  },
  methods: {
    increaseTextSize() {
      this.currentFontSize += 2;
      document.documentElement.style.fontSize = `${this.currentFontSize}px`;
    },
    decreaseTextSize() {
      this.currentFontSize -= 2;
      document.documentElement.style.fontSize = `${this.currentFontSize}px`;
    },
    increaseSpeechRate() {
      this.speechRate = Math.min(this.speechRate + 0.2, 2);
      this.speakText("语速已加快");
    },
    decreaseSpeechRate() {
      this.speechRate = Math.max(this.speechRate - 0.2, 0.5);
      this.speakText("语速已减慢");
    },
    speakText(text) {
      speechSynthesis.cancel(); // 打断当前的朗读
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.rate = this.speechRate;
      speechSynthesis.speak(utterance);
    },
    toggleTextReader() {
      this.isTextReaderActive = !this.isTextReaderActive;
      if (this.isTextReaderActive) {
        document.body.addEventListener('mousemove', this.readTextOnHover);
      } else {
        document.body.removeEventListener('mousemove', this.readTextOnHover);
        speechSynthesis.cancel(); // 停止当前的朗读
      }
    },
    readTextOnHover(event) {
      const element = event.target;
      const elementType = element.tagName.toLowerCase();
      const text = element.innerText || element.alt || element.title || '';
      const readableText = `${this.getElementDescription(elementType)}，内容：${text}`;
      if (readableText) {
        this.speakText(readableText);
      }
    },
    getElementDescription(tag) {
      const descriptions = {
        p: '段落',
        h1: '标题一',
        h2: '标题二',
        h3: '标题三',
        h4: '标题四',
        h5: '标题五',
        h6: '标题六',
        button: '按钮',
        a: '链接',
        img: '图片',
        div: '区块',
        span: '文本'
      };
      return descriptions[tag] || '元素';
    },
    toggleCrosshair() {
      this.isCrosshairActive = !this.isCrosshairActive;
      if (this.isCrosshairActive) {
        document.body.addEventListener('mousemove', this.showCrosshair);
      } else {
        document.body.removeEventListener('mousemove', this.showCrosshair);
        this.removeCrosshair();
      }
    },
    showCrosshair(event) {
      const horizontalLine = document.getElementById('horizontal-line') || document.createElement('div');
      const verticalLine = document.getElementById('vertical-line') || document.createElement('div');

      horizontalLine.id = 'horizontal-line';
      verticalLine.id = 'vertical-line';

      horizontalLine.style.cssText = `position:fixed; top:${event.clientY}px; left:0; width:100%; height:1px; background:red; z-index:9999; pointer-events:none;`;
      verticalLine.style.cssText = `position:fixed; left:${event.clientX}px; top:0; height:100%; width:1px; background:red; z-index:9999; pointer-events:none;`;

      document.body.appendChild(horizontalLine);
      document.body.appendChild(verticalLine);
    },
    removeCrosshair() {
      const horizontalLine = document.getElementById('horizontal-line');
      const verticalLine = document.getElementById('vertical-line');
      if (horizontalLine) document.body.removeChild(horizontalLine);
      if (verticalLine) document.body.removeChild(verticalLine);
    },
    toggleCaption() {
      this.isCaptionActive = !this.isCaptionActive;
      if (this.isCaptionActive) {
        document.body.addEventListener('mousemove', this.updateCaption);
      } else {
        document.body.removeEventListener('mousemove', this.updateCaption);
        this.caption = '';
      }
    },
    updateCaption(event) {
      this.caption = event.target.innerText || event.target.alt || event.target.title || '无文本';
    },
    toggleContrast() {
      this.isHighContrast = (this.isHighContrast + 1) % 3;
      document.body.classList.remove('high-contrast1', 'high-contrast2');
      if (this.isHighContrast === 1) {
        document.body.classList.add('high-contrast1'); // 黑底黄字
      } else if (this.isHighContrast === 2) {
        document.body.classList.add('high-contrast2'); // 白底黑字
      }
    },
    resetSettings() {
      // 恢复字体大小
      this.currentFontSize = this.originalSettings.fontSize;
      document.documentElement.style.fontSize = `${this.currentFontSize}px`;

      // 恢复朗读语速
      this.speechRate = this.originalSettings.speechRate;

      // 恢复高对比度模式
      this.isHighContrast = this.originalSettings.isHighContrast;
      document.body.classList.remove('high-contrast1', 'high-contrast2');
      if (this.isHighContrast === 1) {
        document.body.classList.add('high-contrast1');
      } else if (this.isHighContrast === 2) {
        document.body.classList.add('high-contrast2');
      }

      // 关闭十字辅助线
      if (this.isCrosshairActive) {
        document.body.removeEventListener('mousemove', this.showCrosshair);
        this.removeCrosshair();
        this.isCrosshairActive = false; // 更新状态
      }

      // 关闭字幕显示
      if (this.isCaptionActive) {
        document.body.removeEventListener('mousemove', this.updateCaption);
        this.caption = '';
        this.isCaptionActive = false; // 更新状态
      }

      // 关闭文本朗读
      if (this.isTextReaderActive) {
        document.body.removeEventListener('mousemove', this.readTextOnHover);
        speechSynthesis.cancel(); // 停止当前的朗读
        this.isTextReaderActive = false; // 更新状态
      }

      this.speakText("设置已重置");
    }
  }
};
</script>

<style scoped>
#accessibility-toolbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background-color: #f1f1f1;
  padding: 10px;
  display: flex;
  justify-content: center;
  z-index: 1000;
  transition: opacity 0.3s;
}

#accessibility-toolbar button {
  margin: 0 5px;
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}

#caption-display {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  text-align: center;
  padding: 5px;
  z-index: 1000;
}

#horizontal-line, #vertical-line {
  pointer-events: none;
}
</style>
<style>
html body.high-contrast1 #app {
  background-color: black !important;
  color: orange !important;
}

html body.high-contrast2 #app {
  background-color: blue !important;
  color: yellow !important;
}
</style>