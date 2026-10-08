AI-Based Expiry-Aware Dynamic Discount System 🏷️📉

"Status" (https://img.shields.io/badge/Status-Development-orange)
"Version" (https://img.shields.io/badge/Version-1.0-blue)
"Tech Stack" (https://img.shields.io/badge/Tech_Stack-FastAPI%20%7C%20React%20%7C%20RL-success)

📖 Project Overview

This project is a full-stack, AI-driven inventory management and Point-of-Sale (POS) dashboard.

The system uses an Adaptive Reinforcement Learning (RL) Engine to replace static discount rules with dynamic, expiry-aware pricing.

The AI agent dynamically adjusts discount percentages based on:

- Remaining shelf life
- Current inventory level
- Unsold stock
- Customer purchasing behavior

The main goal is to maximize revenue while minimizing product wastage.

---

🧩 Core Modules

The system is divided into four major modules.

Module 1: Inventory & Product Management

What it does

Tracks:

- Store products
- Product batches
- Manufacturing dates
- Expiry dates
- Initial quantity
- Current stock
- Current discount

How to build it

1. Use PostgreSQL to create "Product" and "Batch" tables.
2. Store "mfg_date" and "expiry_date" for every batch.
3. Build CRUD APIs using FastAPI.
4. Calculate remaining shelf life:

Days Left = Expiry Date - Current Date

5. Run a scheduled background task using APScheduler or Celery.

---

Module 2: Reinforcement Learning Engine

What it does

The RL engine determines the best discount based on the current inventory state.

Example:

Days Left = 3
Stock = 20
       ↓
RL Agent
       ↓
30% Discount

State

The state contains:

[Days Until Expiry, Unsold Stock]

Actions

The agent can select:

0%
15%
30%
50%
75%

Reward

The reward function encourages profitable sales and penalizes expired inventory.

Sale → Positive Reward
Expired Product → Negative Reward

Training

The RL model can be trained offline using simulated retail data.

Possible algorithms:

- Q-Learning
- PPO

The trained model can then be saved and loaded by the FastAPI backend.

---

Module 3: POS & Checkout Simulator

What it does

The POS module simulates a retail checkout system.

When a product is scanned:

Product Scan
     ↓
FastAPI
     ↓
Get Batch State
     ↓
RL Agent
     ↓
Calculate Discount
     ↓
Display Final Price

Main operations

1. Scan product.
2. Fetch batch information.
3. Get current discount from the RL agent.
4. Calculate final price.
5. Add product to cart.
6. Complete checkout.
7. Reduce inventory.
8. Calculate reward.
9. Send reward to the RL agent.

---

Module 4: Full-Stack Admin Dashboard

The dashboard allows store managers to monitor the complete system.

Inventory View

Products can be classified based on remaining shelf life:

Days Left| Status
1–2 days| 🔴 Critical
3–5 days| 🟡 Warning
More than 5 days| 🟢 Safe

POS View

The POS interface contains:

- Product scanner
- Product information
- AI discount
- Shopping cart
- Final price
- Checkout button

Analytics

The dashboard displays:

- Revenue
- Wastage cost
- Revenue saved
- Products sold
- Products expired
- Discount decisions

---

🏗️ System Architecture

1. High-Level Architecture

flowchart TD

    A[React Frontend]

    A1[Inventory Dashboard]
    A2[POS Checkout]
    A3[Analytics Dashboard]

    B[FastAPI Backend]

    B1[Inventory Controller]
    B2[Transaction Processor]
    B3[Pricing API]

    C[RL AI Engine]

    C1[State Observer]
    C2[RL Agent]
    C3[Reward Calculator]

    D[(PostgreSQL Database)]

    D1[(Products & Batches)]
    D2[(Sales Transactions)]
    D3[(RL Training Logs)]

    A --> A1
    A --> A2
    A --> A3

    A1 --> B1
    A2 --> B2
    A2 --> B3
    A3 --> B1

    B1 --> D1
    B2 --> D2

    B3 --> C2
    C2 --> C1
    C1 --> D1

    B2 --> C3
    C3 --> C2

    C2 --> D3

---

2. Database ER Diagram

erDiagram

    PRODUCT {
        int ProductID PK
        string Name
        string Category
        float BasePrice
        float CostPrice
    }

    BATCH {
        int BatchID PK
        int ProductID FK
        int InitialQty
        int CurrentQty
        date MfgDate
        date ExpiryDate
        float CurrentDiscountPct
    }

    TRANSACTION {
        int TransactionID PK
        int BatchID FK
        int QtySold
        float FinalPricePaid
        datetime Timestamp
    }

    RL_LOG {
        int LogID PK
        int BatchID FK
        string StateVector
        float ActionTaken
        float RewardReceived
        datetime Timestamp
    }

    PRODUCT ||--o{ BATCH : contains
    BATCH ||--o{ TRANSACTION : records
    BATCH ||--o{ RL_LOG : generates

---

3. Object-Oriented Class Diagram

classDiagram

    class InventoryController {
        +addBatch(productID, qty, mfgDate, expiryDate)
        +updateStock(batchID, qtySold)
        +getExpiringBatches(thresholdDays)
    }

    class PricingAgent {
        -discountBrackets
        +observeState(batchID)
        +chooseAction(state)
        +updatePolicy(state, action, reward)
    }

    class POSCheckout {
        +scanItem(batchID)
        +fetchDiscount(batchID)
        +completeTransaction(cart)
    }

    class RewardCalculator {
        +calculateSaleReward(margin)
        +calculateWastePenalty(cost, qty)
    }

    POSCheckout --> InventoryController : updates stock
    POSCheckout --> PricingAgent : requests discount
    POSCheckout --> RewardCalculator : calculates reward
    RewardCalculator --> PricingAgent : sends reward

---

4. POS Checkout & AI Learning Flow

sequenceDiagram

    autonumber

    actor Customer as POS Simulator
    participant API as FastAPI Backend
    participant DB as PostgreSQL
    participant AI as RL Pricing Agent

    Customer->>API: GET /api/price?batchID=105

    API->>DB: Get stock and expiry
    DB-->>API: DaysLeft=3, Stock=20

    API->>AI: Request action for state

    AI-->>API: Return 30% discount

    API-->>Customer: Display discounted price

    Customer->>API: POST /api/checkout

    API->>DB: Deduct 2 units
    DB-->>API: Stock updated

    API->>AI: Send sale reward

    AI->>AI: Update learning policy

    API-->>Customer: Transaction successful

---

5. Product Batch Lifecycle

stateDiagram-v2

    [*] --> InStock

    InStock --> RegularPrice : More than 10 days

    RegularPrice --> AIDiscount : 10 days or less

    AIDiscount --> MildDiscount : 10-20% discount

    MildDiscount --> HighDiscount : Expiry approaching

    HighDiscount --> Clearance : 1-2 days left

    RegularPrice --> SoldOut : Stock reaches zero

    AIDiscount --> SoldOut : Stock reaches zero

    AIDiscount --> Expired : Days left reaches zero

    Expired --> [*] : Negative reward

    SoldOut --> [*]

---

🔄 Overall System Workflow

flowchart LR

    A[Product Added]
    B[Batch Created]
    C[Monitor Expiry]
    D[Observe Stock]
    E[RL Agent]
    F[Select Discount]
    G[Customer Purchase]
    H[Update Inventory]
    I[Calculate Reward]
    J[Update RL Policy]
    K[Analytics]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
    J --> E
    H --> K
    I --> K

---

🛠️ Tech Stack

Frontend

- React.js
- Vite
- Tailwind CSS

Data Visualization

- Recharts
- Chart.js

Backend

- FastAPI
- Python
- Uvicorn
- SQLAlchemy

AI / Machine Learning

- Python
- Gymnasium
- PyTorch
- NumPy

Database

- PostgreSQL for production
- SQLite for local development

---

📁 Suggested Project Structure

dynamic-discount-system/
│
├── backend/
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   ├── train_rl_agent.py
│   │
│   ├── api/
│   │   ├── products.py
│   │   ├── batches.py
│   │   ├── pricing.py
│   │   └── checkout.py
│   │
│   ├── rl/
│   │   ├── environment.py
│   │   ├── agent.py
│   │   └── reward.py
│   │
│   ├── models/
│   │   └── discount_agent.pkl
│   │
│   ├── requirements.txt
│   └── alembic.ini
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── hooks/
│   │   └── App.jsx
│   │
│   ├── package.json
│   └── vite.config.js
│
├── README.md
└── .gitignore

---

🚀 Step-by-Step Setup

Prerequisites

Install:

- Python 3.9+
- Node.js 16+
- Git
- PostgreSQL

---

1. Backend & AI Setup

Open Terminal 1.

Clone the repository

git clone https://github.com/yourusername/dynamic-discount-system.git

Enter the backend directory

cd dynamic-discount-system/backend

Create virtual environment

python -m venv venv

Activate virtual environment

Windows

venv\Scripts\activate

Linux / macOS

source venv/bin/activate

Install dependencies

pip install -r requirements.txt

Run database migrations

alembic upgrade head

Train the RL model

python train_rl_agent.py

Start FastAPI

uvicorn main:app --reload --port 8000

FastAPI documentation:

http://localhost:8000/docs

---

2. Frontend Setup

Open Terminal 2.

cd dynamic-discount-system/frontend

Install dependencies:

npm install

Start React:

npm run dev

Open:

http://localhost:5173

---

🤖 Example AI Decision

Suppose a batch contains:

Product: Milk
Stock: 20 units
Days Left: 3
Base Price: ₹50

The RL agent observes:

State = [3, 20]

It selects:

Action = 30% discount

The final price becomes:

₹50 × (1 - 0.30)

= ₹35

Therefore:

Original Price : ₹50
Discount       : 30%
Final Price    : ₹35

---

📊 Expected Benefits

The system aims to:

- Reduce product wastage
- Improve inventory turnover
- Increase revenue recovery
- Automate discount decisions
- Adapt discounts according to demand
- Provide real-time inventory monitoring
- Help store managers make better pricing decisions

---

🔮 Future Improvements

Possible future enhancements include:

- Demand forecasting
- Customer behavior prediction
- Multi-product optimization
- Deep Reinforcement Learning
- Barcode scanner integration
- Real-time notifications
- Cloud deployment
- Mobile application
- Advanced sales forecasting
- Explainable AI for discount decisions

---

📌 Project Status

Status: 🚧 Development

Version: 1.0

The project is currently under development. The main objective is to build an expiry-aware dynamic pricing system using Reinforcement Learning and demonstrate its effectiveness through a full-stack retail dashboard.

---

📚 Research Inspiration

This project is based on research principles related to:

- Expiration-aware pricing
- Dynamic discounting
- Reinforcement Learning
- Inventory optimization
- Food and product waste reduction
- Adaptive pricing strategies

---

👨‍💻 Project Goal

The ultimate goal is to create an intelligent retail system where pricing decisions are automatically adjusted according to inventory conditions, remaining shelf life, and customer purchasing behavior.

Inventory
    ↓
Expiry Monitoring
    ↓
State Observation
    ↓
RL Agent
    ↓
Dynamic Discount
    ↓
Customer Purchase
    ↓
Reward Calculation
    ↓
Policy Update
    ↓
Better Future Decisions

---

⭐ Key Concept

«Instead of using fixed discount rules, the system learns which discount is most effective for different inventory and expiry conditions.»
