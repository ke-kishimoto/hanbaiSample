<template>
  <div class="product-list">
    <h2>商品管理</h2>
    
    <!-- 検索・フィルターエリア -->
    <div class="filter-section">
      <div class="filter-row">
        <div class="search-group">
          <input
            type="text"
            v-model="searchKeyword"
            placeholder="商品コード、商品名で検索..."
            class="search-input"
          />
        </div>
        
        <div class="filter-group">
          <select v-model="filterCategory" class="filter-select">
            <option value="">すべてのカテゴリ</option>
            <option value="パソコン">パソコン</option>
            <option value="周辺機器">周辺機器</option>
            <option value="オフィス用品">オフィス用品</option>
            <option value="消耗品">消耗品</option>
            <option value="ソフトウェア">ソフトウェア</option>
            <option value="その他">その他</option>
          </select>
        </div>
        
        <div class="filter-group">
          <select v-model="filterStatus" class="filter-select">
            <option value="">すべてのステータス</option>
            <option value="有効">有効</option>
            <option value="無効">無効</option>
          </select>
        </div>
        
        <button @click="openCreateModal" class="btn-create">
          ➕ 新規登録
        </button>
      </div>
    </div>
    
    <!-- 商品一覧テーブル -->
    <div class="table-container">
      <table class="product-table">
        <thead>
          <tr>
            <th style="width: 100px">商品コード</th>
            <th style="width: 200px">商品名</th>
            <th style="width: 120px">カテゴリ</th>
            <th style="width: 100px">単価</th>
            <th style="width: 80px">在庫数</th>
            <th style="width: 60px">単位</th>
            <th style="width: 80px">税区分</th>
            <th style="width: 80px">ステータス</th>
            <th style="width: 180px">操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="filteredProducts.length === 0">
            <td colspan="9" class="no-data">該当する商品がありません</td>
          </tr>
          <tr v-for="product in filteredProducts" :key="product.code">
            <td>{{ product.code }}</td>
            <td>{{ product.name }}</td>
            <td>{{ product.category || '-' }}</td>
            <td class="text-right">¥{{ formatCurrency(product.price) }}</td>
            <td class="text-right">{{ product.stock || 0 }}</td>
            <td class="text-center">{{ product.unit }}</td>
            <td class="text-center">{{ product.taxType }}</td>
            <td class="text-center">
              <span :class="['status-badge', product.status === '有効' ? 'status-active' : 'status-inactive']">
                {{ product.status }}
              </span>
            </td>
            <td class="action-cell">
              <button @click="openViewModal(product)" class="btn-action btn-view" title="詳細">
                👁️
              </button>
              <button @click="openEditModal(product)" class="btn-action btn-edit" title="編集">
                ✏️
              </button>
              <button @click="confirmDelete(product)" class="btn-action btn-delete" title="削除">
                🗑️
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    
    <!-- 商品数表示 -->
    <div class="table-footer">
      <p class="product-count">
        全{{ products.length }}件中 {{ filteredProducts.length }}件を表示
      </p>
    </div>
  </div>
  
  <!-- 商品編集モーダル -->
  <ProductEditModal
    :isOpen="isModalOpen"
    :mode="modalMode"
    :product="selectedProduct"
    @close="closeModal"
    @save="handleSave"
  />
</template>

<script setup>
import { ref, computed } from 'vue'
import ProductEditModal from './ProductEditModal.vue'

const props = defineProps({
  products: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['update:products'])

// モーダル制御
const isModalOpen = ref(false)
const modalMode = ref('create') // 'create', 'edit', 'view'
const selectedProduct = ref(null)

// 検索・フィルター
const searchKeyword = ref('')
const filterCategory = ref('')
const filterStatus = ref('')

// 商品リスト（ローカル状態）
const products = ref([...props.products])

// フィルタリングされた商品リスト
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    // 検索キーワード
    const keyword = searchKeyword.value.toLowerCase()
    const matchKeyword = keyword === '' || 
      product.code.toLowerCase().includes(keyword) ||
      product.name.toLowerCase().includes(keyword) ||
      (product.kana && product.kana.toLowerCase().includes(keyword))
    
    // カテゴリフィルター
    const matchCategory = filterCategory.value === '' || product.category === filterCategory.value
    
    // ステータスフィルター
    const matchStatus = filterStatus.value === '' || product.status === filterStatus.value
    
    return matchKeyword && matchCategory && matchStatus
  })
})

// 通貨フォーマット
const formatCurrency = (amount) => {
  if (amount == null) return '0'
  return amount.toLocaleString('ja-JP')
}

// 新規登録モーダルを開く
const openCreateModal = () => {
  modalMode.value = 'create'
  selectedProduct.value = null
  isModalOpen.value = true
}

// 詳細モーダルを開く
const openViewModal = (product) => {
  modalMode.value = 'view'
  selectedProduct.value = { ...product }
  isModalOpen.value = true
}

// 編集モーダルを開く
const openEditModal = (product) => {
  modalMode.value = 'edit'
  selectedProduct.value = { ...product }
  isModalOpen.value = true
}

// モーダルを閉じる
const closeModal = () => {
  isModalOpen.value = false
  selectedProduct.value = null
}

// 保存処理
const handleSave = (productData) => {
  if (modalMode.value === 'create') {
    // 商品コードの重複チェック
    const exists = products.value.some(p => p.code === productData.code)
    if (exists) {
      alert(`商品コード「${productData.code}」は既に登録されています。`)
      return
    }
    
    // 新規追加
    products.value.push(productData)
    alert('商品を登録しました。')
  } else if (modalMode.value === 'edit') {
    // 更新
    const index = products.value.findIndex(p => p.code === selectedProduct.value.code)
    if (index !== -1) {
      products.value[index] = productData
      alert('商品を更新しました。')
    }
  }
  
  // 親コンポーネントに通知
  emit('update:products', products.value)
  
  closeModal()
}

// 削除確認
const confirmDelete = (product) => {
  if (confirm(`商品「${product.name}（${product.code}）」を削除してもよろしいですか？`)) {
    const index = products.value.findIndex(p => p.code === product.code)
    if (index !== -1) {
      products.value.splice(index, 1)
      alert('商品を削除しました。')
      
      // 親コンポーネントに通知
      emit('update:products', products.value)
    }
  }
}
</script>

<style scoped>
.product-list {
  margin-top: 20px;
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 24px;
}

/* フィルターセクション */
.filter-section {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 20px;
}

.filter-row {
  display: flex;
  gap: 15px;
  align-items: center;
}

.search-group {
  flex: 1;
}

.search-input {
  width: 100%;
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #4CAF50;
}

.filter-group {
  min-width: 180px;
}

.filter-select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background-color: white;
  cursor: pointer;
  transition: border-color 0.3s;
}

.filter-select:focus {
  outline: none;
  border-color: #4CAF50;
}

.btn-create {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  white-space: nowrap;
  transition: all 0.3s;
}

.btn-create:hover {
  background-color: #45a049;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

/* テーブル */
.table-container {
  background-color: white;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.product-table {
  width: 100%;
  border-collapse: collapse;
}

.product-table thead {
  background-color: #f5f5f5;
}

.product-table th {
  padding: 15px 12px;
  text-align: left;
  font-weight: 600;
  color: #555;
  border-bottom: 2px solid #ddd;
  font-size: 14px;
}

.product-table td {
  padding: 12px;
  border-bottom: 1px solid #eee;
  font-size: 14px;
}

.product-table tbody tr:hover {
  background-color: #fafafa;
}

.text-center {
  text-align: center;
}

.text-right {
  text-align: right;
}

.no-data {
  text-align: center;
  color: #999;
  padding: 40px !important;
  font-size: 16px;
}

/* ステータスバッジ */
.status-badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.status-active {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.status-inactive {
  background-color: #ffebee;
  color: #c62828;
}

/* アクションボタン */
.action-cell {
  display: flex;
  gap: 8px;
  justify-content: center;
}

.btn-action {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 18px;
  padding: 6px 10px;
  border-radius: 4px;
  transition: all 0.3s;
}

.btn-view:hover {
  background-color: #e3f2fd;
}

.btn-edit:hover {
  background-color: #fff3e0;
}

.btn-delete:hover {
  background-color: #ffebee;
}

/* テーブルフッター */
.table-footer {
  margin-top: 15px;
  text-align: right;
}

.product-count {
  color: #666;
  font-size: 14px;
  margin: 0;
}
</style>
