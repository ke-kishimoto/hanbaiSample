<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>見積検索</h3>
          <button @click="closeModal" class="close-button">&times;</button>
        </div>
        
        <div class="modal-body">
          <!-- 検索フォーム -->
          <div class="search-form">
            <div class="search-group">
              <label>見積番号</label>
              <input
                type="text"
                v-model="searchConditions.quotationNo"
                placeholder="見積番号で検索"
                @input="filterQuotations"
              />
            </div>
            
            <div class="search-group">
              <label>見積日（開始）</label>
              <input
                type="date"
                v-model="searchConditions.dateFrom"
                @change="filterQuotations"
              />
            </div>
            
            <div class="search-group">
              <label>見積日（終了）</label>
              <input
                type="date"
                v-model="searchConditions.dateTo"
                @change="filterQuotations"
              />
            </div>
            
            <div class="search-group">
              <label>得意先</label>
              <input
                type="text"
                v-model="searchConditions.customer"
                placeholder="得意先名で検索"
                @input="filterQuotations"
              />
            </div>
            
            <div class="search-actions">
              <button @click="resetSearch" class="btn-reset">クリア</button>
            </div>
          </div>
          
          <!-- 検索結果 -->
          <div class="search-results">
            <div class="results-header">
              <span>検索結果: {{ filteredQuotations.length }}件</span>
            </div>
            
            <div class="results-table-container">
              <table class="results-table">
                <thead>
                  <tr>
                    <th style="width: 140px">見積番号</th>
                    <th style="width: 100px">見積日</th>
                    <th>得意先</th>
                    <th style="width: 100px">担当者</th>
                    <th style="width: 120px">金額</th>
                    <th style="width: 80px">操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="quotation in filteredQuotations" :key="quotation.quotationNo">
                    <td>{{ quotation.quotationNo }}</td>
                    <td>{{ formatDate(quotation.date) }}</td>
                    <td>{{ quotation.customerName }}</td>
                    <td>{{ quotation.staffName }}</td>
                    <td class="text-right">{{ formatCurrency(quotation.grandTotal) }}</td>
                    <td class="text-center">
                      <button @click="selectQuotation(quotation)" class="btn-select">選択</button>
                    </td>
                  </tr>
                  <tr v-if="filteredQuotations.length === 0">
                    <td colspan="6" class="no-data">該当する見積データがありません</td>
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
import { ref, watch } from 'vue'

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

// サンプル見積データ
const quotations = ref([
  {
    quotationNo: 'QU-2026-001',
    date: '2026-01-15',
    customerCode: 'C001',
    customerName: '株式会社山田商事',
    customer: props.customers.find(c => c.code === 'C001'),
    staffCode: 'S001',
    staffName: '田中太郎',
    staff: props.staffList.find(s => s.code === 'S001'),
    details: [
      { id: 1, productCode: 'P001', productName: 'ノートPC', product: props.products.find(p => p.code === 'P001'), quantity: 5, unitPrice: 120000 },
      { id: 2, productCode: 'P002', productName: 'デスクトップPC', product: props.products.find(p => p.code === 'P002'), quantity: 3, unitPrice: 150000 }
    ],
    grandTotal: 1155000
  },
  {
    quotationNo: 'QU-2026-002',
    date: '2026-01-18',
    customerCode: 'C002',
    customerName: '鈴木物産株式会社',
    customer: props.customers.find(c => c.code === 'C002'),
    staffCode: 'S002',
    staffName: '佐藤花子',
    staff: props.staffList.find(s => s.code === 'S002'),
    details: [
      { id: 1, productCode: 'P003', productName: 'プリンター', product: props.products.find(p => p.code === 'P003'), quantity: 2, unitPrice: 35000 },
      { id: 2, productCode: 'P004', productName: 'モニター', product: props.products.find(p => p.code === 'P004'), quantity: 10, unitPrice: 25000 }
    ],
    grandTotal: 352000
  },
  {
    quotationNo: 'QU-2026-003',
    date: '2026-01-20',
    customerCode: 'C003',
    customerName: '佐藤商店',
    customer: props.customers.find(c => c.code === 'C003'),
    staffCode: 'S003',
    staffName: '鈴木次郎',
    staff: props.staffList.find(s => s.code === 'S003'),
    details: [
      { id: 1, productCode: 'P005', productName: 'マウス', product: props.products.find(p => p.code === 'P005'), quantity: 20, unitPrice: 2000 },
      { id: 2, productCode: 'P006', productName: 'キーボード', product: props.products.find(p => p.code === 'P006'), quantity: 15, unitPrice: 5000 }
    ],
    grandTotal: 126500
  },
  {
    quotationNo: 'QU-2026-004',
    date: '2026-01-22',
    customerCode: 'C004',
    customerName: '高橋工業株式会社',
    customer: props.customers.find(c => c.code === 'C004'),
    staffCode: 'S004',
    staffName: '高橋美咲',
    staff: props.staffList.find(s => s.code === 'S004'),
    details: [
      { id: 1, productCode: 'P007', productName: 'USBメモリ', product: props.products.find(p => p.code === 'P007'), quantity: 50, unitPrice: 1500 },
      { id: 2, productCode: 'P008', productName: '外付けHDD', product: props.products.find(p => p.code === 'P008'), quantity: 5, unitPrice: 12000 }
    ],
    grandTotal: 148500
  },
  {
    quotationNo: 'QU-2026-005',
    date: '2026-01-25',
    customerCode: 'C005',
    customerName: '株式会社田中エンタープライズ',
    customer: props.customers.find(c => c.code === 'C005'),
    staffCode: 'S005',
    staffName: '中村誠',
    staff: props.staffList.find(s => s.code === 'S005'),
    details: [
      { id: 1, productCode: 'P009', productName: 'Webカメラ', product: props.products.find(p => p.code === 'P009'), quantity: 8, unitPrice: 8000 },
      { id: 2, productCode: 'P010', productName: 'ヘッドセット', product: props.products.find(p => p.code === 'P010'), quantity: 12, unitPrice: 6000 }
    ],
    grandTotal: 149600
  }
])

// 検索条件
const searchConditions = ref({
  quotationNo: '',
  dateFrom: '',
  dateTo: '',
  customer: ''
})

// フィルタリングされた見積データ
const filteredQuotations = ref([...quotations.value])

// 見積データのフィルタリング
const filterQuotations = () => {
  filteredQuotations.value = quotations.value.filter(quotation => {
    // 見積番号での絞り込み
    if (searchConditions.value.quotationNo) {
      if (!quotation.quotationNo.toLowerCase().includes(searchConditions.value.quotationNo.toLowerCase())) {
        return false
      }
    }
    
    // 見積日（開始）での絞り込み
    if (searchConditions.value.dateFrom) {
      if (quotation.date < searchConditions.value.dateFrom) {
        return false
      }
    }
    
    // 見積日（終了）での絞り込み
    if (searchConditions.value.dateTo) {
      if (quotation.date > searchConditions.value.dateTo) {
        return false
      }
    }
    
    // 得意先での絞り込み
    if (searchConditions.value.customer) {
      const customerName = quotation.customerName.toLowerCase()
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
    quotationNo: '',
    dateFrom: '',
    dateTo: '',
    customer: ''
  }
  filteredQuotations.value = [...quotations.value]
}

// モーダルが開いたときに検索結果をリセット
watch(() => props.isOpen, (newValue) => {
  if (newValue) {
    resetSearch()
  }
})

// 見積の選択
const selectQuotation = (quotation) => {
  emit('select', quotation)
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
