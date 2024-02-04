<script>
	import Transaction from "./Transaction.svelte";
	import { onMount } from "svelte";
	import Chart from "chart.js/auto";

	let chartCanvas;
	let transactions = getSavedTransactions();
	let newTransaction = { description: "", amount: 0 };
	let transactionFilter = "";
	let totalBalance;
	let expensesByCategory = {};
  
	onMount(() => {
	  const ctx = chartCanvas.getContext("2d");
	  const expensesByMonth = calculateExpensesByMonth();
	  new Chart(ctx, {
		type: "bar",
		data: {
		  labels: Object.keys(expensesByMonth),
		  datasets: [
			{
			  label: "Monthly Expenses",
			  data: Object.values(expensesByMonth),
			  backgroundColor: "rgba(75, 192, 192, 0.2)",
			  borderColor: "rgba(75, 192, 192, 1)",
			  borderWidth: 1,
			},
		  ],
		},
		options: {
		  scales: {
			y: {
			  beginAtZero: true,
			},
		  },
		},
	  });
	});

	$: filteredTransactions = transactions.filter((transaction) =>
	  transaction.description
		.toLowerCase()
		.includes(transactionFilter.toLowerCase())
	);
  
	$: {
	  totalBalance = transactions.reduce(
		(total, transaction) => total + transaction.amount,
		0
	  );
  
	  expensesByCategory = filteredTransactions.reduce((acc, transaction) => {
		const category = transaction.category || "Uncategorized";
		acc[category] = (acc[category] || 0) + transaction.amount;
		return acc;
	  }, {});
	}

	function addTransaction() {
	  transactions = [...transactions, { ...newTransaction, id: Date.now(), date: new Date() }];
	  newTransaction = { description: "", amount: 0 };
	  saveTransactions();
	}
  
	function saveTransactions() {
	  localStorage.setItem("transactions", JSON.stringify(transactions));
	  calculateExpensesByMonth();
	}
  
	function getSavedTransactions() {
	  const savedTransactions = localStorage.getItem("transactions");
	  return savedTransactions ? JSON.parse(savedTransactions) : [];
	}
  
	function removeTransaction(removedTransaction) {
	  transactions = transactions.filter((t) => t.id !== removedTransaction.id);
	  saveTransactions();
	}
  
	function calculateExpensesByMonth() {
		return transactions.reduce((acc, transaction) => {
		  const month = new Date(transaction.date).toLocaleString("default", { month: "long" });
		  acc[month] = (acc[month] || 0) + transaction.amount;
		  return acc;
		}, {});
	}
  </script>

  <body>
	<div>
		<h1>Personal Finance Dashboard</h1>
	
		<div>
		  <h2>Overview</h2>
		  <p>Total Expenses: ${totalBalance.toFixed(2)}</p>
		  <h3>Expenses by Category</h3>
		  <ul>
			{#each Object.entries(expensesByCategory) as [category, amount]}
			  <li>{category}: ${amount.toFixed(2)}</li>
			{/each}
		  </ul>
		  <canvas bind:this={chartCanvas}></canvas>
		</div>
	
		<div>
		  <h2>Transactions</h2>
		  <label for="filter">Filter Transactions:</label>
		  <input type="text" id="filter" bind:value={transactionFilter} />
		  <form on:submit|preventDefault={addTransaction}>
			<label for="description">Description:</label>
			<input type="text" id="description" bind:value={newTransaction.description} />
			<label for="amount">Amount:</label>
			<input type="number" id="amount" step="1" bind:value={newTransaction.amount} />
			<label for="category">Category:</label>
			<input type="text" id="category" bind:value={newTransaction.category} />
			<button type="submit">Add Transaction</button>
		  </form>
		  {#each filteredTransactions as transaction (transaction.id)}
			<Transaction {transaction} {removeTransaction} />
		  {/each}
		</div>
	</div>
  </body>

<style>
	body {
	  font-family: "Roboto", sans-serif;
	  margin: 0;
	  padding: 0;
	}
  
	div {
	  max-width: 800px;
	  margin: 20px auto;
	  padding: 30px;
	  border: 1px solid #e0e0e0;
	  border-radius: 12px;
	  background-color: #fff;
	  box-shadow: 0 0 10px rgba(0, 0, 0, 0.05);
	  transition: box-shadow 0.3s ease;
	}
  
	div:hover {
	  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
	}
  
	h1, h2 {
	  color: #424242;
	}
  
	p, li {
	  color: #616161;
	}
  
	button {
	  background-color: #4caf50;
	  color: white;
	  border: none;
	  padding: 10px 20px;
	  text-align: center;
	  text-decoration: none;
	  display: inline-block;
	  font-size: 16px;
	  margin: 4px 2px;
	  transition-duration: 0.4s;
	  cursor: pointer;
	  border-radius: 12px;
	}
  
	button:hover {
	  background-color: #45a049;
	}
  
	@media (max-width: 600px) {
	  div {
		max-width: 90%;
		padding: 20px;
	  }
	}
  </style>