<template>
  <div class="sales-form">
    <h2>売上入力</h2>
    
    <!-- ヘッダー情報 -->
    <div class="header-section">
      <!-- 1行目：売上番号、日付、受注番号 -->
      <div class="header-row-1">
        <div class="form-group">
          <label>売上番号</label>
          <div class="sales-no-input-group">
            <input
              type="text"
              v-model="salesHeader.salesNo"
              readonly
              class="input-readonly"
            />
            <button @click="openSalesSearch" class="btn-search-small">
              <span>🔍</span>
            </button>
          </div>
        </div>
        
        <div class="form-group">
          <label>日付 <span class="required">*</span></label>
          <input type="date" v-model="salesHeader.date" />
        </div>
        
        <div class="form-group">
          <label>受注番号</label>
          <div class="order-no-input-group">
            <input 
              type="text" 
              v-model="salesHeader.orderNo" 
              placeholder="受注番号"
              readonly
            />
            <button @click="openOrderSearch" class="btn-search">
              <span>🔍</span> 受注検索
            </button>
          </div>
        </div>
      </div>
      
      <!-- 2行目：得意先コード、得意先名、担当者コード、担当者名 -->
      <div class="header-row-2">
        <div class="form-group">
          <label>得意先コード <span class="required">*</span></label>
          <div class="code-input-group">
            <input
              type="text"
              v-model="salesHeader.customerCode"
              placeholder="得意先コード"
              class="input-code"
              @blur="onCustomerCodeInput"
            />
            <button
              @click="openCustomerSearch"
              class="btn-search-small"
              title="得意先検索"
            >
              🔍
            </button>
          </div>
        </div>
        
        <div class="form-group">
          <label>得意先名</label>
          <input
            type="text"
            v-model="salesHeader.customerName"
            readonly
            class="input-readonly"
            placeholder="得意先名"
          />
        </div>
        
        <div class="form-group">
          <label>担当者コード <span class="required">*</span></label>
          <div class="code-input-group">
            <input
              type="text"
              v-model="salesHeader.staffCode"
              placeholder="担当者コード"
              class="input-code"
              @blur="onStaffCodeInput"
            />
            <button
              @click="openStaffSearch"
              class="btn-search-small"
              title="担当者検索"
            >
              🔍
            </button>
          </div>
        </div>
        
        <div class="form-group">
          <label>担当者名</label>
          <input
            type="text"
            v-model="salesHeader.staffName"
            readonly
            class="input-readonly"
            placeholder="担当者名"
          />
        </div>
      </div>
    </div>
    
    <!-- 得意先検索モーダル -->
    <CustomerSearchModal
      :isOpen="isCustomerSearchOpen"
      :customers="props.customers"
      @close="closeCustomerSearch"
      @select="onCustomerSelected"
    />
    
    <!-- 担当者検索モーダル -->
    <StaffSearchModal
      :isOpen="isStaffSearchOpen"
      :staffList="props.staffList"
      @close="closeStaffSearch"
      @select="onStaffSelected"
    />
    
    <!-- 受注検索モーダル -->
    <OrderSearchModal
      :isOpen="isOrderSearchOpen"
      :customers="props.customers"
      :staffList="props.staffList"
      :products="props.products"
      @close="closeOrderSearch"
      @select="onOrderSelected"
    />
    
    <!-- 売上検索モーダル -->
    <SalesSearchModal
      :isOpen="isSalesSearchOpen"
      :salesList="salesList"
      @close="closeSalesSearch"
      @select="onSalesSelected"
    />
    
    <!-- 商品検索モーダル -->
    <ProductSearchModal
      :isOpen="isProductSearchOpen"
      :products="props.products"
      @close="closeProductSearch"
      @select="onProductSelected"
    />

    <!-- 明細セクション -->
    <div class="detail-section">
      <div class="section-header">
        <h3>明細</h3>
        <button @click="addDetailRow" class="btn-add">+ 行追加</button>
      </div>
      
      <div class="table-container">
        <table class="detail-table">
          <thead>
            <tr>
              <th style="width: 50px">No.</th>
              <th style="width: 150px">商品コード <span class="required">*</span></th>
              <th style="width: 250px">商品名</th>
              <th style="width: 120px">個数 <span class="required">*</span></th>
              <th style="width: 150px">単価 <span class="required">*</span></th>
              <th style="width: 150px">金額</th>
              <th style="width: 80px">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in salesDetails" :key="row.id">
              <td class="text-center">{{ index + 1 }}</td>
              <td>
                <div class="product-code-group">
                  <input
                    type="text"
                    v-model="row.productCode"
                    placeholder="商品コード"
                    class="input-code"
                    @blur="onProductCodeInput(index)"
                  />
                  <button
                    @click="openProductSearch(index)"
                    class="btn-product-search"
                    title="商品検索"
                  >
                    🔍
                  </button>
                </div>
              </td>
              <td>
                <input
                  type="text"
                  v-model="row.productName"
                  readonly
                  class="input-readonly"
                  placeholder="商品名"
                />
              </td>
              <td>
                <input
                  type="number"
                  v-model.number="row.quantity"
                  min="0"
                  class="input-number"
                  @input="calculateRowAmount(index)"
                />
              </td>
              <td>
                <input
                  type="number"
                  v-model.number="row.unitPrice"
                  min="0"
                  class="input-number"
                  @input="calculateRowAmount(index)"
                />
              </td>
              <td class="text-right">
                {{ formatCurrency(row.amount) }}
              </td>
              <td class="text-center">
                <button
                  @click="deleteDetailRow(index)"
                  class="btn-delete"
                  :disabled="salesDetails.length === 1"
                >
                  削除
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- 合計セクション -->
    <div class="total-section">
      <div class="total-row">
        <span class="total-label">小計:</span>
        <span class="total-value">{{ formatCurrency(subtotal) }}</span>
      </div>
      <div class="total-row">
        <span class="total-label">消費税 (10%):</span>
        <span class="total-value">{{ formatCurrency(tax) }}</span>
      </div>
      <div class="total-row grand-total">
        <span class="total-label">合計:</span>
        <span class="total-value">{{ formatCurrency(grandTotal) }}</span>
      </div>
    </div>

    <!-- アクションボタン -->
    <div class="action-buttons">
      <button @click="resetForm" class="btn-secondary">クリア</button>
      <button @click="submitForm" class="btn-primary">登録</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import OrderSearchModal from './OrderSearchModal.vue'
import ProductSearchModal from './ProductSearchModal.vue'
import CustomerSearchModal from './CustomerSearchModal.vue'
import StaffSearchModal from './StaffSearchModal.vue'
import SalesSearchModal from './SalesSearchModal.vue'

// Propsを追加
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

// モーダルの表示状態
const isOrderSearchOpen = ref(false)
const isProductSearchOpen = ref(false)
const isCustomerSearchOpen = ref(false)
const isStaffSearchOpen = ref(false)
const isSalesSearchOpen = ref(false)
const selectedDetailIndex = ref(null) // 商品検索中の明細行インデックス

// ヘッダー情報
const salesHeader = ref({
  salesNo: generateSalesNo(), // 売上番号（自動採番）
  date: new Date().toISOString().split('T')[0],
  orderNo: '',
  customerCode: '',
  customerName: '',
  customer: null, // 内部管理用
  staffCode: '',
  staffName: '',
  staff: null // 内部管理用
})

// 売上番号を生成
function generateSalesNo() {
  const date = new Date()
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const random = Math.floor(Math.random() * 10000).toString().padStart(4, '0')
  return `UR-${year}${month}${day}-${random}`
}

// 明細情報
let detailIdCounter = 1
const salesDetails = ref([
  {
    id: detailIdCounter++,
    productCode: '',
    productName: '',
    product: null, // 商品マスタのオブジェクト（内部管理用）
    quantity: 0,
    unitPrice: 0,
    amount: 0
  }
])

// サンプル売上データ
const salesList = ref([
  {
    salesNo: 'UR-20260130-0001',
    date: '2026-01-30',
    orderNo: 'OR-2026-001',
    customerCode: 'C001',
    customerName: '株式会社山田商事',
    customer: props.customers.find(c => c.code === 'C001'),
    staffCode: 'S001',
    staffName: '田中太郎',
    staff: props.staffList.find(s => s.code === 'S001'),
    details: [
      { id: 1, productCode: 'P001', productName: 'ノートPC', product: props.products.find(p => p.code === 'P001'), quantity: 5, unitPrice: 120000, amount: 600000 },
      { id: 2, productCode: 'P002', productName: 'デスクトップPC', product: props.products.find(p => p.code === 'P002'), quantity: 3, unitPrice: 150000, amount: 450000 }
    ],
    subtotal: 1050000,
    tax: 105000,
    grandTotal: 1155000
  },
  {
    salesNo: 'UR-20260131-0002',
    date: '2026-01-31',
    orderNo: 'OR-2026-003',
    customerCode: 'C002',
    customerName: '鈴木物産株式会社',
    customer: props.customers.find(c => c.code === 'C002'),
    staffCode: 'S002',
    staffName: '佐藤花子',
    staff: props.staffList.find(s => s.code === 'S002'),
    details: [
      { id: 1, productCode: 'P003', productName: 'プリンター', product: props.products.find(p => p.code === 'P003'), quantity: 2, unitPrice: 35000, amount: 70000 },
      { id: 2, productCode: 'P004', productName: 'モニター', product: props.products.find(p => p.code === 'P004'), quantity: 10, unitPrice: 25000, amount: 250000 }
    ],
    subtotal: 320000,
    tax: 32000,
    grandTotal: 352000
  },
  {
    salesNo: 'UR-20260201-0003',
    date: '2026-02-01',
    orderNo: '',
    customerCode: 'C003',
    customerName: '佐藤商店',
    customer: props.customers.find(c => c.code === 'C003'),
    staffCode: 'S003',
    staffName: '鈴木次郎',
    staff: props.staffList.find(s => s.code === 'S003'),
    details: [
      { id: 1, productCode: 'P005', productName: 'マウス', product: props.products.find(p => p.code === 'P005'), quantity: 20, unitPrice: 2000, amount: 40000 },
      { id: 2, productCode: 'P006', productName: 'キーボード', product: props.products.find(p => p.code === 'P006'), quantity: 15, unitPrice: 5000, amount: 75000 }
    ],
    subtotal: 115000,
    tax: 11500,
    grandTotal: 126500
  },
  {
    salesNo: 'UR-20260202-0004',
    date: '2026-02-02',
    orderNo: 'OR-2026-007',
    customerCode: 'C004',
    customerName: '高橋工業株式会社',
    customer: props.customers.find(c => c.code === 'C004'),
    staffCode: 'S004',
    staffName: '高橋美咲',
    staff: props.staffList.find(s => s.code === 'S004'),
    details: [
      { id: 1, productCode: 'P007', productName: 'USBメモリ', product: props.products.find(p => p.code === 'P007'), quantity: 50, unitPrice: 1500, amount: 75000 },
      { id: 2, productCode: 'P008', productName: '外付けHDD', product: props.products.find(p => p.code === 'P008'), quantity: 5, unitPrice: 12000, amount: 60000 }
    ],
    subtotal: 135000,
    tax: 13500,
    grandTotal: 148500
  },
  {
    salesNo: 'UR-20260203-0005',
    date: '2026-02-03',
    orderNo: 'OR-2026-009',
    customerCode: 'C005',
    customerName: '株式会社田中エンタープライズ',
    customer: props.customers.find(c => c.code === 'C005'),
    staffCode: 'S005',
    staffName: '中村誠',
    staff: props.staffList.find(s => s.code === 'S005'),
    details: [
      { id: 1, productCode: 'P009', productName: 'Webカメラ', product: props.products.find(p => p.code === 'P009'), quantity: 8, unitPrice: 8000, amount: 64000 },
      { id: 2, productCode: 'P010', productName: 'ヘッドセット', product: props.products.find(p => p.code === 'P010'), quantity: 12, unitPrice: 6000, amount: 72000 }
    ],
    subtotal: 136000,
    tax: 13600,
    grandTotal: 149600
  }
])

// マスタデータはpropsから削除（App.vueから渡される）
// 以下の定義を削除

// 明細行の追加
const addDetailRow = () => {
  salesDetails.value.push({
    id: detailIdCounter++,
    productCode: '',
    productName: '',
    product: null,
    quantity: 0,
    unitPrice: 0,
    amount: 0
  })
}

// 明細行の削除
const deleteDetailRow = (index) => {
  if (salesDetails.value.length > 1) {
    salesDetails.value.splice(index, 1)
  }
}

// 商品検索モーダルを開く
const openProductSearch = (index) => {
  selectedDetailIndex.value = index
  isProductSearchOpen.value = true
}

// 商品検索モーダルを閉じる
const closeProductSearch = () => {
  isProductSearchOpen.value = false
  selectedDetailIndex.value = null
}

// 商品選択時の処理（モーダルから）
const onProductSelected = (product) => {
  if (selectedDetailIndex.value !== null) {
    const row = salesDetails.value[selectedDetailIndex.value]
    row.productCode = product.code
    row.productName = product.name
    row.product = product
    row.unitPrice = product.price
    calculateRowAmount(selectedDetailIndex.value)
  }
}

// 商品コード直接入力時の処理
const onProductCodeInput = (index) => {
  const row = salesDetails.value[index]
  const productCode = row.productCode.trim()
  
  if (!productCode) {
    row.productName = ''
    row.product = null
    row.unitPrice = 0
    calculateRowAmount(index)
    return
  }
  
  // 商品マスタから商品を検索
  const product = props.products.find(
    p => p.code.toLowerCase() === productCode.toLowerCase()
  )
  
  if (product) {
    row.productName = product.name
    row.product = product
    row.unitPrice = product.price
    calculateRowAmount(index)
  } else {
    // 商品が見つからない場合
    row.productName = '（商品コード不正）'
    row.product = null
    row.unitPrice = 0
    calculateRowAmount(index)
  }
}

// 商品選択時の処理（旧Autocomplete用 - 削除予定だが受注連携で使用）
const onProductChange = (index) => {
  const row = salesDetails.value[index]
  if (row.product && row.product.price) {
    row.unitPrice = row.product.price
    calculateRowAmount(index)
  }
}

// 行の金額計算
const calculateRowAmount = (index) => {
  const row = salesDetails.value[index]
  row.amount = (row.quantity || 0) * (row.unitPrice || 0)
}

// 小計の計算
const subtotal = computed(() => {
  return salesDetails.value.reduce((sum, row) => sum + (row.amount || 0), 0)
})

// 消費税の計算（10%）
const tax = computed(() => {
  return Math.floor(subtotal.value * 0.1)
})

// 合計金額の計算
const grandTotal = computed(() => {
  return subtotal.value + tax.value
})

// 通貨フォーマット
const formatCurrency = (value) => {
  return '¥' + value.toLocaleString('ja-JP')
}

// 受注検索モーダルを開く
const openOrderSearch = () => {
  isOrderSearchOpen.value = true
}

// 受注検索モーダルを閉じる
const closeOrderSearch = () => {
  isOrderSearchOpen.value = false
}

// 売上検索モーダルを開く
const openSalesSearch = () => {
  isSalesSearchOpen.value = true
}

// 売上検索モーダルを閉じる
const closeSalesSearch = () => {
  isSalesSearchOpen.value = false
}

// 売上が選択されたときの処理
const onSalesSelected = (sales) => {
  // ヘッダー情報を設定
  salesHeader.value.salesNo = sales.salesNo
  salesHeader.value.date = sales.date
  salesHeader.value.orderNo = sales.orderNo
  salesHeader.value.customerCode = sales.customerCode
  salesHeader.value.customerName = sales.customerName
  salesHeader.value.customer = sales.customer
  salesHeader.value.staffCode = sales.staffCode
  salesHeader.value.staffName = sales.staffName
  salesHeader.value.staff = sales.staff
  
  // 明細データを設定
  salesDetails.value = sales.details.map(detail => ({
    ...detail,
    id: detailIdCounter++
  }))
  
  isSalesSearchOpen.value = false
}

// 得意先検索モーダルを開く
const openCustomerSearch = () => {
  isCustomerSearchOpen.value = true
}

// 得意先検索モーダルを閉じる
const closeCustomerSearch = () => {
  isCustomerSearchOpen.value = false
}

// 得意先選択時の処理（モーダルから）
const onCustomerSelected = (customer) => {
  salesHeader.value.customerCode = customer.code
  salesHeader.value.customerName = customer.name
  salesHeader.value.customer = customer
}

// 得意先コード直接入力時の処理
const onCustomerCodeInput = () => {
  const customerCode = salesHeader.value.customerCode.trim()
  
  if (!customerCode) {
    salesHeader.value.customerName = ''
    salesHeader.value.customer = null
    return
  }
  
  // 得意先マスタから検索
  const customer = props.customers.find(
    c => c.code.toLowerCase() === customerCode.toLowerCase()
  )
  
  if (customer) {
    salesHeader.value.customerName = customer.name
    salesHeader.value.customer = customer
  } else {
    salesHeader.value.customerName = '（得意先コード不正）'
    salesHeader.value.customer = null
  }
}

// 担当者検索モーダルを開く
const openStaffSearch = () => {
  isStaffSearchOpen.value = true
}

// 担当者検索モーダルを閉じる
const closeStaffSearch = () => {
  isStaffSearchOpen.value = false
}

// 担当者選択時の処理（モーダルから）
const onStaffSelected = (staff) => {
  salesHeader.value.staffCode = staff.code
  salesHeader.value.staffName = staff.name
  salesHeader.value.staff = staff
}

// 担当者コード直接入力時の処理
const onStaffCodeInput = () => {
  const staffCode = salesHeader.value.staffCode.trim()
  
  if (!staffCode) {
    salesHeader.value.staffName = ''
    salesHeader.value.staff = null
    return
  }
  
  // 担当者マスタから検索
  const staff = props.staffList.find(
    s => s.code.toLowerCase() === staffCode.toLowerCase()
  )
  
  if (staff) {
    salesHeader.value.staffName = staff.name
    salesHeader.value.staff = staff
  } else {
    salesHeader.value.staffName = '（担当者コード不正）'
    salesHeader.value.staff = null
  }
}

// 受注選択時の処理
const onOrderSelected = (order) => {
  // ヘッダー情報を設定
  salesHeader.value.salesNo = generateSalesNo() // 新しい売上番号を生成
  salesHeader.value.orderNo = order.orderNo
  salesHeader.value.date = order.orderDate
  salesHeader.value.customerCode = order.customer.code
  salesHeader.value.customerName = order.customer.name
  salesHeader.value.customer = order.customer
  salesHeader.value.staffCode = order.staff.code
  salesHeader.value.staffName = order.staff.name
  salesHeader.value.staff = order.staff
  
  // 明細をクリアして受注明細をコピー
  salesDetails.value = order.details.map(detail => ({
    id: detailIdCounter++,
    product: detail.product,
    quantity: detail.quantity,
    unitPrice: detail.unitPrice,
    amount: detail.quantity * detail.unitPrice
  }))
  
  // メッセージ表示
  alert(`受注番号: ${order.orderNo} のデータを読み込みました。`)
}

// フォームのリセット
const resetForm = () => {
  if (confirm('入力内容をクリアしてもよろしいですか？')) {
    salesHeader.value = {
      salesNo: generateSalesNo(),
      date: new Date().toISOString().split('T')[0],
      orderNo: '',
      customerCode: '',
      customerName: '',
      customer: null,
      staffCode: '',
      staffName: '',
      staff: null
    }
    salesDetails.value = [
      {
        id: detailIdCounter++,
        productCode: '',
        productName: '',
        product: null,
        quantity: 0,
        unitPrice: 0,
        amount: 0
      }
    ]
  }
}

// フォームの送信
const submitForm = () => {
  // 簡易バリデーション
  if (!salesHeader.value.date) {
    alert('日付を入力してください。')
    return
  }
  if (!salesHeader.value.customerCode || !salesHeader.value.customer) {
    alert('得意先を選択してください。')
    return
  }
  if (!salesHeader.value.staffCode || !salesHeader.value.staff) {
    alert('担当者を選択してください。')
    return
  }
  
  // 明細のバリデーション
  for (let i = 0; i < salesDetails.value.length; i++) {
    const row = salesDetails.value[i]
    if (!row.productCode || !row.product) {
      alert(`明細${i + 1}行目: 商品を選択してください。`)
      return
    }
    if (!row.quantity || row.quantity <= 0) {
      alert(`明細${i + 1}行目: 個数を入力してください。`)
      return
    }
    if (!row.unitPrice || row.unitPrice <= 0) {
      alert(`明細${i + 1}行目: 単価を入力してください。`)
      return
    }
  }

  // デモ用：データを表示
  const salesData = {
    header: {
      salesNo: salesHeader.value.salesNo,
      date: salesHeader.value.date,
      orderNo: salesHeader.value.orderNo,
      customerCode: salesHeader.value.customerCode,
      customerName: salesHeader.value.customerName,
      staffCode: salesHeader.value.staffCode,
      staffName: salesHeader.value.staffName
    },
    details: salesDetails.value.map(row => ({
      productCode: row.productCode,
      productName: row.productName,
      quantity: row.quantity,
      unitPrice: row.unitPrice,
      amount: row.amount
    })),
    summary: {
      subtotal: subtotal.value,
      tax: tax.value,
      grandTotal: grandTotal.value
    }
  }
  
  console.log('売上データ:', salesData)
  alert('売上データが登録されました！\n（デモのため実際には保存されません）\n\nコンソールにデータを出力しました。')
}

// 受注一覧から受注データを読み込む（外部から呼び出し可能）
const loadOrderData = (order) => {
  salesHeader.value.salesNo = generateSalesNo() // 新しい売上番号を生成
  salesHeader.value.date = order.orderDate
  salesHeader.value.orderNo = order.orderNo
  salesHeader.value.customerCode = order.customer.code
  salesHeader.value.customerName = order.customer.name
  salesHeader.value.customer = order.customer
  salesHeader.value.staffCode = order.staff.code
  salesHeader.value.staffName = order.staff.name
  salesHeader.value.staff = order.staff
  
  salesDetails.value = order.details.map(detail => ({
    id: detailIdCounter++,
    productCode: detail.product.code,
    productName: detail.product.name,
    product: detail.product,
    quantity: detail.quantity,
    unitPrice: detail.unitPrice,
    amount: detail.quantity * detail.unitPrice
  }))
  
  alert(`受注番号: ${order.orderNo} から売上データを作成しました。`)
}

// 外部から呼び出し可能な関数を公開
defineExpose({
  loadOrderData
})
</script>

<style scoped>
.sales-form {
  margin-top: 20px;
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 24px;
}

h3 {
  color: #555;
  font-size: 18px;
  margin: 0;
}

.required {
  color: #f44336;
}

/* ヘッダーセクション */
.header-section {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 30px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.header-row-1 {
  display: grid;
  grid-template-columns: 1fr 1fr 1.5fr;
  gap: 20px;
}

.header-row-2 {
  display: grid;
  grid-template-columns: 1fr 1.5fr 1fr 1.5fr;
  gap: 20px;
}

.order-no-input-group {
  display: flex;
  gap: 10px;
}

.order-no-input-group input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background-color: #f5f5f5;
  color: #666;
}

.sales-no-input-group {
  display: flex;
  gap: 5px;
}

.sales-no-input-group input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background-color: #f5f5f5;
  color: #666;
}

.btn-search {
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
  gap: 5px;
  white-space: nowrap;
}

.btn-search:hover {
  background-color: #1976D2;
}

.btn-search span {
  font-size: 16px;
}

.code-input-group {
  display: flex;
  gap: 5px;
}

.btn-search-small {
  background-color: #2196F3;
  color: white;
  border: none;
  padding: 10px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
  white-space: nowrap;
  min-width: 40px;
}

.btn-search-small:hover {
  background-color: #1976D2;
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

.form-group input[type="date"] {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.form-group input[type="date"]:focus {
  outline: none;
  border-color: #4CAF50;
}

/* 明細セクション */
.detail-section {
  margin-bottom: 30px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.btn-add {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: background-color 0.3s;
}

.btn-add:hover {
  background-color: #45a049;
}

.table-container {
  overflow-x: auto;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.detail-table {
  width: 100%;
  border-collapse: collapse;
  background-color: white;
}

.detail-table thead {
  background-color: #f5f5f5;
}

.detail-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #555;
  border-bottom: 2px solid #ddd;
}

.detail-table td {
  padding: 10px 12px;
  border-bottom: 1px solid #eee;
}

.detail-table tbody tr:hover {
  background-color: #fafafa;
}

.text-center {
  text-align: center;
}

.text-right {
  text-align: right;
  font-weight: 600;
  color: #333;
}

.product-code-group {
  display: flex;
  gap: 5px;
}

.input-code {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.input-code:focus {
  outline: none;
  border-color: #4CAF50;
}

.input-readonly {
  width: 100%;
  padding: 8px;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  font-size: 14px;
  background-color: #f9f9f9;
  color: #666;
  cursor: default;
}

.btn-product-search {
  background-color: #2196F3;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
  white-space: nowrap;
  min-width: 40px;
}

.btn-product-search:hover {
  background-color: #1976D2;
}

.input-number {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.input-number:focus {
  outline: none;
  border-color: #4CAF50;
}

.btn-delete {
  background-color: #f44336;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
}

.btn-delete:hover:not(:disabled) {
  background-color: #da190b;
}

.btn-delete:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

/* 合計セクション */
.total-section {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 30px;
  max-width: 400px;
  margin-left: auto;
}

.total-row {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
}

.total-row:last-child {
  border-bottom: none;
}

.total-label {
  font-weight: 600;
  color: #555;
}

.total-value {
  font-weight: 700;
  color: #333;
  font-size: 16px;
}

.grand-total {
  margin-top: 10px;
  padding-top: 15px;
  border-top: 2px solid #4CAF50;
}

.grand-total .total-label {
  font-size: 18px;
  color: #333;
}

.grand-total .total-value {
  font-size: 20px;
  color: #4CAF50;
}

/* アクションボタン */
.action-buttons {
  display: flex;
  gap: 15px;
  justify-content: flex-end;
}

.btn-primary,
.btn-secondary {
  padding: 12px 30px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background-color: #4CAF50;
  color: white;
}

.btn-primary:hover {
  background-color: #45a049;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.btn-secondary {
  background-color: #9E9E9E;
  color: white;
}

.btn-secondary:hover {
  background-color: #757575;
}
</style>
