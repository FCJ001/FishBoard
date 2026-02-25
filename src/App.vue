<script setup>
import { ref } from 'vue'
import FishboneDiagram from './components/FishboneDiagram.vue'

const fishboneData = ref({
  problem: '项目延期',
  nodes: [
    {
      id: 1,
      top: { id: 101, name: '人员', items: [{ id: 11, name: '人手不足' }, { id: 12, name: '技能不匹配' }] },
      bottom: { id: 201, name: '方法', items: [{ id: 31, name: '需求变更频繁' }] }
    },
    {
      id: 2,
      top: { id: 102, name: '机器', items: [{ id: 21, name: '服务器宕机' }] },
      bottom: { id: 202, name: '环境', items: [{ id: 41, name: '办公室噪音' }] }
    }
  ]
})
</script>

<template>
  <div class="app-container">
    <h1>鱼骨图</h1>
    <div class="diagram-wrapper">
      <FishboneDiagram v-model="fishboneData" />
    </div>
    
    <div class="data-preview">
      <h3>数据预览:</h3>
      <pre>{{ JSON.stringify(fishboneData, null, 2) }}</pre>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.diagram-wrapper {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  background: white;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  margin-bottom: 30px;
  overflow-x: auto;
}

.data-preview {
  background: #f4f4f4;
  padding: 15px;
  border-radius: 8px;
  text-align: left;
}

pre {
  white-space: pre-wrap;
  font-size: 0.85em;
  color: #555;
}
</style>
