<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue';
import QrReader from './components/QrReader.vue'

const qrReaderRef = ref<InstanceType<typeof QrReader> | null>(null);
const fileInput = ref<HTMLInputElement | null>(null);
const dragActive = ref<boolean>(false);
const showQrReader = ref<boolean>(false);

// コンポーネントがマウントされた後に実行
onMounted(() => {
  console.log('Appコンポーネントがマウントされました');
});

// カメラ読み取りを開始する
const startCamera = () => {
  showQrReader.value = true;
  
  // nextTickを使用してDOMの更新後に処理を実行
  nextTick(() => {
    // さらにタイムアウトを設定して確実にコンポーネントが描画されるようにする
    setTimeout(() => {
      console.log('QrReaderコンポーネントの参照:', qrReaderRef.value ? '存在します' : '存在しません');
      if (qrReaderRef.value) {
        console.log('カメラを起動します');
        qrReaderRef.value.startCameraFromParent();
      } else {
        console.error('QrReaderコンポーネントが見つかりません');
      }
    }, 300);
  });
};

// ファイル選択ハンドラ
const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files.length > 0) {
    try {
      const file = target.files[0];
      console.log('ファイルが選択されました:', file.name, 'タイプ:', file.type);
      
      // ファイルタイプが画像かどうかを確認
      if (!file.type.startsWith('image/')) {
        console.error('選択されたファイルは画像ではありません');
        return;
      }
      
      showQrReader.value = true;
      
      // nextTickを使用してDOMの更新後に処理を実行
      nextTick(() => {
        // さらにタイムアウトを設定して確実にコンポーネントが描画されるようにする
        setTimeout(() => {
          console.log('QrReaderコンポーネントの参照:', qrReaderRef.value ? '存在します' : '存在しません');
          if (qrReaderRef.value) {
            console.log('QrReaderコンポーネントにファイルを渡します');
            qrReaderRef.value.processImageFromParent(file);
          } else {
            console.error('QrReaderコンポーネントが見つかりません');
          }
        }, 500);
      });
    } catch (error) {
      console.error('選択されたファイルの処理中にエラーが発生しました:', error);
    }
  }
};

// ドラッグ&ドロップハンドラ
const handleDragOver = (event: DragEvent) => {
  event.preventDefault();
  dragActive.value = true;
};

const handleDragLeave = (event: DragEvent) => {
  event.preventDefault();
  const relatedTarget = event.relatedTarget as Node;
  // ドロップエリア内の要素にドラッグした場合は無視
  if (event.currentTarget && event.currentTarget.contains(relatedTarget)) {
    return;
  }
  dragActive.value = false;
};

const handleDrop = (event: DragEvent) => {
  event.preventDefault();
  dragActive.value = false;
  
  if (event.dataTransfer?.files && event.dataTransfer.files.length > 0) {
    try {
      const file = event.dataTransfer.files[0];
      console.log('ファイルがドロップされました:', file.name, 'タイプ:', file.type);
      
      // ファイルタイプが画像かどうかを確認
      if (!file.type.startsWith('image/')) {
        console.error('ドロップされたファイルは画像ではありません');
        return;
      }
      
      showQrReader.value = true;
      
      // nextTickを使用してDOMの更新後に処理を実行
      nextTick(() => {
        // さらにタイムアウトを設定して確実にコンポーネントが描画されるようにする
        setTimeout(() => {
          console.log('QrReaderコンポーネントの参照:', qrReaderRef.value ? '存在します' : '存在しません');
          if (qrReaderRef.value) {
            console.log('QrReaderコンポーネントにドロップされたファイルを渡します');
            qrReaderRef.value.processImageFromParent(file);
          } else {
            console.error('QrReaderコンポーネントが見つかりません');
          }
        }, 500);
      });
    } catch (error) {
      console.error('ドロップされたファイルの処理中にエラーが発生しました:', error);
    }
  }
};

// QrReaderコンポーネントから戻るイベントハンドラ
const handleBack = () => {
  showQrReader.value = false;
};
</script>

<template>
  <div class="app-container">
    <header>
      <h1>QRコードリーダー</h1>
      <p>QRコードを簡単に読み取ることができます</p>
    </header>
    
    <main>
      <!-- QRリーダーコンポーネント -->
      <QrReader 
        v-if="showQrReader" 
        ref="qrReaderRef" 
        @back="handleBack"
      />
      
      <!-- 入力方法選択画面 -->
      <div v-else class="method-selector">
        <!-- カメラで読み取る -->
        <div class="method-card">
          <div class="method-icon">📷</div>
          <h3>カメラで読み取る</h3>
          <p>デバイスのカメラを使用してQRコードをスキャンします</p>
          <button @click="startCamera" class="method-button camera-button">
            カメラを起動
          </button>
        </div>
        
        <!-- ファイル選択で読み取る -->
        <div class="method-card">
          <div class="method-icon">📁</div>
          <h3>ファイルから読み取る</h3>
          <p>保存された画像からQRコードを読み取ります</p>
          <input 
            type="file" 
            ref="fileInput" 
            accept="image/*" 
            @change="handleFileSelect" 
            class="file-input"
          />
          <button @click="() => fileInput?.click()" class="method-button file-button">
            ファイルを選択
          </button>
        </div>
        
        <!-- ドラッグ&ドロップで読み取る -->
        <div 
          class="method-card drop-zone"
          :class="{ 'drag-active': dragActive }"
          @dragover="handleDragOver"
          @dragleave="handleDragLeave"
          @drop="handleDrop"
        >
          <div class="method-icon">📄</div>
          <h3>ドラッグ&ドロップ</h3>
          <p>画像ファイルをここにドロップしてQRコードを読み取ります</p>
          <div class="drop-message">
            ここにドロップ
          </div>
        </div>
      </div>
    </main>
    
    <footer>
      <p>© 2024 QRコードリーダーWebアプリ</p>
      <p>
        <a href="https://github.com/SatoshiIwasada/qrreader-webapp" target="_blank">GitHub</a>
      </p>
    </footer>
  </div>
</template>

<style>
:root {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  
  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;
  
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  margin: 0;
  display: flex;
  place-items: center;
  min-width: 320px;
  min-height: 100vh;
}

#app {
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: 20px;
  position: relative;
}

header {
  margin-bottom: 40px;
  text-align: center;
}

h1 {
  font-size: 2.5em;
  line-height: 1.1;
  margin-bottom: 0.5em;
  color: #42b883;
}

main {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 20px 0;
}

.method-selector {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  width: 100%;
  max-width: 1000px;
}

.method-card {
  flex: 1;
  min-width: 250px;
  max-width: 300px;
  background-color: #2d3033;
  border-radius: 12px;
  padding: 25px;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.method-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.method-icon {
  font-size: 3em;
  margin-bottom: 15px;
}

.method-card h3 {
  margin: 0 0 10px 0;
  font-size: 1.5em;
  color: #42b883;
}

.method-card p {
  margin-bottom: 20px;
  color: #adb5bd;
  flex-grow: 1;
}

.method-button {
  width: 100%;
  padding: 12px 0;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s;
}

.camera-button {
  background-color: #42b883;
  color: white;
}

.camera-button:hover {
  background-color: #3aa876;
}

.file-button {
  background-color: #6c757d;
  color: white;
}

.file-button:hover {
  background-color: #5a6268;
}

.file-input {
  display: none;
}

.drop-zone {
  border: 2px dashed #6c757d;
  position: relative;
  cursor: pointer;
}

.drop-zone.drag-active {
  border-color: #42b883;
  background-color: rgba(66, 184, 131, 0.1);
}

.drop-message {
  width: 100%;
  padding: 12px 0;
  background-color: rgba(108, 117, 125, 0.2);
  border-radius: 8px;
  font-weight: bold;
  color: #adb5bd;
}

.drop-zone.drag-active .drop-message {
  background-color: rgba(66, 184, 131, 0.3);
  color: #42b883;
}

footer {
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid #444;
  font-size: 0.8em;
  color: #888;
  text-align: center;
}

footer a {
  color: #42b883;
  text-decoration: none;
}

footer a:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .method-selector {
    flex-direction: column;
    align-items: center;
  }
  
  .method-card {
    width: 100%;
    max-width: 100%;
  }
}

@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
  
  .method-card {
    background-color: #f8f9fa;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  }
  
  .method-card p {
    color: #6c757d;
  }
  
  .drop-message {
    color: #6c757d;
  }
  
  footer {
    border-top-color: #ddd;
  }
}
</style>
