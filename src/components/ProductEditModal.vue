<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="closeModal">
    <div class="modal-content">
      <div class="modal-header">
        <h2>{{ modalTitle }}</h2>
        <button @click="closeModal" class="btn-close">×</button>
      </div>
      
      <div class="modal-body">
        <form @submit.prevent="handleSubmit">
          <div class="form-grid">
            <div class="form-group">
              <label>商品コード <span class="required">*</span></label>
              <input
                type="text"
                v-model="formData.code"
                :readonly="mode === 'view' || (mode === 'edit' && product)"
                :class="{ 'input-readonly': mode === 'view' || (mode === 'edit' && product) }"
                placeholder="P001"
                required
              />
            </div>
            
            <div class="form-group">
              <label>商品名 <span class="required">*</span></label>
              <input
                type="text"
                v-model="formData.name"
                :readonly="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
                placeholder="ノートPC"
                required
              />
            </div>
            
            <div class="form-group">
              <label>商品名カナ</label>
              <input
                type="text"
                v-model="formData.kana"
                :readonly="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
                placeholder="ノートピーシー"
              />
            </div>
            
            <div class="form-group">
              <label>カテゴリ</label>
              <select
                v-model="formData.category"
                :disabled="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
              >
                <option value="">選択してください</option>
                <option value="パソコン">パソコン</option>
                <option value="周辺機器">周辺機器</option>
                <option value="オフィス用品">オフィス用品</option>
                <option value="消耗品">消耗品</option>
                <option value="ソフトウェア">ソフトウェア</option>
                <option value="その他">その他</option>
              </select>
            </div>
            
            <div class="form-group">
              <label>単価 <span class="required">*</span></label>
              <input
                type="number"
                v-model.number="formData.price"
                :readonly="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
                placeholder="120000"
                min="0"
                required
              />
            </div>
            
            <div class="form-group">
              <label>原価</label>
              <input
                type="number"
                v-model.number="formData.cost"
                :readonly="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
                placeholder="100000"
                min="0"
              />
            </div>
            
            <div class="form-group">
              <label>単位</label>
              <select
                v-model="formData.unit"
                :disabled="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
              >
                <option value="個">個</option>
                <option value="台">台</option>
                <option value="本">本</option>
                <option value="箱">箱</option>
                <option value="セット">セット</option>
                <option value="枚">枚</option>
              </select>
            </div>
            
            <div class="form-group">
              <label>在庫数</label>
              <input
                type="number"
                v-model.number="formData.stock"
                :readonly="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
                placeholder="0"
                min="0"
              />
            </div>
            
            <div class="form-group">
              <label>税区分</label>
              <select
                v-model="formData.taxType"
                :disabled="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
              >
                <option value="課税">課税</option>
                <option value="非課税">非課税</option>
                <option value="免税">免税</option>
              </select>
            </div>
            
            <div class="form-group">
              <label>ステータス</label>
              <select
                v-model="formData.status"
                :disabled="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
              >
                <option value="有効">有効</option>
                <option value="無効">無効</option>
              </select>
            </div>
            
            <div class="form-group full-width">
              <label>備考</label>
              <textarea
                v-model="formData.remarks"
                :readonly="mode === 'view'"
                :class="{ 'input-readonly': mode === 'view' }"
                placeholder="商品に関する備考を入力"
                rows="3"
              ></textarea>
            </div>
          </div>
        </form>
      </div>
      
      <div class="modal-footer">
        <button v-if="mode === 'view'" @click="closeModal" class="btn-secondary">閉じる</button>
        <template v-else>
          <button @click="closeModal" class="btn-secondary">キャンセル</button>
          <button @click="handleSubmit" class="btn-primary">
            {{ mode === 'create' ? '登録' : '更新' }}
          </button>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  isOpen: {
    type: Boolean,
    required: true
  },
  mode: {
    type: String, // 'create', 'edit', 'view'
    required: true
  },
  product: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close', 'save'])

const formData = ref({
  code: '',
  name: '',
  kana: '',
  category: '',
  price: 0,
  cost: 0,
  unit: '個',
  stock: 0,
  taxType: '課税',
  status: '有効',
  remarks: ''
})

const modalTitle = computed(() => {
  switch (props.mode) {
    case 'create':
      return '商品新規登録'
    case 'edit':
      return '商品編集'
    case 'view':
      return '商品詳細'
    default:
      return '商品'
  }
})

// propsの変更を監視してフォームデータを更新
watch(() => props.product, (newProduct) => {
  if (newProduct) {
    formData.value = {
      code: newProduct.code || '',
      name: newProduct.name || '',
      kana: newProduct.kana || '',
      category: newProduct.category || '',
      price: newProduct.price || 0,
      cost: newProduct.cost || 0,
      unit: newProduct.unit || '個',
      stock: newProduct.stock || 0,
      taxType: newProduct.taxType || '課税',
      status: newProduct.status || '有効',
      remarks: newProduct.remarks || ''
    }
  } else {
    resetForm()
  }
}, { immediate: true })

// モーダルが開いた時にフォームをリセット（新規作成の場合）
watch(() => props.isOpen, (isOpen) => {
  if (isOpen && props.mode === 'create') {
    resetForm()
  }
})

const resetForm = () => {
  formData.value = {
    code: '',
    name: '',
    kana: '',
    category: '',
    price: 0,
    cost: 0,
    unit: '個',
    stock: 0,
    taxType: '課税',
    status: '有効',
    remarks: ''
  }
}

const closeModal = () => {
  emit('close')
}

const handleSubmit = () => {
  if (props.mode === 'view') return
  
  // バリデーション
  if (!formData.value.code.trim()) {
    alert('商品コードを入力してください。')
    return
  }
  if (!formData.value.name.trim()) {
    alert('商品名を入力してください。')
    return
  }
  if (formData.value.price < 0) {
    alert('単価は0以上を入力してください。')
    return
  }
  
  emit('save', { ...formData.value })
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  border-radius: 8px;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 2px solid #eee;
}

.modal-header h2 {
  margin: 0;
  color: #333;
  font-size: 22px;
}

.btn-close {
  background: none;
  border: none;
  font-size: 32px;
  color: #999;
  cursor: pointer;
  padding: 0;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.3s;
}

.btn-close:hover {
  color: #f44336;
}

.modal-body {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-group.full-width {
  grid-column: 1 / -1;
}

.form-group label {
  margin-bottom: 8px;
  font-weight: 600;
  color: #555;
  font-size: 14px;
}

.required {
  color: #f44336;
}

.form-group input,
.form-group select,
.form-group textarea {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #4CAF50;
}

.input-readonly {
  background-color: #f5f5f5;
  color: #666;
  cursor: default;
}

.form-group textarea {
  resize: vertical;
  font-family: inherit;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 20px;
  border-top: 2px solid #eee;
}

.btn-primary,
.btn-secondary {
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: all 0.3s;
}

.btn-primary {
  background-color: #4CAF50;
  color: white;
}

.btn-primary:hover {
  background-color: #45a049;
}

.btn-secondary {
  background-color: #9E9E9E;
  color: white;
}

.btn-secondary:hover {
  background-color: #757575;
}
</style>
