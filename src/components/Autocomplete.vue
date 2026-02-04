<template>
  <div class="autocomplete" ref="autocompleteRef">
    <input
      type="text"
      v-model="searchText"
      @input="onInput"
      @focus="onFocus"
      @blur="onBlur"
      :placeholder="placeholder"
      class="autocomplete-input"
    />
    
    <div v-if="showDropdown && filteredItems.length > 0" class="autocomplete-dropdown">
      <div
        v-for="item in filteredItems"
        :key="item.id"
        @mousedown.prevent="selectItem(item)"
        class="autocomplete-item"
      >
        <span class="item-code">{{ item.code }}</span>
        <span class="item-name">{{ item.name }}</span>
      </div>
    </div>
    
    <div v-if="showDropdown && searchText && filteredItems.length === 0" class="autocomplete-dropdown">
      <div class="autocomplete-item no-results">
        該当する項目がありません
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  modelValue: {
    type: Object,
    default: null
  },
  items: {
    type: Array,
    required: true
  },
  placeholder: {
    type: String,
    default: '選択してください'
  }
})

const emit = defineEmits(['update:modelValue'])

const searchText = ref('')
const showDropdown = ref(false)
const autocompleteRef = ref(null)

// 選択されたアイテムが変更されたら、表示テキストを更新
watch(() => props.modelValue, (newValue) => {
  if (newValue) {
    searchText.value = `${newValue.code} - ${newValue.name}`
  } else {
    searchText.value = ''
  }
}, { immediate: true })

// フィルタリングされたアイテム
const filteredItems = computed(() => {
  if (!searchText.value) {
    return props.items
  }
  
  const search = searchText.value.toLowerCase()
  return props.items.filter(item => {
    const code = item.code?.toLowerCase() || ''
    const name = item.name?.toLowerCase() || ''
    return code.includes(search) || name.includes(search)
  })
})

// 入力時の処理
const onInput = () => {
  showDropdown.value = true
  // 入力中は選択をクリア
  if (props.modelValue) {
    emit('update:modelValue', null)
  }
}

// フォーカス時の処理
const onFocus = () => {
  showDropdown.value = true
}

// ブラー時の処理
const onBlur = () => {
  setTimeout(() => {
    showDropdown.value = false
    
    // 選択されていない場合はテキストをクリア
    if (!props.modelValue) {
      searchText.value = ''
    }
  }, 200)
}

// アイテム選択時の処理
const selectItem = (item) => {
  emit('update:modelValue', item)
  searchText.value = `${item.code} - ${item.name}`
  showDropdown.value = false
}
</script>

<style scoped>
.autocomplete {
  position: relative;
  width: 100%;
}

.autocomplete-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.autocomplete-input:focus {
  outline: none;
  border-color: #4CAF50;
}

.autocomplete-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  max-height: 250px;
  overflow-y: auto;
  background-color: white;
  border: 1px solid #ddd;
  border-top: none;
  border-radius: 0 0 4px 4px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  margin-top: 2px;
}

.autocomplete-item {
  padding: 10px;
  cursor: pointer;
  transition: background-color 0.2s;
  display: flex;
  gap: 10px;
}

.autocomplete-item:hover {
  background-color: #f5f5f5;
}

.autocomplete-item.no-results {
  color: #999;
  cursor: default;
  justify-content: center;
}

.autocomplete-item.no-results:hover {
  background-color: white;
}

.item-code {
  font-weight: 600;
  color: #666;
  min-width: 60px;
}

.item-name {
  color: #333;
}
</style>
