
# Razorpay Integration in React Application

This document explains how to integrate Razorpay payment gateway into a React application. The process includes setting up both frontend and backend components to handle transactions securely.

---

## Prerequisites

1. A Razorpay account ([Sign up here](https://razorpay.com/)).
2. Basic knowledge of JavaScript, React, and Node.js.
3. Installed Node.js environment.

---

## Steps for Integration

### 1. Create a Razorpay Account
- Sign up on the [Razorpay website](https://razorpay.com/).
- Switch to **Test Mode** to simulate transactions without real money.
- Navigate to **Settings** > **API Keys** and generate your API keys.
  - **Key ID**
  - **Key Secret**

---

### 2. Set Up the Backend

#### 2.1 Initialize a Node.js Project
```bash
mkdir razorpay-backend
cd razorpay-backend
npm init -y
```

#### 2.2 Install Dependencies
```bash
npm install express razorpay dotenv crypto cors
```

#### 2.3 Create Environment Variables
Create a `.env` file in the root directory:
```
RAZORPAY_KEY_ID=your_key_id
RAZORPAY_KEY_SECRET=your_key_secret
```

#### 2.4 Develop Backend Server
Create `server.js`:
```javascript
const express = require('express');
const Razorpay = require('razorpay');
const crypto = require('crypto');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET,
});

app.post('/create-order', async (req, res) => {
  const options = {
    amount: req.body.amount * 100,
    currency: 'INR',
    receipt: `receipt_${Date.now()}`,
  };
  try {
    const order = await razorpay.orders.create(options);
    res.json(order);
  } catch (error) {
    res.status(500).send(error);
  }
});

app.post('/verify-signature', (req, res) => {
  const { order_id, payment_id, signature } = req.body;
  const body = order_id + '|' + payment_id;
  const expectedSignature = crypto
    .createHmac('sha256', process.env.RAZORPAY_KEY_SECRET)
    .update(body.toString())
    .digest('hex');
  if (expectedSignature === signature) {
    res.json({ status: 'success' });
  } else {
    res.status(400).json({ status: 'failure' });
  }
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

### 3. Set Up the Frontend

#### 3.1 Initialize a React Application
```bash
npx create-react-app razorpay-frontend
cd razorpay-frontend
```

#### 3.2 Install Axios
```bash
npm install axios
```

#### 3.3 Implement Payment Handler in `App.js`
```javascript
import React from 'react';
import axios from 'axios';

function App() {
  const loadRazorpayScript = () => {
    return new Promise((resolve) => {
      const script = document.createElement('script');
      script.src = 'https://checkout.razorpay.com/v1/checkout.js';
      script.onload = () => resolve(true);
      script.onerror = () => resolve(false);
      document.body.appendChild(script);
    });
  };

  const handlePayment = async () => {
    const res = await loadRazorpayScript();
    if (!res) {
      alert('Razorpay SDK failed to load. Are you online?');
      return;
    }

    const orderResult = await axios.post('http://localhost:5000/create-order', {
      amount: 500,
    });

    const { amount, id: order_id, currency } = orderResult.data;

    const options = {
      key: process.env.REACT_APP_RAZORPAY_KEY_ID,
      amount: amount.toString(),
      currency: currency,
      name: 'Your Company Name',
      description: 'Test Transaction',
      order_id: order_id,
      handler: async function (response) {
        const data = {
          order_id: order_id,
          payment_id: response.razorpay_payment_id,
          signature: response.razorpay_signature,
        };
        const result = await axios.post('http://localhost:5000/verify-signature', data);
        alert(result.data.status);
      },
      prefill: {
        name: 'Your Name',
        email: 'youremail@example.com',
        contact: '9999999999',
      },
      notes: {
        address: 'Your Address',
      },
      theme: {
        color: '#3399cc',
      },
    };

    const paymentObject = new window.Razorpay(options);
    paymentObject.open();
  };

  return (
    <div className="App">
      <header className="App-header">
        <h1>Razorpay Integration in React</h1>
        <button onClick={handlePayment}>Pay ₹500</button>
      </header>
    </div>
  );
}

export default App;
```

---

### 4. Test the Integration

#### 4.1 Run Backend Server
```bash
node server.js
```

#### 4.2 Start React Application
```bash
npm start
```

#### 4.3 Navigate to `http://localhost:3000` and click the **Pay ₹500** button.

---

### 5. Move to Production

1. Switch Razorpay account to **Live Mode**.
2. Update `.env` files in both backend and frontend with live keys.
3. Ensure the application is ready for real transactions.

---

For further details, visit [Razorpay Documentation](https://razorpay.com/docs/).

