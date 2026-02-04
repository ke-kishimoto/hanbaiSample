<template>
  <div class="app">
    <h1>販売管理システム</h1>
    
    <!-- タブナビゲーション -->
    <div class="tab-navigation">
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
    </div>
    
    <!-- タブコンテンツ -->
    <div class="tab-content">
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
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SalesInputForm from './components/SalesInputForm.vue'
import OrderList from './components/OrderList.vue'
import InvoiceGenerator from './components/InvoiceGenerator.vue'

// アクティブタブ
const activeTab = ref('sales')

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

const products = [
  { id: 1, name: 'ノートパソコン Core i5', code: 'P001', price: 98000 },
  { id: 2, name: 'ノートパソコン Core i7', code: 'P002', price: 128000 },
  { id: 3, name: 'デスクトップPC', code: 'P003', price: 85000 },
  { id: 4, name: '液晶ディスプレイ 24インチ', code: 'P004', price: 25000 },
  { id: 5, name: '液晶ディスプレイ 27インチ', code: 'P005', price: 35000 },
  { id: 6, name: 'キーボード', code: 'P006', price: 3500 },
  { id: 7, name: 'マウス', code: 'P007', price: 2000 },
  { id: 8, name: 'プリンター', code: 'P008', price: 45000 },
  { id: 9, name: 'スキャナー', code: 'P009', price: 28000 },
  { id: 10, name: 'Webカメラ', code: 'P010', price: 8000 }
]

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
