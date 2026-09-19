# ABARISH
shopping is available with cash on delivery
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpServer;

import java.io.*;
import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;
import java.util.*;

public class JRAShops {

    static List<Product> products = new ArrayList<>();
    static List<CartItem> cart = new ArrayList<>();

    public static void main(String[] args) throws Exception {

        // Sample products
        products.add(new Product(
                1,
                "Smart Watch",
                "Bluetooth Smart Watch",
                1499,
                20,
                "https://via.placeholder.com/250"
        ));

        products.add(new Product(
                2,
                "Wireless Headphones",
                "High quality wireless headphones",
                1999,
                15,
                "https://via.placeholder.com/250"
        ));

        HttpServer server =
                HttpServer.create(
                        new InetSocketAddress(8080),
                        0
                );

        server.createContext("/api/products",
                JRAShops::products);

        server.createContext("/api/add-product",
                JRAShops::addProduct);

        server.createContext("/api/cart",
                JRAShops::cart);

        server.createContext("/api/order",
                JRAShops::order);

        server.start();

        System.out.println("--------------------------------");
        System.out.println("       JRA SHOPS");
        System.out.println("--------------------------------");
        System.out.println(
                "Server running at http://localhost:8080"
        );
    }

    // GET products
    static void products(HttpExchange exchange)
            throws IOException {

        if (!exchange.getRequestMethod().equals("GET")) {
            send(exchange, "Method Not Allowed", 405);
            return;
        }

        StringBuilder json = new StringBuilder("[");
        boolean first = true;

        for (Product p : products) {

            if (!first) {
                json.append(",");
            }

            json.append(p.toJson());
            first = false;
        }

        json.append("]");

        sendJson(exchange, json.toString());
    }

    // Add new product
    static void addProduct(HttpExchange exchange)
            throws IOException {

        if (!exchange.getRequestMethod().equals("POST")) {
            send(exchange, "Method Not Allowed", 405);
            return;
        }

        String body = readBody(exchange);

        try {

            String name =
                    getValue(body, "name");

            String description =
                    getValue(body, "description");

            double price =
                    Double.parseDouble(
                            getValue(body, "price")
                    );

            int stock =
                    Integer.parseInt(
                            getValue(body, "stock")
                    );

            String image =
                    getValue(body, "image")
                            .replace("\\/", "/");

            int id = products.size() + 1;

            Product product =
                    new Product(
                            id,
                            name,
                            description,
                            price,
                            stock,
                            image
                    );

            products.add(product);

            sendJson(
                    exchange,
                    "{\"message\":\"Product added successfully\"}"
            );

        } catch (Exception e) {

            sendJson(
                    exchange,
                    "{\"message\":\"Invalid product data\"}"
            );
        }
    }

    // Cart
    static void cart(HttpExchange exchange)
            throws IOException {

        if (exchange.getRequestMethod().equals("POST")) {

            String body = readBody(exchange);

            try {

                int productId =
                        Integer.parseInt(
                                getValue(body, "productId")
                        );

                int quantity =
                        Integer.parseInt(
                                getValue(body, "quantity")
                        );

                Product product = null;

                for (Product p : products) {

                    if (p.id == productId) {
                        product = p;
                        break;
                    }
                }

                if (product == null) {

                    sendJson(
                            exchange,
                            "{\"message\":\"Product not found\"}"
                    );

                    return;
                }

                cart.add(
                        new CartItem(
                                product,
                                quantity
                        )
                );

                sendJson(
                        exchange,
                        "{\"message\":\"Added to cart\"}"
                );

            } catch (Exception e) {

                sendJson(
                        exchange,
                        "{\"message\":\"Invalid request\"}"
                );
            }

        } else {

            StringBuilder json =
                    new StringBuilder("[");

            boolean first = true;

            for (CartItem item : cart) {

                if (!first) {
                    json.append(",");
                }

                json.append(item.toJson());

                first = false;
            }

            json.append("]");

            sendJson(exchange, json.toString());
        }
    }

    // Order
    static void order(HttpExchange exchange)
            throws IOException {

        if (!exchange.getRequestMethod().equals("POST")) {

            send(exchange, "Method Not Allowed", 405);

            return;
        }

        if (cart.isEmpty()) {

            sendJson(
                    exchange,
                    "{\"message\":\"Cart is empty\"}"
            );

            return;
        }

        double total = 0;

        for (CartItem item : cart) {

            total +=
                    item.product.price *
                    item.quantity;
        }

        cart.clear();

        sendJson(
                exchange,
                "{\"message\":\"Order placed successfully\","
                        + "\"total\":" + total + "}"
        );
    }

    static String readBody(HttpExchange exchange)
            throws IOException {

        InputStream input =
                exchange.getRequestBody();

        return new String(
                input.readAllBytes(),
                StandardCharsets.UTF_8
        );
    }

    static String getValue(
            String json,
            String key) {

        String search =
                "\"" + key + "\"";

        int start =
                json.indexOf(search);

        if (start == -1)
            return "";

        start =
                json.indexOf(":",
                        start) + 1;

        while (
                start < json.length()
                        &&
                (json.charAt(start) == ' '
                        ||
                 json.charAt(start) == '"')
        ) {
            start++;
        }

        int end;

        if (json.charAt(start - 1) == '"') {

            end =
                    json.indexOf(
                            "\"",
                            start
                    );

        } else {

            end =
                    json.indexOf(
                            ",",
                            start
                    );

            if (end == -1) {
                end =
                        json.indexOf(
                                "}",
                                start
                        );
            }
        }

        return json.substring(
                start,
                end
        ).trim();
    }

    static void sendJson(
            HttpExchange exchange,
            String response)
            throws IOException {

        exchange.getResponseHeaders()
                .set(
                        "Content-Type",
                        "application/json"
                );

        send(exchange, response, 200);
    }

    static void send(
            HttpExchange exchange,
            String response,
            int code)
            throws IOException {

        byte[] data =
                response.getBytes(
                        StandardCharsets.UTF_8
                );

        exchange.sendResponseHeaders(
                code,
                data.length
        );

        OutputStream output =
                exchange.getResponseBody();

        output.write(data);
        output.close();
    }

    // Product class
    static class Product {

        int id;
        String name;
        String description;
        double price;
        int stock;
        String image;

        Product(
                int id,
                String name,
                String description,
                double price,
                int stock,
                String image) {

            this.id = id;
            this.name = name;
            this.description = description;
            this.price = price;
            this.stock = stock;
            this.image = image;
        }

        String toJson() {

            return "{"
                    + "\"id\":" + id + ","
                    + "\"name\":\"" + escape(name) + "\","
                    + "\"description\":\""
                    + escape(description) + "\","
                    + "\"price\":" + price + ","
                    + "\"stock\":" + stock + ","
                    + "\"image\":\""
                    + escape(image) + "\""
                    + "}";
        }
    }

    static class CartItem {

        Product product;
        int quantity;

        CartItem(
                Product product,
                int quantity) {

            this.product = product;
            this.quantity = quantity;
        }

        String toJson() {

            return "{"
                    + "\"productId\":"
                    + product.id + ","
                    + "\"name\":\""
                    + escape(product.name) + "\","
                    + "\"price\":"
                    + product.price + ","
                    + "\"quantity\":"
                    + quantity
                    + "}";
        }
    }

    static String escape(String text) {

        return text
                .replace("\\", "\\\\")
                .replace("\"", "\\\"");
    }
}


<!DOCTYPE html>
<html>

<head>
    <title>JRA SHOPS</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>

<header>

    <h1>JRA SHOPS</h1>

    <nav>
        <a href="index.html">Home</a>
        <a href="products.html">Products</a>
        <a href="admin.html">Sell Product</a>
        <a href="cart.html">Cart</a>
    </nav>

</header>

<section class="hero">

    <h2>Welcome to JRA SHOPS</h2>

    <p>
        Your trusted online shopping store
    </p>

    <a href="products.html">
        <button>Shop Now</button>
    </a>

</section>

<footer>
    © 2026 JRA SHOPS
</footer>

</body>

</html>


<!DOCTYPE html>
<html>

<head>

    <title>Products - JRA SHOPS</title>

    <link rel="stylesheet" href="style.css">

</head>

<body>

<header>

    <h1>JRA SHOPS</h1>

    <nav>

        <a href="index.html">
            Home
        </a>

        <a href="products.html">
            Products
        </a>

        <a href="admin.html">
            Sell Product
        </a>

        <a href="cart.html">
            Cart
        </a>

    </nav>

</header>

<h2 class="title">
    Our Products
</h2>

<div id="products"
     class="products">

    Loading products...

</div>

<script>

async function loadProducts() {

    const response =
        await fetch(
            "http://localhost:8080/api/products"
        );

    const products =
        await response.json();

    const container =
        document.getElementById(
            "products"
        );

    container.innerHTML = "";

    products.forEach(product => {

        container.innerHTML += `

            <div class="card">

                <img
                    src="${product.image}"
                    alt="${product.name}"
                >

                <h2>
                    ${product.name}
                </h2>

                <p>
                    ${product.description}
                </p>

                <h3>
                    ₹${product.price}
                </h3>

                <p>
                    Stock: ${product.stock}
                </p>

                <button
                    onclick="addToCart(${product.id})">

                    Add to Cart

                </button>

            </div>

        `;
    });
}

async function addToCart(id) {

    const response =
        await fetch(
            "http://localhost:8080/api/cart",
            {

                method: "POST",

                headers: {
                    "Content-Type":
                        "application/json"
                },

                body: JSON.stringify({

                    productId: id,

                    quantity: 1

                })

            }
        );

    const result =
        await response.json();

    alert(result.message);
}

loadProducts();

</script>

</body>

</html>


<!DOCTYPE html>
<html>

<head>

    <title>Sell Product - JRA SHOPS</title>

    <link rel="stylesheet"
          href="style.css">

</head>

<body>

<header>

    <h1>JRA SHOPS</h1>

    <nav>

        <a href="index.html">
            Home
        </a>

        <a href="products.html">
            Products
        </a>

        <a href="cart.html">
            Cart
        </a>

    </nav>

</header>

<div class="form">

    <h2>
        Add Product for Sale
    </h2>

    <input
        id="name"
        placeholder="Product Name"
    >

    <textarea
        id="description"
        placeholder="Product Description">
    </textarea>

    <input
        id="price"
        type="number"
        placeholder="Price ₹"
    >

    <input
        id="stock"
        type="number"
        placeholder="Stock Quantity"
    >

    <input
        id="image"
        placeholder="Product Image URL"
    >

    <button onclick="addProduct()">
        Publish Product
    </button>

</div>

<script>

async function addProduct() {

    const product = {

        name:
            document.getElementById(
                "name"
            ).value,

        description:
            document.getElementById(
                "description"
            ).value,

        price:
            Number(
                document.getElementById(
                    "price"
                ).value
            ),

        stock:
            Number(
                document.getElementById(
                    "stock"
                ).value
            ),

        image:
            document.getElementById(
                "image"
            ).value
    };

    const response =
        await fetch(
            "http://localhost:8080/api/add-product",
            {

                method: "POST",

                headers: {
                    "Content-Type":
                        "application/json"
                },

                body:
                    JSON.stringify(product)
            }
        );

    const result =
        await response.json();

    alert(result.message);

}

</script>

</body>

</html>

<!DOCTYPE html>
<html>

<head>

    <title>Cart - JRA SHOPS</title>

    <link rel="stylesheet"
          href="style.css">

</head>

<body>

<header>

    <h1>JRA SHOPS</h1>

    <nav>

        <a href="index.html">
            Home
        </a>

        <a href="products.html">
            Products
        </a>

        <a href="cart.html">
            Cart
        </a>

    </nav>

</header>

<div class="cart">

    <h2>Your Shopping Cart</h2>

    <div id="cartItems">
        Loading...
    </div>

    <button onclick="placeOrder()">
        Place Order
    </button>

</div>

<script>

async function loadCart() {

    const response =
        await fetch(
            "http://localhost:8080/api/cart"
        );

    const cart =
        await response.json();

    const container =
        document.getElementById(
            "cartItems"
        );

    container.innerHTML = "";

    if (cart.length === 0) {

        container.innerHTML =
            "<h3>Cart is empty</h3>";

        return;
    }

    let total = 0;

    cart.forEach(item => {

        const amount =
            item.price *
            item.quantity;

        total += amount;

        container.innerHTML += `

            <div class="cart-item">

                <h3>
                    ${item.name}
                </h3>

                <p>
                    ₹${item.price}
                </p>

                <p>
                    Quantity:
                    ${item.quantity}
                </p>

                <p>
                    Amount:
                    ₹${amount}
                </p>

            </div>

        `;

    });

    container.innerHTML += `

        <h2>
            Total: ₹${total}
        </h2>

    `;
}

async function placeOrder() {

    const response =
        await fetch(
            "http://localhost:8080/api/order",
            {
                method: "POST"
            }
        );

    const result =
        await response.json();

    alert(
        result.message +
        (
            result.total
                ? " Total: ₹" + result.total
                : ""
        )
    );

    loadCart();
}

loadCart();

</script>

</body>

</html>


* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f3f4f6;
}

header {
    background: #111827;
    color: white;
    padding: 20px 7%;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

header h1 {
    margin: 0;
}

nav a {
    color: white;
    text-decoration: none;
    margin: 0 10px;
}

nav a:hover {
    color: #38bdf8;
}

.hero {
    text-align: center;
    padding: 120px 20px;
    background: white;
}

.hero h2 {
    font-size: 45px;
}

.hero p {
    font-size: 20px;
}

button {
    background: #2563eb;
    color: white;
    border: none;
    padding: 12px 22px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
}

button:hover {
    background: #1d4ed8;
}

.title {
    text-align: center;
    margin: 40px;
}

.products {
    display: grid;
    grid-template-columns:
        repeat(auto-fit, minmax(250px, 1fr));

    gap: 25px;

    padding: 30px;
}

.card {
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow:
        0 2px 10px rgba(0,0,0,0.1);
}

.card img {
    width: 100%;
    height: 220px;
    object-fit: contain;
}

.card h3 {
    color: #2563eb;
}

.form {
    max-width: 500px;
    margin: 50px auto;
    background: white;
    padding: 30px;
    border-radius: 10px;
}

.form input,
.form textarea {
    width: 100%;
    padding: 12px;
    margin: 10px 0;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.form textarea {
    height: 100px;
}

.cart {
    max-width: 800px;
    margin: 50px auto;
    background: white;
    padding: 30px;
    border-radius: 10px;
}

.cart-item {
    border-bottom: 1px solid #ddd;
    padding: 15px;
}

footer {
    text-align: center;
    background: #111827;
    color: white;
    padding: 25px;
    margin-top: 50px;
}

@media (max-width: 700px) {

    header {
        flex-direction: column;
        gap: 15px;
    }

    nav {
        margin-top: 10px;
    }

    .hero h2 {
        font-size: 32px;
    }
}
