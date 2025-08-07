<template>
  <Header />
  <div class="container">
    <Balance :total="total" />
    <IncomeExpenses :income="+income" :expenses="+expenses" />

    <TransactionList
        :transactions="transactions"
        :editingId="editingId"
        :editingText="editingText"
        @editTransaction="editTransaction"
        @saveEdit="saveEdit"
        @transactionDeleted="handleTransactionDeleted"
    />

    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import { ref, computed, onMounted } from 'vue';
import { useToast } from 'vue-toastification';

const toast = useToast();

const transactions = ref([]);
const editingId = ref(null);
const editingText = ref('');

// Load saved transactions
onMounted(() => {
  const saved = localStorage.getItem('transactions');
  if (saved) {
    transactions.value = JSON.parse(saved);
  }
});

// Computed values
const total = computed(() =>
    transactions.value.reduce((acc, t) => acc + t.amount, 0)
);

const income = computed(() =>
    transactions.value
        .filter((t) => t.amount > 0)
        .reduce((acc, t) => acc + t.amount, 0)
        .toFixed(2)
);

const expenses = computed(() =>
    transactions.value
        .filter((t) => t.amount < 0)
        .reduce((acc, t) => acc + t.amount, 0)
        .toFixed(2)
);

// Add new transaction
function handleTransactionSubmitted({ text, amount }) {
  transactions.value.push({
    id: generateUniqueId(),
    text,
    amount
  });
  saveTransactions();
  toast.success('Transaction added.');
}

// Delete transaction
function handleTransactionDeleted(id) {
  transactions.value = transactions.value.filter((t) => t.id !== id);
  saveTransactions();
  toast.success('Transaction deleted.');
}

// Edit transaction
function editTransaction(transaction) {
  editingId.value = transaction.id;
  editingText.value = transaction.text;
}

// Save edited transaction
function saveEdit(transaction) {
  const index = transactions.value.findIndex((t) => t.id === transaction.id);
  if (index !== -1) {
    transactions.value[index].text = transaction.text;
    transactions.value[index].amount = transaction.amount; // ✅ تحديث الرقم كمان
    saveTransactions();
    toast.success('Transaction updated.');
  }
  editingId.value = null;
  editingText.value = '';
}

// Save to localStorage
function saveTransactions() {
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
}

// Generate unique ID
function generateUniqueId() {
  return Date.now();
}
</script>
