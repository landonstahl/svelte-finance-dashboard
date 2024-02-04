<div>
    <p>{transaction.description}</p>
    <p>{transaction.amount}</p>
    <p>Category: {transaction.category}</p>
    <p>Date: {new Date(transaction.date).toLocaleDateString()}</p>
    <button on:click={remove}>Remove</button>
  </div>
  
  <script>
    export let transaction;
    export let removeTransaction;
  
    function remove() {
      removeTransaction(transaction);
    }
  </script>
  