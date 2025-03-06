<script setup lang="ts">
import { ref, onUnmounted, computed, defineExpose, defineEmits } from 'vue';
import jsQR from 'jsqr';

// イベントの定義
const emit = defineEmits(['back']);

const videoElement = ref<HTMLVideoElement | null>(null);
const canvasElement = ref<HTMLCanvasElement | null>(null);
const fileInput = ref<HTMLInputElement | null>(null);
const qrResult = ref<string>('');
const scanning = ref<boolean>(false);
const errorMessage = ref<string>('');
const inputMethod = ref<'camera' | 'file' | null>(null);
const dragActive = ref<boolean>(false);
const scannedImageSrc = ref<string | null>(null);

// JWTかどうかを判定する
const isJwt = computed(() => {
  const jwt = qrResult.value;
  // JWTは通常、3つのセクション（ヘッダー、ペイロード、署名）がドットで区切られている
  return jwt && /^[A-Za-z0-9-_]+\.[A-Za-z0-9-_]+\.[A-Za-z0-9-_]*$/.test(jwt);
});

// JWTのヘッダーを解析する
const jwtHeader = computed(() => {
  if (!isJwt.value) return null;
  
  try {
    const [headerBase64] = qrResult.value.split('.');
    // Base64 URLエンコードをデコード
    const headerJson = atob(headerBase64.replace(/-/g, '+').replace(/_/g, '/'));
    return JSON.parse(headerJson);
  } catch (error) {
    console.error('JWTヘッダーの解析に失敗しました:', error);
    return null;
  }
});

// JWTのペイロードを解析する
const jwtPayload = computed(() => {
  if (!isJwt.value) return null;
  
  try {
    const [, payloadBase64] = qrResult.value.split('.');
    // Base64 URLエンコードをデコード
    const payloadJson = atob(payloadBase64.replace(/-/g, '+').replace(/_/g, '/'));
    return JSON.parse(payloadJson);
  } catch (error) {
    console.error('JWTペイロードの解析に失敗しました:', error);
    return null;
  }
});

// 日付フィールドかどうかをチェックする関数
const isDateField = (fieldName: string): boolean => {
  return ['exp', 'iat', 'nbf'].includes(fieldName);
};

// カメラストリームを保持する変数
let videoStream: MediaStream | null = null;

// 入力方法を選択する関数
const selectInputMethod = (method: 'camera' | 'file') => {
  resetState();
  inputMethod.value = method;
  
  if (method === 'camera') {
    startScanning();
  }
};

// 状態をリセットする関数
const resetState = () => {
  qrResult.value = '';
  errorMessage.value = '';
  scannedImageSrc.value = null;
  stopScanning();
};

// QRコードのスキャンを開始する関数
const startScanning = async () => {
  try {
    errorMessage.value = '';
    scanning.value = true;
    
    // カメラへのアクセスを要求
    videoStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'environment' }
    });
    
    if (videoElement.value) {
      videoElement.value.srcObject = videoStream;
      requestAnimationFrame(scanQRCode);
    }
  } catch (error) {
    console.error('カメラへのアクセスに失敗しました:', error);
    errorMessage.value = 'カメラへのアクセスに失敗しました。カメラの使用許可を確認してください。';
    scanning.value = false;
  }
};

// 親コンポーネントからカメラを起動する関数
const startCameraFromParent = () => {
  selectInputMethod('camera');
};

// QRコードのスキャンを停止する関数
const stopScanning = () => {
  scanning.value = false;
  if (videoStream) {
    videoStream.getTracks().forEach(track => track.stop());
    videoStream = null;
  }
  if (videoElement.value) {
    videoElement.value.srcObject = null;
  }
};

// QRコードをスキャンする関数
const scanQRCode = () => {
  if (!scanning.value) return;
  
  if (videoElement.value && canvasElement.value && videoElement.value.readyState === videoElement.value.HAVE_ENOUGH_DATA) {
    const canvas = canvasElement.value;
    const context = canvas.getContext('2d');
    
    if (context) {
      canvas.width = videoElement.value.videoWidth;
      canvas.height = videoElement.value.videoHeight;
      
      // 映像を左右反転して描画
      context.translate(canvas.width, 0);
      context.scale(-1, 1);
      context.drawImage(videoElement.value, 0, 0, canvas.width, canvas.height);
      // 座標系を元に戻す
      context.setTransform(1, 0, 0, 1, 0, 0);
      
      const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: 'dontInvert',
      });
      
      if (code) {
        qrResult.value = code.data;
        // カメラで撮影した画像を保存
        scannedImageSrc.value = canvas.toDataURL('image/png');
        stopScanning();
        return;
      }
    }
  }
  
  requestAnimationFrame(scanQRCode);
};

// 画像ファイルからQRコードを読み取る関数
const processImage = (file: File) => {
  console.log('processImage関数が呼ばれました:', file.name, 'タイプ:', file.type);
  
  // ファイルタイプが画像かどうかを確認
  if (!file.type.startsWith('image/')) {
    console.error('ファイルは画像ではありません:', file.type);
    errorMessage.value = '画像ファイルを選択してください。';
    return;
  }
  
  resetState();
  inputMethod.value = 'file';
  
  const reader = new FileReader();
  
  reader.onload = (e) => {
    console.log('ファイルの読み込みが完了しました');
    const img = new Image();
    
    img.onload = () => {
      console.log('画像の読み込みが完了しました:', img.width, 'x', img.height);
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      
      if (context) {
        canvas.width = img.width;
        canvas.height = img.height;
        context.drawImage(img, 0, 0, img.width, img.height);
        
        try {
          const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
          console.log('QRコードの検出を試みます...', imageData.width, 'x', imageData.height);
          
          const code = jsQR(imageData.data, imageData.width, imageData.height, {
            inversionAttempts: 'dontInvert',
          });
          
          if (code) {
            console.log('QRコードを検出しました:', code.data);
            qrResult.value = code.data;
            // 読み込んだ画像を保存
            scannedImageSrc.value = e.target?.result as string;
          } else {
            console.error('QRコードが見つかりませんでした');
            errorMessage.value = 'QRコードが見つかりませんでした。別の画像を試してください。';
          }
        } catch (error) {
          console.error('QRコード検出中にエラーが発生しました:', error);
          errorMessage.value = 'QRコードの検出中にエラーが発生しました。';
        }
      } else {
        console.error('キャンバスコンテキストを取得できませんでした');
        errorMessage.value = '画像の処理に失敗しました。';
      }
    };
    
    img.onerror = (error) => {
      console.error('画像の読み込みに失敗しました:', error);
      errorMessage.value = '画像の読み込みに失敗しました。';
    };
    
    // 画像のソースを設定
    if (e.target?.result) {
      img.src = e.target.result as string;
    } else {
      console.error('ファイル読み込み結果がnullです');
      errorMessage.value = 'ファイルの読み込みに失敗しました。';
    }
  };
  
  reader.onerror = (error) => {
    console.error('ファイルの読み込みに失敗しました:', error);
    errorMessage.value = 'ファイルの読み込みに失敗しました。';
  };
  
  try {
    reader.readAsDataURL(file);
  } catch (error) {
    console.error('ファイル読み込み開始時にエラーが発生しました:', error);
    errorMessage.value = 'ファイルの読み込みに失敗しました。';
  }
};

// 親コンポーネントからファイルを受け取る関数
const processImageFromParent = (file: File) => {
  console.log('processImageFromParent関数が呼ばれました:', file.name, 'タイプ:', file.type);
  
  // ファイルタイプの確認
  if (!file.type.startsWith('image/')) {
    console.error('親コンポーネントから受け取ったファイルは画像ではありません:', file.type);
    errorMessage.value = '画像ファイルを選択してください。';
    return;
  }
  
  // 状態をリセットしてから処理
  resetState();
  inputMethod.value = 'file';
  
  // 少し遅延させてから処理を実行（コンポーネントの描画を確実にするため）
  setTimeout(() => {
    processImage(file);
  }, 100);
};

// ファイル選択ハンドラ
const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files.length > 0) {
    processImage(target.files[0]);
  }
};

// ドラッグ&ドロップハンドラ
const handleDragOver = (event: DragEvent) => {
  event.preventDefault();
  dragActive.value = true;
};

const handleDragLeave = () => {
  dragActive.value = false;
};

const handleDrop = (event: DragEvent) => {
  event.preventDefault();
  dragActive.value = false;
  
  if (event.dataTransfer?.files && event.dataTransfer.files.length > 0) {
    console.log('QrReaderコンポーネント内でファイルがドロップされました:', event.dataTransfer.files[0].name);
    try {
      const file = event.dataTransfer.files[0];
      // ファイルタイプが画像かどうかを確認
      if (file.type.startsWith('image/')) {
        processImage(file);
      } else {
        console.error('ドロップされたファイルは画像ではありません:', file.type);
        errorMessage.value = '画像ファイルをドロップしてください。';
      }
    } catch (error) {
      console.error('ドロップされたファイルの処理中にエラーが発生しました:', error);
      errorMessage.value = 'ファイルの処理中にエラーが発生しました。';
    }
  }
};

// 再スキャンボタンのハンドラ
const resetScan = () => {
  resetState();
  if (inputMethod.value === 'camera') {
    startScanning();
  } else {
    inputMethod.value = null;
  }
};

// 戻るボタンのハンドラ
const goBack = () => {
  resetState();
  emit('back');
};

// コンポーネントがアンマウントされたときにスキャンを停止
onUnmounted(() => {
  stopScanning();
});

// 親コンポーネントに公開するメソッド
defineExpose({
  processImageFromParent,
  startCameraFromParent
});
</script>

<template>
  <div class="qr-reader">
    <div class="qr-reader-header">
      <button @click="goBack" class="back-button">
        <span class="back-icon">←</span> 戻る
      </button>
      <h2>QRコードリーダー</h2>
    </div>
    
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    
    <!-- 入力方法選択画面 -->
    <div v-if="!inputMethod && !qrResult" class="input-method-selector">
      <h3>QRコードの読み取り方法を選択してください</h3>
      <div class="method-buttons">
        <button @click="selectInputMethod('camera')" class="method-button">
          <span class="icon">📷</span>
          カメラで読み取る
        </button>
        <button @click="selectInputMethod('file')" class="method-button">
          <span class="icon">📁</span>
          画像ファイルから読み取る
        </button>
      </div>
    </div>
    
    <!-- カメラスキャン画面 -->
    <div v-if="inputMethod === 'camera' && !qrResult" class="scanner-container">
      <video ref="videoElement" autoplay playsinline class="camera-video"></video>
      <canvas ref="canvasElement" class="scanner-canvas"></canvas>
      <div class="scanner-overlay">
        <div class="scanner-frame"></div>
      </div>
    </div>
    
    <!-- ファイル選択・ドラッグ&ドロップ画面 -->
    <div 
      v-if="inputMethod === 'file' && !qrResult" 
      class="file-drop-area"
      :class="{ 'drag-active': dragActive }"
      @dragover="handleDragOver"
      @dragleave="handleDragLeave"
      @drop="handleDrop"
    >
      <div class="file-drop-content">
        <span class="icon large-icon">📄</span>
        <p>画像ファイルをドラッグ&ドロップするか、</p>
        <input 
          type="file" 
          ref="fileInput" 
          accept="image/*" 
          @change="handleFileSelect" 
          class="file-input"
        />
        <button @click="() => fileInput?.click()" class="file-select-button">
          ファイルを選択
        </button>
        <p class="file-hint">※ QRコードが含まれる画像を選択してください</p>
      </div>
    </div>
    
    <!-- 結果表示画面 -->
    <div v-if="qrResult" class="result-container">
      <h3>スキャン結果:</h3>
      
      <!-- スキャンした画像の表示 -->
      <div v-if="scannedImageSrc" class="scanned-image-container">
        <img :src="scannedImageSrc" alt="スキャンした画像" class="scanned-image" />
      </div>
      
      <div class="result-content">
        <p>{{ qrResult }}</p>
        <a v-if="qrResult.startsWith('http')" :href="qrResult" target="_blank" class="result-link">
          リンクを開く
        </a>
      </div>
      
      <!-- JWT解析結果 -->
      <div v-if="isJwt" class="jwt-container">
        <div class="jwt-header">
          <h4>JWT検出</h4>
        </div>
        
        <div class="jwt-content">
          <!-- ヘッダー -->
          <div class="jwt-section">
            <h5>ヘッダー</h5>
            <div class="jwt-data">
              <div v-for="(value, key) in jwtHeader" :key="`header-${key}`" class="jwt-field">
                <span class="jwt-key">{{ key }}:</span>
                <span class="jwt-value">{{ value }}</span>
              </div>
            </div>
          </div>
          
          <!-- ペイロード -->
          <div class="jwt-section">
            <h5>ペイロード</h5>
            <div class="jwt-data">
              <div v-for="(value, key) in jwtPayload" :key="`payload-${key}`" class="jwt-field">
                <span class="jwt-key">{{ key }}:</span>
                <span class="jwt-value">
                  <!-- 日付フィールドの場合は変換して表示 -->
                  <template v-if="isDateField(String(key))">
                    {{ typeof value === 'number' ? new Date(value * 1000).toLocaleString('ja-JP') : String(value) }}
                    <span v-if="typeof value === 'number'" class="jwt-timestamp">({{ value }})</span>
                  </template>
                  <template v-else>{{ value }}</template>
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <div class="result-actions">
        <button @click="resetScan" class="scan-button">再スキャン</button>
        <button @click="goBack" class="scan-button secondary">
          トップに戻る
        </button>
      </div>
    </div>
    
    <div v-if="inputMethod === 'camera' && !qrResult && !scanning" class="controls">
      <button @click="startScanning" class="scan-button">スキャン開始</button>
    </div>
  </div>
</template>

<style scoped>
.qr-reader {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
}

.qr-reader-header {
  display: flex;
  align-items: center;
  width: 100%;
  margin-bottom: 20px;
  position: relative;
}

.qr-reader-header h2 {
  flex: 1;
  text-align: center;
  margin: 0;
}

.back-button {
  display: flex;
  align-items: center;
  background: none;
  border: none;
  color: #42b883;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  padding: 8px 12px;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.back-button:hover {
  background-color: rgba(66, 184, 131, 0.1);
}

.back-icon {
  margin-right: 5px;
  font-size: 1.2em;
}

.input-method-selector {
  width: 100%;
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
  margin: 20px 0;
  text-align: center;
}

.method-buttons {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-top: 20px;
}

.method-button {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 15px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: background-color 0.3s;
}

.method-button:hover {
  background-color: #3aa876;
}

.icon {
  margin-right: 10px;
  font-size: 1.2em;
}

.large-icon {
  font-size: 3em;
  margin-bottom: 15px;
}

.scanner-container {
  position: relative;
  width: 100%;
  max-width: 300px;
  height: 300px;
  overflow: hidden;
  border-radius: 8px;
  margin: 20px 0;
}

.file-drop-area {
  width: 100%;
  height: 300px;
  border: 2px dashed #ccc;
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px 0;
  transition: all 0.3s;
}

.file-drop-area.drag-active {
  border-color: #42b883;
  background-color: rgba(66, 184, 131, 0.1);
}

.file-drop-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 20px;
}

.file-input {
  display: none;
}

.file-select-button {
  margin-top: 15px;
  padding: 10px 20px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: background-color 0.3s;
}

.file-select-button:hover {
  background-color: #3aa876;
}

.file-hint {
  margin-top: 10px;
  font-size: 0.9em;
  color: #6c757d;
  font-style: italic;
}

.scanned-image-container {
  width: 100%;
  margin: 15px 0;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.scanned-image {
  width: 100%;
  height: auto;
  display: block;
}

.jwt-container {
  width: 100%;
  margin: 15px 0;
  border-radius: 8px;
  background-color: #f8f9fa;
  border: 1px solid #ddd;
  overflow: hidden;
}

.jwt-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  background-color: #e9ecef;
  border-bottom: 1px solid #ddd;
}

.jwt-header h4 {
  margin: 0;
  color: #495057;
}

.jwt-content {
  padding: 15px;
}

.jwt-section {
  margin-bottom: 15px;
}

.jwt-section h5 {
  margin: 0 0 10px 0;
  color: #495057;
  border-bottom: 1px solid #dee2e6;
  padding-bottom: 5px;
}

.jwt-data {
  background-color: white;
  border-radius: 4px;
  padding: 10px;
  border: 1px solid #dee2e6;
}

.jwt-field {
  margin-bottom: 8px;
  line-height: 1.4;
}

.jwt-key {
  font-weight: bold;
  color: #495057;
  margin-right: 5px;
}

.jwt-value {
  word-break: break-all;
}

.jwt-timestamp {
  font-size: 0.8em;
  color: #6c757d;
  margin-left: 5px;
}

.camera-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transform: scaleX(-1); /* 左右反転 */
}

video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.scanner-canvas {
  display: none;
}

.scanner-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.2);
}

.scanner-frame {
  width: 70%;
  height: 70%;
  border: 2px solid #42b883;
  border-radius: 8px;
  box-shadow: 0 0 0 100vmax rgba(0, 0, 0, 0.3);
}

.result-container {
  width: 100%;
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
  margin-top: 20px;
  text-align: center;
}

.result-content {
  word-break: break-all;
  margin: 15px 0;
  padding: 10px;
  background-color: white;
  border-radius: 4px;
  border: 1px solid #ddd;
}

.result-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 15px;
}

.result-link {
  display: inline-block;
  margin-top: 10px;
  padding: 8px 16px;
  background-color: #42b883;
  color: white;
  text-decoration: none;
  border-radius: 4px;
  font-weight: bold;
}

.scan-button {
  padding: 10px 20px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: background-color 0.3s;
}

.scan-button:hover {
  background-color: #3aa876;
}

.scan-button.secondary {
  background-color: #6c757d;
}

.scan-button.secondary:hover {
  background-color: #5a6268;
}

.error-message {
  color: #e74c3c;
  background-color: #fdecea;
  padding: 10px;
  border-radius: 4px;
  margin: 10px 0;
  width: 100%;
  text-align: center;
}

.camera-hint {
  position: absolute;
  bottom: 10px;
  left: 0;
  right: 0;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 5px;
  font-size: 0.8em;
  border-radius: 0 0 8px 8px;
}

.hint-icon {
  margin-right: 5px;
}

@media (prefers-color-scheme: dark) {
  .input-method-selector,
  .result-container {
    background-color: #333;
  }
  
  .result-content {
    background-color: #444;
    border-color: #555;
  }
  
  .file-drop-area {
    border-color: #555;
  }
  
  .file-drop-area.drag-active {
    background-color: rgba(66, 184, 131, 0.2);
  }
  
  .scanned-image-container {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  }
  
  .jwt-container {
    background-color: #2d3033;
    border-color: #444;
  }
  
  .jwt-header {
    background-color: #383c40;
    border-color: #444;
  }
  
  .jwt-header h4,
  .jwt-section h5 {
    color: #e9ecef;
  }
  
  .jwt-section h5 {
    border-color: #444;
  }
  
  .jwt-data {
    background-color: #444;
    border-color: #555;
  }
  
  .jwt-key {
    color: #e9ecef;
  }
  
  .jwt-timestamp {
    color: #adb5bd;
  }
}
</style> 