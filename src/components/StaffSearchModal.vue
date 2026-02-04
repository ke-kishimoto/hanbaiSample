<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>担当者検索</h3>
          <button @click="closeModal" class="close-button">&times;</button>
        </div>
        
        <div class="modal-body">
          <!-- 検索フォーム -->
          <div class="search-form">
            <div class="search-group">
              <label>担当者コード</label>
              <input
                type="text"
                v-model="searchConditions.code"
                placeholder="担当者コードで検索"
                @input="filterStaff"
              />
            </div>
            
            <div class="search-group">
              <label>担当者名</label>
              <input
                type="text"
                v-model="searchConditions.name"
                placeholder="担当者名で検索"
                @input="filterStaff"
              />
            </div>
            
            <div class="search-actions">
              <button @click="resetSearch" class="btn-reset">クリア</button>
            </div>
          </div>
          
          <!-- 検索結果 -->
          <div class="search-results">
            <div class="results-header">
              <span>検索結果: {{ filteredStaff.length }}件</span>
            </div>
            
            <div class="results-table-container">
              <table class="results-table">
                <thead>
                  <tr>
                    <th style="width: 120px">担当者コード</th>
                    <th>担当者名</th>
                    <th style="width: 100px">操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="staff in filteredStaff" :key="staff.id">
                    <td>{{ staff.code }}</td>
                    <td>{{ staff.name }}</td>
                    <td class="text-center">
                      <button @click="selectStaff(staff)" class="btn-select">選択</button>
                    </td>
                  </tr>
                  <tr v-if="filteredStaff.length === 0">
                    <td colspan="3" class="no-data">該当する担当者がありません</td>
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
  staffList: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['close', 'select'])

// 検索条件
const searchConditions = ref({
  code: '',
  name: ''
})

// フィルタリングされた担当者データ
const filteredStaff = ref([...props.staffList])

// 担当者データのフィルタリング
const filterStaff = () => {
  filteredStaff.value = props.staffList.filter(staff => {
    // 担当者コードでの絞り込み
    if (searchConditions.value.code) {
      if (!staff.code.toLowerCase().includes(searchConditions.value.code.toLowerCase())) {
        return false
      }
    }
    
    // 担当者名での絞り込み
    if (searchConditions.value.name) {
      const staffName = staff.name.toLowerCase()
      const searchName = searchConditions.value.name.toLowerCase()
      if (!staffName.includes(searchName)) {
        return false
      }
    }
    
    return true
  })
}

// 検索条件のリセット
const resetSearch = () => {
  searchConditions.value = {
    code: '',
    name: ''
  }
  filteredStaff.value = [...props.staffList]
}

// 担当者の選択
const selectStaff = (staff) => {
  emit('select', staff)
  closeModal()
}

// モーダルを閉じる
const closeModal = () => {
  emit('close')
}

// モーダルが開かれたときに検索結果をリセット
const resetWhenOpen = () => {
  resetSearch()
}

// 親コンポーネントから呼び出せるようにする
defineExpose({
  resetWhenOpen
})
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
  max-width: 700px;
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
  grid-template-columns: 1fr 1fr auto;
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
  max-height: 400px;
  overflow-y: auto;
}

.results-table {
  width: 100%;
  border-collapse: collapse;
}

.results-table thead {
  background-color: #f5f5f5;
  position: sticky;
  top: 0;
  z-index: 1;
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
