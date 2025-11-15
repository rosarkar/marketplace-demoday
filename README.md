# Canvas Protocol - Complete API Reference for Frontend

**All API calls made by frontend files**  
**Base URL:** `http://localhost:8080` (configurable in each file)  
**Last Updated:** November 15, 2025

---

## 📊 Quick Overview

| File | API Calls | Purpose |
|------|-----------|---------|
| advertiser-signup.html | 1 | Create advertiser account |
| advertiser-dashboard.html | 3 | Login, fetch campaigns, create campaign |
| publisher-signup.html | 1 | Create publisher account |
| publisher-dashboard.html | 3 | Login, fetch earnings, withdraw |
| canvas-widget.html | 2 | Start CAPTCHA, complete CAPTCHA |
| index.html | 2 | Check publisher registration, check active campaigns |

**Total Unique Endpoints:** 8  
**Total API Calls Across All Files:** 12

---

## 1️⃣ advertiser-signup.html

### API Call #1: Create Advertiser Account

**Endpoint:** `POST /advertiser`

**When Called:**
- User clicks "Create Advertiser Account" button
- Form validates successfully

**Request:**
```javascript
POST http://localhost:8080/advertiser
Content-Type: application/json

{
    "name": "Acme Corp",
    "ethereumAddress": "0x1234567890123456789012345678901234567890",
    "description": "Consumer goods company",
    "campaignFundsWei": "5000000000000000000"  // 5 ETH in wei
}
```

**Response (201 - Success):**
```json
{
    "success": true,
    "advertiser": {
        "id": "uuid",
        "name": "Acme Corp",
        "ethereum_address": "0x1234567890123456789012345678901234567890",
        "description": "Consumer goods company",
        "campaign_funds_wei": "5000000000000000000",
        "created_at": "2025-11-15T12:00:00.000Z",
        "updated_at": "2025-11-15T12:00:00.000Z"
    }
}
```

**Response (400 - Invalid Address):**
```json
{
    "error": "Invalid Ethereum address format"
}
```

**Response (409 - Already Exists):**
```json
{
    "error": "Advertiser already exists"
}
```

**Code Location:** Line ~300-350

**What Happens After:**
- Success popup shown
- Advertiser data stored in localStorage
- Redirects to advertiser-dashboard.html after 2 seconds

---

## 2️⃣ advertiser-dashboard.html

### API Call #1: Login / Verify Advertiser

**Endpoint:** `GET /advertiser/{ethereumAddress}`

**When Called:**
- User submits login form with wallet address
- On page load if wallet stored in session

**Request:**
```javascript
GET http://localhost:8080/advertiser/0x1234567890123456789012345678901234567890
```

**Response (200 - Success):**
```json
{
    "success": true,
    "advertiser": {
        "id": "uuid",
        "name": "Acme Corp",
        "ethereum_address": "0x1234567890123456789012345678901234567890",
        "description": "Consumer goods company",
        "campaign_funds_wei": "5000000000000000000",
        "created_at": "2025-11-15T12:00:00.000Z",
        "updated_at": "2025-11-15T12:00:00.000Z"
    }
}
```

**Response (404 - Not Found):**
```json
{
    "error": "Advertiser not found"
}
```

**Code Location:** Line ~450-490

**What Happens After:**
- Login screen hidden
- Dashboard shown
- Wallet address displayed in header
- Triggers API Call #2 (fetch campaigns)

---

### API Call #2: Fetch Advertiser's Campaigns

**Endpoint:** `GET /advertiser/{ethereumAddress}/campaigns`

**When Called:**
- Immediately after successful login
- Every 30 seconds (auto-refresh)
- After creating a new campaign

**Request:**
```javascript
GET http://localhost:8080/advertiser/0x1234567890123456789012345678901234567890/campaigns
```

**Response (200 - Success):**
```json
{
    "success": true,
    "campaigns": [
        {
            "id": "campaign-uuid-1",
            "advertiser_id": "advertiser-uuid",
            "name": "Summer Sale 2025",
            "description": "Promotional campaign",
            "cost_per_captcha_wei": "50000000000000000",
            "total_budget_wei": "2000000000000000000",
            "spent_budget_wei": "100000000000000000",
            "completed_ads_count": 2,
            "status": "active",
            "created_at": "2025-11-15T12:00:00.000Z",
            "updated_at": "2025-11-15T12:05:00.000Z"
        },
        {
            "id": "campaign-uuid-2",
            "advertiser_id": "advertiser-uuid",
            "name": "Black Friday",
            "description": "Holiday promotion",
            "cost_per_captcha_wei": "80000000000000000",
            "total_budget_wei": "3000000000000000000",
            "spent_budget_wei": "0",
            "completed_ads_count": 0,
            "status": "active",
            "created_at": "2025-11-15T13:00:00.000Z",
            "updated_at": "2025-11-15T13:00:00.000Z"
        }
    ]
}
```

**Code Location:** Line ~520-560

**What Happens After:**
- Campaign cards rendered on dashboard
- Stats calculated and displayed:
  - Total Budget (sum of all total_budget_wei)
  - Budget Spent (sum of all spent_budget_wei)
  - Active Campaigns (count where status = 'active')
  - Total Completions (sum of all completed_ads_count)

---

### API Call #3: Create New Campaign

**Endpoint:** `POST /advertiser/{ethereumAddress}/campaigns`

**When Called:**
- User clicks "Create Campaign" in modal
- Form validates successfully

**Request:**
```javascript
POST http://localhost:8080/advertiser/0x1234567890123456789012345678901234567890/campaigns
Content-Type: application/json

{
    "name": "Summer Sale 2025",
    "description": "Promotional summer campaign",
    "costPerCaptchaWei": "50000000000000000",     // 0.05 ETH
    "totalBudgetWei": "2000000000000000000"       // 2 ETH
}
```

**Response (201 - Success):**
```json
{
    "success": true,
    "campaign": {
        "id": "new-campaign-uuid",
        "advertiser_id": "advertiser-uuid",
        "name": "Summer Sale 2025",
        "description": "Promotional summer campaign",
        "cost_per_captcha_wei": "50000000000000000",
        "total_budget_wei": "2000000000000000000",
        "spent_budget_wei": "0",
        "completed_ads_count": 0,
        "status": "active",
        "created_at": "2025-11-15T14:00:00.000Z",
        "updated_at": "2025-11-15T14:00:00.000Z"
    }
}
```

**Response (400 - Insufficient Funds):**
```json
{
    "error": "Insufficient advertiser funds for campaign budget"
}
```

**Code Location:** Line ~680-730

**What Happens After:**
- Modal closes
- Triggers API Call #2 (reload campaigns)
- New campaign appears in list
- Stats update automatically

---

## 3️⃣ publisher-signup.html

### API Call #1: Create Publisher Account

**Endpoint:** `POST /publisher`

**When Called:**
- User clicks "Create Publisher Account" button
- Form validates successfully

**Request:**
```javascript
POST http://localhost:8080/publisher
Content-Type: application/json

{
    "name": "Rohit's Tech Blog",
    "ethereumAddress": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
    "description": "Technology blog covering blockchain and AI",
    "websiteUrl": "http://techblog.example.com"
}
```

**Response (201 - Success):**
```json
{
    "success": true,
    "publisher": {
        "id": "uuid",
        "name": "Rohit's Tech Blog",
        "ethereum_address": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
        "description": "Technology blog covering blockchain and AI",
        "website_url": "http://techblog.example.com",
        "assigned_campaign_id": null,
        "total_earnings_wei": "0",
        "created_at": "2025-11-15T12:00:00.000Z",
        "updated_at": "2025-11-15T12:00:00.000Z"
    }
}
```

**Response (400 - Invalid Address or Duplicate):**
```json
{
    "error": "Invalid Ethereum address" 
    // OR
    "error": "Publisher already exists"
}
```

**Code Location:** Line ~300-350

**What Happens After:**
- Success popup shown
- Publisher data stored in localStorage
- Redirects to publisher-dashboard.html after 2 seconds

---

## 4️⃣ publisher-dashboard.html

### API Call #1: Login / Verify Publisher

**Endpoint:** `GET /publisher/{ethereumAddress}`

**When Called:**
- User submits login form with wallet address
- On page load if wallet stored in session
- Every 30 seconds (auto-refresh)

**Request:**
```javascript
GET http://localhost:8080/publisher/0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1
```

**Response (200 - Success):**
```json
{
    "success": true,
    "publisher": {
        "id": "uuid",
        "name": "Rohit's Tech Blog",
        "ethereum_address": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
        "description": "Technology blog covering blockchain and AI",
        "website_url": "http://techblog.example.com",
        "assigned_campaign_id": "campaign-uuid",
        "total_earnings_wei": "250000000000000000",
        "created_at": "2025-11-15T12:00:00.000Z",
        "updated_at": "2025-11-15T15:00:00.000Z"
    }
}
```

**Response (404 - Not Found):**
```json
{
    "error": "Publisher not found"
}
```

**Code Location:** Line ~450-490

**What Happens After:**
- Login screen hidden
- Dashboard shown
- Wallet address displayed in header
- Site info card populated
- Triggers API Call #2 (fetch earnings)

---

### API Call #2: Fetch Publisher Earnings

**Endpoint:** `GET /publisher/{ethereumAddress}/earnings`

**When Called:**
- Immediately after successful login
- Every 30 seconds (auto-refresh)
- After withdrawal completes

**Request:**
```javascript
GET http://localhost:8080/publisher/0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1/earnings
```

**Response (200 - Success):**
```json
{
    "success": true,
    "earnings": [
        {
            "id": "earning-uuid-1",
            "publisher_address": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
            "publisher_name": "Rohit's Tech Blog",
            "campaign_id": "campaign-uuid",
            "campaign_name": "Summer Sale 2025",
            "captcha_id": "captcha-uuid-1",
            "amount_wei": "50000000000000000",
            "cost_per_captcha_wei": "50000000000000000",
            "earned_at": "2025-11-15T14:30:00.000Z"
        },
        {
            "id": "earning-uuid-2",
            "publisher_address": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
            "publisher_name": "Rohit's Tech Blog",
            "campaign_id": "campaign-uuid",
            "campaign_name": "Summer Sale 2025",
            "captcha_id": "captcha-uuid-2",
            "amount_wei": "50000000000000000",
            "cost_per_captcha_wei": "50000000000000000",
            "earned_at": "2025-11-15T14:35:00.000Z"
        }
    ],
    "totalEarningsWei": "100000000000000000"
}
```

**Code Location:** Line ~520-560

**What Happens After:**
- Earnings table populated with last 20 transactions
- Stats calculated:
  - Total Earnings (from totalEarningsWei)
  - Verifications (earnings.length)
  - This Week (sum of earnings from last 7 days)
  - Avg per CAPTCHA (total / count)

---

### API Call #3: Process Withdrawal

**Endpoint:** `POST /publisher/{ethereumAddress}/withdraw`

**When Called:**
- User clicks "Withdraw" in modal
- Form validates successfully
- Amount validated against available balance

**Request:**
```javascript
POST http://localhost:8080/publisher/0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1/withdraw
Content-Type: application/json

{
    "amountWei": "100000000000000000",                           // 0.1 ETH
    "withdrawalAddress": "0x9876543210987654321098765432109876543210"  // Optional
}
```

**Response (200 - Success):**
```json
{
    "success": true,
    "withdrawal": {
        "id": "withdrawal-uuid",
        "publisher_id": "publisher-uuid",
        "amount_wei": "100000000000000000",
        "withdrawal_address": "0x9876543210987654321098765432109876543210",
        "status": "completed",
        "transaction_hash": "0xabc123def456...",
        "failure_reason": null,
        "created_at": "2025-11-15T16:00:00.000Z",
        "processed_at": "2025-11-15T16:00:05.000Z"
    },
    "transactionHash": "0xabc123def456..."
}
```

**Response (400 - Insufficient Balance):**
```json
{
    "error": "Withdrawal amount exceeds available balance"
}
```

**Response (500 - Transaction Failed):**
```json
{
    "error": "Failed to process withdrawal request"
}
```

**Code Location:** Line ~600-680

**What Happens After:**
- Success message shown with transaction hash
- Link to block explorer displayed
- Dashboard reloads after 2 seconds
- Updated balance reflected
- Modal closes

---

## 5️⃣ canvas-widget.html

### API Call #1: Start CAPTCHA Session

**Endpoint:** `POST /captcha/start`

**When Called:**
- User clicks "Start Verification" button in widget
- Publisher address and campaign ID passed from parent window

**Request:**
```javascript
POST http://localhost:8080/captcha/start
Content-Type: application/json

{
    "campaignId": "campaign-uuid",
    "publisherAddress": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
    "sessionId": "session_1731676800123_abc123xyz",
    "userIpHash": null
}
```

**Response (201 - Success):**
```json
{
    "success": true,
    "captcha": {
        "id": "captcha-uuid",
        "campaign_id": "campaign-uuid",
        "publisher_id": "publisher-uuid",
        "session_id": "session_1731676800123_abc123xyz",
        "user_ip_hash": null,
        "status": "started",
        "payment_amount_wei": "50000000000000000",
        "payment_processed": false,
        "started_at": "2025-11-15T15:00:00.000Z",
        "completed_at": null
    }
}
```

**Response (400 - Campaign Not Active):**
```json
{
    "error": "Campaign is not active"
}
```

**Response (400 - Insufficient Budget):**
```json
{
    "error": "Campaign has insufficient budget"
}
```

**Code Location:** Line ~340-390

**What Happens After:**
- Captcha ID stored
- Game interface shown
- Timer starts
- User begins collecting Bitcoin items

---

### API Call #2: Complete CAPTCHA Session

**Endpoint:** `POST /captcha/{captchaId}/complete`

**When Called:**
- User successfully collects all Bitcoin items in game
- All 4 Bitcoin symbols clicked

**Request:**
```javascript
POST http://localhost:8080/captcha/captcha-uuid/complete
Content-Type: application/json

{
    "sessionId": "session_1731676800123_abc123xyz"
}
```

**Response (200 - Success):**
```json
{
    "success": true,
    "captcha": {
        "id": "captcha-uuid",
        "campaign_id": "campaign-uuid",
        "publisher_id": "publisher-uuid",
        "session_id": "session_1731676800123_abc123xyz",
        "user_ip_hash": null,
        "status": "completed",
        "payment_amount_wei": "50000000000000000",
        "payment_processed": true,
        "started_at": "2025-11-15T15:00:00.000Z",
        "completed_at": "2025-11-15T15:00:03.450Z"
    },
    "paymentProcessed": true,
    "paymentAmountWei": "50000000000000000"
}
```

**Response (400 - Already Completed):**
```json
{
    "error": "Cannot complete captcha (wrong status)"
}
```

**Response (400 - Session Mismatch):**
```json
{
    "error": "Invalid session ID"
}
```

**Code Location:** Line ~440-490

**What Happens After:**
- Success screen shown
- Publisher earnings amount displayed
- Message posted to parent window: `{type: 'CAPTCHA_COMPLETE'}`
- Parent window stores 24hr verification token
- Parent window hides overlay
- User can access site

---

## 6️⃣ index.html

### API Call #1: Fetch All Publishers (for domain matching)

**Endpoint:** `GET /publishers`

**When Called:**
- On page load (DOMContentLoaded event)
- Only if user doesn't have valid 24hr verification token

**Request:**
```javascript
GET http://localhost:8080/publishers
```

**Response (200 - Success):**
```json
{
    "success": true,
    "publishers": [
        {
            "id": "uuid-1",
            "name": "Rohit's Tech Blog",
            "ethereum_address": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
            "description": "Technology blog",
            "website_url": "http://techblog.example.com",
            "assigned_campaign_id": "campaign-uuid",
            "total_earnings_wei": "250000000000000000",
            "created_at": "2025-11-15T12:00:00.000Z",
            "updated_at": "2025-11-15T15:00:00.000Z"
        },
        {
            "id": "uuid-2",
            "name": "Crypto Weekly",
            "ethereum_address": "0x9876543210987654321098765432109876543210",
            "description": "Crypto news",
            "website_url": "https://cryptoweekly.com",
            "assigned_campaign_id": null,
            "total_earnings_wei": "0",
            "created_at": "2025-11-15T13:00:00.000Z",
            "updated_at": "2025-11-15T13:00:00.000Z"
        }
    ]
}
```

**Code Location:** Line ~220-280

**What Happens After:**
- Domain extracted from each website_url
- Compared with current domain (techblog.example.com)
- If match found → Continue to API Call #2
- If no match → Allow free browsing, no CAPTCHA

---

### API Call #2: Fetch Active Campaigns (budget validation)

**Endpoint:** `GET /campaigns`

**When Called:**
- Only if publisher found for current domain (after API Call #1 succeeds)

**Request:**
```javascript
GET http://localhost:8080/campaigns
```

**Response (200 - Success):**
```json
{
    "success": true,
    "campaigns": [
        {
            "id": "campaign-uuid-1",
            "advertiser_id": "advertiser-uuid",
            "name": "Summer Sale 2025",
            "description": "Promotional campaign",
            "cost_per_captcha_wei": "50000000000000000",
            "total_budget_wei": "2000000000000000000",
            "spent_budget_wei": "100000000000000000",
            "status": "active",
            "created_at": "2025-11-15T12:00:00.000Z",
            "updated_at": "2025-11-15T15:05:00.000Z",
            "advertisers": {
                "name": "Acme Corp",
                "ethereum_address": "0x1234567890123456789012345678901234567890"
            }
        },
        {
            "id": "campaign-uuid-2",
            "advertiser_id": "advertiser-uuid-2",
            "name": "Black Friday",
            "description": "Holiday sale",
            "cost_per_captcha_wei": "80000000000000000",
            "total_budget_wei": "50000000000000000",    // Only 0.05 ETH left
            "spent_budget_wei": "2950000000000000000",  // 2.95 ETH spent
            "status": "active",
            "created_at": "2025-11-14T10:00:00.000Z",
            "updated_at": "2025-11-15T14:00:00.000Z",
            "advertisers": {
                "name": "Nike",
                "ethereum_address": "0xabcdef1234567890abcdef1234567890abcdef12"
            }
        }
    ]
}
```

**Code Location:** Line ~290-360

**What Happens After:**
- Filter campaigns where status === 'active'
- For each active campaign, calculate:
  ```javascript
  remainingBudget = total_budget_wei - spent_budget_wei
  hasEnoughBudget = remainingBudget >= cost_per_captcha_wei
  ```
- Campaign #1: 1.9 ETH remaining >= 0.05 ETH → ✅ Valid
- Campaign #2: 0.05 ETH remaining >= 0.08 ETH → ❌ Invalid
- If ANY campaign has sufficient budget → Show CAPTCHA widget
- If NO campaigns have budget → Allow free browsing, no CAPTCHA
- Selected campaign passed to canvas-widget.html via URL params

---

## 📊 Complete API Flow Diagram

### User Journey: Advertiser Setup → Publisher Verification

```
1. ADVERTISER CREATES ACCOUNT
   POST /advertiser
   ↓
   
2. ADVERTISER LOGS IN
   GET /advertiser/{address}
   ↓
   GET /advertiser/{address}/campaigns (loads existing campaigns)
   ↓
   
3. ADVERTISER CREATES CAMPAIGN
   POST /advertiser/{address}/campaigns
   ↓
   GET /advertiser/{address}/campaigns (refresh)
   
4. PUBLISHER CREATES ACCOUNT
   POST /publisher
   ↓
   
5. PUBLISHER LOGS IN
   GET /publisher/{address}
   ↓
   GET /publisher/{address}/earnings (loads earnings history)
   
6. USER VISITS PUBLISHER SITE
   GET /publishers (check if site registered)
   ↓
   GET /campaigns (check for active campaigns with budget)
   ↓
   
7. CAPTCHA WIDGET LOADS
   POST /captcha/start (user clicks "Start Verification")
   ↓
   
8. USER COMPLETES GAME
   POST /captcha/{id}/complete
   ↓
   
9. PUBLISHER CHECKS EARNINGS
   GET /publisher/{address}/earnings (sees new transaction)
   ↓
   
10. PUBLISHER WITHDRAWS
    POST /publisher/{address}/withdraw
    ↓
    GET /publisher/{address}/earnings (balance updated)
```

---

## 🔢 API Call Statistics

### By File:
```
advertiser-signup.html:     1 API call
advertiser-dashboard.html:  3 API calls (1 login + 1 fetch + 1 create + auto-refresh)
publisher-signup.html:      1 API call
publisher-dashboard.html:   3 API calls (1 login + 1 fetch + 1 withdraw + auto-refresh)
canvas-widget.html:         2 API calls (1 start + 1 complete)
index.html:                 2 API calls (1 publishers + 1 campaigns)
```

### By HTTP Method:
```
GET:  6 endpoints (login, fetch campaigns, fetch earnings, publishers, campaigns, wallet)
POST: 5 endpoints (create advertiser, create publisher, create campaign, start captcha, complete captcha, withdraw)
```

### By Frequency:
```
One-time:
- POST /advertiser
- POST /publisher

Per Session:
- GET /advertiser/{address}
- GET /publisher/{address}
- GET /publishers
- GET /campaigns

Per Action:
- POST /advertiser/{address}/campaigns (each campaign creation)
- POST /publisher/{address}/withdraw (each withdrawal)
- POST /captcha/start (each verification start)
- POST /captcha/{id}/complete (each verification complete)

Auto-Refresh (Every 30s):
- GET /advertiser/{address}/campaigns (in advertiser dashboard)
- GET /publisher/{address}/earnings (in publisher dashboard)
```

---

## 🔐 Required Headers

All API calls include:
```javascript
headers: {
    'Content-Type': 'application/json'
}
```

No authentication headers currently implemented.

---

## ⚠️ Error Handling Pattern

All API calls follow this pattern:

```javascript
try {
    const response = await fetch(API_BASE_URL + endpoint, options);
    
    if (!response.ok) {
        const errorData = await response.json().catch(() => ({
            error: `HTTP ${response.status}: ${response.statusText}`
        }));
        throw new Error(errorData.error || 'API request failed');
    }
    
    const data = await response.json();
    
    if (!data.success) {
        throw new Error(data.error || 'Request failed');
    }
    
    // Process successful response
    
} catch (error) {
    console.error('API Error:', error);
    // Show user-friendly error message
}
```

---

## 📝 Notes for Deep

### Auto-Refresh Implementation:
```javascript
// In advertiser-dashboard.html and publisher-dashboard.html
setInterval(async () => {
    if (currentWalletAddress) {
        await loadCampaigns(); // or loadEarnings()
        updateStats();
    }
}, 30000); // 30 seconds
```

### Wei ↔ ETH Conversion:
```javascript
// Used before all API calls involving amounts
function ethToWei(eth) {
    return (parseFloat(eth) * 1e18).toString();
}

function weiToEth(wei) {
    return (parseFloat(wei) / 1e18).toFixed(6);
}
```

### Session ID Generation:
```javascript
// Used in canvas-widget.html
function generateSessionId() {
    return `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
}
```

---

## 🧪 Testing Each Endpoint

### Quick Test Commands (using curl):

**1. Create Advertiser:**
```bash
curl -X POST http://localhost:8080/advertiser \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Corp",
    "ethereumAddress": "0x1234567890123456789012345678901234567890",
    "description": "Test company",
    "campaignFundsWei": "5000000000000000000"
  }'
```

**2. Get Advertiser:**
```bash
curl http://localhost:8080/advertiser/0x1234567890123456789012345678901234567890
```

**3. Create Campaign:**
```bash
curl -X POST http://localhost:8080/advertiser/0x1234567890123456789012345678901234567890/campaigns \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Campaign",
    "costPerCaptchaWei": "50000000000000000",
    "totalBudgetWei": "1000000000000000000"
  }'
```

**4. Get Campaigns:**
```bash
curl http://localhost:8080/advertiser/0x1234567890123456789012345678901234567890/campaigns
```

**5. Create Publisher:**
```bash
curl -X POST http://localhost:8080/publisher \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Blog",
    "ethereumAddress": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
    "description": "Test blog",
    "websiteUrl": "http://testblog.com"
  }'
```

**6. Get Publisher:**
```bash
curl http://localhost:8080/publisher/0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1
```

**7. Get All Publishers:**
```bash
curl http://localhost:8080/publishers
```

**8. Get Active Campaigns:**
```bash
curl http://localhost:8080/campaigns
```

**9. Start CAPTCHA:**
```bash
curl -X POST http://localhost:8080/captcha/start \
  -H "Content-Type: application/json" \
  -d '{
    "campaignId": "campaign-uuid",
    "publisherAddress": "0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1",
    "sessionId": "test-session-123"
  }'
```

**10. Complete CAPTCHA:**
```bash
curl -X POST http://localhost:8080/captcha/captcha-uuid/complete \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "test-session-123"
  }'
```

**11. Get Earnings:**
```bash
curl http://localhost:8080/publisher/0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1/earnings
```

**12. Withdraw:**
```bash
curl -X POST http://localhost:8080/publisher/0x742d35cc6473c4ae23c9a24e0b365b7cef0623b1/withdraw \
  -H "Content-Type: application/json" \
  -d '{
    "amountWei": "100000000000000000"
  }'
```

---

**All 12 API calls documented and ready for integration!** 🚀
