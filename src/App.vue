<template>
  <Header />

  <div class="container">
    <Balance :total="+total" />
    <IncomeExpense :income="+income" :expenses="+expenses" />
    <TransactionList :transactions="transactions" @delete-transaction="handleDeleteTransaction" />
    <AddTransaction @transaction-submitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import AddTransaction from './components/AddTransaction.vue';
import Balance from './components/Balance.vue';
import Header from './components/Header.vue';
import IncomeExpense from './components/IncomeExpense.vue';
import TransactionList from './components/TransactionList.vue';
import { ref, computed, onMounted } from 'vue';
import { useToast } from "vue-toastification";

const toast = useToast();

const transactions = ref([]);

// console.log(transactions.value);

// Get Total Amount 
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get Total Income 
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Get Total Expenses 
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Add Trnsaction 
function handleTransactionSubmitted(transactionData) {
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount
  });
  // console.log(transactionData);
  // console.log(generateUniqueId());

  toast.success('Transaction added');

  saveTransactionsToLocalStorage();
}

// Generate unique Id for transactions
function generateUniqueId(params) {
  return Math.floor(Math.random() * 1000000) + '-' + Date.now();
}

// Delete Transation 
function handleDeleteTransaction(id) {
  // console.log(id);
  transactions.value = transactions.value.filter((transaction) => transaction.id !== id);
  toast.success('Transaction deleted');

  saveTransactionsToLocalStorage();
}

onMounted(() => {
  const savedTransactions = JSON.parse(sessionStorage.getItem('transactions'));

  if(savedTransactions) {
    transactions.value = savedTransactions;
  }
});

// Save transactions to local storage 
function saveTransactionsToLocalStorage(params) {
  sessionStorage.setItem('transactions', JSON.stringify(transactions.value));
}

</script>