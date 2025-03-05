<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import jsQR from 'jsqr';

const videoElement = ref<HTMLVideoElement | null>(null);
const canvasElement = ref<HTMLCanvasElement | null>(null);
const qrResult = ref<string>('');
const scanning = ref<boolean>(false);
const errorMessage = ref<string>('');

// カメラストリームを保持する変数
let videoStream: MediaStream | null = null;

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
      context.drawImage(videoElement.value, 0, 0, canvas.width, canvas.height);
      
      const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: 'dontInvert',
      });
      
      if (code) {
        qrResult.value = code.data;
        stopScanning();
        return;
      }
    }
  }
  
  requestAnimationFrame(scanQRCode);
};

// 再スキャンボタンのハンドラ
const resetScan = () => {
  qrResult.value = '';
  startScanning();
};

// コンポーネントがマウントされたときにスキャンを開始
onMounted(() => {
  startScanning();
});

// コンポーネントがアンマウントされたときにスキャンを停止
onUnmounted(() => {
  stopScanning();
});
</script>

<template>
  <div class="qr-reader">
    <h2>QRコードリーダー</h2>
    
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    
    <div v-if="!qrResult" class="scanner-container">
      <video ref="videoElement" autoplay playsinline></video>
      <canvas ref="canvasElement" class="scanner-canvas"></canvas>
      <div class="scanner-overlay">
        <div class="scanner-frame"></div>
      </div>
    </div>
    
    <div v-if="qrResult" class="result-container">
      <h3>スキャン結果:</h3>
      <div class="result-content">
        <p>{{ qrResult }}</p>
        <a v-if="qrResult.startsWith('http')" :href="qrResult" target="_blank" class="result-link">
          リンクを開く
        </a>
      </div>
      <button @click="resetScan" class="scan-button">再スキャン</button>
    </div>
    
    <div v-if="!qrResult && !scanning" class="controls">
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

.scanner-container {
  position: relative;
  width: 100%;
  max-width: 300px;
  height: 300px;
  overflow: hidden;
  border-radius: 8px;
  margin: 20px 0;
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

.scan-button:hover {
  background-color: #3aa876;
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
</style> 