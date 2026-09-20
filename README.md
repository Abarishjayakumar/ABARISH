<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JRA SHOPPING</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #f4f6f8;
            margin: 0;
            padding: 20px;
        }
        .header {
            background-color: #2c3e50;
            color: white;
            padding: 20px;
            text-align: center;
            border-radius: 8px;
        }
        .status-card {
            background-color: white;
            padding: 20px;
            margin: 20px 0;
            border-radius: 8px;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .status-badge {
            font-size: 1.5rem;
            font-weight: bold;
            padding: 8px 16px;
            border-radius: 20px;
            display: inline-block;
        }
        .status-open {
            background-color: #d4edda;
            color: #155724;
        }
        .status-closed {
            background-color: #f8d7da;
            color: #721c24;
        }
        .btn {
            padding: 10px 20px;
            margin: 10px 5px;
            font-size: 1rem;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: 0.2s;
        }
        .btn-open { background-color: #28a745; color: white; }
        .btn-close { background-color: #dc3545; color: white; }
        .btn-buy { background-color: #007bff; color: white; }
        .btn:disabled { background-color: #cccccc; cursor: not-allowed; }
        .products-grid {
            display: flex;
            gap: 20px;
            justify-content: center;
            margin-top: 20px;
        }
        .product-card {
            background: white;
            border-radius: 8px;
            padding: 20px;
            width: 200px;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>

    <div class="header">
        <h1>Welcome to JRA SHOPPING</h1>
    </div>

    <div class="status-card">
        <h2>Store Status: <span id="statusBadge" class="status-badge status-open">OPEN</span></h2>
        <div>
            <button class="btn btn-open" onclick="updateStoreStatus(true)">Open Shop</button>
            <button class="btn btn-close" onclick="updateStoreStatus(false)">Close Shop</button>
        </div>
    </div>

    <h2 style="text-align: center;">Featured Products</h2>
    <div class="products-grid">
        <div class="product-card">
            <h3>Wireless Headphones</h3>
            <p><strong>$49.99</strong></p>
            <button class="btn btn-buy shop-btn" onclick="buyProduct('Wireless Headphones')">Buy Now</button>
        </div>
        <div class="product-card">
            <h3>Smart Watch</h3>
            <p><strong>$99.99</strong></p>
            <button class="btn btn-buy shop-btn" onclick="buyProduct('Smart Watch')">Buy Now</button>
        </div>
    </div>

    <script>
        let isStoreOpen = true;

        function updateStoreStatus(isOpen) {
            isStoreOpen = isOpen;
            const badge = document.getElementById('statusBadge');
            const buyButtons = document.querySelectorAll('.shop-btn');

            if (isStoreOpen) {
                badge.innerText = 'OPEN';
                badge.className = 'status-badge status-open';
                buyButtons.forEach(btn => btn.disabled = false);
            } else {
                badge.innerText = 'CLOSED';
                badge.className = 'status-badge status-closed';
                buyButtons.forEach(btn => btn.disabled = true);
            }
        }

        function buyProduct(productName) {
            if (!isStoreOpen) {
                alert("JRA SHOPPING is currently closed!");
                return;
            }
            alert(`Thank you for purchasing ${productName} from JRA SHOPPING!`);
        }
    </script>

</body>
</html>
