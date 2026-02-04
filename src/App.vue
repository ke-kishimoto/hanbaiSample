<template>
  <div class="app">
    <h1>販売管理システム</h1>
    
    <!-- タブナビゲーション -->
    <div class="tab-navigation">
      <button
        :class="['tab-button', { active: activeTab === 'quotation' }]"
        @click="activeTab = 'quotation'"
      >
        📝 見積入力
      </button>
      <button
        :class="['tab-button', { active: activeTab === 'order' }]"
        @click="activeTab = 'order'"
      >
        📝 受注入力
      </button>
      <button
        :class="['tab-button', { active: activeTab === 'sales' }]"
        @click="activeTab = 'sales'"
      >
        📝 売上入力
      </button>
      <button
        :class="['tab-button', { active: activeTab === 'orders' }]"
        @click="activeTab = 'orders'"
      >
        📋 受注一覧
      </button>
      <button
        :class="['tab-button', { active: activeTab === 'invoice' }]"
        @click="activeTab = 'invoice'"
      >
        📄 請求書発行
      </button>
      <button
        :class="['tab-button', { active: activeTab === 'products' }]"
        @click="activeTab = 'products'"
      >
        📦 商品管理
      </button>
    </div>
    
    <!-- タブコンテンツ -->
    <div class="tab-content">
      <QuotationInputForm 
        v-if="activeTab === 'quotation'" 
        ref="quotationFormRef"
        :customers="customers"
        :staffList="staffList"
        :products="products"
      />
      <OrderInputForm 
        v-if="activeTab === 'order'" 
        ref="orderFormRef"
        :customers="customers"
        :staffList="staffList"
        :products="products"
      />
      <SalesInputForm 
        v-if="activeTab === 'sales'" 
        ref="salesFormRef"
        :customers="customers"
        :staffList="staffList"
        :products="products"
      />
      <OrderList 
        v-if="activeTab === 'orders'"
        ref="orderListRef"
        :customers="customers"
        :staffList="staffList"
        :products="products"
        @createSales="handleCreateSales"
      />
      <InvoiceGenerator
        v-if="activeTab === 'invoice'"
        :customers="customers"
        :staffList="staffList"
        :products="products"
      />
      <ProductList
        v-if="activeTab === 'products'"
        :products="products"
        @update:products="updateProducts"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import QuotationInputForm from './components/QuotationInputForm.vue'
import OrderInputForm from './components/OrderInputForm.vue'
import SalesInputForm from './components/SalesInputForm.vue'
import OrderList from './components/OrderList.vue'
import InvoiceGenerator from './components/InvoiceGenerator.vue'
import ProductList from './components/ProductList.vue'

// アクティブタブ
const activeTab = ref('quotation')

// コンポーネントの参照
const salesFormRef = ref(null)
const orderListRef = ref(null)

// マスタデータ
const customers = [
  { id: 1, name: '株式会社山田商事', code: 'C001' },
  { id: 2, name: '佐藤工業株式会社', code: 'C002' },
  { id: 3, name: '田中物産株式会社', code: 'C003' },
  { id: 4, name: '鈴木建設株式会社', code: 'C004' },
  { id: 5, name: '高橋電機株式会社', code: 'C005' },
  { id: 6, name: '伊藤製作所', code: 'C006' },
  { id: 7, name: '渡辺商店', code: 'C007' },
  { id: 8, name: '小林運輸株式会社', code: 'C008' }
]

const staffList = [
  { id: 1, name: '山田太郎', code: 'S001' },
  { id: 2, name: '佐藤花子', code: 'S002' },
  { id: 3, name: '田中一郎', code: 'S003' },
  { id: 4, name: '鈴木二郎', code: 'S004' },
  { id: 5, name: '高橋三郎', code: 'S005' }
]

const products = ref([
  { id: 1, code: 'P001', name: 'ノートパソコン Core i5', kana: 'ノートパソコンコアアイファイブ', category: 'パソコン', price: 98000, cost: 78000, unit: '台', stock: 15, taxType: '課税', status: '有効', remarks: 'ビジネス向けスタンダードモデル' },
  { id: 2, code: 'P002', name: 'ノートパソコン Core i7', kana: 'ノートパソコンコアアイセブン', category: 'パソコン', price: 128000, cost: 105000, unit: '台', stock: 10, taxType: '課税', status: '有効', remarks: 'ハイパフォーマンスモデル' },
  { id: 3, code: 'P003', name: 'デスクトップPC', kana: 'デスクトップピーシー', category: 'パソコン', price: 85000, cost: 68000, unit: '台', stock: 8, taxType: '課税', status: '有効', remarks: '' },
  { id: 4, code: 'P004', name: '液晶ディスプレイ 24インチ', kana: 'エキショウディスプレイニジュウヨンインチ', category: '周辺機器', price: 25000, cost: 18000, unit: '台', stock: 20, taxType: '課税', status: '有効', remarks: '' },
  { id: 5, code: 'P005', name: '液晶ディスプレイ 27インチ', kana: 'エキショウディスプレイニジュウナナインチ', category: '周辺機器', price: 35000, cost: 26000, unit: '台', stock: 12, taxType: '課税', status: '有効', remarks: '' },
  { id: 6, code: 'P006', name: 'キーボード', kana: 'キーボード', category: '周辺機器', price: 3500, cost: 2000, unit: '個', stock: 50, taxType: '課税', status: '有効', remarks: '' },
  { id: 7, code: 'P007', name: 'マウス', kana: 'マウス', category: '周辺機器', price: 2000, cost: 1200, unit: '個', stock: 60, taxType: '課税', status: '有効', remarks: '' },
  { id: 8, code: 'P008', name: 'プリンター', kana: 'プリンター', category: '周辺機器', price: 45000, cost: 35000, unit: '台', stock: 7, taxType: '課税', status: '有効', remarks: '複合機タイプ' },
  { id: 9, code: 'P009', name: 'スキャナー', kana: 'スキャナー', category: '周辺機器', price: 28000, cost: 21000, unit: '台', stock: 5, taxType: '課税', status: '有効', remarks: '' },
  { id: 10, code: 'P010', name: 'Webカメラ', kana: 'ウェブカメラ', category: '周辺機器', price: 8000, cost: 5000, unit: '個', stock: 25, taxType: '課税', status: '有効', remarks: 'リモート会議用' }
])

// 商品データ更新
const updateProducts = (newProducts) => {
  products.value = newProducts
}

// 受注一覧から売上作成
const handleCreateSales = (order) => {
  // 売上入力タブに切り替え
  activeTab.value = 'sales'
  
  // 少し待ってから売上フォームに受注データを設定
  setTimeout(() => {
    if (salesFormRef.value && salesFormRef.value.loadOrderData) {
      salesFormRef.value.loadOrderData(order)
      
      // 受注一覧の連携状態を更新
      if (orderListRef.value && orderListRef.value.updateOrderStatus) {
        orderListRef.value.updateOrderStatus(order.orderNo)
      }
    }
  }, 100)
}
</script>

<style scoped>
.app {
  background-color: white;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h1 {
  color: #333;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 3px solid #4CAF50;
}

/* タブナビゲーション */
.tab-navigation {
  display: flex;
  gap: 0;
  margin-bottom: 30px;
  border-bottom: 2px solid #e0e0e0;
}

.tab-button {
  background-color: transparent;
  border: none;
  padding: 12px 24px;
  font-size: 16px;
  font-weight: 600;
  color: #666;
  cursor: pointer;
  transition: all 0.3s;
  border-bottom: 3px solid transparent;
  margin-bottom: -2px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.tab-button:hover {
  color: #4CAF50;
  background-color: #f5f5f5;
}

.tab-button.active {
  color: #4CAF50;
  border-bottom-color: #4CAF50;
  background-color: #f9f9f9;
}

/* タブコンテンツ */
.tab-content {
  min-height: 400px;
}
</style>
