<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>売上検索</h3>
          <button @click="closeModal" class="close-button">&times;</button>
        </div>
        
        <div class="modal-body">
          <!-- 検索フォーム -->
          <div class="search-form">
            <div class="search-group">
              <label>売上番号</label>
              <input
                type="text"
                v-model="searchConditions.salesNo"
                placeholder="売上番号で検索"
                @input="filterSales"
              />
            </div>
            
            <div class="search-group">
              <label>売上日（開始）</label>
              <input
                type="date"
                v-model="searchConditions.dateFrom"
                @change="filterSales"
              />
            </div>
            
            <div class="search-group">
              <label>売上日（終了）</label>
              <input
                type="date"
                v-model="searchConditions.dateTo"
                @change="filterSales"
              />
            </div>
            
            <div class="search-group">
              <label>得意先</label>
              <input
                type="text"
                v-model="searchConditions.customer"
                placeholder="得意先名で検索"
                @input="filterSales"
              />
            </div>
            
            <div class="search-actions">
              <button @click="resetSearch" class="btn-reset">クリア</button>
            </div>
          </div>
          
          <!-- 検索結果 -->
          <div class="search-results">
            <div class="results-header">
              <span>検索結果: {{ filteredSales.length }}件</span>
            </div>
            
            <div class="results-table-container">
              <table class="results-table">
                <thead>
                  <tr>
                    <th style="width: 140px">売上番号</th>
                    <th style="width: 100px">売上日</th>
                    <th style="width: 120px">受注番号</th>
                    <th>得意先</th>
                    <th style="width: 100px">担当者</th>
                    <th style="width: 120px">金額</th>
                    <th style="width: 80px">操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="sales in filteredSales" :key="sales.salesNo">
                    <td>{{ sales.salesNo }}</td>
                    <td>{{ formatDate(sales.date) }}</td>
                    <td>{{ sales.orderNo || '-' }}</td>
                    <td>{{ sales.customerName }}</td>
                    <td>{{ sales.staffName }}</td>
                    <td class="text-right">{{ formatCurrency(sales.grandTotal) }}</td>
                    <td class="text-center">
                      <button @click="selectSales(sales)" class="btn-select">選択</button>
                    </td>
                  </tr>
                  <tr v-if="filteredSales.length === 0">
                    <td colspan="7" class="no-data">該当する売上データがありません</td>
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
import { ref } from 'vue'

const props = defineProps({
  isOpen: {
    type: Boolean,
    required: true
  },
  salesList: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['close', 'select'])

// 検索条件
const searchConditions = ref({
  salesNo: '',
  dateFrom: '',
  dateTo: '',
  customer: ''
})

// フィルタリングされた売上データ
const filteredSales = ref([...props.salesList])

// 売上データのフィルタリング
const filterSales = () => {
  filteredSales.value = props.salesList.filter(sales => {
    // 売上番号での絞り込み
    if (searchConditions.value.salesNo) {
      if (!sales.salesNo.toLowerCase().includes(searchConditions.value.salesNo.toLowerCase())) {
        return false
      }
    }
    
    // 売上日（開始）での絞り込み
    if (searchConditions.value.dateFrom) {
      if (sales.date < searchConditions.value.dateFrom) {
        return false
      }
    }
    
    // 売上日（終了）での絞り込み
    if (searchConditions.value.dateTo) {
      if (sales.date > searchConditions.value.dateTo) {
        return false
      }
    }
    
    // 得意先での絞り込み
    if (searchConditions.value.customer) {
      const customerName = sales.customerName.toLowerCase()
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
    salesNo: '',
    dateFrom: '',
    dateTo: '',
    customer: ''
  }
  filteredSales.value = [...props.salesList]
}

// 売上の選択
const selectSales = (sales) => {
  emit('select', sales)
  closeModal()
}

// モーダルを閉じる
const closeModal = () => {
  emit('close')
}

// 日付フォーマット
const formatDate = (dateString) => {
  const date = new Date(dateString)
  return `${date.getMonth() + 1}/${date.getDate()}`
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
