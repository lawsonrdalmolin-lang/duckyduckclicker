<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ducky Duck Clicker</title>
    <!-- Load Brython Engine -->
    <script type="text/javascript" src="https://cloudflare.com"></script>
    <script type="text/javascript" src="https://cloudflare.com"></script>
    <style>
        body { background-color: #1c1c24; color: white; font-family: Arial, sans-serif; text-align: center; margin: 0; padding: 20px; }
        .container { max-width: 600px; margin: 0 auto; }
        h1 { color: #ffd43b; }
        #duck-btn { font-size: 50px; background: #ffd43b; border: none; padding: 20px 40px; border-radius: 10px; cursor: pointer; margin: 20px; }
        #duck-btn:active { background: #ffe066; }
        .shop { background: #282834; padding: 15px; border-radius: 8px; max-height: 300px; overflow-y: scroll; margin-top: 20px; }
        .upgrade-item { display: flex; justify-content: space-between; background: #3f3f4e; margin: 5px 0; padding: 10px; border-radius: 4px; align-items: center; }
        .buy-btn { background: #6366f1; color: white; border: none; padding: 8px 12px; border-radius: 4px; cursor: pointer; }
        .secret-panel { margin: 20px 0; padding: 10px; background: #282834; border-radius: 6px; }
        input { padding: 5px; font-size: 16px; width: 120px; text-align: center; }
    </style>
</head>
<body onload="brython()">

<div class="container">
    <h1>🦆 DUCKY DUCK CLICKER 🦆</h1>
    <h2 id="duck-counter">0 Ducks</h2>
    <p id="dps-counter">per second: 0.0 | click power: 1</p>
    
    <div id="secret-status" style="color: #707080; font-style: italic;">💡 Tip: Enter the secret code to unlock the resting macro...</div>
    
    <div style="display: flex; justify-content: center; align-items: center;">
        <button id="duck-btn">🦆<br><span style="font-size:18px;">QUACK!</span></button>
        
        <div class="secret-panel" id="code-box">
            <label>Secret Code:</label><br>
            <input type="text" id="code-input"><br>
            <button id="unlock-btn" style="margin-top:5px; background:#4ade80; border:none; padding:5px 10px; border-radius:4px; cursor:pointer;">Unlock</button>
        </div>
    </div>

    <h3>--- UPGRADE SHOP ---</h3>
    <div class="shop" id="shop-container"></div>
</div>

<!-- Python Logic Running Directly in the Web Browser -->
<script type="text/python">
from browser import document, window, timer
import math

# Game State
ducks = 0.0
click_power = 1.0
dps = 0.0
secret_unlocked = False
auto_click_active = False

# Mouse position variables
last_mouse_x = 0
last_mouse_y = 0
last_move_time = window.Date.new().getTime()
rest_time_required = 1000 # 1 second in milliseconds

# Generate 250 upgrades
upgrades = []
for i in range(1, 126):
    upgrades.append({"id": len(upgrades), "name": f"Mega Quacker Mk.{i}", "type": "click", "cost": int(10 * (1.35 ** i)), "value": int(1 * (1.25 ** i)), "level": 0})
for i in range(1, 126):
    buildings = ["Duck Nest", "Pond Colony", "Bread Bakery", "Duck Factory", "Quack Universe"]
    upgrades.append({"id": len(upgrades), "name": f"{buildings[(i-1)%5]} v.{i}", "type": "dps", "cost": int(25 * (1.40 ** i)), "value": round(0.5 * (1.30 ** i), 1), "level": 0})

def update_ui():
    document["duck-counter"].text = f"{int(ducks)} Ducks"
    document["dps-counter"].text = f"per second: {round(dps, 1)} | click power: {int(click_power)}"

def manual_click(ev):
    global ducks
    ducks += click_power
    update_ui()

document["duck-btn"].bind("click", manual_click)

def check_code(ev):
    global secret_unlocked
    if document["code-input"].value.strip() == "12345678919":
        secret_unlocked = True
        document["secret-status"].text = "🔓 RESTING CLICKER UNLOCKED! Press the '|' key to toggle."
        document["secret-status"].style.color = "#4ade80"
        document["code-box"].style.display = "none"

document["unlock-btn"].bind("click", check_code)

def key_handler(ev):
    global auto_click_active
    if not secret_unlocked: return
    if ev.key == "|":
        auto_click_active = not auto_click_active
        if auto_click_active:
            document["secret-status"].text = "Auto-Clicker: ACTIVE 🟢 (Rest mouse to generate ducks)"
            document["secret-status"].style.color = "#4ade80"
        else:
            document["secret-status"].text = "Auto-Clicker: PAUSED 🔴 (Press | to Resume)"
            document["secret-status"].style.color = "#fb923c"

document.bind("keydown", key_handler)

def mouse_handler(ev):
    global last_mouse_x, last_mouse_y, last_move_time
    if ev.clientX != last_mouse_x or ev.clientY != last_mouse_y:
        last_mouse_x = ev.clientX
        last_mouse_y = ev.clientY
        last_move_time = window.Date.new().getTime()

document.bind("mousemove", mouse_handler)

def buy_upgrade(ev):
    global ducks, click_power, dps
    up_id = int(ev.target.attrs["data-id"])
    item = upgrades[up_id]
    if ducks >= item["cost"]:
        ducks -= item["cost"]
        item["level"] += 1
        if item["type"] == "click":
            click_power += item["value"]
        else:
            dps += item["value"]
        item["cost"] = int(item["cost"] * 1.3)
        ev.target.text = f"Cost: {item['cost']}"
        document[f"name-{up_id}"].text = f"{item['name']} (Lvl {item['level']})"
        update_ui()

# Render Shop
shop = document["shop-container"]
for item in upgrades:
    row = document.createElement("div")
    row.className = "upgrade-item"
    row.innerHTML = f"<span id='name-{item['id']}'>{item['name']} (Lvl 0)</span> <span>{item['value']} {'Power' if item['type']=='click' else 'DPS'}</span>"
    btn = document.createElement("button")
    btn.className = "buy-btn"
    btn.text = f"Cost: {item['cost']}"
    btn.attrs["data-id"] = item["id"]
    btn.bind("click", buy_upgrade)
    row.appendChild(btn)
    shop.appendChild(row)

# Game Loops
def game_loop():
    global ducks
    if dps > 0:
        ducks += dps * 0.1
        update_ui()

def macro_loop():
    global ducks
    now = window.Date.new().getTime()
    if secret_unlocked and auto_click_active and (now - last_move_time >= rest_time_required):
        ducks += max(click_power, dps * 0.5)
        update_ui()

timer.set_interval(game_loop, 100)
timer.set_interval(macro_loop, 100)
</script>
</body>
</html>
