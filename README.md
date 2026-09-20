<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JRA Shops</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 20px;
            text-align: center;
        }
        header {
            background-color: #2c3e50;
            color: white;
            padding: 15px;
            border-radius: 8px;
        }
        .container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 30px;
            flex-wrap: wrap;
        }
        .card {
            background-color: white;
            border: 1px solid #ddd;
            border-radius: 8px;
            width: 200px;
            padding: 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        .card h3 {
            margin-top: 0;
            color: #333;
        }
        .price {
            color: #27ae60;
            font-size: 1.2rem;
            font-weight: bold;
        }
        button {
            background-color: #2980b9;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1rem;
        }
        button:hover {
            background-color: #3498db;
        }
    </style>
</head>
<body>

    <header>
        <h1>Welcome to JRA Shops</h1>
    </header>

    <h2>Featured Products</h2>

    <div class="container">
        <div class="card">
            <h3>Laptop</h3>
            <p class="price">$799.99</p>
            <button onclick="buyItem('Laptop', 799.99)">Buy Now</button>
        </div>

        <div class="card">
            <h3>Headphones</h3>
            <p class="price">$49.99</p>
            <button onclick="buyItem('Headphones', 49.99)">Buy Now</button>
        </div>

        <div class="card">
            <h3>Smart Watch</h3>
            <p class="price">$199.99</p>
            <button onclick="buyItem('Smart Watch', 199.99)">Buy Now</button>
        </div>
    </div>

    <script>
        function buyItem(name, price) {
            alert("Thank you for shopping at JRA Shops!\nYou bought: " + name + " for $" + price);
        }
    </script>

</body>
</html>






import java.util.Scanner;

public class JraShops {

    static class Product {
        int id;
        String name;
        double price;

        Product(int id, String name, double price) {
            this.id = id;
            this.name = name;
            this.price = price;
        }
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Define available products in JRA Shops
        Product[] products = {
            new Product(1, "Laptop", 799.99),
            new Product(2, "Headphones", 49.99),
            new Product(3, "Smart Watch", 199.99)
        };

        System.out.println("=================================");
        System.out.println("      WELCOME TO JRA SHOPS       ");
        System.out.println("=================================");

        while (true) {
            System.out.println("\nAvailable Products:");
            for (Product p : products) {
                System.out.println(p.id + ". " + p.name + " - $" + p.price);
            }
            System.out.println("0. Exit");

            System.out.print("\nEnter the Product ID you want to buy: ");
            int choice = scanner.nextInt();

            if (choice == 0) {
                System.out.println("\nThank you for visiting JRA Shops! Goodbye!");
                break;
            }

            boolean found = false;
            for (Product p : products) {
                if (p.id == choice) {
                    System.out.println("\nSuccess: You purchased " + p.name + " for $" + p.price);
                    found = true;
                    break;
                }
            }

            if (!found) {
                System.out.println("\nInvalid Product ID. Please try again.");
            }
        }

        scanner.close();
    }
}
