<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>JRA SHOPPING</title>
  <style>
    :root {
      --primary: #1a73e8;
      --success: #2ea44f;
      --danger: #cb2431;
      --bg: #f6f8fa;
      --card-bg: #ffffff;
      --text: #24292e;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    }

    body {
      background-color: var(--bg);
      color: var(--text);
      line-height: 1.5;
    }

    header {
      background-color: #0f172a;
      color: white;
      padding: 1rem 2rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    }

    .brand {
      display: flex;
      align-items: center;
      gap: 0.75rem;
    }

    .brand h1 {
      font-size: 1.5rem;
      letter-spacing: 1px;
    }

    .status-badge {
      padding: 0.25rem 0.75rem;
      border-radius: 999px;
      font-weight: bold;
      font-size: 0.85rem;
      text-transform: uppercase;
    }

    .status-open {
      background-color: #dcfce7;
      color: #15803d;
    }

    .status-closed {
      background-color: #fee2e2;
      color: #b91c1c;
    }

    .container {
      max-width: 1200px;
      margin: 2rem auto;
      padding: 0 1rem;
      display: grid;
      grid-template-columns: 2fr 1fr;
      gap: 2rem;
    }

    .products-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
      gap: 1.5rem;
    }

    .card {
      background: var(--card-bg);
      border: 1px solid #e1e4e8;
      border-radius: 8px;
      padding: 1rem;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      box-shadow: 0 1px 3px rgba(0,0,0,0.05);
    }

    .card img {
      width: 100%;
      height: 140px;
      object-fit: cover;
      border-radius: 4px;
      margin-bottom: 0.75rem;
    }

    .card h3 {
      font-size: 1.1rem;
      margin-bottom: 0.5rem;
    }

    .card .price {
      font-size: 1.2rem;
      font-weight: bold;
      color: var(--primary);
      margin-bottom: 1rem;
    }

    button {
      background-color: var(--primary);
      color: white;
      border: none;
      padding: 0.6rem 1rem;
      border-radius: 6px;
      cursor: pointer;
      font-weight: 600;
      transition: background 0.2s;
    }

    button:hover {
      opacity: 0.9;
    }

    button:disabled {
      background-color: #94a3b8;
      cursor: not-allowed;
    }

    .cart-section, .admin-section {
      background: var(--card-bg);
      border: 1px solid #e1e4e8;
      border-radius: 8px;
      padding: 1.5rem;
      margin-bottom: 1.5rem;
    }

    .cart-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0.75rem 0;
      border-bottom: 1px solid #f0f0f0;
    }

    .cart-total {
      margin-top: 1rem;
      font-size: 1.25rem;
      font-weight: bold;
      display: flex;
      justify-content: space-between;
    }

    .toggle-btn {
      width: 100%;
      padding: 0.75rem;
      margin-top: 0.5rem;
    }

    .btn-open { background-color: var(--success); }
    .btn-close { background-color: var(--danger); }

    .banner-closed {
      grid-column: 1 / -1;
      background-color: #fef2f2;
      border: 1px solid #fecaca;
      color: #991b1b;
      padding: 1rem;
      border-radius: 8px;
      text-align: center;
      font-weight: 500;
    }
  </style>
</head>
<body>

  <header>
    <div class="brand">
      <h1>JRA SHOPPING</h1>
    </div>
    <div id="statusBadge" class="status-badge status-open">Store Open</div>
  </header>

  <div class="container">
    <main>
      <div id="closedNotice" class="banner-closed" style="display: none;">
        🏬 <strong>JRA SHOPPING</strong> is currently closed. Ordering is unavailable.
      </div>
      <h2 style="margin-bottom: 1rem;">Products</h2>
      <div class="products-grid" id="productList"></div>
    </main>

    <aside>
      <div class="cart-section">
        <h2>Your Cart</h2>
        <div id="cartItems" style="margin-top: 1rem;"></div>
        <div class="cart-total">
          <span>Total:</span>
          <span id="cartTotal">$0.00</span>
        </div>
        <button id="checkoutBtn" style="width: 100%; margin-top: 1rem;" onclick="checkout()">Checkout</button>
      </div>

      <div class="admin-section">
        <h3>Store Admin Panel</h3>
        <p style="font-size: 0.85rem; color: #666; margin-bottom: 0.75rem;">
          Toggle operational status for JRA SHOPPING:
        </p>
        <button id="toggleStoreBtn" class="toggle-btn btn-close" onclick="toggleStoreStatus()">
          Close Store
        </button>
      </div>
    </aside>
  </div>

  <script>
    let isStoreOpen = true;
    let cart = {};

    const products = [
      { id: 1, name: "Wireless Headphones", price: 99.99, img: "https://picsum.photos/id/1/200/140" },
      { id: 2, name: "Smart Watch", price: 149.50, img: "https://picsum.photos/id/2/200/140" },
      { id: 3, name: "Mechanical Keyboard", price: 85.00, img: "https://picsum.photos/id/3/200/140" },
      { id: 4, name: "Gaming Mouse", price: 45.25, img: "https://picsum.photos/id/4/200/140" }
    ];

    async function syncStoreStatus() {
      try {
        const response = await fetch('/api/status');
        const data = await response.json();
        isStoreOpen = data.isOpen;
        updateUI();
      } catch (err) {
        console.warn("Server backend offline, using local state.");
      }
    }

    function renderProducts() {
      const container = document.getElementById("productList");
      container.innerHTML = products.map(p => `
        <div class="card">
          <div>
            <img src="${p.img}" alt="${p.name}">
            <h3>${p.name}</h3>
            <div class="price">$${p.price.toFixed(2)}</div>
          </div>
          <button 
            onclick="addToCart(${p.id})" 
            ${!isStoreOpen ? 'disabled' : ''}>
            Add to Cart
          </button>
        </div>
      `).join('');
    }

    function addToCart(productId) {
      if (!isStoreOpen) return;
      cart[productId] = (cart[productId] || 0) + 1;
      renderCart();
    }

    function renderCart() {
      const cartContainer = document.getElementById("cartItems");
      let total = 0;
      let html = "";

      const keys = Object.keys(cart);
      if (keys.length === 0) {
        cartContainer.innerHTML = "<p style='color: #888;'>Cart is empty.</p>";
        document.getElementById("cartTotal").innerText = "$0.00";
        return;
      }

      keys.forEach(id => {
        const product = products.find(p => p.id == id);
        const qty = cart[id];
        const subtotal = product.price * qty;
        total += subtotal;

        html += `
          <div class="cart-item">
            <div>
              <strong>${product.name}</strong><br>
              <small>$${product.price.toFixed(2)} x ${qty}</small>
            </div>
            <div>
              <strong>$${subtotal.toFixed(2)}</strong>
            </div>
          </div>
        `;
      });

      cartContainer.innerHTML = html;
      document.getElementById("cartTotal").innerText = `$${total.toFixed(2)}`;
    }

    function updateUI() {
      const badge = document.getElementById("statusBadge");
      const notice = document.getElementById("closedNotice");
      const toggleBtn = document.getElementById("toggleStoreBtn");
      const checkoutBtn = document.getElementById("checkoutBtn");

      if (isStoreOpen) {
        badge.innerText = "Store Open";
        badge.className = "status-badge status-open";
        notice.style.display = "none";
        toggleBtn.innerText = "Close Store";
        toggleBtn.className = "toggle-btn btn-close";
        checkoutBtn.disabled = false;
      } else {
        badge.innerText = "Store Closed";
        badge.className = "status-badge status-closed";
        notice.style.display = "block";
        toggleBtn.innerText = "Open Store";
        toggleBtn.className = "toggle-btn btn-open";
        checkoutBtn.disabled = true;
      }

      renderProducts();
    }

    async function toggleStoreStatus() {
      try {
        const response = await fetch('/api/toggle', { method: 'POST' });
        const data = await response.json();
        isStoreOpen = data.isOpen;
      } catch (err) {
        isStoreOpen = !isStoreOpen;
      }
      updateUI();
    }

    function checkout() {
      if (!isStoreOpen) {
        alert("JRA SHOPPING is currently closed.");
        return;
      }
      if (Object.keys(cart).length === 0) {
        alert("Your cart is empty!");
        return;
      }
      alert("Thank you for shopping with JRA SHOPPING! Your order has been placed.");
      cart = {};
      renderCart();
    }

    // Initial Setup
    syncStoreStatus();
    renderProducts();
    renderCart();
  </script>
</body>
</html>
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;

public class JraShoppingServer {

    private static final int PORT = 8080;
    private static boolean isStoreOpen = true;

    public static void main(String[] args) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(PORT), 0);

        // API & Static Resource Mapping
        server.createContext("/", new StaticFileHandler());
        server.createContext("/api/status", new StatusHandler());
        server.createContext("/api/toggle", new ToggleHandler());

        server.setExecutor(null);
        System.out.println("=========================================");
        System.out.println("  JRA SHOPPING SERVER RUNNING");
        System.out.println("  Access URL: http://localhost:" + PORT);
        System.out.println("=========================================");
        server.start();
    }

    // Handler to serve index.html
    static class StaticFileHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            File file = new File("index.html");
            if (!file.exists()) {
                String response = "404 Not Found: Ensure index.html is in the same directory.";
                exchange.sendResponseHeaders(404, response.length());
                try (OutputStream os = exchange.getResponseBody()) {
                    os.write(response.getBytes());
                }
                return;
            }

            exchange.getResponseHeaders().set("Content-Type", "text/html; charset=UTF-8");
            exchange.sendResponseHeaders(200, file.length());
            try (FileInputStream fs = new FileInputStream(file);
                 OutputStream os = exchange.getResponseBody()) {
                byte[] buffer = new byte[1024];
                int count;
                while ((count = fs.read(buffer)) >= 0) {
                    os.write(buffer, 0, count);
                }
            }
        }
    }

    // Handler for fetching store state
    static class StatusHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            String response = "{\"store\": \"JRA SHOPPING\", \"isOpen\": " + isStoreOpen + "}";
            sendJsonResponse(exchange, 200, response);
        }
    }

    // Handler for opening/closing store
    static class ToggleHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            if ("POST".equalsIgnoreCase(exchange.getRequestMethod())) {
                isStoreOpen = !isStoreOpen;
                String response = "{\"message\": \"Store status updated\", \"isOpen\": " + isStoreOpen + "}";
                sendJsonResponse(exchange, 200, response);
            } else {
                sendJsonResponse(exchange, 405, "{\"error\": \"Method not allowed. Use POST.\"}");
            }
        }
    }

    private static void sendJsonResponse(HttpExchange exchange, int statusCode, String response) throws IOException {
        exchange.getResponseHeaders().set("Content-Type", "application/json");
        byte[] bytes = response.getBytes(StandardCharsets.UTF_8);
        exchange.sendResponseHeaders(statusCode, bytes.length);
        try (OutputStream os = exchange.getResponseBody()) {
            os.write(bytes);
        }
    }
}
