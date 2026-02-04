<template>
  <div class="invoice-generator">
    <h2>請求書発行</h2>
    
    <!-- 検索条件 -->
    <div class="search-section">
      <div class="search-form">
        <div class="form-group">
          <label>締め日 <span class="required">*</span></label>
          <input
            type="date"
            v-model="searchConditions.closingDate"
          />
        </div>
        
        <div class="form-group">
          <label>得意先 <span class="required">*</span></label>
          <Autocomplete
            v-model="searchConditions.customer"
            :items="props.customers"
            placeholder="得意先を選択してください"
          />
        </div>
        
        <div class="form-actions">
          <button @click="generateInvoice" class="btn-generate">
            📄 請求書作成
          </button>
          <button @click="clearConditions" class="btn-clear">
            クリア
          </button>
        </div>
      </div>
      
      <div v-if="invoiceData" class="invoice-summary">
        <div class="summary-item">
          <span class="summary-label">請求対象期間:</span>
          <span class="summary-value">{{ formatDate(invoiceData.periodFrom) }} 〜 {{ formatDate(invoiceData.periodTo) }}</span>
        </div>
        <div class="summary-item">
          <span class="summary-label">売上件数:</span>
          <span class="summary-value">{{ invoiceData.items.length }}件</span>
        </div>
        <div class="summary-item">
          <span class="summary-label">請求金額:</span>
          <span class="summary-value total">{{ formatCurrency(invoiceData.grandTotal) }}</span>
        </div>
      </div>
    </div>
    
    <!-- 請求書プレビュー -->
    <div v-if="invoiceData" class="invoice-preview">
      <div class="preview-actions">
        <button @click="printInvoice" class="btn-action">
          🖨️ 印刷
        </button>
        <button @click="downloadPDF" class="btn-action">
          📥 PDF出力
        </button>
      </div>
      
      <div class="invoice-document" ref="invoiceDocumentRef">
        <!-- 請求書ヘッダー -->
        <div class="invoice-header">
          <div class="invoice-title">
            <h1>請求書</h1>
            <div class="invoice-number">No. {{ invoiceData.invoiceNo }}</div>
          </div>
          
          <div class="company-info">
            <h3>株式会社サンプル商事</h3>
            <p>〒100-0001 東京都千代田区千代田1-1-1</p>
            <p>TEL: 03-1234-5678 / FAX: 03-1234-5679</p>
            <p>Email: info@sample-shoji.co.jp</p>
          </div>
        </div>
        
        <div class="invoice-info-section">
          <div class="customer-info">
            <div class="customer-name">{{ invoiceData.customer.name }} 御中</div>
            <div class="customer-code">得意先コード: {{ invoiceData.customer.code }}</div>
          </div>
          
          <div class="invoice-meta">
            <table class="meta-table">
              <tr>
                <th>発行日:</th>
                <td>{{ formatDate(invoiceData.issueDate) }}</td>
              </tr>
              <tr>
                <th>締め日:</th>
                <td>{{ formatDate(invoiceData.closingDate) }}</td>
              </tr>
              <tr>
                <th>対象期間:</th>
                <td>{{ formatDate(invoiceData.periodFrom) }} 〜 {{ formatDate(invoiceData.periodTo) }}</td>
              </tr>
              <tr>
                <th>支払期限:</th>
                <td>{{ formatDate(invoiceData.paymentDue) }}</td>
              </tr>
            </table>
          </div>
        </div>
        
        <!-- 請求金額サマリー -->
        <div class="invoice-summary-box">
          <div class="summary-row">
            <span class="summary-label">ご請求金額</span>
            <span class="summary-amount">{{ formatCurrency(invoiceData.grandTotal) }}</span>
          </div>
        </div>
        
        <!-- 明細 -->
        <div class="invoice-details">
          <h3>明細</h3>
          <table class="details-table">
            <thead>
              <tr>
                <th style="width: 100px">売上日</th>
                <th style="width: 120px">受注番号</th>
                <th>摘要</th>
                <th style="width: 120px" class="text-right">金額</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in invoiceData.items" :key="index">
                <td>{{ formatDate(item.salesDate) }}</td>
                <td>{{ item.orderNo }}</td>
                <td>{{ item.description }}</td>
                <td class="text-right">{{ formatCurrency(item.amount) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        
        <!-- 合計 -->
        <div class="invoice-totals">
          <table class="totals-table">
            <tr>
              <th>小計:</th>
              <td>{{ formatCurrency(invoiceData.subtotal) }}</td>
            </tr>
            <tr>
              <th>消費税 (10%):</th>
              <td>{{ formatCurrency(invoiceData.tax) }}</td>
            </tr>
            <tr class="grand-total">
              <th>合計金額:</th>
              <td>{{ formatCurrency(invoiceData.grandTotal) }}</td>
            </tr>
          </table>
        </div>
        
        <!-- フッター -->
        <div class="invoice-footer">
          <div class="payment-info">
            <h4>お振込先</h4>
            <p>銀行名: サンプル銀行 本店</p>
            <p>口座種別: 普通</p>
            <p>口座番号: 1234567</p>
            <p>口座名義: カ）サンプルショウジ</p>
          </div>
          
          <div class="notes">
            <p>※お振込手数料はお客様のご負担でお願いいたします。</p>
            <p>※ご不明な点がございましたら、お気軽にお問い合わせください。</p>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 未選択時のメッセージ -->
    <div v-if="!invoiceData" class="empty-state">
      <div class="empty-icon">📄</div>
      <p>締め日と得意先を選択して、請求書を作成してください。</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Autocomplete from './Autocomplete.vue'

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

// 検索条件
const searchConditions = ref({
  closingDate: new Date().toISOString().split('T')[0],
  customer: null
})

// 請求書データ
const invoiceData = ref(null)
const invoiceDocumentRef = ref(null)

// サンプル売上データ（実際は受注から生成される）
const generateSalesData = (customer, closingDate) => {
  // 締め日から対象期間を計算（前月の締め日翌日から当月締め日まで）
  const closing = new Date(closingDate)
  const periodTo = new Date(closing)
  const periodFrom = new Date(closing)
  periodFrom.setMonth(periodFrom.getMonth() - 1)
  periodFrom.setDate(periodFrom.getDate() + 1)
  
  // サンプル売上データを生成（得意先ごとに異なるデータ）
  const salesData = []
  const customerIndex = props.customers.findIndex(c => c.id === customer.id)
  
  // 各得意先に3-5件の売上データを生成
  const salesCount = 3 + (customerIndex % 3)
  
  for (let i = 0; i < salesCount; i++) {
    const date = new Date(periodFrom)
    date.setDate(date.getDate() + Math.floor((periodTo - periodFrom) / (1000 * 60 * 60 * 24) * (i + 1) / (salesCount + 1)))
    
    const products = []
    const productCount = 1 + (i % 3)
    for (let j = 0; j < productCount; j++) {
      const product = props.products[(customerIndex * 3 + i * 2 + j) % props.products.length]
      products.push(product.name)
    }
    
    const amount = 50000 + (customerIndex * 10000) + (i * 20000) + ((customerIndex + i) % 5) * 5000
    
    salesData.push({
      salesDate: date.toISOString().split('T')[0],
      orderNo: `OR-2026-${String(customerIndex * 10 + i + 1).padStart(3, '0')}`,
      description: products.join(', '),
      amount: amount
    })
  }
  
  return salesData
}

// 請求書番号を生成
const generateInvoiceNo = (closingDate) => {
  const date = new Date(closingDate)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const random = Math.floor(Math.random() * 1000).toString().padStart(3, '0')
  return `INV-${year}${month}-${random}`
}

// 請求書作成
const generateInvoice = () => {
  if (!searchConditions.value.closingDate) {
    alert('締め日を選択してください。')
    return
  }
  
  if (!searchConditions.value.customer) {
    alert('得意先を選択してください。')
    return
  }
  
  // 対象期間を計算
  const closing = new Date(searchConditions.value.closingDate)
  const periodTo = new Date(closing)
  const periodFrom = new Date(closing)
  periodFrom.setMonth(periodFrom.getMonth() - 1)
  periodFrom.setDate(periodFrom.getDate() + 1)
  
  // 支払期限を計算（締め日の翌月末）
  const paymentDue = new Date(closing)
  paymentDue.setMonth(paymentDue.getMonth() + 1)
  paymentDue.setDate(0) // 前月の最終日 = 当月末
  
  // 売上データを生成
  const salesItems = generateSalesData(searchConditions.value.customer, searchConditions.value.closingDate)
  
  // 小計を計算
  const subtotal = salesItems.reduce((sum, item) => sum + item.amount, 0)
  
  // 消費税を計算
  const tax = Math.floor(subtotal * 0.1)
  
  // 合計を計算
  const grandTotal = subtotal + tax
  
  // 請求書データを作成
  invoiceData.value = {
    invoiceNo: generateInvoiceNo(searchConditions.value.closingDate),
    issueDate: new Date().toISOString().split('T')[0],
    closingDate: searchConditions.value.closingDate,
    periodFrom: periodFrom.toISOString().split('T')[0],
    periodTo: periodTo.toISOString().split('T')[0],
    paymentDue: paymentDue.toISOString().split('T')[0],
    customer: searchConditions.value.customer,
    items: salesItems,
    subtotal: subtotal,
    tax: tax,
    grandTotal: grandTotal
  }
  
  // スクロールしてプレビューを表示
  setTimeout(() => {
    if (invoiceDocumentRef.value) {
      invoiceDocumentRef.value.scrollIntoView({ behavior: 'smooth', block: 'start' })
    }
  }, 100)
}

// 条件クリア
const clearConditions = () => {
  searchConditions.value = {
    closingDate: new Date().toISOString().split('T')[0],
    customer: null
  }
  invoiceData.value = null
}

// 印刷
const printInvoice = () => {
  window.print()
}

// PDF出力（デモ）
const downloadPDF = () => {
  alert('PDF出力機能はデモのため実装されていません。\n実際のシステムでは、サーバー側でPDFを生成します。')
}

// 日付フォーマット
const formatDate = (dateString) => {
  const date = new Date(dateString)
  return `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日`
}

// 通貨フォーマット
const formatCurrency = (value) => {
  return '¥' + value.toLocaleString('ja-JP')
}
</script>

<style scoped>
.invoice-generator {
  margin-top: 20px;
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 24px;
}

/* 検索セクション */
.search-section {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 30px;
}

.search-form {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  gap: 20px;
  align-items: end;
  margin-bottom: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-group label {
  font-weight: 600;
  margin-bottom: 8px;
  color: #555;
}

.form-group input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.form-group input:focus {
  outline: none;
  border-color: #4CAF50;
}

.required {
  color: #f44336;
}

.form-actions {
  display: flex;
  gap: 10px;
}

.btn-generate {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: background-color 0.3s;
  white-space: nowrap;
}

.btn-generate:hover {
  background-color: #45a049;
}

.btn-clear {
  background-color: #9E9E9E;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.btn-clear:hover {
  background-color: #757575;
}

/* サマリー */
.invoice-summary {
  display: flex;
  gap: 30px;
  padding: 15px 20px;
  background-color: #fff;
  border-radius: 4px;
  border: 1px solid #e0e0e0;
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
  color: #333;
  font-size: 16px;
}

.summary-value.total {
  color: #4CAF50;
  font-size: 20px;
}

/* プレビューアクション */
.invoice-preview {
  margin-top: 30px;
}

.preview-actions {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  justify-content: flex-end;
}

.btn-action {
  background-color: #2196F3;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: background-color 0.3s;
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn-action:hover {
  background-color: #1976D2;
}

/* 請求書ドキュメント */
.invoice-document {
  background-color: white;
  padding: 40px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  max-width: 900px;
  margin: 0 auto;
}

.invoice-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 3px solid #333;
}

.invoice-title h1 {
  font-size: 32px;
  margin: 0 0 10px 0;
  color: #333;
}

.invoice-number {
  font-size: 14px;
  color: #666;
}

.company-info {
  text-align: right;
}

.company-info h3 {
  margin: 0 0 10px 0;
  font-size: 18px;
  color: #333;
}

.company-info p {
  margin: 3px 0;
  font-size: 12px;
  color: #666;
}

.invoice-info-section {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30px;
}

.customer-info {
  flex: 1;
}

.customer-name {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 5px;
  color: #333;
}

.customer-code {
  font-size: 13px;
  color: #666;
}

.invoice-meta {
  flex: 1;
  text-align: right;
}

.meta-table {
  display: inline-block;
  text-align: left;
}

.meta-table th {
  font-weight: 600;
  color: #666;
  padding: 5px 15px 5px 0;
  font-size: 13px;
  text-align: left;
}

.meta-table td {
  padding: 5px 0;
  font-size: 13px;
  color: #333;
}

/* 請求金額サマリーボックス */
.invoice-summary-box {
  background-color: #f5f5f5;
  padding: 20px;
  margin-bottom: 30px;
  border-radius: 4px;
  border: 2px solid #4CAF50;
}

.invoice-summary-box .summary-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.invoice-summary-box .summary-label {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.invoice-summary-box .summary-amount {
  font-size: 32px;
  font-weight: 700;
  color: #4CAF50;
}

/* 明細 */
.invoice-details {
  margin-bottom: 30px;
}

.invoice-details h3 {
  font-size: 16px;
  margin-bottom: 15px;
  color: #333;
  padding-bottom: 10px;
  border-bottom: 2px solid #e0e0e0;
}

.details-table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid #ddd;
}

.details-table thead {
  background-color: #f5f5f5;
}

.details-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #555;
  border-bottom: 2px solid #ddd;
  font-size: 13px;
}

.details-table td {
  padding: 10px 12px;
  border-bottom: 1px solid #eee;
  font-size: 13px;
  color: #333;
}

.details-table tbody tr:last-child td {
  border-bottom: none;
}

.text-right {
  text-align: right;
}

/* 合計 */
.invoice-totals {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 30px;
}

.totals-table {
  min-width: 300px;
}

.totals-table tr {
  border-bottom: 1px solid #e0e0e0;
}

.totals-table th {
  padding: 10px 20px;
  text-align: left;
  font-weight: 600;
  color: #555;
  font-size: 14px;
}

.totals-table td {
  padding: 10px 20px;
  text-align: right;
  font-weight: 600;
  color: #333;
  font-size: 14px;
}

.totals-table .grand-total {
  border-top: 2px solid #333;
  border-bottom: 3px double #333;
}

.totals-table .grand-total th,
.totals-table .grand-total td {
  padding: 15px 20px;
  font-size: 18px;
  color: #4CAF50;
}

/* フッター */
.invoice-footer {
  margin-top: 40px;
  padding-top: 20px;
  border-top: 2px solid #e0e0e0;
}

.payment-info {
  margin-bottom: 20px;
}

.payment-info h4 {
  font-size: 14px;
  margin-bottom: 10px;
  color: #333;
}

.payment-info p {
  margin: 3px 0;
  font-size: 13px;
  color: #666;
}

.notes p {
  margin: 5px 0;
  font-size: 12px;
  color: #999;
}

/* 空の状態 */
.empty-state {
  text-align: center;
  padding: 80px 20px;
  color: #999;
}

.empty-icon {
  font-size: 64px;
  margin-bottom: 20px;
}

.empty-state p {
  font-size: 16px;
}

/* 印刷時のスタイル */
@media print {
  .search-section,
  .preview-actions {
    display: none !important;
  }
  
  .invoice-document {
    box-shadow: none;
    border: none;
    padding: 20px;
  }
}
</style>
