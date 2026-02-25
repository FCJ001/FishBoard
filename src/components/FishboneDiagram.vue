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
  const baseSpacing = 80; // 再次增加间距
  const startOffset = 40; 
  const offset = startOffset + (index * baseSpacing);
  
  // 计算水平偏移量以保持在斜线上
  // tan(30) 约为 0.577
  const x = offset * 0.577; 
  
  if (position === 'top') {
    // 上方骨线 (/)
    return {
      bottom: `${offset}px`,
      left: '50%',
      transform: `translateX(${-x - 50}px)`,
      width: '120px'
    }
  } else {
    // 下方骨线 (\)
    return {
      top: `${offset}px`,
      left: '50%',
      transform: `translateX(${-x - 50}px)`,
      width: '120px'
    }
  }
}

/* 根据项目数量计算斜线高度 (数值) */
const calcDiagonalHeightNum = (itemCount) => {
  const itemHeight = 100; // 对应 baseSpacing + margin，确保足够
  const minHeight = 200; 
  
  // 基础高度 + (数量 * 单项高度)
  // 预留一些空间给加号按钮
  const requiredHeight = (itemCount * itemHeight) + 100;
  
  return Math.max(minHeight, requiredHeight);
}

/* 获取斜线高度样式 */
const getDiagonalHeight = (itemCount) => {
  return calcDiagonalHeightNum(itemCount) + 'px';
}

const getHeaderStyle = (itemCount, position) => {
  const height = calcDiagonalHeightNum(itemCount);
  const x = height * 0.577; // horizontal shift due to skew
  
  if (position === 'top') {
    // Top Line: SkewX(30deg).
    // This pushes the top of the line to the LEFT relative to the bottom origin.
    // So we must move the header LEFT to follow it.
    return {
      transform: `translateX(${-x}px)`
    };
  } else {
    // Bottom Line: SkewX(-30deg).
    // This pushes the bottom of the line to the LEFT relative to the top origin.
    // Wait... if Bottom moves left, then Header (which is at top origin) stays put?
    // No, Header is *below* Line in DOM for bottom slot?
    // Let's check DOM order.
    
    // Bottom Slot:
    // <line>
    // <header>
    // <items>
    
    // Line is FIRST. Header is SECOND.
    // So Header is *below* Line.
    // Line origin is Top Center.
    // Line bottom is shifted LEFT (SkewX(-30deg)).
    // Header is at Line bottom.
    // So Header must move LEFT.
    
    return {
      transform: `translateX(${-x}px)`
    };
  }
}

/* 获取节点槽位的最小高度，确保上下对称 */
const getNodeSlotStyle = (node) => {
  const topCount = node.top ? node.top.items.length : 0;
  const bottomCount = node.bottom ? node.bottom.items.length : 0;
  
  const hTop = calcDiagonalHeightNum(topCount);
  const hBottom = calcDiagonalHeightNum(bottomCount);
  
  // 取最大值作为两个槽位的高度，加上头部和padding的预估高度
  // 头部约 40px，padding 约 20px
  const maxH = Math.max(hTop, hBottom) + 80;
  
  return {
    minHeight: `${maxH}px`
  };
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
          <div class="node-slot top" :style="getNodeSlotStyle(node)">
            <template v-if="node.top">
              <div class="category-group">
                <!-- Category Header -->
                <div class="category-header" :style="getHeaderStyle(node.top.items.length, 'top')">
                  <a-input v-model="node.top.name" size="small" placeholder="分类名称" class="category-input" />
                  <div class="header-actions">
                    <a-button type="text" status="danger" shape="circle" size="mini" @click="removeCategoryFromNode(node, 'top')">
                      <template #icon><icon-delete /></template>
                    </a-button>
                  </div>
                </div>

                <!-- Diagonal Line -->
                <div class="diagonal-line" :style="{ height: getDiagonalHeight(node.top.items.length) }">
                  <div class="add-item-btn" @click.stop="addItem(node.top)" title="添加原因">
                    <icon-plus />
                  </div>
                </div>
                
                <!-- Items List -->
                <div class="items-list">
                  <div v-for="(item, i) in node.top.items" :key="item.id" class="item-wrapper" :style="getItemStyle(i, 'top')">
                    <!-- The content box (Left) -->
                    <div class="item-content">
                      <a-input v-model="item.name" size="mini" class="item-input" />
                      <div class="delete-item-btn" @click="removeItem(node.top, i)">
                        <icon-close />
                      </div>
                    </div>
                    <!-- The horizontal bone line (Right) -->
                    <div class="item-bone"></div>
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
          <div class="node-slot bottom" :style="getNodeSlotStyle(node)">
            <template v-if="node.bottom">
              <div class="category-group">
                <!-- Diagonal Line -->
                <div class="diagonal-line" :style="{ height: getDiagonalHeight(node.bottom.items.length) }">
                  <div class="add-item-btn" @click.stop="addItem(node.bottom)" title="添加原因">
                    <icon-plus />
                  </div>
                </div>
                
                <!-- Category Header -->
                <div class="category-header" :style="getHeaderStyle(node.bottom.items.length, 'bottom')">
                  <a-input v-model="node.bottom.name" size="small" placeholder="分类名称" class="category-input" />
                  <div class="header-actions">
                    <a-button type="text" status="danger" shape="circle" size="mini" @click="removeCategoryFromNode(node, 'bottom')">
                      <template #icon><icon-delete /></template>
                    </a-button>
                  </div>
                </div>

                <!-- Items List -->
                <div class="items-list">
                  <div v-for="(item, i) in node.bottom.items" :key="item.id" class="item-wrapper" :style="getItemStyle(i, 'bottom')">
                    <!-- The content box (Left) -->
                    <div class="item-content">
                      <a-input v-model="item.name" size="mini" class="item-input" />
                      <div class="delete-item-btn" @click="removeItem(node.bottom, i)">
                        <icon-close />
                      </div>
                    </div>
                    <!-- The horizontal bone line (Right) -->
                    <div class="item-bone"></div>
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
  align-items: stretch; /* Stretch nodes to full height */
  position: relative;
  z-index: 2;
  gap: 60px; /* Gap between nodes */
  padding: 0 40px;
  min-height: 100%; /* Fill parent */
}

.spine-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 160px;
  justify-content: center; /* Center content vertically around spine */
}

/* Slot containers */
.node-slot {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  flex: 1 0 auto; /* Grow, don't shrink, basis auto */
  min-height: 200px; /* Minimum height */
  position: relative;
}

.node-slot.top {
  justify-content: flex-end; /* Align content to bottom (near spine) */
  padding-bottom: 0;
}

.node-slot.bottom {
  justify-content: flex-start; /* Align content to top (near spine) */
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

/* Add Item Button on Diagonal Line */
.add-item-btn {
  position: absolute;
  width: 16px;
  height: 16px;
  background: white;
  border: 1px solid #555;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  left: 50%;
  margin-left: -8px; /* Center (width/2) */
  z-index: 10;
  font-size: 10px;
  color: #555;
  transition: all 0.2s;
}

.add-item-btn:hover {
  background: #eee;
  color: #000;
  transform: scale(1.2);
}

/* Position for Top Line */
.top .diagonal-line .add-item-btn {
  top: 10px; /* Near the top (Header side) */
  transform: skewX(-30deg); /* Undo skew */
}

.top .diagonal-line .add-item-btn:hover {
  transform: skewX(-30deg) scale(1.2);
}

/* Position for Bottom Line */
.bottom .diagonal-line .add-item-btn {
  bottom: 10px; /* Near the bottom (Header side) */
  transform: skewX(30deg); /* Undo skew */
}

.bottom .diagonal-line .add-item-btn:hover {
  transform: skewX(30deg) scale(1.2);
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

/* Item Wrapper: Positioned relative to diagonal line */
.item-wrapper {
  position: absolute;
  /* width: 120px;  Total width: Bone (40px) + Content (80px) */
  display: flex;
  align-items: center;
  justify-content: flex-end; /* Align everything to right */
  pointer-events: auto;
  z-index: 5;
}

/* The Horizontal Bone Line */
.item-bone {
  width: 40px; /* Fixed length */
  height: 2px; /* Same thickness as diagonal line */
  background-color: #555;
  flex-shrink: 0;
  /* margin-left: -1px;  Overlap slightly if needed */
}

/* The Content Box */
.item-content {
  position: relative;
  display: flex;
  align-items: center;
  margin-right: 0; /* No margin needed now as it's to the left */
}

.item-input :deep(.arco-input) {
  background: white !important;
  border: 1px solid #555 !important; /* Match line color */
  border-radius: 4px;
  padding: 0 4px;
  font-size: 0.8em;
  text-align: center;
  height: 24px;
  line-height: 22px;
  width: 80px;
}

.item-input :deep(.arco-input-wrapper) {
  background: transparent !important;
  border: none !important;
  padding: 0;
  box-shadow: none !important;
}

/* Delete Button for Item */
.delete-item-btn {
  position: absolute;
  left: -20px;
  top: 50%;
  transform: translateY(-50%);
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  opacity: 0;
  transition: opacity 0.2s;
  color: #999;
}

.item-wrapper:hover .delete-item-btn {
  opacity: 1;
}

.delete-item-btn:hover {
  color: #f53f3f;
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