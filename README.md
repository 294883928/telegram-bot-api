from flask import Flask, request
import requests
import os

# Use your bot token here (for learning/demo - in production, use env vars)
TOKEN = "7801617115:AAFg_Onl9I8Ewm_nQ6ZIIE-2n6iPf_-8IpY"
TELEGRAM_API = f'https://api.telegram.org/bot{TOKEN}'

app = Flask(__Meme DOCF__)

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.get_json()
    message = data.get("message")
    if not message or 'text' not in message:
        return '', 200
    
    chat_id = message["chat"]["id"]
    user_text = message["text"].strip()

    if user_text.startswith('/tokens'):
        parts = user_text.split()
        if len(parts) == 2:
            wallet_address = parts[1]
            tokens = get_solana_wallet_tokens(wallet_address)
            reply = format_tokens_reply(tokens, wallet_address)
        else:
            reply = "Usage: /tokens WALLET_ADDRESS"
    elif user_text.startswith('/pair'):
        parts = user_text.split()
        if len(parts) == 3:
            chain = parts[1]
            pair_address = parts[2]
            info = get_dexscreener_pair_info(chain, pair_address)
            reply = format_pair_reply(info, pair_address)
        else:
            reply = "Usage: /pair CHAIN PAIR_ADDRESS"
    else:
        reply = "Commands:\n/tokens WALLET_ADDRESS\n/pair CHAIN PAIR_ADDRESS"
    
    send_message(chat_id, reply)
    return '', 200

def send_message(chat_id, text):
    url = f"{TELEGRAM_API}/sendMessage"
    payload = {
        "chat_id": chat_id,
        "text": text
    }
    requests.post(url, json=payload)

def get_solana_wallet_tokens(wallet_address):
    url = f"https://public-api.solscan.io/account/tokens?account={wallet_address}"
    r = requests.get(url)
    if r.status_code == 200:
        return r.json()
    return None

def format_tokens_reply(tokens, wallet_address):
    if not tokens or isinstance(tokens, dict) and tokens.get("message"):
        return f"Could not fetch tokens for {wallet_address}."
    reply = f"Tokens for {wallet_address}:\n"
    for t in tokens[:10]:  # Show only first 10 for brevity
        mint = t.get("tokenAddress", "N/A")
        amount = t.get("tokenAmount", {}).get("uiAmountString", "0")
        symbol = t.get("tokenSymbol", "")
        reply += f"{symbol or mint}: {amount}\n"
    if len(tokens) > 10:
        reply += f"...and {len(tokens)-10} more."
    return reply

def get_dexscreener_pair_info(chain, pair_address):
    url = f"https://api.dexscreener.com/latest/dex/pairs/{chain}/{pair_address}"
    r = requests.get(url)
    if r.status_code == 200:
        return r.json()
    return None

def format_pair_reply(info, pair_address):
    if not info or not info.get("pair"):
        return f"Could not fetch info for pair {pair_address}."
    pair = info["pair"]
    base = pair.get("baseToken", {}).get("symbol", "")
    quote = pair.get("quoteToken", {}).get("symbol", "")
    price = pair.get("priceUsd", "N/A")
    vol24h = pair.get("volume", {}).get("h24", "N/A")
    reply = (
        f"{base}/{quote} ({pair_address})\n"
        f"Price: ${price}\n"
        f"24h Volume: {vol24h}\n"
        f"More: {pair.get('url', 'N/A')}"
    )
    return reply

if __name__ == "__main__":
    PORT = int(os.environ.get("PORT", 3000))
    app.run(host="0.0.0.0", port=PORT)
    https://api.telegram.org/bot<BOT_TOKEN>/setWebhook?url=<7801617115:AAFg_Onl9I8Ewm_nQ6ZIIE-2n6iPf_-8IpY>
