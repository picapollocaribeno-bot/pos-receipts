# pos-receipts
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Receipt</title>
  <style>
    /* 80mm thermal-friendly styles */
    :root { --w: 300px; }
    * { box-sizing: border-box; }
    body {
      font-family: "Courier New", monospace;
      color: #000;
      background: #fff;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
    }
    .paper {
      width: var(--w);
      padding: 10px 12px;
    }
    .center { text-align: center; }
    .right { text-align: right; }
    .small { font-size: 12px; }
    .big { font-size: 14px; font-weight: 700; }
    .divider { border-top: 1px dashed #000; margin: 6px 0; }
    .row { display: flex; gap: 6px; }
    .row > div { flex: 1; }
    .row .qty { width: 20px; flex: 0 0 20px; }
    .row .price { width: 80px; flex: 0 0 80px; text-align: right; }
    img.logo { display:block; margin: 0 auto 6px auto; max-width: 90px; height: auto; }
    button#printBtn {
      width: 100%;
      padding: 8px 10px;
      margin-top: 10px;
      font-family: inherit;
      font-size: 14px;
    }
    @media print {
      button#printBtn, .noprint { display: none !important; }
      body { background: #fff; }
      .paper { box-shadow: none; }
    }
  </style>
</head>
<body>
  <div class="paper">
    <div class="center">
      <!-- Replace the logo URL with your own if desired -->
      <img class="logo" id="logo" alt="Logo" />
      <div class="big" id="company">PICA POLLO CARIBEÑO</div>
      <div class="small" id="slogan">El sabor del Caribe en cada mordida</div>
      <div class="small" id="address"></div>
    </div>
    <div class="divider"></div>

    <div class="small">Order #: <span id="orderId"></span></div>
    <div class="small">Date: <span id="orderDate"></span></div>
    <div class="small">Cashier: <span id="cashier"></span></div>
    <div class="small">Payment: <span id="payment"></span></div>

    <div class="divider"></div>

    <div id="items"></div>

    <div class="divider"></div>

    <div class="row small">
      <div class="label">Subtotal</div>
      <div class="price" id="subtotal">0.00</div>
    </div>
    <div class="row small">
      <div class="label">Tax</div>
      <div class="price" id="tax">0.00</div>
    </div>
    <div class="row small" id="discountRow" style="display:none;">
      <div class="label">Discount</div>
      <div class="price" id="discount">0.00</div>
    </div>
    <div class="row big">
      <div class="label">TOTAL</div>
      <div class="price" id="total">0.00</div>
    </div>

    <div class="divider"></div>

    <div class="center small" id="footer">¡Gracias por su compra!</div>

    <button id="printBtn" onclick="window.print()">🖨️ Imprimir / Print</button>
  </div>

<script>
/**
 * This template reads values from URL parameters so you can pass
 * Adalo Magic Text directly in the WebView URL.
 *
 * Supported parameters:
 *  - logo (URL)
 *  - company, slogan, address, footer
 *  - order_id, order_date, cashier, payment
 *  - subtotal, tax, discount, total (numbers or strings)
 *  - items: Base64-encoded JSON array of { name, qty, price } (price per line)
 *  - auto: if '1', auto calls window.print() on load
 *
 * Example items JSON (before base64):
 *  [ {"name":"Chicken 2 pcs","qty":1,"price":90.00},
 *    {"name":"Boiled Yuca","qty":1,"price":150.00} ]
 */
function getParams() {
  const p = new URLSearchParams(window.location.search);
  const obj = {};
  for (const [k,v] of p.entries()) obj[k] = v;
  return obj;
}
function decodeItems(b64) {
  try {
    const json = atob(b64);
    const arr = JSON.parse(json);
    if (Array.isArray(arr)) return arr;
  } catch (e) {}
  return [];
}
function fmt(x) {
  if (x === undefined || x === null || x === "") return "";
  const n = Number(x);
  if (!isNaN(n)) return n.toFixed(2);
  return String(x);
}

(function init(){
  const p = getParams();

  // Header fields
  const logo = p.logo || "https://i.imgur.com/6wX7ZqG.png";
  const company = p.company || "PICA POLLO CARIBEÑO";
  const slogan = p.slogan || "El sabor del Caribe en cada mordida";
  const address = p.address || "";
  const footer = p.footer || "¡Gracias por su compra!";

  document.getElementById("logo").src = logo;
  document.getElementById("company").textContent = company;
  document.getElementById("slogan").textContent = slogan;
  document.getElementById("address").textContent = address;
  document.getElementById("footer").textContent = footer;

  // Order meta
  document.getElementById("orderId").textContent = p.order_id || "";
  document.getElementById("orderDate").textContent = p.order_date || "";
  document.getElementById("cashier").textContent = p.cashier || "";
  document.getElementById("payment").textContent = p.payment || "";

  // Totals
  document.getElementById("subtotal").textContent = fmt(p.subtotal);
  document.getElementById("tax").textContent = fmt(p.tax);
  if (p.discount && Number(p.discount) > 0) {
    document.getElementById("discount").textContent = fmt(p.discount);
    document.getElementById("discountRow").style.display = "";
  }
  document.getElementById("total").textContent = fmt(p.total);

  // Items
  const itemsEl = document.getElementById("items");
  let rows = [];
  const items = p.items ? decodeItems(p.items) : [];
  if (items.length) {
    items.forEach(it => {
      const name = it.name || "";
      const qty = it.qty != null ? String(it.qty) : "";
      const price = it.price != null ? Number(it.price) : 0;
      rows.push(`
        <div class="row small">
          <div class="qty">${qty}</div>
          <div class="name">${name}</div>
          <div class="price">${fmt(price)}</div>
        </div>
      `);
    });
  } else {
    // Fallback single row if no items provided
    rows.push(`<div class="small">Items not provided</div>`);
  }
  itemsEl.innerHTML = rows.join("");

  if (p.auto === "1") {
    setTimeout(() => window.print(), 300);
  }
})();
</script>
</body>
</html>
