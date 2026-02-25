<script setup>
import { ref, watch, computed } from 'vue'
import { IconPlus, IconClose, IconDelete } from '@arco-design/web-vue/es/icon'

const props = defineProps({
  modelValue: {
    type: Object,
    default: () => ({
      problem: '此处填写问题',
      nodes: [
        {
          id: 1,
          top: { id: 101, name: '人', items: [{ id: 11, name: '技能不足' }] },
          bottom: { id: 201, name: '法', items: [{ id: 41, name: '流程繁琐' }] }
        }
      ]
    })
  }
})

const emit = defineEmits(['update:modelValue'])

const fishboneData = ref(JSON.parse(JSON.stringify(props.modelValue)))

watch(fishboneData, (newValue) => {
  emit('update:modelValue', newValue)
}, { deep: true })

watch(() => props.modelValue, (newVal) => {
  if (JSON.stringify(newVal) !== JSON.stringify(fishboneData.value)) {
    fishboneData.value = JSON.parse(JSON.stringify(newVal))
  }
}, { deep: true })

// 添加一个新的主骨节点（包含上下空槽位）到尾部（左侧）
const addNode = () => {
  if (!fishboneData.value.nodes) {
    fishboneData.value.nodes = []
  }
  fishboneData.value.nodes.unshift({
    id: Date.now(),
    top: null,
    bottom: null
  })
}

// 在节点的指定位置添加分类
const addCategoryToNode = (node, position) => {
  node[position] = {
    id: Date.now(),
    name: '新分类',
    items: []
  }
}

// 移除节点指定位置的分类
const removeCategoryFromNode = (node, position) => {
  node[position] = null
}

// 移除整个节点
const removeNode = (index) => {
  fishboneData.value.nodes.splice(index, 1)
}

const addItem = (category) => {
  category.items.push({
    id: Date.now(),
    name: '新原因'
  })
}

const removeItem = (category, index) => {
  category.items.splice(index, 1)
}

const getItemStyle = (index, position) => {
  // 基础间距
  const baseSpacing = 20;
  const startOffset = 15;
  const offset = startOffset + (index * baseSpacing);
  
  // 计算水平偏移量以保持在斜线上
  // tan(30) 约为 0.577
  const x = offset * 0.577; 
  
  if (position === 'top') {
    // 上方骨线 (/)
    return {
      bottom: `${offset}px`,
      left: '50%',
      transform: `translateX(${x - 2}px) translateX(-100%)`, 
      width: '60px'
    }
  } else {
    // 下方骨线 (\)
    return {
      top: `${offset}px`,
      left: '50%',
      transform: `translateX(${-x - 2}px) translateX(-100%)`, 
      width: '60px'
    }
  }
}

/* 根据项目数量计算斜线高度 */
const getDiagonalHeight = (itemCount) => {
  const baseHeight = 100;
  const itemSpacing = 20; 
  const requiredHeight = (itemCount * 20) + 40;
  return Math.max(baseHeight, requiredHeight) + 'px';
}
</script>

<template>
  <div class="fishbone-container">
    <div class="fishbone-tail">
      <a-button type="primary" shape="circle" size="large" @click="addNode" title="添加主骨节点">
        <template #icon><icon-plus /></template>
      </a-button>
    </div>

    <div class="fishbone-body">
      <!-- Spine Background Line -->
      <div class="spine">
        <div class="spine-line"></div>
        <div class="spine-arrow"></div>
      </div>

      <!-- Nodes Container -->
      <div class="nodes-container">
        <div v-for="(node, index) in fishboneData.nodes" :key="node.id" class="spine-node">
          
          <!-- Top Slot -->
          <div class="node-slot top">
            <template v-if="node.top">
              <div class="category-group">
                <!-- Category Header -->
                <div class="category-header">
                  <a-input v-model="node.top.name" size="small" placeholder="分类名称" class="category-input" />
                  <div class="header-actions">
                    <a-button type="text" shape="circle" size="mini" @click="addItem(node.top)" title="添加原因">
                      <template #icon><icon-plus /></template>
                    </a-button>
                    <a-button type="text" status="danger" shape="circle" size="mini" @click="removeCategoryFromNode(node, 'top')">
                      <template #icon><icon-delete /></template>
                    </a-button>
                  </div>
                </div>

                <!-- Diagonal Line -->
                <div class="diagonal-line" :style="{ height: getDiagonalHeight(node.top.items.length) }"></div>
                
                <!-- Items List -->
                <div class="items-list">
                  <div v-for="(item, i) in node.top.items" :key="item.id" class="item-row" :style="getItemStyle(i, 'top')">
                    <a-input v-model="item.name" size="mini" class="item-input" />
                    <a-button type="text" status="danger" shape="circle" size="mini" @click="removeItem(node.top, i)">
                      <template #icon><icon-close /></template>
                    </a-button>
                  </div>
                </div>
              </div>
            </template>
            <template v-else>
              <div class="add-slot-btn">
                <a-button type="dashed" shape="circle" size="small" @click="addCategoryToNode(node, 'top')">
                  <template #icon><icon-plus /></template>
                </a-button>
              </div>
            </template>
          </div>

          <!-- Spine Point / Center -->
          <div class="spine-point">
             <!-- Only show delete button if both slots are empty -->
             <div v-if="!node.top && !node.bottom" class="delete-node-wrapper" @click="removeNode(index)">
                <icon-close class="delete-node-icon" />
             </div>
             <!-- Otherwise nothing -->
          </div>

          <!-- Bottom Slot -->
          <div class="node-slot bottom">
            <template v-if="node.bottom">
              <div class="category-group">
                <!-- Diagonal Line -->
                <div class="diagonal-line" :style="{ height: getDiagonalHeight(node.bottom.items.length) }"></div>
                
                <!-- Category Header -->
                <div class="category-header">
                  <a-input v-model="node.bottom.name" size="small" placeholder="分类名称" class="category-input" />
                  <div class="header-actions">
                    <a-button type="text" shape="circle" size="mini" @click="addItem(node.bottom)" title="添加原因">
                      <template #icon><icon-plus /></template>
                    </a-button>
                    <a-button type="text" status="danger" shape="circle" size="mini" @click="removeCategoryFromNode(node, 'bottom')">
                      <template #icon><icon-delete /></template>
                    </a-button>
                  </div>
                </div>

                <!-- Items List -->
                <div class="items-list">
                  <div v-for="(item, i) in node.bottom.items" :key="item.id" class="item-row" :style="getItemStyle(i, 'bottom')">
                    <a-input v-model="item.name" size="mini" class="item-input" />
                    <a-button type="text" status="danger" shape="circle" size="mini" @click="removeItem(node.bottom, i)">
                      <template #icon><icon-close /></template>
                    </a-button>
                  </div>
                </div>
              </div>
            </template>
            <template v-else>
              <div class="add-slot-btn">
                <a-button type="dashed" shape="circle" size="small" @click="addCategoryToNode(node, 'bottom')">
                  <template #icon><icon-plus /></template>
                </a-button>
              </div>
            </template>
          </div>

        </div>
      </div>
    </div>

    <!-- Head -->
    <div class="fishbone-head">
      <div class="head-box">
        <a-textarea 
          v-model="fishboneData.problem" 
          placeholder="输入主要问题..." 
          :auto-size="{ minRows: 2, maxRows: 4 }"
          class="head-textarea"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Container adjustment to handle scrolling properly */
.fishbone-container {
  display: flex;
  align-items: center;
  overflow-x: auto;
  padding: 40px;
  background: #f9f9f9;
  border-radius: 8px;
  min-height: 500px;
}

/* Tail */
.fishbone-tail {
  margin-right: 20px;
  z-index: 2;
  display: flex;
  align-items: center;
  flex-shrink: 0; 
}

/* Body */
.fishbone-body {
  display: flex;
  flex-direction: column;
  position: relative;
  /* Ensure it takes enough width */
  min-width: 200px;
  margin-right: 40px;
}

.spine {
  height: 4px;
  background-color: #333;
  width: 100%;
  position: absolute; 
  top: 50%; 
  transform: translateY(-50%);
  z-index: 1;
  left: 0;
  right: 0;
}

.spine-arrow {
  position: absolute;
  right: -10px;
  top: -8px;
  width: 0;
  height: 0;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
  border-left: 15px solid #333;
}

/* Nodes Container */
.nodes-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  position: relative;
  z-index: 2;
  gap: 60px; /* Gap between nodes */
  padding: 0 40px;
}

.spine-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 160px; /* Width of a category group */
}

.node-slot {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end; /* Align to bottom for top slot */
  min-height: 200px; /* Height for bone */
  width: 100%;
}

.node-slot.top {
  justify-content: flex-end;
  padding-bottom: 0;
}

.node-slot.bottom {
  justify-content: flex-start; /* Align to top for bottom slot */
  padding-top: 0;
}

.add-slot-btn {
  padding: 10px;
  opacity: 0.5;
  transition: opacity 0.3s;
}

.add-slot-btn:hover {
  opacity: 1;
}

/* Spine Point / Center */
.spine-point {
  width: 24px;
  height: 24px;
  margin: -12px 0; /* Center vertically on spine (height/2) */
  z-index: 5; /* Above everything */
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  /* Make sure it doesn't take up space if empty or hidden */
}

.delete-node-wrapper {
  width: 24px;
  height: 24px;
  background: white; 
  border-radius: 50%;
  border: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
  color: #999;
}

.delete-node-wrapper:hover {
  border-color: #f53f3f;
  color: #f53f3f;
  background-color: #fff0f0;
}

.delete-node-icon {
  font-size: 14px;
}

.category-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  width: 160px;
}

/* Diagonal Lines */
.diagonal-line {
  width: 2px;
  min-height: 100px;
  background-color: #555;
  position: relative;
  left: 50%;
  transform: translateX(-50%);
}

.top .diagonal-line {
  transform: translateX(-50%) skewX(30deg);
  transform-origin: bottom center;
  margin-bottom: -2px; /* Connect to spine */
}

.bottom .diagonal-line {
  transform: translateX(-50%) skewX(-30deg);
  transform-origin: top center;
  margin-top: -2px; /* Connect to spine */
}

/* Category Content Styling */
.category-header {
  background: white; 
  padding: 4px 8px;
  width: 160px; 
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: space-between; 
  gap: 4px;
  border: 1px solid #555; 
  border-radius: 8px; 
  box-shadow: 0 2px 4px rgba(0,0,0,0.1); 
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 2px;
  flex-shrink: 0;
}

.category-input {
  width: 100%;
}

.category-input :deep(.arco-input) {
  background: transparent !important;
  border: none !important;
  font-weight: bold;
  font-size: 1.1em;
  padding: 0;
  text-align: center;
}

.category-input :deep(.arco-input-wrapper) {
  background: transparent !important;
  border: none !important;
  padding: 0;
  box-shadow: none !important;
}

.top .category-header {
  margin-bottom: 0;
}

.bottom .category-header {
  margin-top: 0;
}

.items-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

/* Item Styling as Horizontal Line */
.item-row {
  position: absolute;
  border-bottom: 1px solid #777; 
  padding-bottom: 0;
  display: flex;
  align-items: flex-end; 
  gap: 4px;
  pointer-events: auto;
}

.item-input :deep(.arco-input) {
  background: transparent !important;
  border: none !important;
  padding: 0;
  font-size: 0.8em;
  text-align: center; 
}

.item-row .arco-btn {
  position: absolute;
  right: -20px;
  top: -10px;
  opacity: 0; 
  transition: opacity 0.2s;
}

.item-row:hover .arco-btn {
  opacity: 1;
}

.item-input :deep(.arco-input-wrapper) {
  background: transparent !important;
  border: none !important;
  padding: 0;
  box-shadow: none !important;
}

/* Head Styling */
.fishbone-head {
  margin-left: 20px; 
  z-index: 2;
  display: flex;
  align-items: center;
  position: relative;
}

.head-box {
  position: relative;
  z-index: 2;
  background: transparent; 
  width: 150px;
  height: 100px;
  border: 3px solid #333;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  padding: 10px;
  box-shadow: none;
}

.head-textarea {
  width: 100%;
  height: 100%;
  background: transparent !important;
  border: none !important;
  resize: none;
  padding: 0;
  box-shadow: none !important;
}

.head-textarea :deep(.arco-textarea) {
  text-align: center;
  font-weight: bold;
  font-size: 1.1em;
  background: transparent;
  border: none;
  resize: none;
  min-height: 100%;
}
</style>