# Canvas Protocol CORS Configuration Guide

## 🔍 CORS Troubleshooting Steps

### Step 1: Identify CORS Errors

**Browser Console Errors:**
```
Access to fetch at 'http://34.75.24.60:8080/advertiser' from origin 
'https://publisher-site.com' has been blocked by CORS policy: 
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

**Common CORS Error Messages:**
- `No 'Access-Control-Allow-Origin' header`
- `CORS policy: Request header field content-type is not allowed`
- `Method POST is not allowed by Access-Control-Allow-Methods`
- `Preflight request doesn't pass access control check`

### Step 2: Check Network Tab

1. Open Browser DevTools (F12)
2. Go to **Network** tab
3. Submit form and look for:
   - **OPTIONS request** (preflight) - Should return 200
   - **POST/GET request** - Check response headers

**What to look for in Response Headers:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
```

### Step 3: Test Backend Directly

```bash
# Test without CORS (using curl)
curl -X POST http://34.75.24.60:8080/advertiser \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","ethereumAddress":"0x123...","description":"Test"}'

# If this works but browser doesn't → CORS issue
# If this fails → Backend issue (not CORS)
```

### Step 4: Check Backend CORS Configuration

Look for CORS middleware in your backend:
```javascript
app.use(cors({...}));
```

If missing → CORS not configured
If present → Check configuration

---

## 🌐 CORS Solutions for Canvas Protocol

### ❌ Solution 1: Whitelist Specific Domains (NOT SCALABLE)

**Configuration:**
```javascript
const cors = require('cors');

const allowedOrigins = [
  'https://publisher1.com',
  'https://publisher2.com',
  'https://publisher3.com',
  // Need to add every publisher manually
];

app.use(cors({
  origin: function(origin, callback) {
    if (!origin || allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
}));
```

**Problems for Canvas Protocol:**
- ❌ Can't add new publishers without deploying backend
- ❌ Doesn't scale with 1000+ publishers
- ❌ Publishers can't self-serve signup
- ❌ Defeats the purpose of an open marketplace

---

### ✅ Solution 2: Allow All Origins (RECOMMENDED FOR CANVAS)

**Configuration:**
```javascript
const cors = require('cors');

app.use(cors({
  origin: '*',  // Allow requests from ANY domain
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: false  // Important: Must be false when origin is *
}));
```

**Why this works for Canvas Protocol:**
- ✅ Any publisher can integrate immediately
- ✅ Self-service signup works
- ✅ No backend changes needed for new publishers
- ✅ Scales to unlimited publishers
- ✅ Standard for public APIs (Stripe, Twilio, etc. all use this)

**Security Considerations:**
- ✅ Still secure because you validate Ethereum addresses
- ✅ Publishers must have registered Ethereum address
- ✅ Payments require valid signatures
- ✅ Rate limiting prevents abuse
- ✅ No sensitive data exposed in public endpoints

---

### ⚖️ Solution 3: Hybrid Approach (MAXIMUM SECURITY)

Allow all origins BUT protect sensitive operations:

```javascript
const cors = require('cors');

// Public endpoints - allow all origins
const publicCors = cors({
  origin: '*',
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type']
});

// Protected endpoints - require authentication
const authenticatedCors = cors({
  origin: function(origin, callback) {
    // Verify publisher is registered in database
    checkPublisherRegistered(origin).then(isValid => {
      callback(null, isValid);
    });
  },
  credentials: true
});

// Apply different CORS policies to different routes
app.post('/publisher', publicCors, createPublisher);  // Public signup
app.post('/advertiser', publicCors, createAdvertiser);  // Public signup
app.post('/captcha/start', publicCors, startCaptcha);  // Public - any site can verify humans

app.post('/publisher/:address/withdraw', authenticatedCors, requireAuth, withdraw);  // Protected
app.patch('/campaigns/:id', authenticatedCors, requireAuth, updateCampaign);  // Protected
```

---

## 🔒 Security Best Practices (Even with Open CORS)

### 1. API Key Authentication for Sensitive Operations

```javascript
// Publishers get an API key when they register
app.post('/publisher/:address/withdraw', async (req, res) => {
  const apiKey = req.headers['x-api-key'];
  
  // Verify API key belongs to this publisher
  const publisher = await db.query(
    'SELECT * FROM publishers WHERE ethereum_address = $1 AND api_key = $2',
    [req.params.address, apiKey]
  );
  
  if (!publisher.rows[0]) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  // Process withdrawal
});
```

### 2. Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// Prevent spam/abuse
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

app.use('/advertiser', limiter);
app.use('/publisher', limiter);
```

### 3. Input Validation

```javascript
const { body, validationResult } = require('express-validator');

app.post('/advertiser', [
  body('ethereumAddress').matches(/^0x[a-fA-F0-9]{40}$/),
  body('name').isLength({ min: 1, max: 100 }),
  body('campaignFundsWei').isNumeric(),
], async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Process request
});
```

### 4. Ethereum Signature Verification

```javascript
const { ethers } = require('ethers');

// For sensitive operations, require signed messages
app.post('/publisher/:address/withdraw', async (req, res) => {
  const { amountWei, signature } = req.body;
  
  // Verify signature
  const message = `Withdraw ${amountWei} wei`;
  const recoveredAddress = ethers.utils.verifyMessage(message, signature);
  
  if (recoveredAddress.toLowerCase() !== req.params.address.toLowerCase()) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process withdrawal
});
```

### 5. Domain Validation (Optional)

```javascript
// Verify publisher's registered domain matches request origin
app.post('/captcha/start', async (req, res) => {
  const { publisherAddress } = req.body;
  const origin = req.headers.origin;
  
  const publisher = await db.query(
    'SELECT website_url FROM publishers WHERE ethereum_address = $1',
    [publisherAddress]
  );
  
  if (publisher.rows[0] && origin) {
    const registeredDomain = new URL(publisher.rows[0].website_url).origin;
    if (origin !== registeredDomain) {
      console.warn(`Warning: Captcha request from ${origin} but publisher registered ${registeredDomain}`);
      // Log but still allow (publisher might have multiple domains)
    }
  }
  
  // Process captcha
});
```

---

## 🚀 Recommended CORS Configuration for Canvas Protocol

### Full Backend Implementation

```javascript
const express = require('express');
const cors = require('cors');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');

const app = express();

// 1. Security headers
app.use(helmet({
  crossOriginResourcePolicy: { policy: "cross-origin" }
}));

// 2. CORS - Allow all origins (standard for public APIs)
app.use(cors({
  origin: '*',
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key'],
  credentials: false
}));

// 3. Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
});

// 4. Parse JSON
app.use(express.json());

// 5. Apply rate limiting to signup endpoints
app.use('/advertiser', limiter);
app.use('/publisher', limiter);

// 6. More restrictive rate limit for captcha (prevent bot farms)
const captchaLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 10, // 10 captchas per minute per IP
});
app.use('/captcha', captchaLimiter);

// 7. Withdrawal requires stricter rate limiting
const withdrawalLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 withdrawals per hour per IP
});
app.use('/publisher/:address/withdraw', withdrawalLimiter);

// ... Your API routes here

app.listen(8080, () => {
  console.log('Server running on port 8080');
  console.log('CORS enabled for all origins');
});
```

---

## 📊 Comparison: CORS Approaches

| Approach | Scalability | Security | Publisher UX | Recommended for Canvas? |
|----------|-------------|----------|--------------|------------------------|
| Whitelist domains | ❌ Poor | ✅ High | ❌ Manual approval | ❌ No |
| Allow all origins | ✅ Excellent | ⚠️ Medium* | ✅ Self-service | ✅ **YES** |
| Hybrid (public + auth) | ✅ Excellent | ✅ High | ✅ Self-service | ✅ **YES** |

*Security is still strong with proper authentication, validation, and rate limiting

---

## 🎯 Action Items for Canvas Protocol

### Immediate (This Week)

1. **Update backend CORS to allow all origins:**
   ```javascript
   app.use(cors({ origin: '*' }));
   ```

2. **Add rate limiting:**
   ```bash
   npm install express-rate-limit
   ```

3. **Test with your frontend:**
   - Deploy frontend to GitHub Pages
   - Verify signup forms work
   - Check browser console for errors

### Short-term (This Month)

4. **Add API key authentication:**
   - Generate API key when publisher signs up
   - Store in database
   - Require for withdrawals and sensitive operations

5. **Add input validation:**
   ```bash
   npm install express-validator
   ```

6. **Implement signature verification for withdrawals:**
   - Require Ethereum signature for withdrawals
   - Verify signature matches publisher address

### Long-term (Production Ready)

7. **Add monitoring:**
   - Log all API requests
   - Track unusual patterns
   - Alert on rate limit violations

8. **Add domain verification (optional):**
   - Compare request origin to registered website_url
   - Log mismatches for fraud detection

9. **Consider CDN/API Gateway:**
   - Cloudflare, AWS API Gateway, or Kong
   - Advanced rate limiting
   - DDoS protection
   - Geographic distribution

---

## 🔧 Testing Your CORS Configuration

### Test 1: Browser Console
```javascript
// Paste in browser console on ANY website
fetch('http://34.75.24.60:8080/wallet')
  .then(r => r.json())
  .then(d => console.log('Success:', d))
  .catch(e => console.log('Error:', e));

// Should work if CORS is configured correctly
```

### Test 2: Different Domains
```bash
# Test from different domains
curl -H "Origin: https://example.com" \
     -H "Access-Control-Request-Method: POST" \
     -H "Access-Control-Request-Headers: Content-Type" \
     -X OPTIONS \
     http://34.75.24.60:8080/advertiser

# Should return Access-Control-Allow-Origin header
```

### Test 3: Actual Publisher Integration
1. Deploy frontend to GitHub Pages
2. Submit advertiser signup form
3. Check Network tab for successful POST request
4. Verify response in browser console

---

## 💡 Key Insights for Canvas Protocol

**Your business model REQUIRES open CORS:**
- ✅ Publishers self-serve signup
- ✅ Anyone can integrate Canvas widget
- ✅ No manual approval process
- ✅ Scales to thousands of publishers

**This is how successful API platforms work:**
- Stripe: `origin: '*'`
- Twilio: `origin: '*'`
- Plaid: `origin: '*'`
- They protect sensitive operations with API keys and signatures, not CORS

**CORS is NOT your primary security layer:**
- CORS only works in browsers (easily bypassed with curl/Postman)
- Use authentication, validation, rate limiting, and signatures instead
- CORS is just to make browsers happy

---

## 🎓 Further Reading

- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Express CORS Middleware](https://expressjs.com/en/resources/middleware/cors.html)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Ethereum Signature Verification](https://docs.ethers.org/v5/api/utils/signing-key/)

---

## ✅ TL;DR: Recommended Solution

```javascript
// Backend CORS configuration
app.use(cors({ origin: '*' }));

// Security layers:
// 1. Rate limiting (prevent spam)
// 2. Input validation (prevent injection)
// 3. API keys (authenticate publishers)
// 4. Ethereum signatures (verify ownership)
// 5. Database checks (verify registration)

// This allows ANY publisher to integrate while maintaining security
```

**This is the standard approach for public APIs and the only scalable solution for Canvas Protocol's marketplace model.**
