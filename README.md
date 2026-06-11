# -import streamlit as st

import pandas as pd

import matplotlib.pyplot as plt

import matplotlib.ticker as ticker

st.set_page_config(page_title="내신 등급 계산기", layout="centered")

st.markdown("""
<style>

    .main { background-color: #f0f8ff; }

    h1, h2, h3 { color: #1a7abf; }

    .stButton>button {

        background-color: #1a7abf;

        color: white;

        border-radius: 8px;

        border: none;

        padding: 0.4em 1em;

    }

    .stButton>button:hover { background-color: #155f99; }

    .stDataFrame { border-radius: 8px; }
</style>

""", unsafe_allow_html=True)

st.title("내신 등급 계산기")

grade_to_point = {"1등급": 1, "2등급": 2, "3등급": 3, "4등급": 4, "5등급": 5}

if "subjects" not in st.session_state:

    st.session_state.subjects = []

if "counter" not in st.session_state:

    st.session_state.counter = 0

st.subheader("과목 추가")

col1, col2, col3, col4 = st.columns([3, 1, 2, 2])

with col1:

    name = st.text_input("과목명", key=f"name_{st.session_state.counter}")

with col2:

    credit = st.number_input("단위수", min_value=1, max_value=10, value=3, key=f"credit_{st.session_state.counter}")

with col3:

    grade = st.selectbox("등급", list(grade_to_point.keys()), key=f"grade_{st.session_state.counter}")

with col4:

    semester = st.selectbox("학기", ["1-1", "1-2", "2-1", "2-2", "3-1", "3-2"], key=f"semester_{st.session_state.counter}")

if st.button("추가"):

    if name:

        st.session_state.subjects.append({

            "학기": semester,

            "과목명": name,

            "단위수": credit,

            "등급": grade,

            "등급점수": grade_to_point[grade]

        })

        st.session_state.counter += 1

        st.session_state[f"semester_{st.session_state.counter}"] = semester

        st.rerun()


if st.session_state.subjects:

    df = pd.DataFrame(st.session_state.subjects)

    # 전체 합산

    total_credit = sum(s["단위수"] for s in st.session_state.subjects)

    total_weighted = sum(s["단위수"] * s["등급점수"] for s in st.session_state.subjects)

    overall_avg = total_weighted / total_credit

    st.subheader("전체 합산")

    col1, col2 = st.columns(2)

    col1.metric("총 이수단위", f"{total_credit}단위")

    col2.metric("평균 등급", f"{overall_avg:.2f}등급")

    # 학기별 표 + 삭제 버튼

    st.subheader("학기별 성적")

    for sem in sorted(df["학기"].unique()):

        sem_df = df[df["학기"] == sem]

        sem_credit = sem_df["단위수"].sum()

        sem_weighted = (sem_df["단위수"] * sem_df["등급점수"]).sum()

        sem_avg = sem_weighted / sem_credit

        st.markdown(f"**{sem}학기** — 평균 {sem_avg:.2f}등급 ({sem_credit}단위)")

        for i, row in sem_df.iterrows():

            c1, c2, c3, c4 = st.columns([3, 1, 2, 1])

            c1.write(row["과목명"])

            c2.write(f"{row['단위수']}단위")

            c3.write(row["등급"])

            if c4.button("삭제", key=f"del_{i}"):

                st.session_state.subjects.pop(i)

                st.rerun()

    # 그래프

    st.subheader("학기별 평균 등급 추이")

    sem_list = sorted(df["학기"].unique())

    avg_list = []

    for sem in sem_list:

        sem_df = df[df["학기"] == sem]

        avg = (sem_df["단위수"] * sem_df["등급점수"]).sum() / sem_df["단위수"].sum()

        avg_list.append(avg)

    fig, ax = plt.subplots(figsize=(7, 3))

    ax.plot(sem_list, avg_list, marker="o", color="#1a7abf", linewidth=2, markersize=7)

    ax.set_ylim(5.2, 0.8)    

    ax.set_ylabel("Avg Grade (1=best)")

    ax.set_xlabel("Semester")

    ax.yaxis.set_major_locator(ticker.MultipleLocator(1))

    ax.grid(axis="y", linestyle="--", alpha=0.5)

    fig.patch.set_facecolor("#f0f8ff")

    ax.set_facecolor("#f0f8ff")

    st.pyplot(fig)

    st.divider()

    if st.button("전체 초기화"):

        st.session_state.subjects = []

        st.session_state.counter = 0

        st.rerun()
 
설명
