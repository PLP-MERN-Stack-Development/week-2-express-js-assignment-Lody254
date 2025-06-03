  server.js
  require('dotenv').config();
const express = require('express');
const app = express();

// Middleware
const logger = require('./middleware/logger');
const auth = require('./middleware/auth');
const errorHandler = require('./middleware/errorHandler');

// Routes
const productRoutes = require('./routes/products');

// Middleware
app.use(express.json());
app.use(logger);
app.use(auth);

// API Routes
app.use('/api/products', productRoutes);

// Error Handling Middleware (after routes)
app.use(errorHandler);

// Start Server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

logger.js
module.exports = (req, res, next) => {
  console.log(`${req.method} ${req.originalUrl}`);
  next();
};
/auth.js
module.exports = (req, res, next) => {
  const token = req.headers['authorization'];
  if (!token || token !== 'Bearer mysecrettoken') {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  next();
};

errorHandler.js
module.exports = (err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message || 'Something went wrong' });
};

routes/products.js
const express = require('express');
const router = express.Router();
let products = require('../data/products'); // Simulate DB with an array

// GET all products + optional filtering and search
router.get('/', (req, res) => {
  let results = products;

  // Filtering
  if (req.query.category) {
    results = results.filter(p => p.category === req.query.category);
  }

  // Search
  if (req.query.search) {
    const keyword = req.query.search.toLowerCase();
    results = results.filter(p => p.name.toLowerCase().includes(keyword));
  }

  // Pagination
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const start = (page - 1) * limit;
  const end = start + limit;

  res.json({
    page,
    total: results.length,
    data: results.slice(start, end)
  });
});

// GET product by ID
router.get('/:id', (req, res) => {
  const product = products.find(p => p.id === req.params.id);
  if (!product) return res.status(404).json({ error: 'Product not found' });
  res.json(product);
});

// POST create product
router.post('/', (req, res) => {
  const { name, price, category } = req.body;
  if (!name || !price) return res.status(400).json({ error: 'Name and price required' });

  const newProduct = {
    id: Date.now().toString(),
    name,
    price,
    category: category || 'General'
  };

  products.push(newProduct);
  res.status(201).json(newProduct);
});

// PUT update product
router.put('/:id', (req, res) => {
  const product = products.find(p => p.id === req.params.id);
  if (!product) return res.status(404).json({ error: 'Product not found' });

  const { name, price, category } = req.body;
  if (name) product.name = name;
  if (price) product.price = price;
  if (category) product.category = category;

  res.json(product);
});

// DELETE product
router.delete('/:id', (req, res) => {
  products = products.filter(p => p.id !== req.params.id);
  res.status(204).send();
});

module.exports = router;
