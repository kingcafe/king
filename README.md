<head>
  <meta charset="UTF-8" />
  <title>King Cafe Table Orders</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #222;
      color: #fff;
      margin: 0;
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #f39c12;
    }
    .tables {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
      gap: 15px;
      max-width: 1000px;
      margin: 30px auto;
    }
    .table-btn {
      background-color: #444;
      border: 2px solid #f39c12;
      padding: 20px;
      font-size: 16px;
      border-radius: 8px;
      cursor: pointer;
      transition: 0.3s;
    }
    .table-btn:hover {
      background-color: #f39c12;
      color: black;
    }
    .view-orders {
      display: block;
      margin: 10px auto 30px;
      padding: 10px 20px;
      font-size: 18px;
      background-color: #f39c12;
      color: black;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
    .menu {
      display: none;
      max-width: 900px;
      margin: 0 auto;
    }
    .category {
      margin: 20px 0;
    }
    .category h2 {
      color: #f39c12;
      margin-bottom: 10px;
    }
    .items {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
    }
    .item {
      background-color: #333;
      padding: 10px 15px;
      border-radius: 5px;
      cursor: pointer;
      border: 1px solid #f39c12;
      position: relative;
    }
    .item.selected {
      background-color: #f39c12;
      color: black;
    }
    .item::after {
      content: attr(data-qty);
      position: absolute;
      top: -8px;
      right: -8px;
      background: red;
      color: white;
      font-size: 14px;
      padding: 2px 6px;
      border-radius: 50%;
      display: none;
    }
    .item[data-qty]:not([data-qty="0"])::after {
      display: block;
    }
    .back-btn, .save-btn {
      margin: 20px 10px 0 0;
      padding: 10px 20px;
      background-color: #f39c12;
      color: black;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #allOrders {
      display: none;
      max-width: 800px;
      margin: 20px auto;
      background: #333;
      padding: 20px;
      border-radius: 10px;
    }
    h3#totalDisplay {
      color: #0f0;
      font-weight: bold;
      margin-top: 10px;
    }
    #orderSummary {
      margin-top: 15px;
      padding: 10px;
      background: #111;
      border: 1px solid #f39c12;
      border-radius: 8px;
    }
    #orderSummary ul {
      list-style: none;
      padding-left: 0;
    }
    #orderSummary li {
      margin: 5px 0;
    }
  </style>
</head>
<body>

<h1>🍽️ King Cafe - Table Orders</h1>

<button class="view-orders" onclick="showAllOrders()">👀 View All Orders</button>

<div class="tables" id="tables"></div>

<div class="menu" id="menu">
  <h2 id="menuTitle">Ordering for Table</h2>
  <div id="categories"></div>

  <h3 id="totalDisplay">💰 کۆی پارە: <span id="totalPrice">0</span> دینار</h3>

  <div id="orderSummary">
    <h3 style="color:#0ff;">🧾 Table Order Summary</h3>
    <ul id="orderItems"></ul>
  </div>

  <button class="save-btn" onclick="saveOrder()">💾 Save Order</button>
  <button class="back-btn" onclick="backToTables()">↩ Back to Tables</button>
  <button class="back-btn" onclick="undoLastAction()">↩ Undo Last</button>
</div>

<div id="allOrders">
  <h2 style="color: #f39c12;">📋 All Table Orders</h2>
  <div id="orderList"></div>
  <button class="back-btn" onclick="backToTables()">↩ Back to Tables</button>
</div>

<script>
  const tableOrders = {};
  let currentTable = null;
  let undoStack = [];

  // Updated categories with custom prices:
  const categories = {
    Hookah: [
      {name: 'King 1', price: 5000},
      {name: 'King 2', price: 5500},
      {name: 'King 3', price: 6000},
      {name: 'King 4', price: 5200},
      {name: 'King 5', price: 5100},
      {name: 'King 6', price: 5300},
      {name: 'King 7', price: 5800},
      {name: 'King 8', price: 4900},
      {name: 'King 9', price: 4700},
      {name: 'King 10', price: 5500},
      {name: 'King 11', price: 5600},
      {name: 'Dw Sew', price: 4500},
      {name: 'Bnisht', price: 4800},
      {name: 'Baghdadi', price: 5200},
      {name: 'Limo', price: 4700},
    ],
    Foods: [
      {name: 'Pizza', price: 7000},
      {name: 'Burger', price: 6500},
      {name: 'Fries', price: 3000},
      {name: 'Shawarma', price: 5500},
    ],
    Drinks: [
      {name: 'Tea', price: 1500},
      {name: 'Coffee', price: 2500},
      {name: 'Coke', price: 2000},
      {name: 'Water', price: 1000},
    ],
    Drink2: [
      {name: 'Smoothie', price: 4500},
      {name: 'Lemonade', price: 4000},
      {name: 'Iced Tea', price: 3500},
    ],
    Drinkyy: [
      {name: 'Energy Drink', price: 5000},
      {name: 'Milkshake', price: 6000},
      {name: 'Cold Brew', price: 5500},
    ],
    Sweets: [
      {name: 'Pancake', price: 3000},
      {name: 'Cupcake', price: 2500},
      {name: 'Trilece', price: 4000},
      {name: 'Triangle Banana Krep', price: 4500},
    ]
  };

  function createTables() {
    const container = document.getElementById('tables');
    container.innerHTML = ''; // Clear in case
    for (let i = 1; i <= 60; i++) {
      const btn = document.createElement('button');
      btn.textContent = 'Table ' + i;
      btn.className = 'table-btn';
      btn.onclick = () => openMenu(i);
      container.appendChild(btn);
    }
  }

  function openMenu(tableNumber) {
    currentTable = tableNumber;
    document.getElementById('tables').style.display = 'none';
    document.getElementById('menu').style.display = 'block';
    document.getElementById('allOrders').style.display = 'none';
    document.getElementById('menuTitle').textContent = `Ordering for Table ${tableNumber}`;

    const categoriesContainer = document.getElementById('categories');
    categoriesContainer.innerHTML = '';

    const currentOrder = tableOrders[currentTable] || {};

    Object.entries(categories).forEach(([category, items]) => {
      const div = document.createElement('div');
      div.className = 'category';
      const title = document.createElement('h2');
      title.textContent = category;
      div.appendChild(title);

      const itemsDiv = document.createElement('div');
      itemsDiv.className = 'items';

      items.forEach(itemObj => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'item';
        itemDiv.textContent = itemObj.name;
        itemDiv.setAttribute('data-price', itemObj.price);
        const qty = currentOrder[itemObj.name]?.qty || 0;
        if (qty > 0) {
          itemDiv.classList.add('selected');
          itemDiv.setAttribute('data-qty', qty);
        } else {
          itemDiv.setAttribute('data-qty', 0);
        }

        itemDiv.onclick = () => {
          let q = parseInt(itemDiv.getAttribute('data-qty')) || 0;
          undoStack.push({ el: itemDiv, prevQty: q });
          q++;
          itemDiv.setAttribute('data-qty', q);
          itemDiv.classList.add('selected');
          updateTotal();
        };

        itemDiv.oncontextmenu = (e) => {
          e.preventDefault();
          let q = parseInt(itemDiv.getAttribute('data-qty')) || 0;
          undoStack.push({ el: itemDiv, prevQty: q });
          q = Math.max(0, q - 1);
          itemDiv.setAttribute('data-qty', q);
          if (q === 0) itemDiv.classList.remove('selected');
          updateTotal();
        };

        itemsDiv.appendChild(itemDiv);
      });

      div.appendChild(itemsDiv);
      categoriesContainer.appendChild(div);
    });

    updateTotal();
  }

  function updateTotal() {
    const selectedItems = document.querySelectorAll('.item.selected');
    let total = 0;
    const summaryList = document.getElementById('orderItems');
    summaryList.innerHTML = '';

    selectedItems.forEach(el => {
      const qty = parseInt(el.getAttribute('data-qty')) || 0;
      const item = el.textContent;
      const price = parseInt(el.getAttribute('data-price')) || 0;
      total += qty * price;

      const li = document.createElement('li');
      li.textContent = `🍽️ ${item} x${qty} = ${qty * price} دینار`;
      summaryList.appendChild(li);
    });

    document.getElementById('totalPrice').textContent = total.toLocaleString();
  }

  function saveOrder() {
    const selectedItems = document.querySelectorAll('.item.selected');
    const order = {};
    let message = `📥 Order from Table ${currentTable}:\n`;

    selectedItems.forEach(el => {
      const item = el.textContent;
      const qty = parseInt(el.getAttribute('data-qty')) || 0;
      const price = parseInt(el.getAttribute('data-price')) || 0;
      if (qty > 0) {
        order[item] = { qty, price };
        message += `• ${item} x${qty} = ${qty * price} دینار\n`;
      }
    });

    const total = Object.values(order).reduce((sum, o) => sum + o.qty * o.price, 0);
    message += `\n💰 Total: ${total.toLocaleString()} دینار`;

    tableOrders[currentTable] = order;

    sendToTelegram(message);
    alert(`✅ Order saved and sent for Table ${currentTable}`);

    undoStack = [];

    backToTables();
  }

  function sendToTelegram(message) {
    fetch(`https://api.telegram.org/bot7769917776:AAH3S6oshqdyRjg-P0433hGbaUPSeuoAgTk/sendMessage`, {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify({ chat_id: '1881744939', text: message })
    }).catch(err => console.error('Telegram API error:', err));
  }

  function showAllOrders() {
    const ordersDiv = document.getElementById('orderList');
    ordersDiv.innerHTML = '';

    const keys = Object.keys(tableOrders);
    if (keys.length === 0) {
      ordersDiv.innerHTML = '<p>No orders saved yet.</p>';
    } else {
      keys.forEach(tableNum => {
        const order = tableOrders[tableNum];
        const item = document.createElement('div');
        item.style.marginBottom = '10px';
        const items = Object.entries(order).map(([name, obj]) => `${name} x${obj.qty}`);
        item.innerHTML = `<strong>Table ${tableNum}:</strong> ${items.join(', ')}`;
        ordersDiv.appendChild(item);
      });
    }

    document.getElementById('tables').style.display = 'none';
    document.getElementById('menu').style.display = 'none';
    document.getElementById('allOrders').style.display = 'block';
  }

  function backToTables() {
    document.getElementById('menu').style.display = 'none';
    document.getElementById('tables').style.display = 'grid';
    document.getElementById('allOrders').style.display = 'none';
  }

  function undoLastAction() {
    if (undoStack.length === 0) {
      alert("❌ No action to undo.");
      return;
    }
    const last = undoStack.pop();
    last.el.setAttribute('data-qty', last.prevQty);
    if (last.prevQty === 0) {
      last.el.classList.remove('selected');
    } else {
      last.el.classList.add('selected');
    }
    updateTotal();
  }

  createTables();
</script>

</body>
</html>
