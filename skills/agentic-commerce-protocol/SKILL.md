---
name: agentic-commerce-protocol
description: Integrate the Agentic Commerce Protocol (ACP) for AI-driven commerce between buyers, agents, and businesses
triggers:
  - implement agentic commerce protocol
  - integrate ACP checkout flow
  - add AI agent commerce capability
  - setup agentic commerce API
  - create ACP payment handler
  - configure agentic checkout endpoint
  - implement ACP seller integration
  - build AI commerce protocol integration
---

# Agentic Commerce Protocol (ACP) Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

The Agentic Commerce Protocol (ACP) is an open standard for connecting buyers, their AI agents, and businesses to complete purchases seamlessly. Maintained by OpenAI and Stripe, ACP enables AI agents to initiate and complete commerce transactions on behalf of users.

**Key Capabilities:**
- AI agents can discover and purchase products/services
- Businesses expose checkout endpoints to AI agents
- Payment providers handle secure payment token exchange
- Support for carts, orders, fulfillment, and extensions
- Date-based versioning (`YYYY-MM-DD` format)

**Current Version:** `2026-04-17`

## Installation & Setup

### For Merchants (Implementing Checkout Endpoints)

1. **Clone the specification repository:**
```bash
git clone https://github.com/agentic-commerce-protocol/agentic-commerce-protocol.git
cd agentic-commerce-protocol
```

2. **Review the OpenAPI specification:**
```bash
# Latest stable spec
cat spec/2026-04-17/openapi/openapi.agentic_checkout.yaml
```

3. **Install dependencies for your implementation:**

Node.js/Express example:
```bash
npm init -y
npm install express body-parser express-validator
```

Python/Flask example:
```bash
pip install flask flask-cors jsonschema
```

### For AI Agent Developers

Review the specification and integrate with a reference implementation:
- **OpenAI:** https://developers.openai.com/commerce/
- **Stripe:** https://docs.stripe.com/agentic-commerce

## Core Concepts

### 1. Agentic Checkout Flow

The basic checkout flow involves these endpoints:

- `POST /agentic-checkout/capabilities` - Discover seller capabilities
- `POST /agentic-checkout/create-checkout` - Initialize checkout session
- `POST /agentic-checkout/update-checkout` - Update checkout details
- `POST /agentic-checkout/confirm-checkout` - Finalize and confirm purchase

### 2. Payment Handlers

ACP supports multiple payment handler types:

- **Delegate Payment Handler** - Agent handles payment collection
- **Seller-Backed Payment Handler** - Seller collects payment directly
- **Payment Link Handler** - Redirect to payment page

## Implementation Examples

### Basic Express.js Seller Implementation

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const { v4: uuidv4 } = require('uuid');

const app = express();
app.use(bodyParser.json());

// In-memory store (use database in production)
const checkoutSessions = new Map();

// Capabilities endpoint
app.post('/agentic-checkout/capabilities', (req, res) => {
  res.json({
    version: '2026-04-17',
    supported_features: [
      'cart',
      'orders',
      'fulfillment',
      'extensions'
    ],
    payment_handlers: [
      {
        type: 'delegate_payment',
        supported_methods: ['card', 'bank_transfer']
      }
    ],
    extensions: [
      {
        type: 'discount',
        supported_discount_types: ['percentage', 'fixed_amount']
      }
    ]
  });
});

// Create checkout endpoint
app.post('/agentic-checkout/create-checkout', (req, res) => {
  const { items, buyer_info, payment_handler } = req.body;
  
  // Validate required fields
  if (!items || !Array.isArray(items) || items.length === 0) {
    return res.status(400).json({
      error: {
        type: 'invalid_request',
        message: 'Items are required'
      }
    });
  }

  // Create session
  const sessionId = uuidv4();
  const session = {
    id: sessionId,
    status: 'open',
    items: items.map(item => ({
      id: item.id,
      name: item.name,
      quantity: item.quantity || 1,
      unit_price: item.unit_price,
      total: (item.quantity || 1) * item.unit_price
    })),
    buyer_info: buyer_info || {},
    payment_handler,
    created_at: new Date().toISOString()
  };

  // Calculate totals
  session.subtotal = session.items.reduce((sum, item) => sum + item.total, 0);
  session.total = session.subtotal;

  checkoutSessions.set(sessionId, session);

  res.json({
    checkout_id: sessionId,
    status: 'open',
    items: session.items,
    subtotal: session.subtotal,
    total: session.total,
    payment_handler: session.payment_handler
  });
});

// Update checkout endpoint
app.post('/agentic-checkout/update-checkout', (req, res) => {
  const { checkout_id, items, buyer_info, shipping_address } = req.body;

  const session = checkoutSessions.get(checkout_id);
  if (!session) {
    return res.status(404).json({
      error: {
        type: 'checkout_not_found',
        message: 'Checkout session not found'
      }
    });
  }

  if (session.status !== 'open') {
    return res.status(400).json({
      error: {
        type: 'invalid_state',
        message: 'Checkout is not in open state'
      }
    });
  }

  // Update session
  if (items) {
    session.items = items.map(item => ({
      id: item.id,
      name: item.name,
      quantity: item.quantity || 1,
      unit_price: item.unit_price,
      total: (item.quantity || 1) * item.unit_price
    }));
    session.subtotal = session.items.reduce((sum, item) => sum + item.total, 0);
    session.total = session.subtotal;
  }

  if (buyer_info) {
    session.buyer_info = { ...session.buyer_info, ...buyer_info };
  }

  if (shipping_address) {
    session.shipping_address = shipping_address;
  }

  res.json({
    checkout_id: session.id,
    status: session.status,
    items: session.items,
    subtotal: session.subtotal,
    total: session.total
  });
});

// Confirm checkout endpoint
app.post('/agentic-checkout/confirm-checkout', (req, res) => {
  const { checkout_id, payment_token } = req.body;

  const session = checkoutSessions.get(checkout_id);
  if (!session) {
    return res.status(404).json({
      error: {
        type: 'checkout_not_found',
        message: 'Checkout session not found'
      }
    });
  }

  if (session.status !== 'open') {
    return res.status(400).json({
      error: {
        type: 'invalid_state',
        message: 'Checkout is not in open state'
      }
    });
  }

  // Process payment (integrate with payment processor)
  // For demo purposes, we'll simulate success
  session.status = 'completed';
  session.payment_token = payment_token;
  session.completed_at = new Date().toISOString();

  const orderId = `ord_${uuidv4()}`;
  session.order_id = orderId;

  res.json({
    checkout_id: session.id,
    status: 'completed',
    order_id: orderId,
    confirmation: {
      order_number: orderId,
      total: session.total,
      items: session.items
    }
  });
});

app.listen(3000, () => {
  console.log('ACP server listening on port 3000');
});
```

### Python/Flask Seller Implementation

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
import uuid
from datetime import datetime

app = Flask(__name__)
CORS(app)

# In-memory store
checkout_sessions = {}

@app.route('/agentic-checkout/capabilities', methods=['POST'])
def capabilities():
    return jsonify({
        'version': '2026-04-17',
        'supported_features': [
            'cart',
            'orders',
            'fulfillment'
        ],
        'payment_handlers': [
            {
                'type': 'delegate_payment',
                'supported_methods': ['card']
            }
        ]
    })

@app.route('/agentic-checkout/create-checkout', methods=['POST'])
def create_checkout():
    data = request.json
    items = data.get('items', [])
    
    if not items:
        return jsonify({
            'error': {
                'type': 'invalid_request',
                'message': 'Items are required'
            }
        }), 400
    
    session_id = str(uuid.uuid4())
    
    # Process items
    processed_items = []
    subtotal = 0
    
    for item in items:
        quantity = item.get('quantity', 1)
        unit_price = item.get('unit_price', 0)
        total = quantity * unit_price
        
        processed_items.append({
            'id': item.get('id'),
            'name': item.get('name'),
            'quantity': quantity,
            'unit_price': unit_price,
            'total': total
        })
        subtotal += total
    
    session = {
        'id': session_id,
        'status': 'open',
        'items': processed_items,
        'buyer_info': data.get('buyer_info', {}),
        'payment_handler': data.get('payment_handler'),
        'subtotal': subtotal,
        'total': subtotal,
        'created_at': datetime.utcnow().isoformat()
    }
    
    checkout_sessions[session_id] = session
    
    return jsonify({
        'checkout_id': session_id,
        'status': 'open',
        'items': processed_items,
        'subtotal': subtotal,
        'total': subtotal
    })

@app.route('/agentic-checkout/confirm-checkout', methods=['POST'])
def confirm_checkout():
    data = request.json
    checkout_id = data.get('checkout_id')
    payment_token = data.get('payment_token')
    
    session = checkout_sessions.get(checkout_id)
    if not session:
        return jsonify({
            'error': {
                'type': 'checkout_not_found',
                'message': 'Checkout session not found'
            }
        }), 404
    
    if session['status'] != 'open':
        return jsonify({
            'error': {
                'type': 'invalid_state',
                'message': 'Checkout is not in open state'
            }
        }), 400
    
    # Process payment
    session['status'] = 'completed'
    session['payment_token'] = payment_token
    order_id = f"ord_{uuid.uuid4()}"
    session['order_id'] = order_id
    
    return jsonify({
        'checkout_id': checkout_id,
        'status': 'completed',
        'order_id': order_id,
        'confirmation': {
            'order_number': order_id,
            'total': session['total'],
            'items': session['items']
        }
    })

if __name__ == '__main__':
    app.run(port=3000, debug=True)
```

### Capability Negotiation

```javascript
// Client-side capability check
async function checkSellerCapabilities(sellerUrl) {
  const response = await fetch(`${sellerUrl}/agentic-checkout/capabilities`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    }
  });
  
  const capabilities = await response.json();
  
  // Check if seller supports required features
  const requiredFeatures = ['cart', 'orders'];
  const hasRequiredFeatures = requiredFeatures.every(feature =>
    capabilities.supported_features?.includes(feature)
  );
  
  return {
    compatible: hasRequiredFeatures,
    capabilities
  };
}
```

### Extension Support (Discounts)

```javascript
// Apply discount extension
app.post('/agentic-checkout/update-checkout', (req, res) => {
  const { checkout_id, extensions } = req.body;
  const session = checkoutSessions.get(checkout_id);
  
  if (!session) {
    return res.status(404).json({ error: 'Session not found' });
  }
  
  // Process discount extension
  const discountExt = extensions?.find(ext => ext.type === 'discount');
  if (discountExt) {
    const { discount_type, value, code } = discountExt;
    
    // Validate discount code
    if (code === 'SAVE10') {
      if (discount_type === 'percentage') {
        session.discount = session.subtotal * (value / 100);
      } else if (discount_type === 'fixed_amount') {
        session.discount = value;
      }
      
      session.total = session.subtotal - session.discount;
    }
  }
  
  res.json({
    checkout_id: session.id,
    subtotal: session.subtotal,
    discount: session.discount || 0,
    total: session.total
  });
});
```

### Order Management

```javascript
// Get order status
app.post('/agentic-checkout/get-order', (req, res) => {
  const { order_id } = req.body;
  
  // Find session by order_id
  const session = Array.from(checkoutSessions.values())
    .find(s => s.order_id === order_id);
  
  if (!session) {
    return res.status(404).json({
      error: {
        type: 'order_not_found',
        message: 'Order not found'
      }
    });
  }
  
  res.json({
    order_id: session.order_id,
    status: session.status,
    created_at: session.created_at,
    items: session.items,
    total: session.total,
    fulfillment: {
      status: 'pending',
      tracking_number: null
    }
  });
});
```

## Configuration

### Environment Variables

```bash
# Server configuration
PORT=3000
NODE_ENV=production

# Payment processor
PAYMENT_PROCESSOR_API_KEY=your_payment_api_key
PAYMENT_PROCESSOR_SECRET=your_payment_secret

# Security
WEBHOOK_SECRET=your_webhook_secret
ALLOWED_ORIGINS=https://agent1.example.com,https://agent2.example.com

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/acp_db

# ACP version
ACP_VERSION=2026-04-17
```

### Configuration File Example

```javascript
// config/acp.config.js
module.exports = {
  version: process.env.ACP_VERSION || '2026-04-17',
  
  features: {
    cart: true,
    orders: true,
    fulfillment: true,
    extensions: true,
    authentication: false
  },
  
  paymentHandlers: [
    {
      type: 'delegate_payment',
      methods: ['card', 'bank_transfer'],
      processor: 'stripe'
    }
  ],
  
  extensions: [
    {
      type: 'discount',
      supportedTypes: ['percentage', 'fixed_amount', 'code']
    }
  ],
  
  security: {
    requireAuth: false,
    allowedOrigins: process.env.ALLOWED_ORIGINS?.split(',') || [],
    webhookSecret: process.env.WEBHOOK_SECRET
  }
};
```

## Common Patterns

### 1. Idempotent Request Handling

```javascript
const processedRequests = new Map();

app.post('/agentic-checkout/confirm-checkout', (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  
  if (idempotencyKey && processedRequests.has(idempotencyKey)) {
    // Return cached response
    return res.json(processedRequests.get(idempotencyKey));
  }
  
  // Process request
  const result = processCheckout(req.body);
  
  if (idempotencyKey) {
    processedRequests.set(idempotencyKey, result);
  }
  
  res.json(result);
});
```

### 2. Webhook Event Handling

```javascript
app.post('/webhooks/acp', (req, res) => {
  const signature = req.headers['x-acp-signature'];
  
  // Verify signature
  if (!verifyWebhookSignature(req.body, signature)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  const event = req.body;
  
  switch (event.type) {
    case 'checkout.completed':
      handleCheckoutCompleted(event.data);
      break;
    case 'checkout.cancelled':
      handleCheckoutCancelled(event.data);
      break;
    case 'order.fulfilled':
      handleOrderFulfilled(event.data);
      break;
  }
  
  res.json({ received: true });
});
```

### 3. Error Handling

```javascript
// Standard error response format
function sendError(res, type, message, statusCode = 400) {
  res.status(statusCode).json({
    error: {
      type,
      message,
      timestamp: new Date().toISOString()
    }
  });
}

// Usage
app.post('/agentic-checkout/create-checkout', (req, res) => {
  try {
    const { items } = req.body;
    
    if (!items || items.length === 0) {
      return sendError(res, 'invalid_request', 'Items are required');
    }
    
    // Process checkout...
    
  } catch (error) {
    console.error('Checkout error:', error);
    sendError(res, 'internal_error', 'An error occurred', 500);
  }
});
```

### 4. Cart Management

```javascript
// Add item to cart
app.post('/agentic-checkout/add-to-cart', (req, res) => {
  const { checkout_id, item } = req.body;
  const session = checkoutSessions.get(checkout_id);
  
  if (!session) {
    return sendError(res, 'checkout_not_found', 'Session not found', 404);
  }
  
  // Add or update item
  const existingIndex = session.items.findIndex(i => i.id === item.id);
  
  if (existingIndex >= 0) {
    session.items[existingIndex].quantity += item.quantity || 1;
  } else {
    session.items.push({
      id: item.id,
      name: item.name,
      quantity: item.quantity || 1,
      unit_price: item.unit_price,
      total: (item.quantity || 1) * item.unit_price
    });
  }
  
  // Recalculate totals
  session.subtotal = session.items.reduce((sum, i) => sum + i.total, 0);
  session.total = session.subtotal - (session.discount || 0);
  
  res.json({
    checkout_id: session.id,
    items: session.items,
    subtotal: session.subtotal,
    total: session.total
  });
});
```

## Testing

### Test Suite Example

```javascript
const request = require('supertest');
const app = require('./server');

describe('ACP Checkout Flow', () => {
  let checkoutId;
  
  test('GET capabilities', async () => {
    const res = await request(app)
      .post('/agentic-checkout/capabilities')
      .expect(200);
    
    expect(res.body.version).toBe('2026-04-17');
    expect(res.body.supported_features).toContain('cart');
  });
  
  test('POST create-checkout', async () => {
    const res = await request(app)
      .post('/agentic-checkout/create-checkout')
      .send({
        items: [
          {
            id: 'prod_123',
            name: 'Test Product',
            quantity: 2,
            unit_price: 1000
          }
        ],
        payment_handler: {
          type: 'delegate_payment'
        }
      })
      .expect(200);
    
    checkoutId = res.body.checkout_id;
    expect(res.body.status).toBe('open');
    expect(res.body.total).toBe(2000);
  });
  
  test('POST confirm-checkout', async () => {
    const res = await request(app)
      .post('/agentic-checkout/confirm-checkout')
      .send({
        checkout_id: checkoutId,
        payment_token: 'tok_test_12345'
      })
      .expect(200);
    
    expect(res.body.status).toBe('completed');
    expect(res.body.order_id).toBeDefined();
  });
});
```

## Troubleshooting

### Common Issues

**1. Invalid version error**
```
Error: Unsupported ACP version
```
Solution: Ensure your implementation supports the version specified in the request. Check `spec/2026-04-17/` for the current stable version.

**2. Payment handler mismatch**
```
Error: Unsupported payment handler type
```
Solution: Verify the payment handler type in your capabilities response matches what the agent is requesting.

**3. Checkout session not found**
```
Error: checkout_not_found
```
Solution: Implement proper session storage (Redis, database) instead of in-memory maps for production. Sessions should persist across server restarts.

**4. CORS errors**
```
Access-Control-Allow-Origin header missing
```
Solution: Add CORS middleware with appropriate allowed origins:
```javascript
const cors = require('cors');
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS.split(','),
  credentials: true
}));
```

**5. Idempotency issues**
```
Duplicate checkout created
```
Solution: Implement idempotency key handling for all mutating operations.

### Validation

Use the JSON schemas provided in the spec for validation:

```javascript
const Ajv = require('ajv');
const ajv = new Ajv();

const checkoutSchema = require('./spec/2026-04-17/json-schema/checkout.schema.json');
const validate = ajv.compile(checkoutSchema);

app.post('/agentic-checkout/create-checkout', (req, res) => {
  const valid = validate(req.body);
  if (!valid) {
    return res.status(400).json({
      error: {
        type: 'validation_error',
        message: validate.errors
      }
    });
  }
  // Process request...
});
```

## Additional Resources

- **Official Documentation:** https://agenticcommerce.dev
- **OpenAPI Spec:** `spec/2026-04-17/openapi/openapi.agentic_checkout.yaml`
- **Examples:** `examples/2026-04-17/`
- **Governance:** `docs/governance.md`
- **Contributing:** `CONTRIBUTING.md`
