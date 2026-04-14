
import streamlit as st
import pandas as pd
from fpdf import FPDF
from sklearn.metrics import precision_score, recall_score, f1_score, confusion_matrix
import matplotlib.pyplot as plt

st.set_page_config(page_title="AI Resume Builder", layout="wide")

st.title("🤖 AI Chatbot Resume Builder")

# ---------------- CHATBOT ---------------- #
st.sidebar.title("💬 Resume Chatbot")

steps = [
    "name","email","phone","address","objective","skills",
    "education","internships","projects","certificates",
    "achievements","personal"
]

if "step" not in st.session_state:
    st.session_state.step = 0
    st.session_state.data = {}

questions = {
    "name": "What is your full name?",
    "email": "Enter your email ID",
    "phone": "Enter phone number",
    "address": "Enter address",
    "objective": "Career Objective",
    "skills": "Technical skills (comma separated)",
    "education": "Education (UG/PG, Year, Marks)",
    "internships": "Internships (max 5)",
    "projects": "Projects (Major & Minor)",
    "certificates": "Certificates",
    "achievements": "Achievements (max 5)",
    "personal": "Personal details (DOB, hobbies, languages)"
}

current = steps[st.session_state.step]
answer = st.sidebar.text_input(questions[current])

if st.sidebar.button("Next"):
    st.session_state.data[current] = answer
    st.session_state.step += 1

# ---------------- ATS SCORING ---------------- #
def ats_score(data):
    score = 0
    keywords = ["python","java","ldap","azure","iam","security"]
    skill_text = data.get("skills","").lower()

    for k in keywords:
        if k in skill_text:
            score += 10

    completeness = len(data.keys()) * 5
    return min(score + completeness, 100)

# ---------------- FINAL OUTPUT ---------------- #
if st.session_state.step >= len(steps):
    st.success("✅ Resume data collected")

    ats = ats_score(st.session_state.data)
    st.metric("ATS Score", f"{ats}/100")

    y_true = [1,1,1,1]
    y_pred = [1 if ats > 60 else 0]*4

    st.write("### ML Metrics")
    st.write("Precision:", precision_score(y_true, y_pred))
    st.write("Recall:", recall_score(y_true, y_pred))
    st.write("F1 Score:", f1_score(y_true, y_pred))

    cm = confusion_matrix(y_true, y_pred)
    fig, ax = plt.subplots()
    ax.matshow(cm)
    st.pyplot(fig)

    if st.button("📄 Download Resume PDF"):
        pdf = FPDF()
        pdf.add_page()
        pdf.set_font("Arial", size=12)
        for k,v in st.session_state.data.items():
            pdf.multi_cell(0,10,f"{k.upper()}: {v}")
        pdf.output("resume.pdf")
        st.success("✅ Resume PDF Generated")
``
