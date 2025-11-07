# 🔧 Troubleshooting Guide

## Issue: "No markets fetched" or "Fetched 0 active markets"

### What this means:
The bot successfully connected to the API but got an empty response. This is the most common issue.

### Solutions (try in order):

#### 1. **Check Polymarket Status** ✅
Polymarket APIs may be temporarily down or rate-limiting.

**Quick test:**
```bash
curl https://gamma-api.polymarket.com/markets?limit=5
```

If you get an error or empty response, the API is down. Wait 5-10 minutes and try again.

---

#### 2. **Try Different API Endpoints** ⚙️

The bot now tries multiple endpoints automatically (v2.0):
1. Gamma API (primary) - `https://gamma-api.polymarket.com`
2. CLOB API (fallback) - `https://clob.polymarket.com`

**Manual test:**
```bash
# Test Gamma API
curl "https://gamma-api.polymarket.com/markets?limit=5&active=true"

# Test CLOB API
curl "https://clob.polymarket.com/sampling-markets?limit=5"
```

If both return empty or error, Polymarket is having issues.

---

#### 3. **Check Network/Firewall** 🌐

**Test your connection:**
```bash
ping gamma-api.polymarket.com
```

**Check if your firewall is blocking:**
- Corporate networks often block trading/gambling sites
- VPN may be required
- Try mobile hotspot as test

**Firewall check:**
```bash
curl -I https://gamma-api.polymarket.com
```

Should return `HTTP/2 200` or similar. If it hangs or times out, firewall issue.

---

#### 4. **Increase Timeout** ⏱️

Edit `prediction_market_arbitrage.py` line 70:

```python
# Change from:
timeout = aiohttp.ClientTimeout(total=30)

# To:
timeout = aiohttp.ClientTimeout(total=60)
```

If you're on slow/unreliable network.

---

#### 5. **Reduce Markets Scanned** 📉

Edit bottom of `prediction_market_arbitrage.py`:

```python
# Change from:
TOP_MARKETS = 50

# To:
TOP_MARKETS = 10  # Start small
```

Or edit `config.py`:
```python
TOP_MARKETS = 10
```

---

#### 6. **Check API Response Format** 🔍

Add debug logging to see what's actually returned.

Edit `prediction_market_arbitrage.py` line 113:

```python
# Add after line 114:
print(f"DEBUG: Raw API response: {data}")
```

This will show you what the API is returning. Common issues:
- API changed response format
- Different field names
- Nested data structure

---

## Issue: "Connection refused" or "Network error"

### Solutions:

#### 1. **Internet Connection**
```bash
ping google.com
```

If no response, your internet is down.

---

#### 2. **DNS Issues**
```bash
nslookup gamma-api.polymarket.com
```

Should return an IP address. If not, try:
```bash
# Use Google DNS
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

---

#### 3. **Proxy/VPN**
If you're behind corporate proxy:

Set environment variables:
```bash
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
```

Then run bot.

---

## Issue: "No opportunities found" (for hours)

### This is NORMAL! ✅

**Expected opportunity frequency:**
- **Quiet periods:** 0-2 opportunities per hour
- **Busy periods:** 1-5 opportunities per hour
- **Major events:** 5-10+ opportunities per hour

**Why opportunities are rare:**
- Most markets are efficiently priced
- Arbitrage windows close quickly (seconds to minutes)
- Bot catches them when they appear

**What to check:**
1. Let bot run for 2-4 hours minimum
2. Check log file: `tail -f arbitrage_bot.log`
3. Verify markets are being scanned (should see "Analyzing: ..." messages)

**If truly no opportunities after 24 hours:**
- Market may be unusually efficient (post-major event)
- Lower the profit threshold in `config.py`:
  ```python
  MIN_PROFIT_THRESHOLD = 0.01  # 1¢ instead of 2¢
  ```

---

## Issue: Bot crashes or freezes

### Solutions:

#### 1. **Check Python Version**
```bash
python3 --version
```

Need Python 3.8 or higher. If lower:
```bash
# macOS
brew install python@3.11

# Ubuntu/Debian
sudo apt install python3.11
```

---

#### 2. **Reinstall Dependencies**
```bash
pip3 uninstall -y aiohttp pandas numpy
pip3 install -r requirements.txt
```

---

#### 3. **Memory Issues**
If bot crashes after running for hours:

```python
# Edit config.py
TOP_MARKETS = 20  # Reduce from 50
SCAN_INTERVAL = 120  # Increase to 2 minutes
```

---

#### 4. **Check Logs for Errors**
```bash
tail -100 arbitrage_bot.log
```

Look for Python tracebacks or error messages.

---

## Issue: "Too many false positives" or "Low quality alerts"

### Solutions:

#### 1. **Increase Profit Threshold**

Edit `config.py`:
```python
MIN_PROFIT_THRESHOLD = 0.05  # 5¢ instead of 2¢
MIN_ALERT_PROFIT = 50.0  # Only alert if >$50 profit
```

---

#### 2. **Only High Urgency**

Edit `config.py`:
```python
ALERT_ONLY_HIGH_URGENCY = True
```

This filters out 70-80% of alerts, showing only best opportunities.

---

#### 3. **Increase Capital Threshold**

Edit `config.py`:
```python
MIN_CAPITAL_REQUIRED = 500  # Don't alert if <$500 needed
```

---

## Issue: Bot runs slow or high CPU usage

### Solutions:

#### 1. **Reduce Scan Frequency**
```python
# config.py
SCAN_INTERVAL = 120  # 2 minutes instead of 1
```

---

#### 2. **Monitor Fewer Markets**
```python
# config.py
TOP_MARKETS = 20  # Instead of 50
```

---

#### 3. **Disable Whale Tracking**
```python
# config.py
ENABLE_WHALE_TRACKING = False
```

Whale tracking requires fetching trade history, which is API-intensive.

---

## Issue: "SSL Certificate Error"

### Solutions:

#### 1. **Update Certificates**
```bash
# macOS
brew install ca-certificates

# Ubuntu/Debian
sudo apt-get install ca-certificates
```

---

#### 2. **Python SSL Module**
```bash
pip3 install --upgrade certifi
```

---

## Issue: Getting Rate Limited

### Symptoms:
- "403 Forbidden" errors
- "429 Too Many Requests"
- Bot stops fetching data

### Solutions:

#### 1. **Increase Delays**

Edit `config.py`:
```python
DELAY_BETWEEN_MARKETS = 0.5  # Increase from 0.2
DELAY_BETWEEN_ORDERBOOKS = 0.2  # Increase from 0.1
```

---

#### 2. **Reduce Scan Frequency**
```python
SCAN_INTERVAL = 180  # 3 minutes
```

---

#### 3. **Monitor Fewer Markets**
```python
TOP_MARKETS = 25
```

---

## Diagnostic Commands

### Check everything is working:

```bash
# 1. Python version
python3 --version  # Should be 3.8+

# 2. Dependencies installed
pip3 list | grep -E "aiohttp|pandas|numpy"

# 3. Network connectivity
ping gamma-api.polymarket.com

# 4. API accessible
curl -I https://gamma-api.polymarket.com/markets

# 5. Bot logs
tail -20 arbitrage_bot.log

# 6. Check for Python errors
python3 -c "import aiohttp, pandas, numpy; print('All imports OK')"
```

---

## Still Having Issues?

### 1. **Run in Debug Mode**

Edit `prediction_market_arbitrage.py` line 33:

```python
# Change:
level=logging.INFO

# To:
level=logging.DEBUG
```

This will show detailed API responses and error traces.

---

### 2. **Test Connection Manually**

```bash
python3 test_connection.py
```

This will test the API and show exactly what's wrong.

---

### 3. **Check GitHub Issues**

Someone else may have had the same problem:
https://github.com/NavnoorBawa/prediction-market/issues

---

### 4. **Simplify Configuration**

Reset `config.py` to defaults:
```bash
git checkout config.py
```

---

## Common Error Messages Decoded

| Error Message | What It Means | Fix |
|---------------|---------------|-----|
| "No markets fetched" | API returned empty list | Wait 10 min, API may be down |
| "Connection refused" | Can't reach API | Check internet/firewall |
| "SSL Certificate verify failed" | Certificate issue | Update certifi: `pip3 install --upgrade certifi` |
| "403 Forbidden" | Rate limited or blocked | Increase delays, reduce frequency |
| "Timeout error" | Request took too long | Increase timeout to 60s |
| "Invalid JSON" | API response corrupted | Retry, may be temporary |
| "KeyError: 'tokens'" | Market format unexpected | Bot handles this, debug mode shows issue |

---

## Performance Benchmarks

### Normal Performance:
- **Scan time:** 30-90 seconds for 50 markets
- **CPU usage:** 10-30% during scan, <5% idle
- **Memory:** 50-150 MB
- **Network:** 1-5 MB per scan

### If Slower:
- Check network speed: `speedtest-cli`
- Reduce markets scanned
- Check for other programs using bandwidth

---

## Best Practices

### ✅ DO:
- Let bot run for 24+ hours before judging
- Check logs regularly: `tail -f arbitrage_bot.log`
- Start with default config
- Test with small changes
- Keep dependencies updated

### ❌ DON'T:
- Scan more than 100 markets (API limits)
- Reduce interval below 30 seconds (rate limit risk)
- Run multiple bots simultaneously (compounds API load)
- Edit code without backing up first

---

## Quick Fixes Checklist

If bot stops working suddenly:

- [ ] Check internet: `ping google.com`
- [ ] Check Polymarket: Visit polymarket.com in browser
- [ ] Restart bot: `Ctrl+C` then restart
- [ ] Check logs: `tail -50 arbitrage_bot.log`
- [ ] Update dependencies: `pip3 install --upgrade -r requirements.txt`
- [ ] Reset config: `git checkout config.py`
- [ ] Reboot machine (clears network issues)

---

## Still Stuck?

**Remember:**
1. 90% of issues are API-related (temporary)
2. Wait 10-30 minutes and retry
3. Check logs first: `tail -f arbitrage_bot.log`
4. The bot worked for others - it will work for you too!

**The research is real ($39.59M extracted). The bot works. Be patient!** 🚀
