<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="closeModal">
    <div class="modal-content quotation-preview-modal">
      <div class="modal-header">
        <h2>見積書プレビュー</h2>
        <button @click="closeModal" class="btn-close">×</button>
      </div>
      
      <div class="modal-body">
        <!-- 印刷用ボタン（印刷時は非表示） -->
        <div class="action-buttons no-print">
          <button @click="printQuotation" class="btn-print">🖨️ 印刷</button>
          <button @click="downloadPDF" class="btn-pdf">📄 PDF保存</button>
        </div>

        <!-- 見積書本体 -->
        <div class="quotation-document" ref="quotationDoc">
          <div class="document-header">
            <h1>見積書</h1>
            <div class="quotation-number">見積番号: {{ quotationData.quotationNo }}</div>
          </div>

          <div class="document-meta">
            <div class="meta-left">
              <div class="customer-info">
                <p class="customer-name">{{ quotationData.customerName }} 御中</p>
                <p class="quotation-date">見積日: {{ formatDate(quotationData.date) }}</p>
              </div>
            </div>
            <div class="meta-right">
              <div class="company-info">
                <p class="company-name">株式会社サンプル商事</p>
                <p class="company-address">〒100-0001</p>
                <p class="company-address">東京都千代田区千代田1-1-1</p>
                <p class="company-tel">TEL: 03-1234-5678</p>
                <p class="company-担当">担当: {{ quotationData.staffName }}</p>
              </div>
            </div>
          </div>

          <div class="total-amount-section">
            <p class="total-label">お見積金額</p>
            <p class="total-amount">¥{{ formatCurrency(quotationData.grandTotal) }}</p>
            <p class="tax-note">（消費税込）</p>
          </div>

          <div class="quotation-message">
            <p>下記の通り、お見積申し上げます。</p>
          </div>

          <!-- 明細テーブル -->
          <table class="detail-table">
            <thead>
              <tr>
                <th style="width: 40px">No</th>
                <th style="width: 120px">商品コード</th>
                <th>商品名</th>
                <th style="width: 80px">数量</th>
                <th style="width: 120px">単価</th>
                <th style="width: 120px">金額</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in quotationData.details" :key="index">
                <td class="text-center">{{ index + 1 }}</td>
                <td>{{ item.productCode }}</td>
                <td>{{ item.productName }}</td>
                <td class="text-right">{{ item.quantity }}</td>
                <td class="text-right">¥{{ formatCurrency(item.unitPrice) }}</td>
                <td class="text-right">¥{{ formatCurrency(item.amount) }}</td>
              </tr>
            </tbody>
          </table>

          <!-- 合計セクション -->
          <div class="quotation-totals">
            <table class="totals-table">
              <tbody>
                <tr>
                  <th>小計</th>
                  <td>¥{{ formatCurrency(quotationData.subtotal) }}</td>
                </tr>
                <tr>
                  <th>消費税 (10%)</th>
                  <td>¥{{ formatCurrency(quotationData.tax) }}</td>
                </tr>
                <tr class="grand-total-row">
                  <th>合計金額</th>
                  <td>¥{{ formatCurrency(quotationData.grandTotal) }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <div class="quotation-notes">
            <p class="notes-title">【備考】</p>
            <p>・本見積の有効期限は発行日より30日間とさせていただきます。</p>
            <p>・納期については別途ご相談させていただきます。</p>
            <p>・ご不明な点がございましたら、お気軽にお問い合わせください。</p>
          </div>

          <div class="document-footer">
            <p class="footer-text">以上</p>
          </div>
        </div>
      </div>
      
      <div class="modal-footer no-print">
        <button @click="closeModal" class="btn-secondary">閉じる</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  isOpen: {
    type: Boolean,
    required: true
  },
  quotationData: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['close'])

const quotationDoc = ref(null)

const closeModal = () => {
  emit('close')
}

// 日付フォーマット
const formatDate = (dateStr) => {
  if (!dateStr) return ''
  const date = new Date(dateStr)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}年${month}月${day}日`
}

// 通貨フォーマット
const formatCurrency = (amount) => {
  if (amount == null) return '0'
  return amount.toLocaleString('ja-JP')
}

// 印刷機能
const printQuotation = () => {
  window.print()
}

// PDF保存機能（ブラウザの印刷→PDF機能を利用）
const downloadPDF = () => {
  alert('印刷ダイアログで「PDFに保存」を選択してください。')
  window.print()
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
  max-width: 900px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.quotation-preview-modal {
  max-width: 1000px;
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

.action-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 2px dashed #ddd;
}

.btn-print,
.btn-pdf {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.btn-print:hover {
  background-color: #45a049;
}

.btn-pdf {
  background-color: #2196F3;
}

.btn-pdf:hover {
  background-color: #1976D2;
}

/* 見積書本体スタイル */
.quotation-document {
  background: white;
  padding: 40px;
  border: 1px solid #ddd;
  min-height: 800px;
}

.document-header {
  text-align: center;
  margin-bottom: 30px;
  border-bottom: 3px double #333;
  padding-bottom: 20px;
}

.document-header h1 {
  font-size: 32px;
  margin: 0 0 10px 0;
  color: #333;
}

.quotation-number {
  font-size: 14px;
  color: #666;
}

.document-meta {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30px;
}

.meta-left {
  flex: 1;
}

.customer-info {
  margin-bottom: 20px;
}

.customer-name {
  font-size: 20px;
  font-weight: bold;
  margin: 0 0 10px 0;
  color: #333;
}

.quotation-date {
  font-size: 14px;
  color: #666;
  margin: 5px 0;
}

.meta-right {
  text-align: right;
}

.company-info {
  font-size: 14px;
  color: #333;
}

.company-name {
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 5px;
}

.company-info p {
  margin: 3px 0;
}

.total-amount-section {
  background-color: #f5f5f5;
  border: 2px solid #333;
  padding: 20px;
  text-align: center;
  margin-bottom: 30px;
}

.total-label {
  font-size: 16px;
  margin: 0 0 10px 0;
  color: #666;
}

.total-amount {
  font-size: 36px;
  font-weight: bold;
  margin: 10px 0;
  color: #333;
}

.tax-note {
  font-size: 12px;
  color: #666;
  margin: 5px 0 0 0;
}

.quotation-message {
  margin-bottom: 20px;
  font-size: 14px;
  color: #333;
}

/* 明細テーブル */
.detail-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 30px;
  font-size: 14px;
}

.detail-table thead {
  background-color: #f0f0f0;
}

.detail-table th {
  padding: 12px 8px;
  text-align: left;
  font-weight: 600;
  color: #333;
  border: 1px solid #ddd;
}

.detail-table td {
  padding: 10px 8px;
  border: 1px solid #ddd;
}

.detail-table tbody tr:nth-child(even) {
  background-color: #fafafa;
}

.text-center {
  text-align: center;
}

.text-right {
  text-align: right;
}

/* 合計テーブル */
.quotation-totals {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 30px;
}

.totals-table {
  width: 350px;
  border-collapse: collapse;
  font-size: 14px;
}

.totals-table th,
.totals-table td {
  padding: 10px 15px;
  border: 1px solid #ddd;
}

.totals-table th {
  text-align: left;
  background-color: #f5f5f5;
  font-weight: 600;
  width: 150px;
}

.totals-table td {
  text-align: right;
  font-weight: 600;
}

.grand-total-row {
  background-color: #e3f2fd;
  font-weight: bold;
}

.grand-total-row th,
.grand-total-row td {
  font-size: 16px;
  color: #1976D2;
}

.quotation-notes {
  margin-bottom: 30px;
  padding: 15px;
  background-color: #f9f9f9;
  border-left: 4px solid #4CAF50;
  font-size: 13px;
  color: #555;
}

.notes-title {
  font-weight: bold;
  margin: 0 0 10px 0;
  color: #333;
}

.quotation-notes p {
  margin: 5px 0;
}

.document-footer {
  text-align: right;
  margin-top: 30px;
}

.footer-text {
  font-size: 14px;
  margin: 0;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 20px;
  border-top: 2px solid #eee;
}

.btn-secondary {
  background-color: #9E9E9E;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.btn-secondary:hover {
  background-color: #757575;
}

/* 印刷用スタイル */
@media print {
  /* モーダルオーバーレイを非表示 */
  .modal-overlay {
    position: static;
    background: none;
  }

  .modal-content {
    width: 100%;
    max-width: none;
    max-height: none;
    box-shadow: none;
    border-radius: 0;
  }

  .modal-header,
  .modal-footer,
  .no-print {
    display: none !important;
  }

  .modal-body {
    padding: 0;
    overflow: visible;
  }

  .quotation-document {
    border: none;
    padding: 20mm;
  }

  /* ページ区切りの調整 */
  .detail-table {
    page-break-inside: avoid;
  }

  /* 背景色を印刷 */
  .total-amount-section,
  .quotation-notes,
  .grand-total-row {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
}
</style>
