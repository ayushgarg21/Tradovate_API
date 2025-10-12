# Tradovate Trading Bot

A Flask-based webhook receiver that executes trades on Tradovate with trailing stop loss management.

## What it does

- Listens for trade signals via webhook and executes them on Tradovate
- Monitors profit in real-time and places trailing stop loss orders when thresholds are hit
- Adjusts monitoring speed based on NY market hours (faster during 9:30am-4pm ET)
- Auto-exits trades after 12 minutes if still open
- Handles multiple monitoring threads simultaneously

## Requirements

Python 3.7+ and these packages:
```bash
pip install flask requests python-dotenv pytz
```

You'll also need a Tradovate account with API access.

## Setup

1. Clone this repo

2. Create a `bot.env` file with your Tradovate credentials:

```env
TRADOVATE_USERNAME=your_username
TRADOVATE_PASSWORD=your_password
TRADOVATE_APP_ID=your_app_id
TRADOVATE_DEVICE_ID=your_device_id
TRADOVATE_CLIENT_ID=your_client_id
TRADOVATE_CLIENT_SECRET=your_client_secret
TRADOVATE_ACCOUNT_ID=your_account_id
TRADOVATE_ACCOUNT_SPEC=your_account_spec
WEBHOOK_SECRET=your_webhook_secret
```

Don't commit this file - add it to `.gitignore`

3. Install dependencies:
```bash
pip install flask requests python-dotenv pytz
```

4. Run it:
```bash
python bot.py
```

## How to send trades

POST to `/webhook` with this JSON:

```json
{
  "secret": "your_webhook_secret",
  "side": "buy",
  "entry": "21500.00",
  "tp": "21550.00",
  "sl": "21480.00",
  "trail_sl_trigger_profit_dollars": "30",
  "trail_amt": "5",
  "tls": "21530.00"
}
```

**Required fields:**
- `secret` - matches WEBHOOK_SECRET in your .env
- `side` - "buy" or "sell"  
- `entry` - entry price (just for reference)
- `tp` - take profit price
- `sl` - stop loss price

**Optional (for trailing stop loss):**
- `trail_sl_trigger_profit_dollars` - profit amount that triggers TSL
- `trail_amt` - how much the stop trails by
- `tls` - initial trailing stop price

## Settings you can change

**Switch to demo:** Change all `live.tradovateapi.com` URLs to `demo.tradovateapi.com`

**Trading symbol:** Currently set to MNQZ5, update in the order payloads if needed

**Monitoring speeds:**
- NY session (9:30am-4pm ET): checks every 0.3 seconds
- Outside NY hours: checks every 0.5 seconds  
- Auto-exit timer: 12 minutes

**NY session hours:** Modify the `is_ny_session()` function

## Security stuff

All credentials are loaded from environment variables, never hardcoded. The logging system hides sensitive data. There's also a 10 second cooldown between trades and webhook authentication.

## Logs

Check the console output for:
- When trades are placed
- TSL triggers
- Position updates
- Any errors or rate limits
- Current session status

## Notes

Test on demo first. The bot stops TSL monitoring if a trade is losing more than $20. Rate limits are handled but you might still hit them with very aggressive settings.

## Common issues

**Won't start:** Make sure all variables are in `bot.env` and dependencies are installed

**Orders failing:** Check your API credentials and account margin

**Rate limits:** Increase the sleep intervals in `monitor_tsl()` function----> this is very important , during the NY session the the api limit will be hit if ur in a trade for more than 15 minutes iirc.
