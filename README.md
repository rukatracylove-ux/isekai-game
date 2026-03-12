# isekai?game.py
import sys
from pathlib import Path

# -----------------------------
# 初期ステータス
# -----------------------------
def reset_state():
    return {
        "scene": "intro",
        "hp": 30,
        "mp": 10,
        "str": 3,
        "int": 4,
        "inventory": ["古びたペンダント"],
        "affection": {
            "レオ": 0,
            "ミラ": 0,
            "ラズ": 0
        },
        "flags": {}
    }

state = reset_state()

# -----------------------------
# 表示関数
# -----------------------------
def text(msg):
    print("\n" + msg)

def status():
    print("\n--- ステータス ---")
    print(f"HP:{state['hp']}  MP:{state['mp']}")
    print(f"力:{state['str']}  知性:{state['int']}")
    print("好感度:", state["affection"])
    print("所持品:", state["inventory"])
    print("------------------")

def choice(options):
    for i, opt in enumerate(options, 1):
        print(f"{i}. {opt}")
    while True:
        try:
            c = int(input("▶ 選択 > "))
            if 1 <= c <= len(options):
                return c
        except:
            pass

# -----------------------------
# シーン定義
# -----------------------------
def intro():
    text("――目を覚ますと、そこは異世界の森だった。")
    text("前世の記憶を持ったまま、あなたは転生している。")

    c = choice([
        "立ち上がって周囲を見る",
        "もう少し眠る"
    ])

    if c == 1:
        state["scene"] = "leo"
    else:
        state["hp"] -= 5
        state["scene"] = "monster"

def monster():
    text("魔物に襲われた！")

    c = choice([
        "力で応戦する",
        "魔法で逃げる"
    ])

    if c == 1:
        state["str"] += 1
        state["hp"] -= 3
    else:
        state["int"] += 1
        state["mp"] -= 3

    state["scene"] = "leo"

def leo():
    text("旅人の青年『レオ』と出会った。")

    c = choice([
        "笑顔で挨拶する",
        "警戒する"
    ])

    if c == 1:
        state["affection"]["レオ"] += 2

    state["scene"] = "village"

def village():
    text("村の娘『ミラ』があなたに助けを求めている。")

    c = choice([
        "力仕事を手伝う",
        "魔法で畑を修復",
        "報酬を要求する"
    ])

    if c == 1:
        state["str"] += 2
        state["affection"]["ミラ"] += 1
    elif c == 2:
        state["int"] += 2
        state["affection"]["ミラ"] += 2
        state["inventory"].append("魔力の水晶")
    else:
        state["inventory"].append("金貨")
        state["affection"]["ミラ"] -= 1

    state["scene"] = "guild"

def guild():
    text("ギルド副長『ラズ』があなたを試す。")

    c = choice([
        "ギルドに加入する",
        "単独で行動する"
    ])

    if c == 1:
        state["flags"]["guild"] = True
        state["affection"]["ラズ"] += 2
    else:
        state["str"] += 1

    state["scene"] = "battle"

def battle():
    text("魔物討伐が始まった。")

    c = choice([
        "前線で戦う",
        "魔法で援護する",
        "罠を仕掛ける"
    ])

    success = False

    if c == 1 and state["str"] >= 5:
        success = True
        state["affection"]["レオ"] += 3
    elif c == 2 and state["int"] >= 6:
        success = True
        state["affection"]["ミラ"] += 3
    elif c == 3 and state["flags"].get("guild"):
        success = True
        state["affection"]["ラズ"] += 3
    else:
        state["hp"] -= 10

    state["flags"]["battle_success"] = success
    state["scene"] = "ending"

def ending():
    text("――あなたの物語は、一つの結末を迎える。")

    top = max(state["affection"], key=state["affection"].get)
    love = state["affection"][top]

    if state["hp"] <= 0:
        text("あなたは戦いで命を落とした……【BAD END】")
    elif love >= 6:
        text(f"{top}と結ばれ、異世界で生きていく。【恋愛END】")
    elif state["flags"].get("battle_success"):
        text("英雄として名を刻んだ。【英雄END】")
    else:
        text("静かな日々を選んだ。【平穏END】")

    again = choice([
        "最初から遊ぶ",
        "終了する"
    ])

    if again == 1:
        global state
        state = reset_state()
        game_loop()

# -----------------------------
# メインループ
# -----------------------------
def game_loop():
    while True:
        status()

        if state["scene"] == "intro":
            intro()
        elif state["scene"] == "monster":
            monster()
        elif state["scene"] == "leo":
            leo()
        elif state["scene"] == "village":
            village()
        elif state["scene"] == "guild":
            guild()
        elif state["scene"] == "battle":
            battle()
        elif state["scene"] == "ending":
            ending()
            break

# -----------------------------
# 起動
# -----------------------------
print("✨ 異世界転生 × 乙女ゲーム（Python単体版）✨")
game_loop()
