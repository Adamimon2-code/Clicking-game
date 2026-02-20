import json
import os
import time
import random
from kivy.app import App
from kivy.uix.label import Label
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.scrollview import ScrollView
from kivy.uix.behaviors import ButtonBehavior
from kivy.uix.image import Image
from kivy.uix.button import Button
from kivy.uix.popup import Popup
from kivy.uix.widget import Widget
from kivy.clock import Clock
from kivy.core.window import Window
from kivy.graphics import Color, RoundedRectangle, Ellipse, Rectangle
from kivy.animation import Animation
from kivy.uix.progressbar import ProgressBar
from kivy.uix.textinput import TextInput
from kivy.metrics import dp

# ─────────────────────────────────────────────
# Window setup
# ─────────────────────────────────────────────
Window.clearcolor = (0.06, 0.04, 0.02, 1)

SAVE_FILE = "orange_clicker_v3.json"

# ─────────────────────────────────────────────
# COLOR PALETTE
# ─────────────────────────────────────────────
C = {
    "bg":       (0.06, 0.04, 0.02, 1),
    "panel":    (0.12, 0.08, 0.03, 1),
    "orange":   (1.0,  0.55, 0.0,  1),
    "orange2":  (1.0,  0.35, 0.0,  1),
    "gold":     (1.0,  0.80, 0.1,  1),
    "green":    (0.15, 0.85, 0.3,  1),
    "blue":     (0.2,  0.6,  1.0,  1),
    "purple":   (0.7,  0.2,  1.0,  1),
    "red":      (1.0,  0.2,  0.2,  1),
    "white":    (1.0,  1.0,  1.0,  1),
    "dim":      (0.6,  0.5,  0.3,  1),
    "dark":     (0.08, 0.05, 0.01, 1),
}

# ─────────────────────────────────────────────
# GENERATE 250 UPGRADES
# ─────────────────────────────────────────────
def generate_upgrades():
    upgrades = []
    
    # TAP UPGRADES (100 upgrades)
    base_tap_cost = 15
    base_tap_value = 2
    for i in range(1, 101):
        cost = int(base_tap_cost * (1.5 ** (i - 1)))
        value = int(base_tap_value * (1.3 ** (i - 1)))
        icon = ["👆", "💪", "⚡", "🔮", "🦾", "🌩", "☀️", "🌌", "🌠", "💥"][i % 10]
        
        tier = (i - 1) // 10 + 1
        tier_names = ["Basic", "Advanced", "Elite", "Master", "Epic", "Legendary", "Mythic", "Divine", "Cosmic", "Transcendent"]
        tier_name = tier_names[min(tier - 1, 9)]
        
        upgrades.append({
            "id": f"tap{i}",
            "name": f"{tier_name} Tap #{i}",
            "desc": f"+{value} Data/tap",
            "cost": cost,
            "type": "tap",
            "value": value,
            "icon": icon
        })
    
    # AUTO UPGRADES (100 upgrades)
    base_auto_cost = 50
    base_auto_value = 2
    for i in range(1, 101):
        cost = int(base_auto_cost * (1.6 ** (i - 1)))
        value = int(base_auto_value * (1.35 ** (i - 1)))
        icon = ["🚁", "🏭", "🌾", "🧠", "🌀", "⚛️", "🛰️", "🪐", "🌌", "🤖"][i % 10]
        
        tier = (i - 1) // 10 + 1
        tier_names = ["Basic", "Advanced", "Elite", "Master", "Epic", "Legendary", "Mythic", "Divine", "Cosmic", "Transcendent"]
        tier_name = tier_names[min(tier - 1, 9)]
        
        upgrades.append({
            "id": f"auto{i}",
            "name": f"{tier_name} Auto #{i}",
            "desc": f"+{value}/sec",
            "cost": cost,
            "type": "auto",
            "value": value,
            "icon": icon
        })
    
    # MULTIPLIER UPGRADES (30 upgrades)
    base_mult_cost = 500
    for i in range(1, 31):
        cost = int(base_mult_cost * (10 ** (i - 1)))
        value = 1.5 + (i * 0.1)
        icon = ["✨", "🍊", "🌟", "🌌", "💫", "🔆", "🕳️", "⚛️", "🌐", "🔮"][i % 10]
        
        upgrades.append({
            "id": f"mult{i}",
            "name": f"Multiplier Tier {i}",
            "desc": f"x{value:.1f} all income",
            "cost": cost,
            "type": "mult",
            "value": value,
            "icon": icon
        })
    
    # SPECIAL UPGRADES (20 upgrades)
    special_upgrades = [
        {"id": "spec1", "name": "Lucky Peel", "desc": "+5% golden chance", "cost": 2_000, "type": "spec", "value": 0.05, "icon": "🍀"},
        {"id": "spec2", "name": "Extended Zest", "desc": "+10s golden time", "cost": 10_000, "type": "spec", "value": 10, "icon": "⏰"},
        {"id": "spec3", "name": "Double Squeeze", "desc": "2x tap power", "cost": 25_000, "type": "spec", "value": 2, "icon": "✌️"},
        {"id": "spec4", "name": "Vitamin C Surge", "desc": "Faster auto ticks", "cost": 80_000, "type": "spec", "value": 2, "icon": "💊"},
        {"id": "spec5", "name": "Offline Booster", "desc": "24h offline cap", "cost": 200_000, "type": "spec", "value": 24, "icon": "💤"},
        {"id": "spec6", "name": "Data Crystal", "desc": "+2% all data", "cost": 1_000_000, "type": "spec", "value": 0.02, "icon": "💎"},
        {"id": "spec7", "name": "Orange Nova", "desc": "5x tap boost", "cost": 5_000_000, "type": "spec", "value": 5, "icon": "💥"},
        {"id": "spec8", "name": "Infinite Peel", "desc": "x2 all forever", "cost": 50_000_000, "type": "mult", "value": 2.0, "icon": "♾️"},
        {"id": "spec9", "name": "Quantum Zest", "desc": "+10% all income", "cost": 100_000_000, "type": "spec", "value": 0.1, "icon": "🔬"},
        {"id": "spec10", "name": "Cosmic Juice", "desc": "x3 multiplier", "cost": 500_000_000, "type": "mult", "value": 3.0, "icon": "🌠"},
        {"id": "spec11", "name": "Time Warp", "desc": "+50s golden time", "cost": 1_000_000_000, "type": "spec", "value": 50, "icon": "⏳"},
        {"id": "spec12", "name": "Mega Boost", "desc": "x5 all income", "cost": 10_000_000_000, "type": "mult", "value": 5.0, "icon": "🚀"},
        {"id": "spec13", "name": "Ultra Peel", "desc": "+20% golden", "cost": 50_000_000_000, "type": "spec", "value": 0.2, "icon": "✨"},
        {"id": "spec14", "name": "Hyper Drive", "desc": "x10 multiplier", "cost": 100_000_000_000, "type": "mult", "value": 10.0, "icon": "⚡"},
        {"id": "spec15", "name": "God Mode", "desc": "x50 all", "cost": 500_000_000_000, "type": "mult", "value": 50.0, "icon": "👑"},
        {"id": "spec16", "name": "Divine Touch", "desc": "+100 tap power", "cost": 1_000_000_000_000, "type": "spec", "value": 100, "icon": "🙏"},
        {"id": "spec17", "name": "Celestial Boost", "desc": "x100 mult", "cost": 10_000_000_000_000, "type": "mult", "value": 100.0, "icon": "🌟"},
        {"id": "spec18", "name": "Reality Bend", "desc": "+50% all", "cost": 100_000_000_000_000, "type": "spec", "value": 0.5, "icon": "🌀"},
        {"id": "spec19", "name": "Omnipotence", "desc": "x1000 mult", "cost": 1_000_000_000_000_000, "type": "mult", "value": 1000.0, "icon": "💫"},
        {"id": "spec20", "name": "Infinity Stone", "desc": "Ultimate power", "cost": 999_999_999_999_999_999, "type": "mult", "value": 10000.0, "icon": "💠"},
    ]
    upgrades.extend(special_upgrades)
    
    return upgrades

UPGRADES = generate_upgrades()

ACHIEVEMENTS = [
    # ── Data collection milestones ───────────────
    {"id": "ach1",  "name": "First Squeeze",        "desc": "Collect 100 Data",          "req": 100,            "type": "total", "reward": 50},
    {"id": "ach2",  "name": "Juice Addict",          "desc": "Collect 1K Data",           "req": 1_000,          "type": "total", "reward": 200},
    {"id": "ach3",  "name": "Orange Baron",          "desc": "Collect 10K Data",          "req": 10_000,         "type": "total", "reward": 1_000},
    {"id": "ach4",  "name": "Data Lord",             "desc": "Collect 100K Data",         "req": 100_000,        "type": "total", "reward": 5_000},
    {"id": "ach5",  "name": "Citrus God",            "desc": "Collect 1M Data",           "req": 1_000_000,      "type": "total", "reward": 25_000},
    {"id": "ach6",  "name": "Mega Squeezer",         "desc": "Collect 10M Data",          "req": 10_000_000,     "type": "total", "reward": 100_000},
    {"id": "ach7",  "name": "Giga Juicer",           "desc": "Collect 100M Data",         "req": 100_000_000,    "type": "total", "reward": 500_000},
    {"id": "ach8",  "name": "Tera Tapper",           "desc": "Collect 1B Data",           "req": 1_000_000_000,  "type": "total", "reward": 2_000_000},
    {"id": "ach9",  "name": "Cosmic Collector",      "desc": "Collect 10B Data",          "req": 10_000_000_000, "type": "total", "reward": 10_000_000},
    {"id": "ach10", "name": "Universe Hoarder",      "desc": "Collect 1T Data",           "req": 1_000_000_000_000,"type":"total","reward": 50_000_000},
    # ── Auto income milestones ───────────────────
    {"id": "ach11", "name": "Auto Pilot",            "desc": "Reach 50/sec",              "req": 50,             "type": "auto",  "reward": 300},
    {"id": "ach12", "name": "Speed Demon",           "desc": "Reach 500/sec",             "req": 500,            "type": "auto",  "reward": 2_000},
    {"id": "ach13", "name": "Income Machine",        "desc": "Reach 5K/sec",              "req": 5_000,          "type": "auto",  "reward": 15_000},
    {"id": "ach14", "name": "Data Tsunami",          "desc": "Reach 50K/sec",             "req": 50_000,         "type": "auto",  "reward": 100_000},
    {"id": "ach15", "name": "Galaxy Stream",         "desc": "Reach 500K/sec",            "req": 500_000,        "type": "auto",  "reward": 750_000},
    {"id": "ach16", "name": "Infinite River",        "desc": "Reach 5M/sec",              "req": 5_000_000,      "type": "auto",  "reward": 5_000_000},
    # ── Tap milestones ───────────────────────────
    {"id": "ach17", "name": "Button Smasher",        "desc": "Tap 100 times",             "req": 100,            "type": "taps",  "reward": 500},
    {"id": "ach18", "name": "Tap Machine",           "desc": "Tap 1K times",              "req": 1_000,          "type": "taps",  "reward": 2_000},
    {"id": "ach19", "name": "Finger Legend",         "desc": "Tap 10K times",             "req": 10_000,         "type": "taps",  "reward": 20_000},
    {"id": "ach20", "name": "The Prestige Master",   "desc": "Prestige 5 times",          "req": 5,              "type": "prestige","reward":500_000},
    # ── Upgrade milestones ───────────────────────
    {"id": "ach21", "name": "Upgrade Novice",        "desc": "Buy 10 upgrades",           "req": 10,             "type": "upgrades", "reward": 5_000},
    {"id": "ach22", "name": "Upgrade Expert",        "desc": "Buy 50 upgrades",           "req": 50,             "type": "upgrades", "reward": 50_000},
    {"id": "ach23", "name": "Upgrade Master",        "desc": "Buy 100 upgrades",          "req": 100,            "type": "upgrades", "reward": 500_000},
    {"id": "ach24", "name": "Upgrade Legend",        "desc": "Buy 150 upgrades",          "req": 150,            "type": "upgrades", "reward": 5_000_000},
    {"id": "ach25", "name": "Upgrade God",           "desc": "Buy all 250 upgrades",      "req": 250,            "type": "upgrades", "reward": 100_000_000},
]


# ─────────────────────────────────────────────
# DEFAULT GAME STATE
# ─────────────────────────────────────────────
def default_state():
    return {
        "total_data": 0,
        "all_time_data": 0,
        "data_per_click": 1,
        "auto_income": 0,
        "multiplier": 1.0,
        "owned_upgrades": [],
        "unlocked_achievements": [],
        "tap_count": 0,
        "prestige_count": 0,
        "prestige_bonus": 1.0,
        "last_time": time.time(),
        "golden_orange_active": False,
        "achievement_completions": 0,
        "ach_bonus": 1.0,
        "data_saver": False,
    }

game_state = default_state()

# ─────────────────────────────────────────────
# SAVE / LOAD
# ─────────────────────────────────────────────
_last_save_time = 0

def save_game():
    global _last_save_time
    now = time.time()
    if game_state.get("data_saver") and (now - _last_save_time) < 30:
        return
    _last_save_time = now
    game_state["last_time"] = now
    with open(SAVE_FILE, "w") as f:
        json.dump(game_state, f, indent=2)

def force_save():
    game_state["last_time"] = time.time()
    with open(SAVE_FILE, "w") as f:
        json.dump(game_state, f, indent=2)

def load_game():
    global game_state
    if os.path.exists(SAVE_FILE):
        try:
            with open(SAVE_FILE, "r") as f:
                loaded = json.load(f)
            base = default_state()
            base.update(loaded)
            game_state = base
            calculate_offline()
        except Exception as e:
            print(f"Load error: {e}")
            game_state = default_state()
    else:
        game_state = default_state()

def calculate_offline():
    elapsed = min(time.time() - game_state.get("last_time", time.time()), 8 * 3600)
    earned = int(elapsed * game_state["auto_income"] * game_state["multiplier"] * game_state["prestige_bonus"])
    if earned > 0:
        game_state["total_data"] += earned
        game_state["all_time_data"] = game_state.get("all_time_data", 0) + earned
        return earned
    return 0

# ─────────────────────────────────────────────
# HELPERS
# ─────────────────────────────────────────────
def fmt(n):
    """Format large numbers nicely."""
    n = int(n)
    if n >= 1_000_000_000_000_000: return f"{n/1_000_000_000_000_000:.2f}Q"
    if n >= 1_000_000_000_000: return f"{n/1_000_000_000_000:.2f}T"
    if n >= 1_000_000_000: return f"{n/1_000_000_000:.2f}B"
    if n >= 1_000_000:     return f"{n/1_000_000:.2f}M"
    if n >= 1_000:         return f"{n/1_000:.1f}K"
    return str(n)

def get_income_per_sec():
    return game_state["auto_income"] * game_state["multiplier"] * game_state["prestige_bonus"] * game_state.get("ach_bonus", 1.0)

def get_data_per_click():
    return int(game_state["data_per_click"] * game_state["multiplier"] * game_state["prestige_bonus"] * game_state.get("ach_bonus", 1.0))

# ─────────────────────────────────────────────
# CUSTOM WIDGETS
# ─────────────────────────────────────────────
class OrangeButton(ButtonBehavior, Widget):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self._scale = 1.0
        self.bind(pos=self._draw, size=self._draw)

    def _draw(self, *a):
        self.canvas.clear()
        cx, cy = self.center_x, self.center_y
        r = min(self.width, self.height) * 0.42 * self._scale
        with self.canvas:
            # Glow ring
            Color(1, 0.6, 0, 0.18)
            Ellipse(pos=(cx - r*1.35, cy - r*1.35), size=(r*2.7, r*2.7))
            # Shadow
            Color(0.3, 0.15, 0, 0.5)
            Ellipse(pos=(cx - r + 4, cy - r - 4), size=(r*2, r*2))
            # Main orange body
            Color(1.0, 0.5, 0.0, 1)
            Ellipse(pos=(cx - r, cy - r), size=(r*2, r*2))
            # Highlight
            Color(1.0, 0.85, 0.3, 0.55)
            Ellipse(pos=(cx - r*0.55, cy + r*0.25), size=(r*0.7, r*0.42))
            # Small leaf
            Color(0.2, 0.7, 0.1, 1)
            Ellipse(pos=(cx - r*0.12, cy + r*0.78), size=(r*0.25, r*0.35))
            # Face
            Color(0.5, 0.2, 0, 0.8)
            # Eyes
            Ellipse(pos=(cx - r*0.3, cy + r*0.1), size=(r*0.18, r*0.18))
            Ellipse(pos=(cx + r*0.12, cy + r*0.1), size=(r*0.18, r*0.18))
            # Smile
            Color(0.5, 0.2, 0, 0.9)
            Ellipse(pos=(cx - r*0.25, cy - r*0.28), size=(r*0.5, r*0.22))
            Color(1.0, 0.5, 0, 1)
            Ellipse(pos=(cx - r*0.22, cy - r*0.18), size=(r*0.44, r*0.18))

    def on_press(self):
        anim = Animation(_scale=0.88, duration=0.06) + Animation(_scale=1.0, duration=0.1)
        anim.bind(on_progress=lambda *a: self._draw())
        anim.start(self)


# ─────────────────────────────────────────────
# MAIN APP
# ─────────────────────────────────────────────
class OrangeClickerApp(App):
    title = "🍊 Orange Data Clicker - 250 Upgrades"

    def build(self):
        load_game()
        self._float_labels = []
        self._golden_timer = None
        self._offline_earned = calculate_offline()

        # ── Root layout ──────────────────────────
        root = BoxLayout(orientation="vertical", spacing=0)

        # ── TOP BAR ──────────────────────────────
        top_bar = BoxLayout(size_hint_y=None, height=dp(50), padding=(dp(10), dp(4)),
                            spacing=dp(8))
        with top_bar.canvas.before:
            Color(*C["dark"])
            self.top_rect = Rectangle()
        top_bar.bind(pos=lambda *a: setattr(self.top_rect, 'pos', top_bar.pos),
                     size=lambda *a: setattr(self.top_rect, 'size', top_bar.size))

        self.data_label = Label(
            text=f"🍊 {fmt(game_state['total_data'])} Data",
            font_size=dp(22), bold=True, color=C["gold"],
            halign="left", valign="middle"
        )
        self.data_label.bind(size=lambda i,s: setattr(i,'text_size',s))

        self.ips_label = Label(
            text=f"▶ {fmt(get_income_per_sec())}/sec",
            font_size=dp(14), color=C["orange"],
            halign="right", valign="middle"
        )
        self.ips_label.bind(size=lambda i,s: setattr(i,'text_size',s))

        top_bar.add_widget(self.data_label)
        top_bar.add_widget(self.ips_label)

        # ── PRESTIGE BAR ─────────────────────────
        prestige_row = BoxLayout(size_hint_y=None, height=dp(26), padding=(dp(8),dp(2)))
        with prestige_row.canvas.before:
            Color(0.08, 0.05, 0.01, 1)
            self.pres_rect = Rectangle()
        prestige_row.bind(pos=lambda *a: setattr(self.pres_rect,'pos',prestige_row.pos),
                          size=lambda *a: setattr(self.pres_rect,'size',prestige_row.size))

        prestige_cost = self._prestige_cost()
        self.prestige_label = Label(
            text=f"✦ Prestige {game_state['prestige_count']} | Bonus: x{game_state['prestige_bonus']:.1f} | Next: {fmt(prestige_cost)} Data",
            font_size=dp(11), color=C["purple"], halign="center"
        )
        self.prestige_label.bind(size=lambda i,s: setattr(i,'text_size',s))
        prestige_row.add_widget(self.prestige_label)

        # ── ORANGE AREA (SMALLER) ────────────────
        orange_area = BoxLayout(orientation="vertical", size_hint_y=0.28,
                                padding=(dp(8), dp(4)))
        with orange_area.canvas.before:
            Color(*C["bg"])
            self.oa_rect = Rectangle()
        orange_area.bind(pos=lambda *a: setattr(self.oa_rect,'pos',orange_area.pos),
                         size=lambda *a: setattr(self.oa_rect,'size',orange_area.size))

        self.orange_btn = OrangeButton(size_hint=(None, None), size=(dp(90), dp(90)))
        self.orange_btn.bind(on_press=self.tap_orange)

        # Center it
        orange_wrap = BoxLayout()
        orange_wrap.add_widget(Widget())
        orange_wrap.add_widget(self.orange_btn)
        orange_wrap.add_widget(Widget())

        self.click_info = Label(
            text=f"TAP! (+{get_data_per_click()} Data)",
            font_size=dp(12), color=C["orange"], bold=True,
            size_hint_y=None, height=dp(20)
        )

        self.golden_label = Label(
            text="", font_size=dp(11), color=C["gold"],
            size_hint_y=None, height=dp(18)
        )

        orange_area.add_widget(orange_wrap)
        orange_area.add_widget(self.click_info)
        orange_area.add_widget(self.golden_label)

        # ── TAB BAR ───────────────────────────────
        tab_row = BoxLayout(size_hint_y=None, height=dp(38), spacing=dp(2), padding=(dp(4),dp(2)))
        with tab_row.canvas.before:
            Color(*C["dark"])
            self.tab_rect = Rectangle()
        tab_row.bind(pos=lambda *a: setattr(self.tab_rect,'pos',tab_row.pos),
                     size=lambda *a: setattr(self.tab_rect,'size',tab_row.size))

        self._tabs = ["UPGRADES", "ACHIEVEMENTS", "PRESTIGE", "STATS"]
        self._tab_btns = []
        self._current_tab = 0

        for i, name in enumerate(self._tabs):
            btn = Button(text=name, font_size=dp(10), bold=True,
                         background_color=C["orange"] if i==0 else C["panel"],
                         color=C["white"])
            btn.bind(on_press=lambda inst, idx=i: self.switch_tab(idx))
            self._tab_btns.append(btn)
            tab_row.add_widget(btn)

        # ── CONTENT PANEL ─────────────────────────
        self.content_area = BoxLayout(orientation="vertical")

        # ── BOTTOM ASSEMBLE ───────────────────────
        root.add_widget(top_bar)
        root.add_widget(prestige_row)
        root.add_widget(orange_area)
        root.add_widget(tab_row)
        root.add_widget(self.content_area)

        # Build tab contents
        self._build_upgrades_tab()
        self._build_achievements_tab()
        self._build_prestige_tab()
        self._build_stats_tab()

        self._panels = [
            self._upgrades_panel,
            self._achievements_panel,
            self._prestige_panel,
            self._stats_panel,
        ]
        self.switch_tab(0)

        # Timers
        Clock.schedule_interval(self._auto_tick, 1)
        Clock.schedule_interval(self._update_ui, 0.25)
        Clock.schedule_interval(self._try_golden_orange, 30)

        # Show offline earnings popup
        if self._offline_earned > 0:
            Clock.schedule_once(lambda dt: self._show_offline_popup(self._offline_earned), 0.5)

        return root

    # ─────────────────────────────────────
    # TAB BUILDING
    # ─────────────────────────────────────
    def _panel_base(self):
        sv = ScrollView()
        inner = BoxLayout(orientation="vertical", spacing=dp(4),
                          padding=(dp(6), dp(4)), size_hint_y=None)
        inner.bind(minimum_height=inner.setter("height"))
        sv.add_widget(inner)
        return sv, inner

    def _build_upgrades_tab(self):
        self._upgrades_panel, inner = self._panel_base()
        self._upgrade_buttons = {}
        
        # Add header
        header = BoxLayout(size_hint_y=None, height=dp(40), padding=(dp(6),dp(2)))
        header.add_widget(Label(
            text=f"250 UPGRADES | Owned: {len(game_state['owned_upgrades'])}/250",
            font_size=dp(13), bold=True, color=C["gold"]
        ))
        inner.add_widget(header)
        
        for upg in UPGRADES:
            owned = upg["id"] in game_state["owned_upgrades"]
            row = BoxLayout(size_hint_y=None, height=dp(48), spacing=dp(4))

            icon_lbl = Label(text=upg["icon"], font_size=dp(20),
                             size_hint_x=None, width=dp(36))

            info = BoxLayout(orientation="vertical")
            name_lbl = Label(text=upg["name"], font_size=dp(11), bold=True,
                             color=C["gold"], halign="left")
            name_lbl.bind(size=lambda i,s: setattr(i,'text_size',s))
            desc_lbl = Label(text=upg["desc"], font_size=dp(9),
                             color=C["dim"], halign="left")
            desc_lbl.bind(size=lambda i,s: setattr(i,'text_size',s))
            info.add_widget(name_lbl)
            info.add_widget(desc_lbl)

            if owned:
                buy_btn = Button(text="✅", font_size=dp(10),
                                 background_color=(0.2,0.5,0.2,1),
                                 size_hint_x=None, width=dp(50), disabled=True)
            else:
                buy_btn = Button(
                    text=f"{fmt(upg['cost'])}",
                    font_size=dp(9), bold=True,
                    background_color=C["orange2"],
                    size_hint_x=None, width=dp(60)
                )
                buy_btn.bind(on_press=lambda inst, u=upg: self.buy_upgrade(u))
                self._upgrade_buttons[upg["id"]] = buy_btn

            row.add_widget(icon_lbl)
            row.add_widget(info)
            row.add_widget(buy_btn)

            with row.canvas.before:
                Color(0.12, 0.08, 0.03, 1)
                rr = RoundedRectangle(radius=[dp(6)])
            row.bind(pos=lambda inst, val, rr=rr: setattr(rr,'pos',val),
                     size=lambda inst, val, rr=rr: setattr(rr,'size',val))

            inner.add_widget(row)

    def _build_achievements_tab(self):
        self._achievements_panel, inner = self._panel_base()
        self._ach_inner = inner
        self._refresh_achievements()

    def _refresh_achievements(self):
        self._ach_inner.clear_widgets()

        ach_bonus = game_state.get("ach_bonus", 1.0)
        completions = game_state.get("achievement_completions", 0)
        unlocked = len(game_state["unlocked_achievements"])
        total = len(ACHIEVEMENTS)
        next_bonus = ach_bonus * 2

        header = BoxLayout(orientation="vertical", size_hint_y=None, height=dp(56),
                           padding=(dp(6), dp(3)))
        with header.canvas.before:
            Color(0.1, 0.06, 0.18, 1)
            h_rr = RoundedRectangle(radius=[dp(8)])
        header.bind(pos=lambda i,v,r=h_rr: setattr(r,'pos',v),
                    size=lambda i,v,r=h_rr: setattr(r,'size',v))

        bonus_lbl = Label(
            text=f"🌟 Bonus: x{ach_bonus:.0f}  |  Completions: {completions}",
            font_size=dp(12), bold=True, color=C["gold"], halign="center"
        )
        bonus_lbl.bind(size=lambda i,s: setattr(i,'text_size',s))
        prog_lbl = Label(
            text=f"Complete all {total} → x{next_bonus:.0f} bonus  [{unlocked}/{total}]",
            font_size=dp(10), color=C["purple"], halign="center"
        )
        prog_lbl.bind(size=lambda i,s: setattr(i,'text_size',s))
        header.add_widget(bonus_lbl)
        header.add_widget(prog_lbl)
        self._ach_inner.add_widget(header)
        self._ach_inner.add_widget(Widget(size_hint_y=None, height=dp(3)))
        
        for ach in ACHIEVEMENTS:
            unlocked = ach["id"] in game_state["unlocked_achievements"]
            row = BoxLayout(size_hint_y=None, height=dp(48), spacing=dp(4), padding=(dp(4),0))
            with row.canvas.before:
                Color(0.14, 0.09, 0.02, 1) if unlocked else Color(0.08, 0.05, 0.01, 1)
                rr = RoundedRectangle(radius=[dp(6)])
            row.bind(pos=lambda i,v,r=rr: setattr(r,'pos',v),
                     size=lambda i,v,r=rr: setattr(r,'size',v))

            medal = "🏆" if unlocked else "🔒"
            icon = Label(text=medal, font_size=dp(20), size_hint_x=None, width=dp(36))

            info = BoxLayout(orientation="vertical")
            Label_name = Label(text=ach["name"], font_size=dp(11), bold=True,
                               color=C["gold"] if unlocked else C["dim"],
                               halign="left")
            Label_name.bind(size=lambda i,s: setattr(i,'text_size',s))
            Label_desc = Label(text=f"{ach['desc']} | +{fmt(ach['reward'])} Data",
                               font_size=dp(9), color=C["dim"], halign="left")
            Label_desc.bind(size=lambda i,s: setattr(i,'text_size',s))
            info.add_widget(Label_name)
            info.add_widget(Label_desc)

            row.add_widget(icon)
            row.add_widget(info)
            self._ach_inner.add_widget(row)

    def _build_prestige_tab(self):
        self._prestige_panel, inner = self._panel_base()

        title = Label(text="✦ PRESTIGE", font_size=dp(20), bold=True,
                      color=C["purple"], size_hint_y=None, height=dp(36))
        desc = Label(
            text="Reset Data & Upgrades for permanent +0.5x bonus!\nRequired: 50,000 Data",
            font_size=dp(12), color=C["dim"], halign="center",
            size_hint_y=None, height=dp(60)
        )
        desc.bind(size=lambda i,s: setattr(i,'text_size',s))

        self._prestige_info = Label(
            text=self._prestige_text(),
            font_size=dp(14), color=C["gold"], halign="center",
            size_hint_y=None, height=dp(46)
        )
        self._prestige_info.bind(size=lambda i,s: setattr(i,'text_size',s))

        prestige_btn = Button(
            text="✦ PRESTIGE NOW ✦",
            font_size=dp(16), bold=True,
            background_color=C["purple"],
            size_hint_y=None, height=dp(50)
        )
        prestige_btn.bind(on_press=self.do_prestige)

        inner.add_widget(title)
        inner.add_widget(desc)
        inner.add_widget(self._prestige_info)
        inner.add_widget(prestige_btn)

    def _build_stats_tab(self):
        self._stats_panel, inner = self._panel_base()
        self._stats_inner = inner
        self._refresh_stats()

    def _refresh_stats(self):
        self._stats_inner.clear_widgets()

        # Data Saver Toggle
        ds_row = BoxLayout(size_hint_y=None, height=dp(44), spacing=dp(6),
                           padding=(dp(6), dp(3)))
        with ds_row.canvas.before:
            Color(0.08, 0.12, 0.06, 1)
            ds_rr = RoundedRectangle(radius=[dp(6)])
        ds_row.bind(pos=lambda i,v,r=ds_rr: setattr(r,'pos',v),
                    size=lambda i,v,r=ds_rr: setattr(r,'size',v))

        ds_icon = Label(text="💾", font_size=dp(18), size_hint_x=None, width=dp(30))
        ds_label = Label(text="Data Saver (saves every 30s)",
                         font_size=dp(10), color=C["green"] if game_state.get("data_saver") else C["dim"],
                         halign="left")
        ds_label.bind(size=lambda i,s: setattr(i,'text_size',s))

        is_on = game_state.get("data_saver", False)
        ds_btn = Button(
            text="ON" if is_on else "OFF",
            font_size=dp(11), bold=True,
            background_color=C["green"] if is_on else (0.3,0.3,0.3,1),
            size_hint_x=None, width=dp(60)
        )

        def toggle_ds(inst):
            game_state["data_saver"] = not game_state.get("data_saver", False)
            force_save()
            self._refresh_stats()

        ds_btn.bind(on_press=toggle_ds)
        ds_row.add_widget(ds_icon)
        ds_row.add_widget(ds_label)
        ds_row.add_widget(ds_btn)
        self._stats_inner.add_widget(ds_row)

        # Achievement Bonus Display
        ach_completions = game_state.get("achievement_completions", 0)
        ach_bonus = game_state.get("ach_bonus", 1.0)
        unlocked = len(game_state["unlocked_achievements"])
        total_ach = len(ACHIEVEMENTS)

        ach_row = BoxLayout(size_hint_y=None, height=dp(46), spacing=dp(6),
                            padding=(dp(6), dp(3)))
        with ach_row.canvas.before:
            Color(0.12, 0.08, 0.02, 1)
            ar_rr = RoundedRectangle(radius=[dp(6)])
        ach_row.bind(pos=lambda i,v,r=ar_rr: setattr(r,'pos',v),
                     size=lambda i,v,r=ar_rr: setattr(r,'size',v))

        ach_icon = Label(text="🌟", font_size=dp(18), size_hint_x=None, width=dp(30))
        ach_info = BoxLayout(orientation="vertical")
        ach_title = Label(
            text=f"Achievement Bonus: x{ach_bonus:.0f}  (#{ach_completions})",
            font_size=dp(12), bold=True, color=C["gold"], halign="left"
        )
        ach_title.bind(size=lambda i,s: setattr(i,'text_size',s))
        ach_prog = Label(
            text=f"Progress: {unlocked}/{total_ach} — next = x{ach_bonus*2:.0f}",
            font_size=dp(10), color=C["orange"], halign="left"
        )
        ach_prog.bind(size=lambda i,s: setattr(i,'text_size',s))
        ach_info.add_widget(ach_title)
        ach_info.add_widget(ach_prog)
        ach_row.add_widget(ach_icon)
        ach_row.add_widget(ach_info)
        self._stats_inner.add_widget(ach_row)

        # Regular stats
        stats = [
            ("🍊 Total Data",        fmt(game_state["total_data"])),
            ("📊 All-Time Data",     fmt(game_state.get("all_time_data", 0))),
            ("👆 Data/Click",        fmt(get_data_per_click())),
            ("⚡ Auto Income",       f"{fmt(get_income_per_sec())}/sec"),
            ("✨ Multiplier",        f"x{game_state['multiplier']:.1f}"),
            ("✦ Prestige Bonus",    f"x{game_state['prestige_bonus']:.1f}"),
            ("🌟 Ach Bonus",        f"x{game_state.get('ach_bonus',1.0):.0f}"),
            ("🏆 Prestiges",        str(game_state["prestige_count"])),
            ("👆 Total Taps",       fmt(game_state.get("tap_count", 0))),
            ("🔓 Upgrades Owned",   f"{len(game_state['owned_upgrades'])}/250"),
            ("🏅 Achievements",     f"{unlocked}/{total_ach}"),
            ("🔄 Ach Completions",  str(ach_completions)),
        ]
        for label, val in stats:
            row = BoxLayout(size_hint_y=None, height=dp(32), padding=(dp(6),0), spacing=dp(8))
            with row.canvas.before:
                Color(0.10, 0.07, 0.02, 1)
                rr = RoundedRectangle(radius=[dp(5)])
            row.bind(pos=lambda i,v,r=rr: setattr(r,'pos',v),
                     size=lambda i,v,r=rr: setattr(r,'size',v))
            lbl = Label(text=label, font_size=dp(11), color=C["dim"],
                        halign="left", size_hint_x=0.6)
            lbl.bind(size=lambda i,s: setattr(i,'text_size',s))
            val_lbl = Label(text=val, font_size=dp(12), color=C["gold"],
                            bold=True, halign="right", size_hint_x=0.4)
            val_lbl.bind(size=lambda i,s: setattr(i,'text_size',s))
            row.add_widget(lbl)
            row.add_widget(val_lbl)
            self._stats_inner.add_widget(row)

    # ─────────────────────────────────────
    # TAB SWITCHING
    # ─────────────────────────────────────
    def switch_tab(self, idx):
        self._current_tab = idx
        self.content_area.clear_widgets()
        self.content_area.add_widget(self._panels[idx])
        for i, btn in enumerate(self._tab_btns):
            btn.background_color = C["orange"] if i == idx else (0.12, 0.08, 0.03, 1)

        if idx == 1: self._refresh_achievements()
        if idx == 3: self._refresh_stats()

    # ─────────────────────────────────────
    # GAME LOGIC
    # ─────────────────────────────────────
    def tap_orange(self, instance):
        earned = get_data_per_click()
        if game_state.get("golden_orange_active"):
            earned *= 5
        game_state["total_data"] += earned
        game_state["all_time_data"] = game_state.get("all_time_data", 0) + earned
        game_state["tap_count"] = game_state.get("tap_count", 0) + 1
        self._check_achievements()

    def buy_upgrade(self, upg):
        if upg["id"] in game_state["owned_upgrades"]:
            return
        if game_state["total_data"] < upg["cost"]:
            return
        game_state["total_data"] -= upg["cost"]
        game_state["owned_upgrades"].append(upg["id"])

        if upg["type"] == "tap":
            game_state["data_per_click"] += upg["value"]
        elif upg["type"] == "auto":
            game_state["auto_income"] += upg["value"]
        elif upg["type"] == "mult":
            game_state["multiplier"] *= upg["value"]
        elif upg["type"] == "spec":
            sid = upg["id"]
            if "spec" in sid:
                game_state["multiplier"] *= (1 + upg["value"])

        # Update button
        if upg["id"] in self._upgrade_buttons:
            btn = self._upgrade_buttons[upg["id"]]
            btn.text = "✅"
            btn.background_color = (0.2, 0.5, 0.2, 1)
            btn.disabled = True

        # Rebuild upgrades header
        self._build_upgrades_tab()
        self._panels[0] = self._upgrades_panel
        if self._current_tab == 0:
            self.switch_tab(0)

        self._check_achievements()
        save_game()

    def _auto_tick(self, dt):
        earned = get_income_per_sec()
        game_state["total_data"] += earned
        game_state["all_time_data"] = game_state.get("all_time_data", 0) + earned
        self._check_achievements()
        save_game()

    def _update_ui(self, dt):
        game_state["total_data"] = max(0, game_state["total_data"])
        self.data_label.text = f"🍊 {fmt(game_state['total_data'])} Data"
        self.ips_label.text = f"▶ {fmt(get_income_per_sec())}/sec"
        self.click_info.text = f"TAP! (+{fmt(get_data_per_click())} Data)"
        prestige_cost = self._prestige_cost()

        ach_bonus = game_state.get("ach_bonus", 1.0)
        ach_str = f" | 🌟x{ach_bonus:.0f}" if ach_bonus > 1.0 else ""

        self.prestige_label.text = (
            f"✦ Prestige {game_state['prestige_count']} "
            f"| Bonus: x{game_state['prestige_bonus']:.1f}"
            f"{ach_str} "
            f"| Next: {fmt(prestige_cost)}"
        )
        if hasattr(self, '_prestige_info'):
            self._prestige_info.text = self._prestige_text()

        if game_state.get("golden_orange_active"):
            self.golden_label.text = "⭐ GOLDEN 5x ⭐"
        else:
            self.golden_label.text = ""

    def _try_golden_orange(self, dt):
        if not game_state.get("golden_orange_active") and random.random() < 0.05:
            game_state["golden_orange_active"] = True
            Clock.schedule_once(self._end_golden_orange, 10)

    def _end_golden_orange(self, dt):
        game_state["golden_orange_active"] = False

    def _check_achievements(self):
        newly_unlocked = []
        for ach in ACHIEVEMENTS:
            if ach["id"] in game_state["unlocked_achievements"]:
                continue
            unlocked = False
            if ach["type"] == "total" and game_state.get("all_time_data", 0) >= ach["req"]:
                unlocked = True
            elif ach["type"] == "auto" and get_income_per_sec() >= ach["req"]:
                unlocked = True
            elif ach["type"] == "taps" and game_state.get("tap_count", 0) >= ach["req"]:
                unlocked = True
            elif ach["type"] == "prestige" and game_state.get("prestige_count", 0) >= ach["req"]:
                unlocked = True
            elif ach["type"] == "upgrades" and len(game_state["owned_upgrades"]) >= ach["req"]:
                unlocked = True
            if unlocked:
                game_state["unlocked_achievements"].append(ach["id"])
                game_state["total_data"] += ach["reward"]
                game_state["all_time_data"] = game_state.get("all_time_data", 0) + ach["reward"]
                newly_unlocked.append(ach)

        for ach in newly_unlocked:
            self._show_achievement_popup(ach)

        if len(game_state["unlocked_achievements"]) >= len(ACHIEVEMENTS):
            self._handle_achievement_clear()

    def _handle_achievement_clear(self):
        completions = game_state.get("achievement_completions", 0)
        expected_bonus = 2.0 ** completions
        if game_state.get("ach_bonus", 1.0) < expected_bonus * 1.5:
            game_state["achievement_completions"] = completions + 1
            new_bonus = 2.0 ** (completions + 1)
            game_state["ach_bonus"] = new_bonus
            game_state["unlocked_achievements"] = []
            force_save()
            self._refresh_achievements()
            self._show_clear_popup(new_bonus)

    def _show_clear_popup(self, new_bonus):
        box = BoxLayout(orientation="vertical", padding=dp(14), spacing=dp(8))
        with box.canvas.before:
            Color(0.05, 0.02, 0.1, 1)
            rr = RoundedRectangle(radius=[dp(12)])
        box.bind(pos=lambda i,v: setattr(rr,'pos',v),
                 size=lambda i,v: setattr(rr,'size',v))

        box.add_widget(Label(text="🌟 ALL ACHIEVEMENTS CLEARED! 🌟",
                             font_size=dp(15), bold=True, color=C["gold"]))
        box.add_widget(Label(text="BONUS DOUBLED!",
                             font_size=dp(18), bold=True, color=C["orange"]))
        box.add_widget(Label(text=f"New bonus: x{new_bonus:.0f}",
                             font_size=dp(20), bold=True, color=C["green"]))

        close_btn = Button(text="LET'S GO! 🚀", font_size=dp(16), bold=True,
                           background_color=C["gold"], color=(0,0,0,1),
                           size_hint_y=None, height=dp(44))
        box.add_widget(close_btn)

        popup = Popup(title="", content=box, size_hint=(0.85, 0.5),
                      separator_height=0, background="")
        close_btn.bind(on_press=popup.dismiss)
        popup.open()

    def _show_achievement_popup(self, ach):
        box = BoxLayout(orientation="vertical", padding=dp(14), spacing=dp(6))
        with box.canvas.before:
            Color(*C["dark"])
            RoundedRectangle(pos=box.pos, size=box.size, radius=[dp(12)])

        box.add_widget(Label(text="🏆 ACHIEVEMENT!", font_size=dp(16),
                             bold=True, color=C["gold"]))
        box.add_widget(Label(text=ach["name"], font_size=dp(18), bold=True,
                             color=C["orange"]))
        box.add_widget(Label(text=ach["desc"], font_size=dp(12), color=C["dim"]))
        box.add_widget(Label(text=f"+{fmt(ach['reward'])} Data",
                             font_size=dp(14), color=C["green"]))
        close_btn = Button(text="AWESOME!", font_size=dp(14), bold=True,
                           background_color=C["orange"], size_hint_y=None, height=dp(40))
        box.add_widget(close_btn)

        popup = Popup(title="", content=box, size_hint=(0.8, 0.42),
                      separator_height=0, background="")
        close_btn.bind(on_press=popup.dismiss)
        popup.open()

    def _show_offline_popup(self, earned):
        box = BoxLayout(orientation="vertical", padding=dp(14), spacing=dp(8))
        box.add_widget(Label(text="💤 Welcome Back!", font_size=dp(18), bold=True, color=C["gold"]))
        box.add_widget(Label(text=f"You earned:",
                             font_size=dp(12), color=C["dim"]))
        box.add_widget(Label(text=f"+{fmt(earned)} Data", font_size=dp(22),
                             bold=True, color=C["orange"]))
        btn = Button(text="Collect!", font_size=dp(14), bold=True,
                     background_color=C["orange"], size_hint_y=None, height=dp(40))
        box.add_widget(btn)
        popup = Popup(title="", content=box, size_hint=(0.75, 0.38),
                      separator_height=0, background="")
        btn.bind(on_press=popup.dismiss)
        popup.open()

    # ─────────────────────────────────────
    # PRESTIGE
    # ─────────────────────────────────────
    def _prestige_cost(self):
        return 50000 * (3 ** game_state["prestige_count"])

    def _prestige_text(self):
        cost = self._prestige_cost()
        can = game_state["total_data"] >= cost
        status = "✅ READY!" if can else f"Need {fmt(cost)}"
        return f"Current: x{game_state['prestige_bonus']:.1f}\nNext: x{game_state['prestige_bonus']+0.5:.1f}\n{status}"

    def do_prestige(self, instance):
        cost = self._prestige_cost()
        if game_state["total_data"] < cost:
            self._show_toast("Not enough Data!")
            return

        box = BoxLayout(orientation="vertical", padding=dp(14), spacing=dp(6))
        box.add_widget(Label(text="⚠ PRESTIGE", font_size=dp(16), bold=True, color=C["purple"]))
        box.add_widget(Label(
            text="Reset Data & Upgrades\nGain +0.5x bonus\n\nContinue?",
            font_size=dp(12), color=C["dim"], halign="center"))
        box.add_widget(Label(text=f"New: x{game_state['prestige_bonus']+0.5:.1f}",
                             font_size=dp(14), bold=True, color=C["gold"]))
        row = BoxLayout(size_hint_y=None, height=dp(40), spacing=dp(6))
        yes_btn = Button(text="YES", background_color=C["purple"], bold=True)
        no_btn = Button(text="NO", background_color=(0.3,0.3,0.3,1))
        row.add_widget(yes_btn)
        row.add_widget(no_btn)
        box.add_widget(row)

        popup = Popup(title="", content=box, size_hint=(0.8, 0.48),
                      separator_height=0, background="")
        no_btn.bind(on_press=popup.dismiss)

        def confirm(*a):
            popup.dismiss()
            self._execute_prestige()

        yes_btn.bind(on_press=confirm)
        popup.open()

    def _execute_prestige(self):
        game_state["prestige_count"] += 1
        game_state["prestige_bonus"] += 0.5
        all_time = game_state.get("all_time_data", 0)

        game_state["total_data"] = 0
        game_state["data_per_click"] = 1
        game_state["auto_income"] = 0
        game_state["multiplier"] = 1.0
        game_state["owned_upgrades"] = []
        game_state["all_time_data"] = all_time

        self._build_upgrades_tab()
        self._panels[0] = self._upgrades_panel
        if self._current_tab == 0:
            self.switch_tab(0)

        save_game()
        self._show_toast(f"✦ Prestiged! x{game_state['prestige_bonus']:.1f}")

    def _show_toast(self, msg):
        box = BoxLayout(padding=dp(10))
        box.add_widget(Label(text=msg, font_size=dp(12), color=C["gold"],
                             halign="center", bold=True))
        popup = Popup(title="", content=box, size_hint=(0.7, 0.18),
                      separator_height=0, background="")
        popup.open()
        Clock.schedule_once(lambda dt: popup.dismiss(), 2.0)


# ─────────────────────────────────────────────
# RUN
# ─────────────────────────────────────────────
if __name__ == "__main__":
    OrangeClickerApp().run()
