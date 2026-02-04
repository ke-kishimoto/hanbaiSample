<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>受注検索</h3>
          <button @click="closeModal" class="close-button">&times;</button>
        </div>
        
        <div class="modal-body">
          <!-- 検索フォーム -->
          <div class="search-form">
            <div class="search-group">
              <label>受注番号</label>
              <input
                type="text"
                v-model="searchConditions.orderNo"
                placeholder="受注番号で検索"
                @input="filterOrders"
              />
            </div>
            
            <div class="search-group">
              <label>受注日（開始）</label>
              <input
                type="date"
                v-model="searchConditions.dateFrom"
                @change="filterOrders"
              />
            </div>
            
            <div class="search-group">
              <label>受注日（終了）</label>
              <input
                type="date"
                v-model="searchConditions.dateTo"
                @change="filterOrders"
              />
            </div>
            
            <div class="search-group">
              <label>得意先</label>
              <input
                type="text"
                v-model="searchConditions.customer"
                placeholder="得意先名で検索"
                @input="filterOrders"
              />
            </div>
            
            <div class="search-actions">
              <button @click="resetSearch" class="btn-reset">クリア</button>
            </div>
          </div>
          
          <!-- 検索結果 -->
          <div class="search-results">
            <div class="results-header">
              <span>検索結果: {{ filteredOrders.length }}件</span>
            </div>
            
            <div class="results-table-container">
              <table class="results-table">
                <thead>
                  <tr>
                    <th>受注番号</th>
                    <th>受注日</th>
                    <th>得意先</th>
                    <th>担当者</th>
                    <th>金額</th>
                    <th>操作</th>
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
                      <button @click="selectOrder(order)" class="btn-select">選択</button>
                    </td>
                  </tr>
                  <tr v-if="filteredOrders.length === 0">
                    <td colspan="6" class="no-data">該当する受注データがありません</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  isOpen: {
    type: Boolean,
    required: true
  },
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

const emit = defineEmits(['close', 'select'])

// 検索条件
const searchConditions = ref({
  orderNo: '',
  dateFrom: '',
  dateTo: '',
  customer: ''
})

// サンプル受注データ
const orders = ref([
  {
    orderNo: 'OR-2026-001',
    orderDate: '2026-01-15',
    customer: props.customers[0], // 株式会社山田商事
    staff: props.staffList[0], // 山田太郎
    totalAmount: 256000,
    details: [
      { product: props.products[0], quantity: 2, unitPrice: 98000 },
      { product: props.products[5], quantity: 2, unitPrice: 3500 },
      { product: props.products[6], quantity: 3, unitPrice: 2000 }
    ]
  },
  {
    orderNo: 'OR-2026-002',
    orderDate: '2026-01-18',
    customer: props.customers[1], // 佐藤工業株式会社
    staff: props.staffList[1], // 佐藤花子
    totalAmount: 163000,
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
    customer: props.customers[2], // 田中物産株式会社
    staff: props.staffList[2], // 田中一郎
    totalAmount: 145000,
    details: [
      { product: props.products[2], quantity: 1, unitPrice: 85000 },
      { product: props.products[4], quantity: 1, unitPrice: 35000 },
      { product: props.products[3], quantity: 1, unitPrice: 25000 }
    ]
  },
  {
    orderNo: 'OR-2026-004',
    orderDate: '2026-01-22',
    customer: props.customers[3], // 鈴木建設株式会社
    staff: props.staffList[0], // 山田太郎
    totalAmount: 45000,
    details: [
      { product: props.products[7], quantity: 1, unitPrice: 45000 }
    ]
  },
  {
    orderNo: 'OR-2026-005',
    orderDate: '2026-01-25',
    customer: props.customers[4], // 高橋電機株式会社
    staff: props.staffList[3], // 鈴木二郎
    totalAmount: 294000,
    details: [
      { product: props.products[1], quantity: 2, unitPrice: 128000 },
      { product: props.products[8], quantity: 1, unitPrice: 28000 },
      { product: props.products[5], quantity: 3, unitPrice: 3500 }
    ]
  },
  {
    orderNo: 'OR-2026-006',
    orderDate: '2026-01-26',
    customer: props.customers[5], // 伊藤製作所
    staff: props.staffList[4], // 高橋三郎
    totalAmount: 96000,
    details: [
      { product: props.products[9], quantity: 10, unitPrice: 8000 },
      { product: props.products[6], quantity: 8, unitPrice: 2000 }
    ]
  },
  {
    orderNo: 'OR-2026-007',
    orderDate: '2026-01-27',
    customer: props.customers[0], // 株式会社山田商事
    staff: props.staffList[1], // 佐藤花子
    totalAmount: 393000,
    details: [
      { product: props.products[0], quantity: 3, unitPrice: 98000 },
      { product: props.products[3], quantity: 3, unitPrice: 25000 },
      { product: props.products[5], quantity: 3, unitPrice: 3500 }
    ]
  },
  {
    orderNo: 'OR-2026-008',
    orderDate: '2026-01-28',
    customer: props.customers[6], // 渡辺商店
    staff: props.staffList[2], // 田中一郎
    totalAmount: 158500,
    details: [
      { product: props.products[2], quantity: 1, unitPrice: 85000 },
      { product: props.products[4], quantity: 2, unitPrice: 35000 },
      { product: props.products[5], quantity: 1, unitPrice: 3500 }
    ]
  },
  {
    orderNo: 'OR-2026-009',
    orderDate: '2026-01-29',
    customer: props.customers[7], // 小林運輸株式会社
    staff: props.staffList[3], // 鈴木二郎
    totalAmount: 226000,
    details: [
      { product: props.products[1], quantity: 1, unitPrice: 128000 },
      { product: props.products[0], quantity: 1, unitPrice: 98000 }
    ]
  },
  {
    orderNo: 'OR-2026-010',
    orderDate: '2026-01-30',
    customer: props.customers[1], // 佐藤工業株式会社
    staff: props.staffList[0], // 山田太郎
    totalAmount: 73000,
    details: [
      { product: props.products[7], quantity: 1, unitPrice: 45000 },
      { product: props.products[8], quantity: 1, unitPrice: 28000 }
    ]
  }
])

// フィルタリングされた受注データ
const filteredOrders = ref([...orders.value])

// 受注データのフィルタリング
const filterOrders = () => {
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
    
    return true
  })
}

// 検索条件のリセット
const resetSearch = () => {
  searchConditions.value = {
    orderNo: '',
    dateFrom: '',
    dateTo: '',
    customer: ''
  }
  filteredOrders.value = [...orders.value]
}

// 受注の選択
const selectOrder = (order) => {
  emit('select', order)
  closeModal()
}

// モーダルを閉じる
const closeModal = () => {
  emit('close')
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
  z-index: 9999;
  padding: 20px;
}

.modal-content {
  background-color: white;
  border-radius: 8px;
  width: 100%;
  max-width: 1000px;
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

/* 検索フォーム */
.search-form {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 20px;
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

.search-group input {
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.search-group input:focus {
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

/* 検索結果 */
.search-results {
  margin-top: 20px;
}

.results-header {
  margin-bottom: 10px;
  font-weight: 600;
  color: #555;
}

.results-table-container {
  border: 1px solid #ddd;
  border-radius: 6px;
  overflow: hidden;
}

.results-table {
  width: 100%;
  border-collapse: collapse;
}

.results-table thead {
  background-color: #f5f5f5;
}

.results-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #555;
  border-bottom: 2px solid #ddd;
  font-size: 14px;
}

.results-table td {
  padding: 12px;
  border-bottom: 1px solid #eee;
  font-size: 14px;
}

.results-table tbody tr:hover {
  background-color: #fafafa;
}

.results-table tbody tr:last-child td {
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

.btn-select {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 6px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: background-color 0.3s;
}

.btn-select:hover {
  background-color: #45a049;
}
</style>
