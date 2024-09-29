# PERSONAL-FINANCE-TRACKER
FRONTEND(React.js)
1.TransactionForm Component
import { useState } from 'react';

const TransactionForm = ({ onSubmit, transaction }) => {
  const [type, setType] = useState(transaction?.type || 'Income');
  const [amount, setAmount] = useState(transaction?.amount || '');
  const [description, setDescription] = useState(transaction?.description || '');
  const [date, setDate] = useState(transaction?.date || new Date().toISOString().split('T')[0]);

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit({ type, amount: parseFloat(amount), description, date });
  };

  return (
    <form onSubmit={handleSubmit}>
      <select value={type} onChange={(e) => setType(e.target.value)}>
        <option value="Income">Income</option>
        <option value="Expense">Expense</option>
      </select>
      <input type="number" value={amount} onChange={(e) => setAmount(e.target.value)} required />
      <input type="text" value={description} onChange={(e) => setDescription(e.target.value)} required />
      <input type="date" value={date} onChange={(e) => setDate(e.target.value)} required />
      <button type="submit">Save Transaction</button>
    </form>
  );
};

export default TransactionForm;
2.TransactionList Component
const TransactionList = ({ transactions, onDelete, onEdit }) => (
  <ul>
    {transactions.map((transaction) => (
      <li key={transaction._id}>
        {transaction.type}: {transaction.amount} - {transaction.description} on {transaction.date}
        <button onClick={() => onEdit(transaction)}>Edit</button>
        <button onClick={() => onDelete(transaction._id)}>Delete</button>
      </li>
    ))}
  </ul>
);

export default TransactionList;
3.TransactionSummary Component
const TransactionSummary = ({ transactions }) => {
  const income = transactions
    .filter((t) => t.type === 'Income')
    .reduce((acc, t) => acc + t.amount, 0);

  const expenses = transactions
    .filter((t) => t.type === 'Expense')
    .reduce((acc, t) => acc + t.amount, 0);

  return (
    <div>
      <p>Total Income: {income}</p>
      <p>Total Expenses: {expenses}</p>
      <p>Balance: {income - expenses}</p>
    </div>
  );
};

export default TransactionSummary;
4.Filter Component
const Filter = ({ onFilterChange }) => {
  return (
    <div>
      <select onChange={(e) => onFilterChange({ type: e.target.value })}>
        <option value="">All</option>
        <option value="Income">Income</option>
        <option value="Expense">Expense</option>
      </select>
      <input type="date" onChange={(e) => onFilterChange({ date: e.target.value })} />
    </div>
  );
};

export default Filter;
BACKEND(Node.js/Express.js)
1.Transaction Schema(MangoDB Model)
const mongoose = require('mongoose');

const transactionSchema = new mongoose.Schema({
  type: {
    type: String,
    enum: ['Income', 'Expense'],
    required: true
  },
  amount: {
    type: Number,
    required: true
  },
  description: String,
  date: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Transaction', transactionSchema);
2.API Routes(Express.js)
const express = require('express');
const router = express.Router();
const Transaction = require('../models/Transaction');

// Get all transactions
router.get('/', async (req, res) => {
  const transactions = await Transaction.find();
  res.json(transactions);
});

// Add a transaction
router.post('/', async (req, res) => {
  const transaction = new Transaction(req.body);
  await transaction.save();
  res.status(201).json(transaction);
});

// Edit a transaction
router.put('/:id', async (req, res) => {
  const transaction = await Transaction.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(transaction);
});

// Delete a transaction
router.delete('/:id', async (req, res) => {
  await Transaction.findByIdAndDelete(req.params.id);
  res.status(204).end();
});

module.exports = router;
3.Express.js Setup
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const transactions = require('./routes/transactions');

const app = express();

app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost/personal-finance', { useNewUrlParser: true, useUnifiedTopology: true });

app.use('/api/transactions', transactions);

app.listen(5000, () => {
  console.log('Server running on port 5000');
});
