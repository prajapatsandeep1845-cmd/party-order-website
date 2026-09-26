const CATEGORIES = [
  { id: 'earings', name: 'Earings', icon: '💎' },
  { id: 'tops', name: 'Tops', icon: '✨' },
  { id: 'kalanki', name: 'Kalanki', icon: '🌸' },
  { id: 'groom-necklace', name: 'Groom Necklace', icon: '📿' }
];

const STORE_KEY = 'party-order-v1';
const PRODUCT_ICONS = ['💎', '✨', '🌸', '📿', '💠', '🌟', '🪷', '🔶'];

let state = {
  user: null,
  shop: null,
  category: 'earings',
  cart: [],
  records: []
};

function createCatalog() {
  const catalog = [];
  CATEGORIES.forEach((category, categoryIndex) => {
    for (let i = 1; i <= 100; i += 1) {
      catalog.push({
        id: `${category.id}-${i}`,
        category: category.id,
        name: `${category.name} Design ${String(i).padStart(3, '0')}`,
        price: [249, 299, 349, 399][(i + categoryIndex) % 4],
        icon: PRODUCT_ICONS[(i + categoryIndex) % PRODUCT_ICONS.length]
      });
    }
  });
  return catalog;
}

const PRODUCT_CATALOG = createCatalog();

function loadState() {
  try {
    state.user = JSON.parse(localStorage.getItem(`${STORE_KEY}-user`)) || null;
    state.shop = JSON.parse(localStorage.getItem(`${STORE_KEY}-shop`)) || null;
    state.records = JSON.parse(localStorage.getItem(`${STORE_KEY}-records`)) || [];
  } catch (error) {
    state.user = null;
    state.shop = null;
    state.records = [];
  }
}

function saveState() {
  localStorage.setItem(`${STORE_KEY}-user`, JSON.stringify(state.user));
  localStorage.setItem(`${STORE_KEY}-shop`, JSON.stringify(state.shop));
  localStorage.setItem(`${STORE_KEY}-records`, JSON.stringify(state.records));
}

function currency(value) {
  return `₹${Number(value || 0).toLocaleString('en-IN', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  })}`;
}

function escapeHtml(value) {
  return String(value ?? '')
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

function getCartTotals() {
  const subtotal = state.cart.reduce((total, item) => total + (item.price * item.qty), 0);
  const gst = subtotal * 0.03;
  const total = subtotal + gst;
  return { subtotal, gst, total };
}

function renderAuthForm() {
  document.querySelector('#app').innerHTML = `
    <main class="auth-page">
      <section class="auth-card">
        <div class="brand">
          <span class="brand-mark">P</span>
          PartyOrder
        </div>
        <h1>Welcome back</h1>
        <p class="muted">Sign in to create and manage your jewelry orders.</p>

        <form id="login-form">
          <div class="field">
            <label>Email or username</label>
            <input name="identity" type="text" placeholder="owner@example.com" required>
          </div>
          <div class="field">
            <label>Password</label>
            <input name="password" type="password" placeholder="••••••••" required minlength="4">
          </div>
          <div id="login-error" class="error"></div>
          <button class="primary" type="submit" style="width:100%">Sign in</button>
        </form>
      </section>
    </main>
  `;
}

function renderShopSetupForm() {
  document.querySelector('#app').innerHTML = `
    <main class="setup-wrap">
      <section class="setup-card">
        <div class="brand">
          <span class="brand-mark">P</span>
          PartyOrder
        </div>
        <h1>Set up your shop</h1>
        <p class="muted">This information will be used on each invoice.</p>

        <form id="shop-form">
          <div class="field">
            <label>Shop name</label>
            <input name="shopName" type="text" maxlength="80" placeholder="Shree Fashion Store" required>
          </div>
          <div class="field">
            <label>Mobile number</label>
            <input name="mobile" type="tel" maxlength="15" placeholder="9876543210" required>
          </div>
          <div id="shop-error" class="error"></div>
          <button class="primary" type="submit" style="width:100%">Save shop and continue</button>
        </form>
      </section>
    </main>
  `;
}

function renderDashboard() {
  const categoryProducts = PRODUCT_CATALOG.filter((p) => p.category === state.category);
  const { subtotal, gst, total } = getCartTotals();

  document.querySelector('#app').innerHTML = `
    <div class="app-shell">
      <header class="topbar">
        <div class="brand">
          <span class="brand-mark">P</span>
          PartyOrder
        </div>
        <div class="top-actions">
          <span class="shop-pill">${escapeHtml(state.shop.name)}</span>
          <button class="logout" id="logout-btn" type="button">Sign out</button>
        </div>
      </header>

      <main class="content">
        <div class="welcome">
          <div>
            <h1>Good day, ${escapeHtml(state.shop.name)} 👋</h1>
            <p class="muted">Select products and create your party order.</p>
          </div>
          <button class="secondary" id="clear-order-btn" type="button">Clear order</button>
        </div>

        <nav class="tabs">
          ${CATEGORIES.map((category) => `
            <button
              class="tab ${category.id === state.category ? 'active' : ''}"
              type="button"
              data-category="${category.id}"
            >
              ${category.icon} ${category.name}
            </button>
          `).join('')}
        </nav>

        <div class="layout">
          <section>
            <div class="catalog">
              ${categoryProducts.map((product) => `
                <article class="product-card">
                  <div class="product-art">${product.icon}</div>
                  <div class="product-body">
                    <h3>${escapeHtml(product.name)}</h3>
                    <div class="price">${currency(product.price)}</div>
                    <button class="add" type="button" data-add-product="${product.id}">Add to cart</button>
                  </div>
                </article>
              `).join('')}
            </div>
          </section>

          <aside class="cart">
            <h2>Current order <span class="badge">${state.cart.length} items</span></h2>

            ${state.cart.length === 0 ? `
              <div class="cart-empty">
                Your cart is empty.<br>
                Click “Add to cart” to begin.
              </div>
            ` : state.cart.map((item, index) => `
              <div class="cart-row">
                <div class="row-head">
                  <strong>${escapeHtml(item.name)}</strong>
                  <button class="remove" type="button" data-remove-item="${index}">Remove</button>
                </div>
                <div class="muted">${currency(item.price)} each</div>
                <div class="options">
                  <input type="number" data-cart-index="${index}" data-field="qty" min="1" value="${item.qty}">
                  <input type="text" data-cart-index="${index}" data-field="color" placeholder="Color" value="${escapeHtml(item.color || '')}">
                  <select data-cart-index="${index}" data-field="setting">
                    <option value="Standard" ${item.setting === 'Standard' ? 'selected' : ''}>Standard</option>
                    <option value="Gold" ${item.setting === 'Gold' ? 'selected' : ''}>Gold</option>
                    <option value="Silver" ${item.setting === 'Silver' ? 'selected' : ''}>Silver</option>
                    <option value="Rose gold" ${item.setting === 'Rose gold' ? 'selected' : ''}>Rose gold</option>
                  </select>
                </div>
              </div>
            `).join('')}

            <div class="totals">
              <div class="total-line">
                <span>Subtotal</span>
                <strong>${currency(subtotal)}</strong>
              </div>
              <div class="total-line">
                <span>GST (3%)</span>
                <strong>${currency(gst)}</strong>
              </div>
              <div class="total-line grand">
                <span>Total</span>
                <strong>${currency(total)}</strong>
              </div>
            </div>

            <div class="cart-actions">
              <button class="secondary" id="print-invoice-btn" type="button" ${state.cart.length === 0 ? 'disabled' : ''}>Print invoice</button>
              <button class="primary" id="save-order-btn" type="button" ${state.cart.length === 0 ? 'disabled' : ''}>Save customer</button>
            </div>
          </aside>
        </div>

        <section class="records">
          <h2>Saved customers / invoices</h2>
          ${state.records.length === 0 ? `
            <p class="muted">Saved invoices will appear here after you save an order.</p>
          ` : state.records.map((record) => `
            <div class="record">
              <div>
                <strong>${escapeHtml(record.shop)}</strong>
                <span class="muted">${record.date} · ${record.items} products · ${currency(record.total)}</span>
              </div>
              <span class="badge">Saved</span>
            </div>
          `).join('')}
        </section>
      </main>
    </div>
  `;
}

function addToCart(productId) {
  const product = PRODUCT_CATALOG.find((item) => item.id === productId);
  if (!product) return;

  const existing = state.cart.find((item) => item.id === productId);
  if (existing) {
    existing.qty += 1;
  } else {
    state.cart.push({
      ...product,
      qty: 1,
      color: '',
      setting: 'Standard'
    });
  }

  render();
}

function removeFromCart(index) {
  state.cart.splice(index, 1);
  render();
}

function updateCartValue(index, field, value) {
  const item = state.cart[index];
  if (!item) return;

  if (field === 'qty') {
    item.qty = Math.max(1, Number(value) || 1);
  } else {
    item[field] = value;
  }

  render();
}

function saveOrder() {
  if (!state.cart.length) return;

  const { total } = getCartTotals();
  const order = {
    shop: state.shop.name,
    mobile: state.shop.mobile,
    date: new Date().toLocaleString('en-IN'),
    items: state.cart.reduce((sum, item) => sum + item.qty, 0),
    total: Number(total.toFixed(2))
  };

  state.records.push(order);
  saveState();

  const printWindow = window.open('', '_blank');
  if (printWindow) {
    printWindow.document.write(`
      <html>
        <head>
          <title>Invoice - ${escapeHtml(state.shop.name)}</title>
          <style>
            body { font-family: Arial, sans-serif; padding: 28px; color: #111; }
            .header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 10px; margin-bottom: 18px; }
            .title { font-size: 28px; font-weight: 700; }
            .meta { margin: 12px 0; line-height: 1.7; }
            table { width: 100%; border-collapse: collapse; margin-top: 20px; }
            th, td { padding: 10px; border: 1px solid #ddd; text-align: left; }
            .right { text-align: right; }
            .total { font-weight: 700; font-size: 18px; }
          </style>
        </head>
        <body>
          <div class="header">
            <div class="title">${escapeHtml(state.shop.name)}</div>
            <div>Invoice</div>
          </div>

          <div class="meta">
            <div><strong>Mobile:</strong> ${escapeHtml(state.shop.mobile)}</div>
            <div><strong>Date:</strong> ${escapeHtml(order.date)}</div>
          </div>

          <table>
            <thead>
              <tr>
                <th>Product</th>
                <th>Qty</th>
                <th>Color</th>
                <th>Setting</th>
                <th>Price</th>
                <th>Total</th>
              </tr>
            </thead>
            <tbody>
              ${state.cart.map((item) => `
                <tr>
                  <td>${escapeHtml(item.name)}</td>
                  <td>${item.qty}</td>
                  <td>${escapeHtml(item.color || 'N/A')}</td>
                  <td>${escapeHtml(item.setting || 'Standard')}</td>
                  <td>${currency(item.price)}</td>
                  <td>${currency(item.price * item.qty)}</td>
                </tr>
              `).join('')}
            </tbody>
          </table>

          <div style="margin-top: 20px; width: 260px; margin-left: auto;">
            <div><span>Subtotal:</span> <span class="right">${currency(getCartTotals().subtotal)}</span></div>
            <div><span>GST (3%):</span> <span class="right">${currency(getCartTotals().gst)}</span></div>
            <div class="total"><span>Total:</span> <span class="right">${currency(getCartTotals().total)}</span></div>
          </div>
        </body>
      </html>
    `);
    printWindow.document.close();
    printWindow.focus();
    setTimeout(() => printWindow.print(), 300);
  }

  state.cart = [];
  render();
}

function render() {
  if (!state.user) {
    renderAuthForm();
    return;
  }

  if (!state.shop) {
    renderShopSetupForm();
    return;
  }

  renderDashboard();
}

function handleSubmit(event) {
  if (event.target.id === 'login-form') {
    event.preventDefault();
    const form = new FormData(event.target);
    const identity = String(form.get('identity') || '').trim();
    const password = String(form.get('password') || '');

    if (password.length < 4) {
      document.querySelector('#login-error').textContent = 'Password must be at least 4 characters.';
      return;
    }

    state.user = { identity, password };
    saveState();
    render();
  }

  if (event.target.id === 'shop-form') {
    event.preventDefault();
    const form = new FormData(event.target);
    const shopName = String(form.get('shopName') || '').trim();
    const mobile = String(form.get('mobile') || '').replace(/\D/g, '');

    if (!shopName) {
      document.querySelector('#shop-error').textContent = 'Please enter your shop name.';
      return;
    }

    if (mobile.length < 7) {
      document.querySelector('#shop-error').textContent = 'Enter a valid mobile number.';
      return;
    }

    state.shop = {
      name: shopName,
      mobile
    };
    saveState();
    render();
  }
}

function handleClick(event) {
  const categoryButton = event.target.closest('[data-category]');
  if (categoryButton) {
    state.category = categoryButton.dataset.category;
    render();
    return;
  }

  const addProductButton = event.target.closest('[data-add-product]');
  if (addProductButton) {
    addToCart(addProductButton.dataset.addProduct);
    return;
  }

  const removeButton = event.target.closest('[data-remove-item]');
  if (removeButton) {
    removeFromCart(Number(removeButton.dataset.removeItem));
    return;
  }

  if (event.target.id === 'logout-btn') {
    state.user = null;
    state.shop = null;
    state.cart = [];
    saveState();
    render();
    return;
  }

  if (event.target.id === 'clear-order-btn') {
    state.cart = [];
    render();
    return;
  }

  if (event.target.id === 'print-invoice-btn') {
    saveOrder();
    return;
  }

  if (event.target.id === 'save-order-btn') {
    saveOrder();
  }
}

function handleInput(event) {
  const field = event.target.dataset.field;
  const cartIndex = event.target.dataset.cartIndex;

  if (field && cartIndex !== undefined) {
    const index = Number(cartIndex);
    const value = event.target.type === 'number' ? event.target.value : event.target.value;
    updateCartValue(index, field, value);
  }
}

document.addEventListener('DOMContentLoaded', () => {
  loadState();
  render();

  document.addEventListener('submit', handleSubmit);
  document.addEventListener('click', handleClick);
  document.addEventListener('input', handleInput);
  document.addEventListener('change', handleInput);
});

window.addEventListener('beforeunload', () => {
  saveState();
});

