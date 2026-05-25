# soul-reader

import re
import streamlit as st
import pandas as pd

st.set_page_config(page_title="Soul Reader", page_icon="📖", layout="wide")

READER_TYPES = {
    "自己犠牲共鳴型": {
        "keywords": ["守る", "犠牲", "救う", "命", "痛み", "傷", "背負", "失う"],
        "description": "誰かを守るために傷付く主人公へ強く感情移入するタイプ。"
    },
    "中二病美学型": {
        "keywords": ["闇", "焔", "異形", "終焉", "覚醒", "破壊", "運命", "英雄"],
        "description": "スタイリッシュな熱量・美学・必殺演出に強く反応するタイプ。"
    },
    "映像脳バトル好き": {
        "keywords": ["轟音", "閃光", "爆発", "衝撃", "疾る", "拳", "炎", "咆哮"],
        "description": "脳内で映像再生されるような戦闘描写を好むタイプ。"
    },
    "孤独救済型": {
        "keywords": ["孤独", "一人", "救い", "居場所", "涙", "優しさ", "抱える"],
        "description": "孤独や欠落を埋める関係性に深く刺さるタイプ。"
    },
    "文体陶酔型": {
        "keywords": ["——", "……", "静寂", "余韻", "沈む", "零れる", "揺れる"],
        "description": "文章リズムや空気感そのものに酔うタイプ。"
    },
    "ライトテンポ重視型": {
        "keywords": ["軽口", "テンポ", "ギャグ", "日常", "会話", "コミカル"],
        "description": "読みやすさやテンポ感を重視するライト層。"
    }
}

def normalize(text):
    return re.sub(r"\s+", "", text)

def analyze(text):
    text_n = normalize(text)
    results = []

    for reader_type, data in READER_TYPES.items():
        score = 0
        hits = []

        for kw in data["keywords"]:
            count = text_n.count(kw)
            if count > 0:
                score += count * 10
                hits.append(f"{kw}×{count}")

        score = min(score, 100)

        if score >= 80:
            rank = "SSS"
        elif score >= 60:
            rank = "SS"
        elif score >= 40:
            rank = "S"
        elif score >= 20:
            rank = "A"
        else:
            rank = "B"

        results.append({
            "読者タイプ": reader_type,
            "刺さり度": rank,
            "スコア": score,
            "特徴": data["description"],
            "反応要素": ", ".join(hits[:5])
        })

    results.sort(key=lambda x: x["スコア"], reverse=True)
    return results

st.title("📖 Soul Reader")
st.subheader("小説『刺さり読者』シミュレーター")

text = st.text_area(
    "小説本文を貼ってください",
    height=350,
    placeholder="ここに本文を貼ってください..."
)

if st.button("解析する"):
    if not text.strip():
        st.warning("本文を入力してください。")
    else:
        results = analyze(text)

        st.success("解析完了")

        top = results[0]

        st.markdown("## 🔥 最も刺さる読者")
        st.markdown(f"### {top['読者タイプ']} : {top['刺さり度']}")

        st.markdown("---")

        df = pd.DataFrame(results)
        st.dataframe(df, use_container_width=True)

        st.markdown("## 🧠 AI講評")

        commentary = f'''
この文章は「{top["読者タイプ"]}」への反応が特に強く、
{top["特徴"]}

また、
・演出密度
・感情圧
・キーワード反復

によって、特定読者への没入力が強化されています。
'''
        st.write(commentary)
