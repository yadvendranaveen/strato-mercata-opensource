# Demo Setup Guide for MetaMask Integration Presentation

## Prerequisites Setup

### 1. Stripe Test Account Setup
1. Go to [Stripe Dashboard](https://dashboard.stripe.com/test/apikeys)
2. Sign up for a free account
3. Copy your test keys:
   - **Publishable Key**: `pk_test_...`
   - **Secret Key**: `sk_test_...`

### 2. MetaMask Test Setup
1. Install MetaMask browser extension
2. Add Sepolia testnet:
   - Network Name: `Sepolia Testnet`
   - RPC URL: `https://sepolia.infura.io/v3/YOUR_INFURA_KEY` (or use public RPC)
   - Chain ID: `11155111`
   - Currency Symbol: `ETH`
3. Get test ETH from [Sepolia Faucet](https://sepoliafaucet.com/)

### 3. Environment Variables
Create a `.env` file in the payment-server directory:

```bash
# Stripe Test Keys
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key_here
STRIPE_PUBLISHABLE_KEY=pk_test_your_stripe_publishable_key_here

# Database (local)
POSTGRES_SERVER_URL=localhost
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DBNAME=payment_server

# Server Configuration
SERVER_HOST=http://localhost:8018
STRATO_HOST=http://localhost:8080
NODE_ENV=development

# OAuth (for demo, we'll use a simple setup)
OAUTH_DISCOVERY_URL=http://localhost:8080/.well-known/openid-configuration
OAUTH_CLIENT_ID=demo_client
OAUTH_CLIENT_SECRET=demo_secret

# USDST (test token)
USDST_ADDRESS=0x1234567890123456789012345678901234567890
USDST_FEE_RECIPIENT=0x1234567890123456789012345678901234567890

# MetaMask Configuration
METAMASK_SERVICE_NAME_VALUE=MetaMask
METAMASK_IMAGE_URL_VALUE=https://fileserver.mercata-testnet2.blockapps.net/highway/3fe266f64979ff185364131d9f6f3bc96eb272e98691bbc829ccf31f59d956c9.png
```

## Demo Flow

### 1. Start Services
```bash
# Start PostgreSQL
docker run -d --name payment-postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=payment_server \
  -p 5432:5432 \
  postgres:13

# Start payment server
cd payment-server
npm install
npm start
```

### 2. Demo Steps

#### Step 1: Seller Onboarding
1. Open browser to `http://localhost:8018/metamask/onboarding`
2. Connect MetaMask wallet
3. Select supported tokens (ETH, USDC)
4. Complete onboarding

#### Step 2: Create Test Product
1. Use marketplace backend to create a test product
2. Set price in USD (e.g., $50)

#### Step 3: Buyer Checkout
1. Navigate to checkout page
2. Select MetaMask payment
3. Choose ETH or USDC
4. Complete transaction in MetaMask
5. Show successful completion

### 3. Presentation Script

#### Introduction (2 minutes)
- "Today I'll demonstrate the MetaMask integration I implemented for the payment server"
- "This allows users to pay with cryptocurrencies alongside traditional payment methods"

#### Technical Overview (3 minutes)
- Show the code structure
- Explain the key components:
  - MetaMask controller
  - Transaction parameter generation
  - Real-time price conversion
  - Order completion flow

#### Live Demo (5 minutes)
1. **Seller Setup**: Show onboarding process
2. **Product Creation**: Create a test product
3. **Payment Flow**: Complete a MetaMask payment
4. **Transaction Confirmation**: Show successful completion

#### Code Walkthrough (3 minutes)
- Highlight key implementation details
- Show error handling
- Demonstrate test coverage

#### Q&A (2 minutes)

## Troubleshooting

### Common Issues:
1. **MetaMask not connecting**: Check if MetaMask is unlocked
2. **Transaction failing**: Ensure you have test ETH
3. **Database connection**: Verify PostgreSQL is running
4. **CORS issues**: Check server configuration

### Test Data:
- Test ETH address: `0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6`
- Test USDC amount: 100 (represents $100)
- Test product: "Demo NFT" for $50

## Files to Show in Presentation

1. `payment-server/MetaMask/metamask.controller.js` - Main logic
2. `payment-server/MetaMask/README.md` - Documentation
3. `payment-server/tests/metamask.test.js` - Test coverage
4. `payment-server/public/checkout.js` - Frontend integration

## Success Metrics

- ✅ MetaMask wallet connects successfully
- ✅ Transaction parameters generated correctly
- ✅ Payment completes without errors
- ✅ Order status updates properly
- ✅ All tests pass 