<script setup lang="ts">
import QrReader from './components/QrReader.vue'
</script>

<template>
  <div class="app-container">
    <header>
      <h1>QRコードリーダー</h1>
      <p>カメラを使ってQRコードを読み取ります</p>
    </header>
    
    <main>
      <QrReader />
    </main>
    
    <footer>
      <p>© 2024 QRコードリーダーWebアプリ</p>
      <p>
        <a href="https://github.com/yourusername/qrreader-webapp" target="_blank">GitHub</a>
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
}

header {
  margin-bottom: 30px;
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

footer {
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid #444;
  font-size: 0.8em;
  color: #888;
}

footer a {
  color: #42b883;
  text-decoration: none;
}

footer a:hover {
  text-decoration: underline;
}

@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
  footer {
    border-top-color: #ddd;
  }
}
</style>
