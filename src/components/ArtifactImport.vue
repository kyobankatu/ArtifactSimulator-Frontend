<template>
  <div class="artifact-import-header">
    <p class="artifact-import-title">聖遺物インポート</p>
    <button class="info-icon" @click="$emit('show-info')">
      <img src="../assets/info.webp" alt="Info"/>
    </button>
  </div>
  
  <div class="img-upload-area">
    <!-- ファイル選択を隠してラベルをクリック可能にする -->
    <label for="artifact-select" class="def-button artifact-import-label">ファイルを選択</label>
    <input type="file" id="artifact-select" @change="$emit('file-change', $event)" accept="image/png,image/jpeg,image/webp"/>
    <input type="text" :value="fileName" class="artifact-img-name" readonly placeholder="ファイル名" />
    <button class="def-button artifact-import-scan" @click="$emit('scan-image')">スキャン</button>
  </div>
</template>

<script>
export default {
  name: "ArtifactImport",
  props: {
    fileName: String
  },
  emits: ["file-change", "scan-image"]
}
</script>

<style scoped>
.artifact-import-header {
  display: flex;
  align-items: center;
  justify-items: center;
}

.artifact-import-title {
  font-size: min(calc(14vw / 5), 20px);
  color: #3a404f;
}

.artifact-import-label {
  font-size: min(calc(9vw / 5), 16px);
}

#artifact-select {
  display: none;
}

.artifact-img-name {
  padding: 10px;
  width: min(calc(200vw / 5), 250px);
  
  font-size: min(calc(9vw / 5), 16px);
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f9f9f9;
  cursor: not-allowed;
}

.artifact-import-scan {
  font-size: min(calc(9vw / 5), 16px);
}

.img-upload-area {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}
</style>
