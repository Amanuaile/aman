# ============================================
# AmaEyoba Cafe - Automated Setup Script
# ============================================

$projectPath = "C:\Users\Amanuel\Desktop\AmaEyoba-Cafe"
Write-Host "Creating AmaEyoba Cafe project at: $projectPath" -ForegroundColor Green

# Create project directories
New-Item -ItemType Directory -Path "$projectPath\server\models" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\server\routes" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\server\controllers" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\server\config" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\public\css" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\public\js" -Force | Out-Null

Write-Host "✓ Project directories created" -ForegroundColor Green

# Create package.json
@"
{
  "name": "amaeyoba-cafe",
  "version": "1.0.0",
  "description": "Full-stack cafe menu website with shopping cart and orders",
  "main": "server/server.js",
  "scripts": {
    "start": "node server/server.js",
    "dev": "nodemon server/server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "dotenv": "^16.0.3",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
}
"@ | Set-Content -Path "$projectPath\package.json"

Write-Host "✓ package.json created" -ForegroundColor Green

# Create .env file
@"
PORT=5000
MONGODB_URI=mongodb://localhost:27017/amaeyoba-cafe
NODE_ENV=development
"@ | Set-Content -Path "$projectPath\.env"

Write-Host "✓ .env file created" -ForegroundColor Green

# Create server/config/db.js
@"
const mongoose = require('mongoose');
require('dotenv').config();

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('✓ MongoDB connected successfully');
  } catch (err) {
    console.error('✗ MongoDB connection error:', err);
    process.exit(1);
  }
};

module.exports = connectDB;
"@ | Set-Content -Path "$projectPath\server\config\db.js"

Write-Host "✓ server/config/db.js created" -ForegroundColor Green

# Create server/models/Menu.js
@"
const mongoose = require('mongoose');

const menuSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: { type: String, required: true },
  price: { type: Number, required: true },
  category: { type: String, required: true },
  image: { type: String, default: 'https://via.placeholder.com/300x200?text=Menu+Item' },
  available: { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Menu', menuSchema);
"@ | Set-Content -Path "$projectPath\server\models\Menu.js"

Write-Host "✓ server/models/Menu.js created" -ForegroundColor Green

# Create server/models/Order.js
@"
const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema({
  customer: {
    name: { type: String, required: true },
    email: { type: String, required: true },
    phone: { type: String, required: true }
  },
  items: [
    {
      menuId: mongoose.Schema.Types.ObjectId,
      name: String,
      price: Number,
      quantity: Number
    }
  ],
  totalPrice: { type: Number, required: true },
  status: { type: String, default: 'Pending', enum: ['Pending', 'Confirmed', 'Completed', 'Cancelled'] },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Order', orderSchema);
"@ | Set-Content -Path "$projectPath\server\models\Order.js"

Write-Host "✓ server/models/Order.js created" -ForegroundColor Green

# Create server/controllers/menuController.js
@"
const Menu = require('../models/Menu');

// Get all menu items
exports.getAllMenus = async (req, res) => {
  try {
    const menus = await Menu.find();
    res.json(menus);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

// Get menu by ID
exports.getMenuById = async (req, res) => {
  try {
    const menu = await Menu.findById(req.params.id);
    if (!menu) return res.status(404).json({ message: 'Menu not found' });
    res.json(menu);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

// Create menu item
exports.createMenu = async (req, res) => {
  const menu = new Menu({
    name: req.body.name,
    description: req.body.description,
    price: req.body.price,
    category: req.body.category,
    image: req.body.image
  });

  try {
    const newMenu = await menu.save();
    res.status(201).json(newMenu);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

// Update menu item
exports.updateMenu = async (req, res) => {
  try {
    const menu = await Menu.findById(req.params.id);
    if (!menu) return res.status(404).json({ message: 'Menu not found' });
    
    if (req.body.name) menu.name = req.body.name;
    if (req.body.description) menu.description = req.body.description;
    if (req.body.price) menu.price = req.body.price;
    if (req.body.category) menu.category = req.body.category;
    if (req.body.image) menu.image = req.body.image;
    if (req.body.available !== undefined) menu.available = req.body.available;

    const updatedMenu = await menu.save();
    res.json(updatedMenu);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

// Delete menu item
exports.deleteMenu = async (req, res) => {
  try {
    const menu = await Menu.findByIdAndDelete(req.params.id);
    if (!menu) return res.status(404).json({ message: 'Menu not found' });
    res.json({ message: 'Menu deleted' });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};
"@ | Set-Content -Path "$projectPath\server\controllers\menuController.js"

Write-Host "✓ server/controllers/menuController.js created" -ForegroundColor Green

# Create server/controllers/orderController.js
@"
const Order = require('../models/Order');

// Create order
exports.createOrder = async (req, res) => {
  const order = new Order({
    customer: req.body.customer,
    items: req.body.items,
    totalPrice: req.body.totalPrice
  });

  try {
    const newOrder = await order.save();
    res.status(201).json(newOrder);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

// Get all orders
exports.getAllOrders = async (req, res) => {
  try {
    const orders = await Order.find().sort({ createdAt: -1 });
    res.json(orders);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

// Get order by ID
exports.getOrderById = async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ message: 'Order not found' });
    res.json(order);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

// Update order status
exports.updateOrderStatus = async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ message: 'Order not found' });
    
    order.status = req.body.status;
    const updatedOrder = await order.save();
    res.json(updatedOrder);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};
"@ | Set-Content -Path "$projectPath\server\controllers\orderController.js"

Write-Host "✓ server/controllers/orderController.js created" -ForegroundColor Green

# Create server/routes/menuRoutes.js
@"
const express = require('express');
const router = express.Router();
const menuController = require('../controllers/menuController');

router.get('/', menuController.getAllMenus);
router.get('/:id', menuController.getMenuById);
router.post('/', menuController.createMenu);
router.put('/:id', menuController.updateMenu);
router.delete('/:id', menuController.deleteMenu);

module.exports = router;
"@ | Set-Content -Path "$projectPath\server\routes\menuRoutes.js"

Write-Host "✓ server/routes/menuRoutes.js created" -ForegroundColor Green

# Create server/routes/orderRoutes.js
@"
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');

router.post('/', orderController.createOrder);
router.get('/', orderController.getAllOrders);
router.get('/:id', orderController.getOrderById);
router.put('/:id', orderController.updateOrderStatus);

module.exports = router;
"@ | Set-Content -Path "$projectPath\server\routes\orderRoutes.js"

Write-Host "✓ server/routes/orderRoutes.js created" -ForegroundColor Green

# Create server/server.js
@"
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();
const connectDB = require('./config/db');

const menuRoutes = require('./routes/menuRoutes');
const orderRoutes = require('./routes/orderRoutes');

const app = express();

// Middleware
app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public'));

// Connect to MongoDB
connectDB();

// Routes
app.use('/api/menus', menuRoutes);
app.use('/api/orders', orderRoutes);

// Health check
app.get('/api/health', (req, res) => {
  res.json({ status: 'Server is running!' });
});

// Serve index.html for root
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/../public/index.html');
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`✓ AmaEyoba Cafe server running on http://localhost:\${PORT}`);
});
"@ | Set-Content -Path "$projectPath\server\server.js"

Write-Host "✓ server/server.js created" -ForegroundColor Green

# Create public/index.html
@"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AmaEyoba Cafe - Menu</title>
  <link rel="stylesheet" href="css/style.css">
  <link rel="stylesheet" href="css/menu.css">
</head>
<body>
  <div class="container">
    <header class="header">
      <div class="header-content">
        <h1>☕ AmaEyoba Cafe</h1>
        <p>Fresh Coffee, Fresh Vibes</p>
      </div>
      <div class="header-actions">
        <button class="cart-btn" id="cartBtn">
          🛒 Cart <span class="cart-count" id="cartCount">0</span>
        </button>
      </div>
    </header>

    <nav class="filters">
      <button class="filter-btn active" data-category="all">All Items</button>
      <button class="filter-btn" data-category="Coffee">Coffee</button>
      <button class="filter-btn" data-category="Tea">Tea</button>
      <button class="filter-btn" data-category="Pastry">Pastry</button>
      <button class="filter-btn" data-category="Sandwich">Sandwich</button>
    </nav>

    <main class="menu-container" id="menuContainer">
      <div class="loading">Loading menu items...</div>
    </main>
  </div>

  <!-- Cart Modal -->
  <div class="modal" id="cartModal">
    <div class="modal-content">
      <span class="close" id="closeCart">&times;</span>
      <h2>Shopping Cart</h2>
      <div class="cart-items" id="cartItems"></div>
      <div class="cart-summary">
        <p>Total: <strong id="cartTotal">Br 0.00</strong></p>
        <button class="checkout-btn" id="checkoutBtn">Proceed to Checkout</button>
      </div>
    </div>
  </div>

  <!-- Checkout Modal -->
  <div class="modal" id="checkoutModal">
    <div class="modal-content checkout-form">
      <span class="close" id="closeCheckout">&times;</span>
      <h2>Checkout</h2>
      <form id="orderForm">
        <input type="text" placeholder="Full Name" id="customerName" required>
        <input type="email" placeholder="Email" id="customerEmail" required>
        <input type="tel" placeholder="Phone Number" id="customerPhone" required>
        <p>Total: <strong id="checkoutTotal">Br 0.00</strong></p>
        <button type="submit" class="submit-btn">Place Order</button>
      </form>
    </div>
  </div>

  <!-- Order Confirmation Modal -->
  <div class="modal" id="confirmationModal">
    <div class="modal-content confirmation">
      <span class="close" id="closeConfirmation">&times;</span>
      <h2>✓ Order Confirmed!</h2>
      <div id="confirmationContent"></div>
      <button class="btn-primary" id="continueShopping">Continue Shopping</button>
    </div>
  </div>

  <script src="js/api.js"></script>
  <script src="js/menu.js"></script>
  <script src="js/cart.js"></script>
  <script src="js/app.js"></script>
</body>
</html>
"@ | Set-Content -Path "$projectPath\public\index.html"

Write-Host "✓ public/index.html created" -ForegroundColor Green

# Create public/css/style.css
@"
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #f5e6d3 0%, #e8d5c4 100%);
  color: #333;
  line-height: 1.6;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  background: linear-gradient(135deg, #8B4513 0%, #A0522D 100%);
  color: white;
  padding: 30px;
  border-radius: 10px;
  margin-bottom: 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.header h1 {
  font-size: 2.5em;
  margin-bottom: 5px;
}

.header p {
  opacity: 0.9;
  font-style: italic;
}

.header-actions {
  display: flex;
  gap: 10px;
}

.cart-btn {
  background: #FFD700;
  color: #8B4513;
  border: none;
  padding: 12px 20px;
  border-radius: 25px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
}

.cart-btn:hover {
  background: #FFC700;
  transform: scale(1.05);
}

.cart-count {
  background: #FF6B6B;
  color: white;
  border-radius: 50%;
  padding: 2px 6px;
  font-size: 12px;
  margin-left: 5px;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
  flex-wrap: wrap;
  justify-content: center;
}

.filter-btn {
  padding: 10px 20px;
  border: 2px solid #8B4513;
  background: white;
  color: #8B4513;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 500;
}

.filter-btn:hover,
.filter-btn.active {
  background: #8B4513;
  color: white;
}

.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal-content {
  background-color: #fefcfb;
  margin: 50px auto;
  padding: 30px;
  border-radius: 10px;
  width: 90%;
  max-width: 600px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    transform: translateY(-50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.modal-content h2 {
  color: #8B4513;
  margin-bottom: 20px;
  font-size: 1.8em;
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
  transition: color 0.3s ease;
}

.close:hover {
  color: #000;
}

.loading {
  text-align: center;
  padding: 40px;
  font-size: 18px;
  color: #8B4513;
}

.btn-primary {
  background: #8B4513;
  color: white;
  border: none;
  padding: 12px 30px;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-primary:hover {
  background: #A0522D;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

@media (max-width: 768px) {
  .header {
    flex-direction: column;
    text-align: center;
  }

  .header h1 {
    font-size: 2em;
  }

  .header-actions {
    margin-top: 15px;
  }

  .filters {
    justify-content: flex-start;
    overflow-x: auto;
  }

  .modal-content {
    width: 95%;
    margin: 30px auto;
  }
}
"@ | Set-Content -Path "$projectPath\public\css\style.css"

Write-Host "✓ public/css/style.css created" -ForegroundColor Green

# Create public/css/menu.css
@"
.menu-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.menu-card {
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  cursor: pointer;
}

.menu-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.menu-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  background: #f0f0f0;
}

.menu-info {
  padding: 15px;
}

.menu-name {
  font-size: 1.3em;
  font-weight: bold;
  color: #8B4513;
  margin-bottom: 8px;
}

.menu-description {
  font-size: 0.9em;
  color: #666;
  margin-bottom: 12px;
  height: 40px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.menu-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #eee;
  padding-top: 12px;
}

.menu-price {
  font-size: 1.5em;
  font-weight: bold;
  color: #FFD700;
}

.add-to-cart-btn {
  background: #8B4513;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.add-to-cart-btn:hover {
  background: #A0522D;
  transform: scale(1.05);
}

.add-to-cart-btn:active {
  transform: scale(0.98);
}

.unavailable {
  opacity: 0.6;
  pointer-events: none;
}

.unavailable .add-to-cart-btn {
  background: #ccc;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .menu-container {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
  }

  .menu-image {
    height: 150px;
  }

  .menu-name {
    font-size: 1.1em;
  }
}
"@ | Set-Content -Path "$projectPath\public\css\menu.css"

Write-Host "✓ public/css/menu.css created" -ForegroundColor Green

# Create public/css/cart.css
@"
.cart-items {
  max-height: 400px;
  overflow-y: auto;
  margin: 20px 0;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  border-bottom: 1px solid #eee;
  background: #f9f9f9;
  border-radius: 5px;
  margin-bottom: 10px;
}

.item-details {
  flex: 1;
}

.item-name {
  font-weight: bold;
  color: #8B4513;
  margin-bottom: 5px;
}

.item-quantity {
  color: #666;
  font-size: 0.9em;
}

.item-price {
  font-weight: bold;
  color: #FFD700;
  margin: 0 15px;
  min-width: 80px;
  text-align: right;
}

.item-actions {
  display: flex;
  gap: 10px;
}

.qty-btn {
  background: #8B4513;
  color: white;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s ease;
}

.qty-btn:hover {
  background: #A0522D;
}

.remove-btn {
  background: #FF6B6B;
  color: white;
  border: none;
  padding: 5px 12px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.85em;
  transition: all 0.2s ease;
}

.remove-btn:hover {
  background: #FF5252;
}

.cart-empty {
  text-align: center;
  padding: 30px;
  color: #999;
}

.cart-summary {
  border-top: 2px solid #8B4513;
  padding: 20px 0;
  margin-top: 20px;
}

.cart-summary p {
  font-size: 1.2em;
  margin-bottom: 15px;
  color: #333;
}

.checkout-btn {
  width: 100%;
  background: #8B4513;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 5px;
  font-size: 1.1em;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.checkout-btn:hover {
  background: #A0522D;
  transform: translateY(-2px);
}

.checkout-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

.checkout-form {
  max-width: 500px;
}

.checkout-form input {
  width: 100%;
  padding: 12px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1em;
}

.checkout-form input:focus {
  outline: none;
  border-color: #8B4513;
  box-shadow: 0 0 5px rgba(139, 69, 19, 0.3);
}

.submit-btn {
  width: 100%;
  background: #8B4513;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 5px;
  font-size: 1.1em;
  font-weight: bold;
  cursor: pointer;
  margin-top: 20px;
  transition: all 0.3s ease;
}

.submit-btn:hover {
  background: #A0522D;
}

.confirmation {
  text-align: center;
}

.confirmation h2 {
  color: #4CAF50;
  font-size: 2em;
  margin-bottom: 20px;
}

#confirmationContent {
  background: #f0f0f0;
  padding: 20px;
  border-radius: 5px;
  margin: 20px 0;
  text-align: left;
}

#confirmationContent p {
  margin: 8px 0;
}

@media (max-width: 600px) {
  .cart-item {
    flex-wrap: wrap;
  }

  .item-price {
    width: 100%;
    text-align: left;
    margin: 10px 0;
  }

  .item-actions {
    width: 100%;
    justify-content: flex-end;
  }
}
"@ | Set-Content -Path "$projectPath\public\css\cart.css"

Write-Host "✓ public/css/cart.css created" -ForegroundColor Green

# Create public/js/api.js
@"
const API_BASE_URL = '/api';

// Menu API calls
async function fetchAllMenus() {
  try {
    const response = await fetch(\`\${API_BASE_URL}/menus\`);
    if (!response.ok) throw new Error('Failed to fetch menus');
    return await response.json();
  } catch (error) {
    console.error('Error fetching menus:', error);
    return [];
  }
}

async function createOrder(orderData) {
  try {
    const response = await fetch(\`\${API_BASE_URL}/orders\`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(orderData)
    });
    if (!response.ok) throw new Error('Failed to create order');
    return await response.json();
  } catch (error) {
    console.error('Error creating order:', error);
    return null;
  }
}

async function addMenuItems(items) {
  try {
    for (const item of items) {
      const response = await fetch(\`\${API_BASE_URL}/menus\`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(item)
      });
      if (!response.ok) console.error('Failed to add menu item:', item.name);
    }
  } catch (error) {
    console.error('Error adding menu items:', error);
  }
}
"@ | Set-Content -Path "$projectPath\public\js\api.js"

Write-Host "✓ public/js/api.js created" -ForegroundColor Green

# Create public/js/menu.js
@"
let allMenus = [];
let filteredMenus = [];
let selectedCategory = 'all';

async function initializeMenu() {
  try {
    allMenus = await fetchAllMenus();
    
    if (allMenus.length === 0) {
      await loadSampleMenus();
      allMenus = await fetchAllMenus();
    }
    
    displayMenus(allMenus);
  } catch (error) {
    console.error('Error initialLet me provide the complete setup script directly. Copy this entire script into PowerShell:

```powershell
# ============================================
# AmaEyoba Cafe - Complete Automated Setup
# ============================================

$projectPath = "C:\Users\Amanuel\Desktop\AmaEyoba-Cafe"
Write-Host "🔧 Setting up AmaEyoba Cafe..." -ForegroundColor Cyan

# Create directories
New-Item -ItemType Directory -Path "$projectPath\server\models" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\server\routes" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\server\controllers" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\server\config" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\public\css" -Force | Out-Null
New-Item -ItemType Directory -Path "$projectPath\public\js" -Force | Out-Null
Write-Host "✓ Directories created" -ForegroundColor Green

# package.json
@"
{
  "name": "amaeyoba-cafe",
  "version": "1.0.0",
  "description": "Full-stack cafe menu website with shopping cart and orders",
  "main": "server/server.js",
  "scripts": {
    "start": "node server/server.js",
    "dev": "nodemon server/server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "dotenv": "^16.0.3",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
}
"@ | Set-Content -Path "$projectPath\package.json"
Write-Host "✓ package.json" -ForegroundColor Green

# .env
@"
PORT=5000
MONGODB_URI=mongodb://localhost:27017/amaeyoba-cafe
NODE_ENV=development
"@ | Set-Content -Path "$projectPath\.env"
Write-Host "✓ .env" -ForegroundColor Green

# .gitignore
@"
node_modules/
.env
*.log
.DS_Store
"@ | Set-Content -Path "$projectPath\.gitignore"
Write-Host "✓ .gitignore" -ForegroundColor Green

# server/config/db.js
@"
const mongoose = require('mongoose');
require('dotenv').config();

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('✓ MongoDB connected');
  } catch (err) {
    console.error('✗ MongoDB error:', err);
    process.exit(1);
  }
};

module.exports = connectDB;
"@ | Set-Content -Path "$projectPath\server\config\db.js"
Write-Host "✓ server/config/db.js" -ForegroundColor Green

# server/models/Menu.js
@"
const mongoose = require('mongoose');

const menuSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: { type: String, required: true },
  price: { type: Number, required: true },
  category: { type: String, required: true },
  image: { type: String, default: 'https://via.placeholder.com/300x200?text=Menu+Item' },
  available: { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Menu', menuSchema);
"@ | Set-Content -Path "$projectPath\server\models\Menu.js"
Write-Host "✓ server/models/Menu.js" -ForegroundColor Green

# server/models/Order.js
@"
const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema({
  customer: {
    name: { type: String, required: true },
    email: { type: String, required: true },
    phone: { type: String, required: true }
  },
  items: [
    {
      menuId: mongoose.Schema.Types.ObjectId,
      name: String,
      price: Number,
      quantity: Number
    }
  ],
  totalPrice: { type: Number, required: true },
  status: { type: String, default: 'Pending', enum: ['Pending', 'Confirmed', 'Completed', 'Cancelled'] },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Order', orderSchema);
"@ | Set-Content -Path "$projectPath\server\models\Order.js"
Write-Host "✓ server/models/Order.js" -ForegroundColor Green

# server/controllers/menuController.js
@"
const Menu = require('../models/Menu');

exports.getAllMenus = async (req, res) => {
  try {
    const menus = await Menu.find();
    res.json(menus);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

exports.getMenuById = async (req, res) => {
  try {
    const menu = await Menu.findById(req.params.id);
    if (!menu) return res.status(404).json({ message: 'Menu not found' });
    res.json(menu);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

exports.createMenu = async (req, res) => {
  const menu = new Menu({
    name: req.body.name,
    description: req.body.description,
    price: req.body.price,
    category: req.body.category,
    image: req.body.image
  });
  try {
    const newMenu = await menu.save();
    res.status(201).json(newMenu);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

exports.updateMenu = async (req, res) => {
  try {
    const menu = await Menu.findById(req.params.id);
    if (!menu) return res.status(404).json({ message: 'Menu not found' });
    
    if (req.body.name) menu.name = req.body.name;
    if (req.body.description) menu.description = req.body.description;
    if (req.body.price) menu.price = req.body.price;
    if (req.body.category) menu.category = req.body.category;
    if (req.body.image) menu.image = req.body.image;
    if (req.body.available !== undefined) menu.available = req.body.available;

    const updatedMenu = await menu.save();
    res.json(updatedMenu);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

exports.deleteMenu = async (req, res) => {
  try {
    const menu = await Menu.findByIdAndDelete(req.params.id);
    if (!menu) return res.status(404).json({ message: 'Menu not found' });
    res.json({ message: 'Menu deleted' });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};
"@ | Set-Content -Path "$projectPath\server\controllers\menuController.js"
Write-Host "✓ server/controllers/menuController.js" -ForegroundColor Green

# server/controllers/orderController.js
@"
const Order = require('../models/Order');

exports.createOrder = async (req, res) => {
  const order = new Order({
    customer: req.body.customer,
    items: req.body.items,
    totalPrice: req.body.totalPrice
  });
  try {
    const newOrder = await order.save();
    res.status(201).json(newOrder);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

exports.getAllOrders = async (req, res) => {
  try {
    const orders = await Order.find().sort({ createdAt: -1 });
    res.json(orders);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

exports.getOrderById = async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ message: 'Order not found' });
    res.json(order);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

exports.updateOrderStatus = async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ message: 'Order not found' });
    order.status = req.body.status;
    const updatedOrder = await order.save();
    res.json(updatedOrder);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};
"@ | Set-Content -Path "$projectPath\server\controllers\orderController.js"
Write-Host "✓ server/controllers/orderController.js" -ForegroundColor Green

# server/routes/menuRoutes.js
@"
const express = require('express');
const router = express.Router();
const menuController = require('../controllers/menuController');

router.get('/', menuController.getAllMenus);
router.get('/:id', menuController.getMenuById);
router.post('/', menuController.createMenu);
router.put('/:id', menuController.updateMenu);
router.delete('/:id', menuController.deleteMenu);

module.exports = router;
"@ | Set-Content -Path "$projectPath\server\routes\menuRoutes.js"
Write-Host "✓ server/routes/menuRoutes.js" -ForegroundColor Green

# server/routes/orderRoutes.js
@"
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');

router.post('/', orderController.createOrder);
router.get('/', orderController.getAllOrders);
router.get('/:id', orderController.getOrderById);
router.put('/:id', orderController.updateOrderStatus);

module.exports = router;
"@ | Set-Content -Path "$projectPath\server\routes\orderRoutes.js"
Write-Host "✓ server/routes/orderRoutes.js" -ForegroundColor Green

# server/server.js
@"
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();
const connectDB = require('./config/db');

const menuRoutes = require('./routes/menuRoutes');
const orderRoutes = require('./routes/orderRoutes');

const app = express();

app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public'));

connectDB();

app.use('/api/menus', menuRoutes);
app.use('/api/orders', orderRoutes);

app.get('/api/health', (req, res) => {
  res.json({ status: 'Server is running!' });
});

app.get('/', (req, res) => {
  res.sendFile(__dirname + '/../public/index.html');
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`✓ AmaEyoba Cafe server: http://localhost:\${PORT}`);
});
"@ | Set-Content -Path "$projectPath\server\server.js"
Write-Host "✓ server/server.js" -ForegroundColor Green

# public/index.html
@"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AmaEyoba Cafe - Menu</title>
  <link rel="stylesheet" href="css/style.css">
  <link rel="stylesheet" href="css/menu.css">
</head>
<body>
  <div class="container">
    <header class="header">
      <div class="header-content">
        <h1>☕ AmaEyoba Cafe</h1>
        <p>Fresh Coffee, Fresh Vibes</p>
      </div>
      <div class="header-actions">
        <button class="cart-btn" id="cartBtn">
          🛒 Cart <span class="cart-count" id="cartCount">0</span>
        </button>
      </div>
    </header>

    <nav class="filters">
      <button class="filter-btn active" data-category="all">All Items</button>
      <button class="filter-btn" data-category="Coffee">Coffee</button>
      <button class="filter-btn" data-category="Tea">Tea</button>
      <button class="filter-btn" data-category="Pastry">Pastry</button>
      <button class="filter-btn" data-category="Sandwich">Sandwich</button>
    </nav>

    <main class="menu-container" id="menuContainer">
      <div class="loading">Loading menu items...</div>
    </main>
  </div>

  <div class="modal" id="cartModal">
    <div class="modal-content">
      <span class="close" id="closeCart">&times;</span>
      <h2>Shopping Cart</h2>
      <div class="cart-items" id="cartItems"></div>
      <div class="cart-summary">
        <p>Total: <strong id="cartTotal">Br 0.00</strong></p>
        <button class="checkout-btn" id="checkoutBtn">Proceed to Checkout</button>
      </div>
    </div>
  </div>

  <div class="modal" id="checkoutModal">
    <div class="modal-content checkout-form">
      <span class="close" id="closeCheckout">&times;</span>
      <h2>Checkout</h2>
      <form id="orderForm">
        <input type="text" placeholder="Full Name" id="customerName" required>
        <input type="email" placeholder="Email" id="customerEmail" required>
        <input type="tel" placeholder="Phone Number" id="customerPhone" required>
        <p>Total: <strong id="checkoutTotal">Br 0.00</strong></p>
        <button type="submit" class="submit-btn">Place Order</button>
      </form>
    </div>
  </div>

  <div class="modal" id="confirmationModal">
    <div class="modal-content confirmation">
      <span class="close" id="closeConfirmation">&times;</span>
      <h2>✓ Order Confirmed!</h2>
      <div id="confirmationContent"></div>
      <button class="btn-primary" id="continueShopping">Continue Shopping</button>
    </div>
  </div>

  <script src="js/api.js"></script>
  <script src="js/menu.js"></script>
  <script src="js/cart.js"></script>
  <script src="js/app.js"></script>
</body>
</html>
"@ | Set-Content -Path "$projectPath\public\index.html"
Write-Host "✓ public/index.html" -ForegroundColor Green

# public/css/style.css
@"
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #f5e6d3 0%, #e8d5c4 100%);
  color: #333;
  line-height: 1.6;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  background: linear-gradient(135deg, #8B4513 0%, #A0522D 100%);
  color: white;
  padding: 30px;
  border-radius: 10px;
  margin-bottom: 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.header h1 {
  font-size: 2.5em;
  margin-bottom: 5px;
}

.header p {
  opacity: 0.9;
  font-style: italic;
}

.header-actions {
  display: flex;
  gap: 10px;
}

.cart-btn {
  background: #FFD700;
  color: #8B4513;
  border: none;
  padding: 12px 20px;
  border-radius: 25px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.cart-btn:hover {
  background: #FFC700;
  transform: scale(1.05);
}

.cart-count {
  background: #FF6B6B;
  color: white;
  border-radius: 50%;
  padding: 2px 6px;
  font-size: 12px;
  margin-left: 5px;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
  flex-wrap: wrap;
  justify-content: center;
}

.filter-btn {
  padding: 10px 20px;
  border: 2px solid #8B4513;
  background: white;
  color: #8B4513;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 500;
}

.filter-btn:hover,
.filter-btn.active {
  background: #8B4513;
  color: white;
}

.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: #fefcfb;
  margin: 50px auto;
  padding: 30px;
  border-radius: 10px;
  width: 90%;
  max-width: 600px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.modal-content h2 {
  color: #8B4513;
  margin-bottom: 20px;
  font-size: 1.8em;
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.close:hover {
  color: #000;
}

.loading {
  text-align: center;
  padding: 40px;
  font-size: 18px;
  color: #8B4513;
}

.btn-primary {
  background: #8B4513;
  color: white;
  border: none;
  padding: 12px 30px;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
}

.btn-primary:hover {
  background: #A0522D;
}
"@ | Set-Content -Path "$projectPath\public\css\style.css"
Write-Host "✓ public/css/style.css" -ForegroundColor Green

# public/css/menu.css
@"
.menu-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.menu-card {
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.menu-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.menu-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  background: #f0f0f0;
}

.menu-info {
  padding: 15px;
}

.menu-name {
  font-size: 1.3em;
  font-weight: bold;
  color: #8B4513;
  margin-bottom: 8px;
}

.menu-description {
  font-size: 0.9em;
  color: #666;
  margin-bottom: 12px;
  height: 40px;
  overflow: hidden;
}

.menu-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #eee;
  padding-top: 12px;
}

.menu-price {
  font-size: 1.5em;
  font-weight: bold;
  color: #FFD700;
}

.add-to-cart-btn {
  background: #8B4513;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.add-to-cart-btn:hover {
  background: #A0522D;
  transform: scale(1.05);
}

@media (max-width: 768px) {
  .menu-container {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  }

  .menu-image {
    height: 150px;
  }
}
"@ | Set-Content -Path "$projectPath\public\css\menu.css"
Write-Host "✓ public/css/menu.css" -ForegroundColor Green

# public/css/cart.css
@"
.cart-items {
  max-height: 400px;
  overflow-y: auto;
  margin: 20px 0;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  border-bottom: 1px solid #eee;
  background: #f9f9f9;
  border-radius: 5px;
  margin-bottom: 10px;
}

.item-details {
  flex: 1;
}

.item-name {
  font-weight: bold;
  color: #8B4513;
  margin-bottom: 5px;
}

.item-quantity {
  color: #666;
  font-size: 0.9em;
}

.item-price {
  font-weight: bold;
  color: #FFD700;
  margin: 0 15px;
  min-width: 80px;
  text-align: right;
}

.item-actions {
  display: flex;
  gap: 10px;
}

.qty-btn {
  background: #8B4513;
  color: white;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  font-weight: bold;
}

.qty-btn:hover {
  background: #A0522D;
}

.remove-btn {
  background: #FF6B6B;
  color: white;
  border: none;
  padding: 5px 12px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.85em;
}

.remove-btn:hover {
  background: #FF5252;
}

.cart-empty {
  text-align: center;
  padding: 30px;
  color: #999;
}

.cart-summary {
  border-top: 2px solid #8B4513;
  padding: 20px 0;
  margin-top: 20px;
}

.cart-summary p {
  font-size: 1.2em;
  margin-bottom: 15px;
}

.checkout-btn {
  width: 100%;
  background: #8B4513;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 5px;
  font-size: 1.1em;
  font-weight: bold;
  cursor: pointer;
}

.checkout-btn:hover {
  background: #A0522D;
}

.checkout-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.checkout-form input {
  width: 100%;
  padding: 12px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1em;
}

.checkout-form input:focus {
  outline: none;
  border-color: #8B4513;
}

.submit-btn {
  width: 100%;
  background: #8B4513;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 5px;
  font-size: 1.1em;
  font-weight: bold;
  cursor: pointer;
  margin-top: 20px;
}

.submit-btn:hover {
  background: #A0522D;
}

.confirmation {
  text-align: center;
}

.confirmation h2 {
  color: #4CAF50;
  font-size: 2em;
}

#confirmationContent {
  background: #f0f0f0;
  padding: 20px;
  border-radius: 5px;
  margin: 20px 0;
  text-align: left;
}
"@ | Set-Content -Path "$projectPath\public\css\cart.css"
Write-Host "✓ public/css/cart.css" -ForegroundColor Green

# public/js/api.js
@"
const API_BASE_URL = '/api';

async function fetchAllMenus() {
  try {
    const response = await fetch(`\${API_BASE_URL}/menus`);
    if (!response.ok) throw new Error('Failed');
    return await response.json();
  } catch (error) {
    console.error('Error fetching menus:', error);
    return [];
  }
}

async function createOrder(orderData) {
  try {
    const response = await fetch(`\${API_BASE_URL}/orders`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(orderData)
    });
    if (!response.ok) throw new Error('Failed');
    return await response.json();
  } catch (error) {
    console.error('Error:', error);
    return null;
  }
}

async function addMenuItems(items) {
  for (const item of items) {
    try {
      await fetch(`\${API_BASE_URL}/menus`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(item)
      });
    } catch (error) {
      console.error('Error adding:', item.name);
    }
  }
}
"@ | Set-Content -Path "$projectPath\public\js\api.js"
Write-Host "✓ public/js/api.js" -ForegroundColor Green

# public/js/menu.js
@"
let allMenus = [];
let selectedCategory = 'all';

async function initializeMenu() {
  allMenus = await fetchAllMenus();
  if (allMenus.length === 0) {
    await loadSampleMenus();
    allMenus = await fetchAllMenus();
  }
  displayMenus(allMenus);
}

async function loadSampleMenus() {
  const sampleMenus = [
    { name: 'Espresso', description: 'Strong black coffee', price: 45, category: 'Coffee', image: 'https://via.placeholder.com/300x200?text=Espresso' },
    { name: 'Cappuccino', description: 'Coffee with steamed milk', price: 55, category: 'Coffee', image: 'https://via.placeholder.com/300x200?text=Cappuccino' },
    { name: 'Latte', description: 'Smooth and creamy', price: 60, category: 'Coffee', image: 'https://via.placeholder.com/300x200?text=Latte' },
    { name: 'Green Tea', description: 'Fresh green tea', price: 40, category: 'Tea', image: 'https://via.placeholder.com/300x200?text=Green+Tea' },
    { name: 'Chamomile', description: 'Relaxing chamomile', price: 35, category: 'Tea', image: 'https://via.placeholder.com/300x200?text=Chamomile' },
    { name: 'Croissant', description: 'Buttery French pastry', price: 50, category: 'Pastry', image: 'https://via.placeholder.com/300x200?text=Croissant' },
    { name: 'Muffin', description: 'Chocolate chip muffin', price: 40, category: 'Pastry', image: 'https://via.placeholder.com/300x200?text=Muffin' },
    { name: 'Sandwich', description: 'Fresh vegetable sandwich', price: 70, category: 'Sandwich', image: 'https://via.placeholder.com/300x200?text=Sandwich' }
  ];
  await addMenuItems(sampleMenus);
}

function displayMenus(menus) {
  const container = document.getElementById('menuContainer');
  container.innerHTML = '';
  
  menus.forEach(menu => {
    const card = document.createElement('div');
    card.className = 'menu-card';
    card.innerHTML = `
      <img src="\${menu.image}" alt="\${menu.name}" class="menu-image">
      <div class="menu-info">
        <h3 class="menu-name">\${menu.name}</h3>
        <p class="menu-description">\${menu.description}</p>
        <div class="menu-footer">
          <span class="menu-price">Br \${menu.price}</span>
          <button class="add-to-cart-btn" onclick="addToCart('\${menu._id}', '\${menu.name}', \${menu.price})">+ Add</button>
        </div>
      </div>
    `;
    container.appendChild(card);
  });
}

function filterMenus(category) {
  selectedCategory = category;
  if (category === 'all') {
    displayMenus(allMenus);
  } else {
    const filtered = allMenus.filter(m => m.category === category);
    displayMenus(filtered);
  }
}

document.addEventListener('DOMContentLoaded', initializeMenu);
"@ | Set-Content -Path "$projectLet me provide you with the final part of the setup script. Copy and paste this complete section into PowerShell:

```powershell
# ============================================
# PART 2: Complete JavaScript Files & Installation
# ============================================

$projectPath = "C:\Users\Amanuel\Desktop\AmaEyoba-Cafe"

# public/js/cart.js
@"
let cart = [];

function addToCart(id, name, price) {
  const item = cart.find(item => item.id === id);
  if (item) {
    item.quantity++;
  } else {
    cart.push({ id, name, price, quantity: 1 });
  }
  updateCartCount();
  localStorage.setItem('cart', JSON.stringify(cart));
  alert(name + ' added to cart!');
}

function removeFromCart(id) {
  cart = cart.filter(item => item.id !== id);
  updateCartCount();
  localStorage.setItem('cart', JSON.stringify(cart));
  displayCartItems();
}

function updateQuantity(id, change) {
  const item = cart.find(item => item.id === id);
  if (item) {
    item.quantity += change;
    if (item.quantity <= 0) removeFromCart(id);
  }
  updateCartCount();
  localStorage.setItem('cart', JSON.stringify(cart));
  displayCartItems();
}

function updateCartCount() {
  const count = cart.reduce((sum, item) => sum + item.quantity, 0);
  document.getElementById('cartCount').textContent = count;
}

function displayCartItems() {
  const cartItems = document.getElementById('cartItems');
  if (cart.length === 0) {
    cartItems.innerHTML = '<div class="cart-empty">Your cart is empty</div>';
    document.getElementById('checkoutBtn').disabled = true;
    return;
  }
  
  cartItems.innerHTML = cart.map(item => `
    <div class="cart-item">
      <div class="item-details">
        <div class="item-name">\${item.name}</div>
        <div class="item-quantity">Qty: \${item.quantity}</div>
      </div>
      <div class="item-price">Br \${(item.price * item.quantity).toFixed(2)}</div>
      <div class="item-actions">
        <button class="qty-btn" onclick="updateQuantity('\${item.id}', -1)">-</button>
        <button class="qty-btn" onclick="updateQuantity('\${item.id}', 1)">+</button>
        <button class="remove-btn" onclick="removeFromCart('\${item.id}')">Remove</button>
      </div>
    </div>
  `).join('');
  
  const total = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  document.getElementById('cartTotal').textContent = 'Br ' + total.toFixed(2);
  document.getElementById('checkoutTotal').textContent = 'Br ' + total.toFixed(2);
  document.getElementById('checkoutBtn').disabled = false;
}

function loadCartFromStorage() {
  const stored = localStorage.getItem('cart');
  if (stored) {
    cart = JSON.parse(stored);
    updateCartCount();
  }
}

window.addEventListener('load', loadCartFromStorage);
"@ | Set-Content -Path "$projectPath\public\js\cart.js"
Write-Host "✓ public/js/cart.js" -ForegroundColor Green

# public/js/app.js
@"
document.getElementById('cartBtn').addEventListener('click', () => {
  displayCartItems();
  document.getElementById('cartModal').style.display = 'block';
});

document.getElementById('closeCart').addEventListener('click', () => {
  document.getElementById('cartModal').style.display = 'none';
});

document.getElementById('checkoutBtn').addEventListener('click', () => {
  document.getElementById('cartModal').style.display = 'none';
  document.getElementById('checkoutModal').style.display = 'block';
});

document.getElementById('closeCheckout').addEventListener('click', () => {
  document.getElementById('checkoutModal').style.display = 'none';
});

document.getElementById('orderForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  
  const orderData = {
    customer: {
      name: document.getElementById('customerName').value,
      email: document.getElementById('customerEmail').value,
      phone: document.getElementById('customerPhone').value
    },
    items: cart,
    totalPrice: cart.reduce((sum, item) => sum + (item.price * item.quantity), 0)
  };

  const result = await createOrder(orderData);
  
  if (result) {
    document.getElementById('checkoutModal').style.display = 'none';
    
    const confirmContent = `
      <p><strong>Order ID:</strong> \${result._id}</p>
      <p><strong>Customer:</strong> \${result.customer.name}</p>
      <p><strong>Total:</strong> Br \${result.totalPrice}</p>
      <p><strong>Status:</strong> \${result.status}</p>
      <p style="margin-top: 15px; color: #666;">Your order has been received. We will prepare it shortly!</p>
    `;
    document.getElementById('confirmationContent').innerHTML = confirmContent;
    document.getElementById('confirmationModal').style.display = 'block';
    
    cart = [];
    localStorage.setItem('cart', JSON.stringify(cart));
    updateCartCount();
  } else {
    alert('Error placing order. Please try again.');
  }
});

document.getElementById('closeConfirmation').addEventListener('click', () => {
  document.getElementById('confirmationModal').style.display = 'none';
});

document.getElementById('continueShopping').addEventListener('click', () => {
  document.getElementById('confirmationModal').style.display = 'none';
});

document.querySelectorAll('.filter-btn').forEach(btn => {
  btn.addEventListener('click', (e) => {
    document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
    e.target.classList.add('active');
    filterMenus(e.target.dataset.category);
  });
});

window.addEventListener('click', (event) => {
  const cartModal = document.getElementById('cartModal');
  const checkoutModal = document.getElementById('checkoutModal');
  const confirmModal = document.getElementById('confirmationModal');
  
  if (event.target === cartModal) cartModal.style.display = 'none';
  if (event.target === checkoutModal) checkoutModal.style.display = 'none';
  if (event.target === confirmModal) confirmModal.style.display = 'none';
});
"@ | Set-Content -Path "$projectPath\public\js\app.js"
Write-Host "✓ public/js/app.js" -ForegroundColor Green

# README.md
@"
# ☕ AmaEyoba Cafe - Full Stack Menu Website

A complete full-stack web application for AmaEyoba Cafe with menu display, shopping cart, and order management.

## Features

✅ Browse cafe menu items organized by category
✅ Shopping cart with add/remove/quantity features
✅ Checkout with customer information
✅ Order placement and confirmation
✅ Responsive mobile-friendly design
✅ Real-time cart updates

## Tech Stack

- **Backend**: Node.js + Express.js
- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Database**: MongoDB
- **Package Manager**: npm

## Prerequisites

- Node.js (v14+)
- MongoDB installed and running locally OR MongoDB Atlas account

## Installation & Setup

1. **Navigate to project directory**
   \`\`\`bash
   cd C:\Users\Amanuel\Desktop\AmaEyoba-Cafe
   \`\`\`

2. **Install dependencies**
   \`\`\`bash
   npm install
   \`\`\`

3. **Configure MongoDB**
   
   **Option A: Local MongoDB**
   - Install MongoDB Community Edition
   - Start MongoDB service
   - .env is already set to: mongodb://localhost:27017/amaeyoba-cafe

   **Option B: MongoDB Atlas (Cloud)**
   - Create account at mongodb.com
   - Create a cluster
   - Get connection string
   - Update .env: MONGODB_URI=your_connection_string

4. **Start the server**
   \`\`\`bash
   npm start
   \`\`\`

5. **Open in browser**
   \`\`\`
   http://localhost:5000
   \`\`\`

## Project Structure

\`\`\`
AmaEyoba-Cafe/
├── server/
│   ├── config/db.js          (MongoDB connection)
│   ├── models/               (Data schemas)
│   │   ├── Menu.js
│   │   └── Order.js
│   ├── controllers/          (Business logic)
│   │   ├── menuController.js
│   │   └── orderController.js
│   ├── routes/               (API endpoints)
│   │   ├── menuRoutes.js
│   │   └── orderRoutes.js
│   └── server.js             (Main server file)
├── public/
│   ├── index.html            (Main page)
│   ├── css/                  (Stylesheets)
│   │   ├── style.css
│   │   ├── menu.css
│   │   └── cart.css
│   └── js/                   (Client-side logic)
│       ├── api.js
│       ├── menu.js
│       ├── cart.js
│       └── app.js
├── .env                      (Environment variables)
├── package.json              (Dependencies)
└── README.md                 (This file)
\`\`\`

## API Endpoints

### Menus
- \`GET /api/menus\` - Get all menu items
- \`GET /api/menus/:id\` - Get menu by ID
- \`POST /api/menus\` - Create new menu item
- \`PUT /api/menus/:id\` - Update menu item
- \`DELETE /api/menus/:id\` - Delete menu item

### Orders
- \`POST /api/orders\` - Create new order
- \`GET /api/orders\` - Get all orders
- \`GET /api/orders/:id\` - Get order by ID
- \`PUT /api/orders/:id\` - Update order status

## Usage

1. **Browse Menu**: Load the homepage to see all cafe menu items
2. **Filter by Category**: Use category buttons (Coffee, Tea, Pastry, Sandwich)
3. **Add to Cart**: Click "Add" on any menu item
4. **View Cart**: Click the cart button to review items
5. **Checkout**: Provide name, email, and phone number
6. **Confirm Order**: Submit order and receive confirmation

## Menu Items

Sample menu includes:
- **Coffee**: Espresso, Cappuccino, Latte
- **Tea**: Green Tea, Chamomile
- **Pastry**: Croissant, Muffin
- **Sandwich**: Fresh Vegetable Sandwich

## Development

To use nodemon for auto-reload during development:

\`\`\`bash
npm run dev
\`\`\`

## License

ISC

## Support

For issues or questions, contact the development team.
"@ | Set-Content -Path "$projectPath\README.md"
Write-Host "✓ README.md" -ForegroundColor Green

Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "✓ All files created successfully!" -ForegroundColor Green
Write-Host "========================================`n" -ForegroundColor Cyan

# Navigate to project and install dependencies
Write-Host "📦 Installing npm dependencies..." -ForegroundColor Yellow
cd $projectPath
npm install

Write-Host "`n========================================" -ForegroundColor Green
Write-Host "✓ Setup Complete! Ready to run AmaEyoba Cafe" -ForegroundColor Green
Write-Host "========================================`n" -ForegroundColor Green

Write-Host "📋 NEXT STEPS:" -ForegroundColor Cyan
Write-Host "1. Make sure MongoDB is running" -ForegroundColor White
Write-Host "2. Run: npm start" -ForegroundColor White
Write-Host "3. Open: http://localhost:5000" -ForegroundColor White
Write-Host "`n" -ForegroundColor Green# aman
generally this is for store  the code and share experiances with in the co workers
