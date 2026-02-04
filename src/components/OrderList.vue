<template>
  <div class="order-list">
    <h2>受注一覧</h2>
    
    <!-- 検索フォーム -->
    <div class="search-section">
      <div class="search-form">
        <div class="search-group">
          <label>受注番号</label>
          <input
            type="text"
            v-model="searchConditions.orderNo"
            placeholder="受注番号で検索"
            @input="applyFilter"
          />
        </div>
        
        <div class="search-group">
          <label>受注日（開始）</label>
          <input
            type="date"
            v-model="searchConditions.dateFrom"
            @change="applyFilter"
          />
        </div>
        
        <div class="search-group">
          <label>受注日（終了）</label>
          <input
            type="date"
            v-model="searchConditions.dateTo"
            @change="applyFilter"
          />
        </div>
        
        <div class="search-group">
          <label>得意先</label>
          <input
            type="text"
            v-model="searchConditions.customer"
            placeholder="得意先名で検索"
            @input="applyFilter"
          />
        </div>
        
        <div class="search-group">
          <label>売上連携状態</label>
          <select v-model="searchConditions.salesStatus" @change="applyFilter">
            <option value="">すべて</option>
            <option value="linked">連携済み</option>
            <option value="notLinked">未連携</option>
          </select>
        </div>
        
        <div class="search-actions">
          <button @click="resetSearch" class="btn-reset">クリア</button>
        </div>
      </div>
    </div>
    
    <!-- 検索結果サマリー -->
    <div class="results-summary">
      <div class="summary-item">
        <span class="summary-label">検索結果:</span>
        <span class="summary-value">{{ filteredOrders.length }}件</span>
      </div>
      <div class="summary-item">
        <span class="summary-label">未連携:</span>
        <span class="summary-value warning">{{ notLinkedCount }}件</span>
      </div>
      <div class="summary-item">
        <span class="summary-label">連携済み:</span>
        <span class="summary-value success">{{ linkedCount }}件</span>
      </div>
      <div class="summary-item">
        <span class="summary-label">合計金額:</span>
        <span class="summary-value">{{ formatCurrency(totalAmount) }}</span>
      </div>
    </div>
    
    <!-- 一覧テーブル -->
    <div class="table-section">
      <div class="table-container">
        <table class="order-table">
          <thead>
            <tr>
              <th style="width: 120px">受注番号</th>
              <th style="width: 110px">受注日</th>
              <th style="width: 200px">得意先</th>
              <th style="width: 120px">担当者</th>
              <th style="width: 130px">金額</th>
              <th style="width: 100px">売上連携</th>
              <th style="width: 150px">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="order in filteredOrders" :key="order.orderNo">
              <td>{{ order.orderNo }}</td>
              <td>{{ formatDate(order.orderDate) }}</td>
              <td>{{ order.customer.name }}</td>
              <td>{{ order.staff.name }}</td>
              <td class="text-right">{{ formatCurrency(order.totalAmount) }}</td>
              <td class="text-center">
                <span :class="['status-badge', order.salesLinked ? 'linked' : 'not-linked']">
                  {{ order.salesLinked ? '連携済み' : '未連携' }}
                </span>
              </td>
              <td class="text-center">
                <div class="action-buttons">
                  <button @click="viewDetail(order)" class="btn-detail" title="詳細表示">
                    📄
                  </button>
                  <button 
                    @click="createSales(order)" 
                    class="btn-create-sales"
                    :disabled="order.salesLinked"
                    title="売上作成"
                  >
                    {{ order.salesLinked ? '作成済み' : '売上作成' }}
                  </button>
                </div>
              </td>
            </tr>
            <tr v-if="filteredOrders.length === 0">
              <td colspan="7" class="no-data">該当する受注データがありません</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    
    <!-- 詳細モーダル -->
    <Teleport to="body">
      <div v-if="selectedOrder" class="modal-overlay" @click="closeDetail">
        <div class="modal-content" @click.stop>
          <div class="modal-header">
            <h3>受注詳細</h3>
            <button @click="closeDetail" class="close-button">&times;</button>
          </div>
          
          <div class="modal-body">
            <div class="detail-section">
              <h4>基本情報</h4>
              <div class="detail-grid">
                <div class="detail-item">
                  <span class="detail-label">受注番号:</span>
                  <span class="detail-value">{{ selectedOrder.orderNo }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">受注日:</span>
                  <span class="detail-value">{{ formatDate(selectedOrder.orderDate) }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">得意先:</span>
                  <span class="detail-value">{{ selectedOrder.customer.name }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">担当者:</span>
                  <span class="detail-value">{{ selectedOrder.staff.name }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">売上連携:</span>
                  <span :class="['detail-value', 'status-badge', selectedOrder.salesLinked ? 'linked' : 'not-linked']">
                    {{ selectedOrder.salesLinked ? '連携済み' : '未連携' }}
                  </span>
                </div>
              </div>
            </div>
            
            <div class="detail-section">
              <h4>明細</h4>
              <table class="detail-table">
                <thead>
                  <tr>
                    <th>No.</th>
                    <th>商品コード</th>
                    <th>商品名</th>
                    <th class="text-right">個数</th>
                    <th class="text-right">単価</th>
                    <th class="text-right">金額</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(detail, index) in selectedOrder.details" :key="index">
                    <td>{{ index + 1 }}</td>
                    <td>{{ detail.product.code }}</td>
                    <td>{{ detail.product.name }}</td>
                    <td class="text-right">{{ detail.quantity }}</td>
                    <td class="text-right">{{ formatCurrency(detail.unitPrice) }}</td>
                    <td class="text-right">{{ formatCurrency(detail.quantity * detail.unitPrice) }}</td>
                  </tr>
                </tbody>
                <tfoot>
                  <tr>
                    <td colspan="5" class="text-right"><strong>合計:</strong></td>
                    <td class="text-right"><strong>{{ formatCurrency(selectedOrder.totalAmount) }}</strong></td>
                  </tr>
                </tfoot>
              </table>
            </div>
          </div>
          
          <div class="modal-footer">
            <button 
              @click="createSalesFromDetail" 
              class="btn-primary"
              :disabled="selectedOrder.salesLinked"
            >
              {{ selectedOrder.salesLinked ? '売上作成済み' : '売上作成' }}
            </button>
            <button @click="closeDetail" class="btn-secondary">閉じる</button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  customers: {
    type: Array,
    required: true
  },
  staffList: {
    type: Array,
    required: true
  },
  products: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['createSales'])

// 検索条件
const searchConditions = ref({
  orderNo: '',
  dateFrom: '',
  dateTo: '',
  customer: '',
  salesStatus: '' // '', 'linked', 'notLinked'
})

// 選択された受注（詳細表示用）
const selectedOrder = ref(null)

// サンプル受注データ（売上連携フラグを追加）
const orders = ref([
  {
    orderNo: 'OR-2026-001',
    orderDate: '2026-01-15',
    customer: props.customers[0],
    staff: props.staffList[0],
    totalAmount: 256000,
    salesLinked: true,
    details: [
      { product: props.products[0], quantity: 2, unitPrice: 98000 },
      { product: props.products[5], quantity: 2, unitPrice: 3500 },
      { product: props.products[6], quantity: 3, unitPrice: 2000 }
    ]
  },
  {
    orderNo: 'OR-2026-002',
    orderDate: '2026-01-18',
    customer: props.customers[1],
    staff: props.staffList[1],
    totalAmount: 163000,
    salesLinked: false,
    details: [
      { product: props.products[1], quantity: 1, unitPrice: 128000 },
      { product: props.products[3], quantity: 1, unitPrice: 25000 },
      { product: props.products[5], quantity: 1, unitPrice: 3500 },
      { product: props.products[6], quantity: 1, unitPrice: 2000 }
    ]
  },
  {
    orderNo: 'OR-2026-003',
    orderDate: '2026-01-20',
    customer: props.customers[2],
    staff: props.staffList[2],
    totalAmount: 145000,
    salesLinked: true,
    details: [
      { product: props.products[2], quantity: 1, unitPrice: 85000 },
      { product: props.products[4], quantity: 1, unitPrice: 35000 },
      { product: props.products[3], quantity: 1, unitPrice: 25000 }
    ]
  },
  {
    orderNo: 'OR-2026-004',
    orderDate: '2026-01-22',
    customer: props.customers[3],
    staff: props.staffList[0],
    totalAmount: 45000,
    salesLinked: false,
    details: [
      { product: props.products[7], quantity: 1, unitPrice: 45000 }
    ]
  },
  {
    orderNo: 'OR-2026-005',
    orderDate: '2026-01-25',
    customer: props.customers[4],
    staff: props.staffList[3],
    totalAmount: 294000,
    salesLinked: false,
    details: [
      { product: props.products[1], quantity: 2, unitPrice: 128000 },
      { product: props.products[8], quantity: 1, unitPrice: 28000 },
      { product: props.products[5], quantity: 3, unitPrice: 3500 }
    ]
  },
  {
    orderNo: 'OR-2026-006',
    orderDate: '2026-01-26',
    customer: props.customers[5],
    staff: props.staffList[4],
    totalAmount: 96000,
    salesLinked: true,
    details: [
      { product: props.products[9], quantity: 10, unitPrice: 8000 },
      { product: props.products[6], quantity: 8, unitPrice: 2000 }
    ]
  },
  {
    orderNo: 'OR-2026-007',
    orderDate: '2026-01-27',
    customer: props.customers[0],
    staff: props.staffList[1],
    totalAmount: 393000,
    salesLinked: false,
    details: [
      { product: props.products[0], quantity: 3, unitPrice: 98000 },
      { product: props.products[3], quantity: 3, unitPrice: 25000 },
      { product: props.products[5], quantity: 3, unitPrice: 3500 }
    ]
  },
  {
    orderNo: 'OR-2026-008',
    orderDate: '2026-01-28',
    customer: props.customers[6],
    staff: props.staffList[2],
    totalAmount: 158500,
    salesLinked: false,
    details: [
      { product: props.products[2], quantity: 1, unitPrice: 85000 },
      { product: props.products[4], quantity: 2, unitPrice: 35000 },
      { product: props.products[5], quantity: 1, unitPrice: 3500 }
    ]
  },
  {
    orderNo: 'OR-2026-009',
    orderDate: '2026-01-29',
    customer: props.customers[7],
    staff: props.staffList[3],
    totalAmount: 226000,
    salesLinked: false,
    details: [
      { product: props.products[1], quantity: 1, unitPrice: 128000 },
      { product: props.products[0], quantity: 1, unitPrice: 98000 }
    ]
  },
  {
    orderNo: 'OR-2026-010',
    orderDate: '2026-01-30',
    customer: props.customers[1],
    staff: props.staffList[0],
    totalAmount: 73000,
    salesLinked: false,
    details: [
      { product: props.products[7], quantity: 1, unitPrice: 45000 },
      { product: props.products[8], quantity: 1, unitPrice: 28000 }
    ]
  }
])

// フィルタリングされた受注データ
const filteredOrders = ref([...orders.value])

// フィルター適用
const applyFilter = () => {
  filteredOrders.value = orders.value.filter(order => {
    // 受注番号での絞り込み
    if (searchConditions.value.orderNo) {
      if (!order.orderNo.toLowerCase().includes(searchConditions.value.orderNo.toLowerCase())) {
        return false
      }
    }
    
    // 受注日（開始）での絞り込み
    if (searchConditions.value.dateFrom) {
      if (order.orderDate < searchConditions.value.dateFrom) {
        return false
      }
    }
    
    // 受注日（終了）での絞り込み
    if (searchConditions.value.dateTo) {
      if (order.orderDate > searchConditions.value.dateTo) {
        return false
      }
    }
    
    // 得意先での絞り込み
    if (searchConditions.value.customer) {
      const customerName = order.customer.name.toLowerCase()
      const searchCustomer = searchConditions.value.customer.toLowerCase()
      if (!customerName.includes(searchCustomer)) {
        return false
      }
    }
    
    // 売上連携状態での絞り込み
    if (searchConditions.value.salesStatus) {
      if (searchConditions.value.salesStatus === 'linked' && !order.salesLinked) {
        return false
      }
      if (searchConditions.value.salesStatus === 'notLinked' && order.salesLinked) {
        return false
      }
    }
    
    return true
  })
}

// 検索条件のリセット
const resetSearch = () => {
  searchConditions.value = {
    orderNo: '',
    dateFrom: '',
    dateTo: '',
    customer: '',
    salesStatus: ''
  }
  filteredOrders.value = [...orders.value]
}

// 未連携件数
const notLinkedCount = computed(() => {
  return filteredOrders.value.filter(order => !order.salesLinked).length
})

// 連携済み件数
const linkedCount = computed(() => {
  return filteredOrders.value.filter(order => order.salesLinked).length
})

// 合計金額
const totalAmount = computed(() => {
  return filteredOrders.value.reduce((sum, order) => sum + order.totalAmount, 0)
})

// 詳細表示
const viewDetail = (order) => {
  selectedOrder.value = order
}

// 詳細を閉じる
const closeDetail = () => {
  selectedOrder.value = null
}

// 売上作成
const createSales = (order) => {
  if (order.salesLinked) {
    alert('この受注は既に売上連携済みです。')
    return
  }
  emit('createSales', order)
}

// 詳細モーダルから売上作成
const createSalesFromDetail = () => {
  if (selectedOrder.value && !selectedOrder.value.salesLinked) {
    createSales(selectedOrder.value)
    closeDetail()
  }
}

// 日付フォーマット
const formatDate = (dateString) => {
  const date = new Date(dateString)
  return `${date.getFullYear()}/${String(date.getMonth() + 1).padStart(2, '0')}/${String(date.getDate()).padStart(2, '0')}`
}

// 通貨フォーマット
const formatCurrency = (value) => {
  return '¥' + value.toLocaleString('ja-JP')
}

// 売上連携状態の更新（外部から呼び出し可能）
defineExpose({
  updateOrderStatus: (orderNo) => {
    const order = orders.value.find(o => o.orderNo === orderNo)
    if (order) {
      order.salesLinked = true
      applyFilter() // フィルターを再適用
    }
  }
})
</script>

<style scoped>
.order-list {
  margin-top: 20px;
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 24px;
}

h4 {
  color: #555;
  margin-bottom: 15px;
  font-size: 16px;
  padding-bottom: 8px;
  border-bottom: 2px solid #e0e0e0;
}

/* 検索セクション */
.search-section {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 20px;
}

.search-form {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
  align-items: end;
}

.search-group {
  display: flex;
  flex-direction: column;
}

.search-group label {
  font-weight: 600;
  margin-bottom: 5px;
  color: #555;
  font-size: 14px;
}

.search-group input,
.search-group select {
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.search-group input:focus,
.search-group select:focus {
  outline: none;
  border-color: #4CAF50;
}

.search-actions {
  display: flex;
  align-items: flex-end;
}

.btn-reset {
  background-color: #9E9E9E;
  color: white;
  border: none;
  padding: 8px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.btn-reset:hover {
  background-color: #757575;
}

/* 結果サマリー */
.results-summary {
  display: flex;
  gap: 30px;
  margin-bottom: 20px;
  padding: 15px 20px;
  background-color: #f5f5f5;
  border-radius: 6px;
}

.summary-item {
  display: flex;
  gap: 10px;
  align-items: center;
}

.summary-label {
  font-weight: 600;
  color: #666;
  font-size: 14px;
}

.summary-value {
  font-weight: 700;
  font-size: 18px;
  color: #333;
}

.summary-value.warning {
  color: #FF9800;
}

.summary-value.success {
  color: #4CAF50;
}

/* テーブルセクション */
.table-section {
  margin-bottom: 30px;
}

.table-container {
  border: 1px solid #ddd;
  border-radius: 6px;
  overflow: hidden;
}

.order-table {
  width: 100%;
  border-collapse: collapse;
  background-color: white;
}

.order-table thead {
  background-color: #f5f5f5;
}

.order-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #555;
  border-bottom: 2px solid #ddd;
  font-size: 14px;
}

.order-table td {
  padding: 12px;
  border-bottom: 1px solid #eee;
  font-size: 14px;
}

.order-table tbody tr:hover {
  background-color: #fafafa;
}

.order-table tbody tr:last-child td {
  border-bottom: none;
}

.text-center {
  text-align: center;
}

.text-right {
  text-align: right;
  font-weight: 600;
}

.no-data {
  text-align: center;
  color: #999;
  padding: 30px !important;
}

/* ステータスバッジ */
.status-badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.status-badge.linked {
  background-color: #E8F5E9;
  color: #2E7D32;
}

.status-badge.not-linked {
  background-color: #FFF3E0;
  color: #E65100;
}

/* アクションボタン */
.action-buttons {
  display: flex;
  gap: 8px;
  justify-content: center;
}

.btn-detail {
  background-color: #2196F3;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.btn-detail:hover {
  background-color: #1976D2;
}

.btn-create-sales {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
  white-space: nowrap;
}

.btn-create-sales:hover:not(:disabled) {
  background-color: #45a049;
}

.btn-create-sales:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

/* モーダル */
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
  z-index: 9999;
  padding: 20px;
}

.modal-content {
  background-color: white;
  border-radius: 8px;
  width: 100%;
  max-width: 800px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 25px;
  border-bottom: 2px solid #e0e0e0;
}

.modal-header h3 {
  margin: 0;
  color: #333;
  font-size: 20px;
}

.close-button {
  background: none;
  border: none;
  font-size: 32px;
  color: #999;
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.3s;
}

.close-button:hover {
  color: #333;
}

.modal-body {
  padding: 20px 25px;
  overflow-y: auto;
}

.modal-footer {
  padding: 15px 25px;
  border-top: 1px solid #e0e0e0;
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.detail-section {
  margin-bottom: 25px;
}

.detail-section:last-child {
  margin-bottom: 0;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.detail-label {
  font-weight: 600;
  color: #666;
  font-size: 13px;
}

.detail-value {
  color: #333;
  font-size: 15px;
}

.detail-table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.detail-table thead {
  background-color: #f5f5f5;
}

.detail-table th,
.detail-table td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #eee;
  font-size: 14px;
}

.detail-table tfoot {
  background-color: #f9f9f9;
  font-weight: 600;
}

.detail-table tfoot td {
  border-bottom: none;
}

.btn-primary,
.btn-secondary {
  padding: 10px 24px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background-color: #4CAF50;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background-color: #45a049;
}

.btn-primary:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.btn-secondary {
  background-color: #9E9E9E;
  color: white;
}

.btn-secondary:hover {
  background-color: #757575;
}
</style>
