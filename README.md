from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

# Database setup
conn = sqlite3.connect("quickbill.db", check_same_thread=False)
cursor = conn.cursor()
cursor.execute("""
CREATE TABLE IF NOT EXISTS api_keys (
    id INTEGER PRIMARY KEY,
    api_key TEXT
)
""")
conn.commit()

# Helper functions
def is_valid_key(key):
    cursor.execute("SELECT * FROM api_keys WHERE api_key=?", (key,))
    return cursor.fetchone() is not None

def generate_api_key():
    import random, string
    chars = string.ascii_uppercase + string.digits
    key = "QB-" + "".join(random.choice(chars) for _ in range(4)) + "-" + \
                "".join(random.choice(chars) for _ in range(4)) + "-" + \
                "".join(random.choice(chars) for _ in range(4))
    cursor.execute("INSERT INTO api_keys (api_key) VALUES (?)", (key,))
    conn.commit()
    return key

@app.route("/")
def home():
    return "QuickBill API is running"

@app.route("/create_invoice", methods=["POST"])
def create_invoice():
    api_key = request.headers.get("Authorization")
    if not is_valid_key(api_key):
        return jsonify({"error": "Invalid API key"}), 401

    data = request.json
    total = data.get("price", 0) * data.get("quantity", 1)
    return jsonify({
        "invoice_id": "QB-1001",
        "customer": data.get("customer"),
        "item": data.get("item"),
        "total": total,
        "status": "success"
    })

@app.route("/generate_key", methods=["GET"])
def generate_key_route():
    new_key = generate_api_key()
    return jsonify({"api_key": new_key})

if __name__ == "__main__":
    app.run()