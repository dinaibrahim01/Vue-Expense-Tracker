<template>
  <h3>History</h3>
  <ul id="list" class="list">
    <li v-for="transaction in transactions" :key="transaction.id">
      <div :class="transaction.amount < 0 ? 'minus' : 'plus'">
        <template v-if="transaction.id === editingId">
          <input
              v-model="localEditingText"
              placeholder="Edit text"
          />
          <input
              type="number"
              v-model.number="localEditingAmount"
              placeholder="Edit amount"
          />
          <button @click="handleSave(transaction)">✔</button>
        </template>
        <template v-else>
          {{ transaction.text }}
          <button @click="handleEdit(transaction)">✏</button>
        </template>

        <span>${{ transaction.amount }}</span>
        <button class="delete-btn" @click="deleteTransaction(transaction.id)">x</button>
      </div>
    </li>
  </ul>
</template>

<script setup>
import { ref, watch } from 'vue';
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  transactions: Array,
  editingId: Number,
  editingText: String
});

const emit = defineEmits(['editTransaction', 'saveEdit', 'transactionDeleted']);

const localEditingText = ref('');
const localEditingAmount = ref(0);

// Sync with props
watch(() => props.editingText, (val) => {
  localEditingText.value = val;
});

watch(() => props.editingId, (val) => {
  const transaction = props.transactions.find(t => t.id === val);
  if (transaction) {
    localEditingAmount.value = transaction.amount;
  }
});

function handleEdit(transaction) {
  emit('editTransaction', transaction);
}

function handleSave(transaction) {
  emit('saveEdit', {
    ...transaction,
    text: localEditingText.value,
    amount: localEditingAmount.value
  });
}

function deleteTransaction(id) {
  emit('transactionDeleted', id);
}
</script>
