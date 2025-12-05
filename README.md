{
  "name": "eliteucltickets",
  "version": "1.0.0",
  "description": "Elite UCL Tickets - Champions League Final Premium Ticketing Website",
  "main": "server/server.js",
  "scripts": {
    "start": "node server/server.js"
  },
  "author": "Smoke",
  "license": "MIT",
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "body-parser": "^1.20.2",
    "cors": "^2.8.5",
    "dotenv": "^16.4.1",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.1",
    "mongodb": "^6.3.0",
    "mongoose": "^7.6.3",
    "nodemailer": "^6.9.13"
  }
}

import express from "express";
import mongoose from "mongoose";
import cors from "cors";
import dotenv from "dotenv";
import bodyParser from "body-parser";
import path from "path";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";

dotenv.config();

const app = express();
const PORT = process.env.PORT || 10000;

// MIDDLEWARE
app.use(cors());
app.use(bodyParser.json());
app.use(express.urlencoded({ extended: true }));

// SERVE PUBLIC FRONTEND
const __dirname = path.resolve();
app.use(express.static(path.join(__dirname, "public")));

// MONGO CONNECTION
mongoose
  .connect(process.env.MONGO_URI, { dbName: "eliteucltickets" })
  .then(() => console.log("MongoDB Connected"))
  .catch((err) => console.log("MongoDB Error:", err));

// ADMIN SCHEMA
const adminSchema = new mongoose.Schema({
  email: String,
  password: String
});

const Admin = mongoose.model("Admin", adminSchema);

// DEFAULT ADMIN (only created ONCE)
async function createDefaultAdmin() {
  const exists = await Admin.findOne({ email: "elitematchticket@gmail.com" });
  if (!exists) {
    const hashed = await bcrypt.hash("smoke091$$DIOR", 10);
    await Admin.create({
      email: "elitematchticket@gmail.com",
      password: hashed
    });
    console.log("Default admin created");
  }
}
createDefaultAdmin();

// LOGIN ROUTE
app.post("/api/admin/login", async (req, res) => {
  const { email, password } = req.body;

  const admin = await Admin.findOne({ email });
  if (!admin) return res.status(400).json({ message: "Invalid email" });

  const match = await bcrypt.compare(password, admin.password);
  if (!match) return res.status(400).json({ message: "Invalid password" });

  const token = jwt.sign({ id: admin._id }, "SECRET_KEY");

  res.json({ message: "Login successful", token });
});

// TICKET PLACEHOLDER ROUTE
app.get("/api/tickets", (req, res) => {
  res.json({
    tiers: [
      { name: "Immortal", price: 100000 },
      { name: "Elite", price: 55000 },
      { name: "Hospitality", price: 12000 },
      { name: "VIP", price: 750 },
      { name: "Regular", price: "150 - 300" }
    ]
  });
});

// FALLBACK — SERVE FRONTEND
app.get("*", (req, res) => {
  res.sendFile(path.join(__dirname, "public", "index.html"));
});

// START SERVER
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Elite Match Ticket – UCL Final 2026</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>
    <header class="hero">
        <h1>UEFA Champions League Final 2026 – Hungary</h1>
        <p>Your gateway to the most exclusive football experience on Earth.</p>
        <a href="exclusive.html" class="exclusive-btn">🔥 IMMORTAL & ELITE EXCLUSIVE OFFERS →</a>
    </header>

    <section class="tickets">
        <h2>Available Ticket Categories</h2>

        <div class="ticket-card">
            <h3>Standing</h3>
            <p>General standing access.</p>
            <span>$150 – $300</span>
        </div>

        <div class="ticket-card">
            <h3>VIP</h3>
            <p>Premium padded seat + lounge access.</p>
            <span>$750</span>
        </div>

        <div class="ticket-card">
            <h3>Hospitality</h3>
            <p>Includes flight, meals, drinks, and front-row Champions League executive seating.</p>
            <span>$12,000</span>
        </div>

        <a href="exclusive.html" class="highlight-card">
            <h2>🔥 ELITE & IMMORTAL EXCLUSIVE PACKAGES</h2>
            <p>Private jets, 5-star hotels, meet & greet, trophies & more.</p>
        </a>

    </section>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Exclusive Packages – Elite Match Ticket</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>

<header class="hero">
    <h1>Exclusive Immortal & Elite Packages</h1>
    <p>The most powerful experience in Champions League history.</p>
</header>

<section class="exclusive-section">

    <div class="exclusive-card immortal">
        <h2>IMMORTAL PACKAGE – $100,000</h2>
        <ul>
            <li>Private Jet to Hungary</li>
            <li>3 Nights in 5-Star Hotel</li>
            <li>Private Chef</li>
            <li>Chauffeur Service</li>
            <li>Meet & Greet With Football Legends</li>
            <li>Champions League Museum Tour</li>
            <li>Dinner With UCL Winners</li>
            <li>VIP Access to After Party</li>
        </ul>
        <button class="buy-btn">Buy Now</button>
    </div>

    <div class="exclusive-card elite">
        <h2>ELITE PACKAGE – $55,000</h2>
        <ul>
            <li>First Class Flight</li>
            <li>3 Days in 5-Star Hotel</li>
            <li>Early Access to Stadium</li>
            <li>See the Champions League Trophy Before the Match</li>
        </ul>
        <button class="buy-btn">Buy Now</button>
    </div>

</section>

</body>
</html>

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #000020;
    color: white;
}

.hero {
    text-align: center;
    padding: 50px;
    background: linear-gradient(to bottom right, #001060, #000020);
}

.exclusive-btn, .highlight-card {
    display: block;
    margin: 20px auto;
    padding: 15px;
    width: 90%;
    max-width: 500px;
    background: #0028ff;
    color: white;
    text-align: center;
    font-size: 20px;
    border-radius: 10px;
    text-decoration: none;
    font-weight: bold;
}

.ticket-card, .exclusive-card {
    background: #111;
    margin: 20px auto;
    padding: 20px;
    max-width: 600px;
    border-radius: 10px;
}

.exclusive-card h2 {
    color: gold;
}

.immortal h2 {
    color: #ff0059;
}

.buy-btn {
    width: 100%;
    padding: 15px;
    background: gold;
    border: none;
    font-size: 18px;
    border-radius: 8px;
}


