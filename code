import streamlit as st
import time
import base64
import json
import re
from datetime import datetime

# ========== API INTEGRATION ==========
try:
    from api_integration import LangflowAPI
    api = LangflowAPI()
except ImportError:
    class DummyAPI:
        def qna_medical(self, question):
            return {"success": True, "message": f"**Answer:** This would come from your Q&A Langflow flow.\n\nQuestion: {question}"}
        def analyze_report(self, user_message, file_uploaded=False, file_name=None):
            if file_uploaded:
                return {"success": True, "message": f"**Report Analysis:** Analysis for uploaded file: {file_name}\n\nAI analysis of your medical report.\n\nMessage: {user_message}"}
            return {"success": True, "message": f"**Report Analysis:** {user_message}"}
        def find_hospitals(self, query, location=""):
            return {"success": True, "message": f"**Hospital Recommendations:**\n\nLooking for: {query} in {location if location else 'your area'}"}
        def explain_medicines(self, user_message, file_uploaded=False, file_name=None):
            if file_uploaded:
                return {"success": True, "message": f"**Medicine Explanation:** Analysis for uploaded file: {file_name}\n\nAI analysis of your prescription.\n\nMessage: {user_message}"}
            return {"success": True, "message": f"**Medicine Explanation:** {user_message}"}
        def analyze_bill(self, user_message, file_uploaded=False, file_name=None):
            audit_report = """Medical Billing Audit Report

| Bill Item | Billed Price (₹) | Standard/Ref Price (₹) | Potential Overcharge (₹) | Auditor's Expert Analysis |
| :--- | :--- | :--- | :--- | :--- |
| Emergency Room Consultation | ₹800.00 | ₹800.00 | ₹0.00 | Charged fairly |
| Complete Blood Count (CBC) | ₹1,200.00 | ₹1,200.00 | ₹0.00 | Charged fairly |
| Laparoscopic Appendectomy | ₹35,000.00 | ₹30,000.00 | ₹5,000.00 | Potential overcharge |
| Laparoscopic Equipment Fee | ₹5,000.00 | Included in Surgery | ₹0.00 | Double-billed item |
| Inj. Pantoprazole | ₹135.00 | ₹160.00 | ₹25.00 | Price discrepancy |

**Total Potential Overcharge: ₹5,025.00**

**Recommendation:** Request a reduction of ₹5,025.00 from hospital TPA or management."""
            return {"success": False, "error": "API unavailable", "message": audit_report, "demo_mode": True}
        def symptom_check(self, symptoms, age, gender):
            return {"success": True, "message": f"Symptom analysis for: {symptoms}"}
    api = DummyAPI()

# ========== PAGE CONFIG ==========
st.set_page_config(
    page_title="CHRONOCHECK — Medical AI",
    page_icon="⚕️",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ========== GLOBAL CSS ==========
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700;800;900&family=JetBrains+Mono:wght@400;500&display=swap');

:root {
    --bg-base:    #060b14;
    --bg-card:    #0d1829;
    --bg-glass:   rgba(13,24,41,0.85);
    --border:     rgba(56,189,248,0.12);
    --border-hi:  rgba(56,189,248,0.35);
    --accent:     #38bdf8;
    --accent2:    #818cf8;
    --accent3:    #34d399;
    --danger:     #f87171;
    --warn:       #fbbf24;
    --text-pri:   #e2e8f0;
    --text-sec:   #94a3b8;
    --text-muted: #475569;
    --glow:       0 0 40px rgba(56,189,248,0.15);
    --glow-strong:0 0 60px rgba(56,189,248,0.3);
}

html, body, .stApp {
    background: var(--bg-base) !important;
    font-family: 'Outfit', sans-serif;
    color: var(--text-pri);
}

/* Animated mesh background */
.stApp::before {
    content: '';
    position: fixed;
    inset: 0;
    background:
        radial-gradient(ellipse 80% 60% at 20% 10%, rgba(56,189,248,0.07) 0%, transparent 60%),
        radial-gradient(ellipse 60% 80% at 80% 80%, rgba(129,140,248,0.06) 0%, transparent 60%),
        radial-gradient(ellipse 40% 40% at 50% 50%, rgba(52,211,153,0.04) 0%, transparent 60%);
    pointer-events: none;
    z-index: 0;
}

/* Hide streamlit chrome */
#MainMenu, footer, header { visibility: hidden; }
.stDeployButton { display: none; }

/* Scrollbar */
::-webkit-scrollbar { width: 4px; }
::-webkit-scrollbar-track { background: var(--bg-base); }
::-webkit-scrollbar-thumb { background: var(--border-hi); border-radius: 4px; }

/* ===== SIDEBAR ===== */
[data-testid="stSidebar"] {
    background: linear-gradient(180deg, #0a1520 0%, #060b14 100%) !important;
    border-right: 1px solid var(--border) !important;
}
[data-testid="stSidebar"] > div { padding: 0 !important; }

.sidebar-brand {
    padding: 28px 20px 20px;
    border-bottom: 1px solid var(--border);
    margin-bottom: 8px;
}
.brand-logo {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 6px;
}
.brand-icon {
    width: 40px; height: 40px;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border-radius: 10px;
    display: flex; align-items: center; justify-content: center;
    font-size: 20px;
    box-shadow: 0 0 20px rgba(56,189,248,0.4);
}
.brand-name {
    font-size: 20px; font-weight: 800;
    letter-spacing: 2px;
    background: linear-gradient(90deg, var(--accent), var(--accent2));
    -webkit-background-clip: text; -webkit-text-fill-color: transparent;
}
.brand-tagline {
    font-size: 10px; color: var(--text-muted);
    letter-spacing: 3px; font-weight: 500;
    padding-left: 50px;
}

/* Nav items */
.stRadio > label { display: none !important; }
.stRadio > div {
    display: flex;
    flex-direction: column;
    gap: 2px;
    padding: 8px 12px;
}
.stRadio > div > label {
    padding: 10px 14px !important;
    border-radius: 10px !important;
    cursor: pointer !important;
    font-weight: 500 !important;
    font-size: 14px !important;
    color: var(--text-sec) !important;
    transition: all 0.2s ease !important;
    border: 1px solid transparent !important;
}
.stRadio > div > label:hover {
    background: rgba(56,189,248,0.08) !important;
    color: var(--accent) !important;
    border-color: var(--border) !important;
}
.stRadio > div > label[data-checked="true"],
.stRadio > div > label[aria-checked="true"] {
    background: linear-gradient(135deg, rgba(56,189,248,0.15), rgba(129,140,248,0.1)) !important;
    color: var(--accent) !important;
    border-color: var(--border-hi) !important;
}
.stRadio [data-testid="stMarkdownContainer"] p { margin: 0; }

/* ===== MAIN CONTENT ===== */
.main .block-container {
    padding: 24px 32px !important;
    max-width: 1400px !important;
}

/* ===== PAGE HEADER ===== */
.page-header {
    margin-bottom: 32px;
    padding-bottom: 20px;
    border-bottom: 1px solid var(--border);
}
.page-header h1 {
    font-size: 32px; font-weight: 800;
    margin: 0 0 6px;
    background: linear-gradient(90deg, var(--text-pri), var(--text-sec));
    -webkit-background-clip: text; -webkit-text-fill-color: transparent;
}
.page-header p {
    color: var(--text-muted); font-size: 14px; margin: 0;
}
.page-badge {
    display: inline-flex; align-items: center; gap: 6px;
    background: rgba(56,189,248,0.1); border: 1px solid var(--border-hi);
    padding: 4px 12px; border-radius: 20px;
    font-size: 11px; font-weight: 600; letter-spacing: 1px;
    color: var(--accent); margin-bottom: 12px;
}
.page-badge::before {
    content: ''; width: 6px; height: 6px;
    background: var(--accent3); border-radius: 50%;
    box-shadow: 0 0 6px var(--accent3);
    animation: pulse-dot 2s infinite;
}
@keyframes pulse-dot { 0%,100%{opacity:1} 50%{opacity:0.3} }

/* ===== GLASS CARD ===== */
.glass-card {
    background: var(--bg-glass);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 24px;
    backdrop-filter: blur(20px);
    transition: border-color 0.3s, box-shadow 0.3s;
    position: relative; overflow: hidden;
}
.glass-card::before {
    content: '';
    position: absolute; top: 0; left: 0; right: 0; height: 1px;
    background: linear-gradient(90deg, transparent, var(--accent), transparent);
    opacity: 0.5;
}
.glass-card:hover {
    border-color: var(--border-hi);
    box-shadow: var(--glow);
}

/* ===== TOOL CARD (dashboard) ===== */
.tool-card {
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: 16px; padding: 24px;
    text-align: center; cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4,0,0.2,1);
    position: relative; overflow: hidden;
    height: 100%;
}
.tool-card::after {
    content: '';
    position: absolute; inset: 0;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    opacity: 0;
    transition: opacity 0.3s;
}
.tool-card:hover { transform: translateY(-4px); border-color: var(--border-hi); box-shadow: var(--glow); }
.tool-icon { font-size: 36px; margin-bottom: 12px; position: relative; z-index:1; }
.tool-title { font-size: 15px; font-weight: 700; color: var(--text-pri); margin-bottom: 8px; position:relative;z-index:1; }
.tool-desc { font-size: 12px; color: var(--text-sec); line-height: 1.6; position:relative;z-index:1; }
.tool-badge {
    position: absolute; top: 12px; right: 12px;
    background: rgba(52,211,153,0.15); border: 1px solid rgba(52,211,153,0.3);
    color: var(--accent3); font-size: 9px; font-weight: 700;
    padding: 2px 8px; border-radius: 10px; letter-spacing: 1px;
}

/* ===== STAT CARDS ===== */
.stat-row { display: flex; gap: 16px; margin-bottom: 24px; flex-wrap: wrap; }
.stat-card {
    flex: 1; min-width: 140px;
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: 12px; padding: 16px 20px;
}
.stat-label { font-size: 11px; color: var(--text-muted); letter-spacing: 1px; font-weight: 600; margin-bottom: 6px; }
.stat-value { font-size: 28px; font-weight: 800; line-height: 1; }
.stat-sub { font-size: 11px; color: var(--text-muted); margin-top: 4px; }

/* ===== INPUTS ===== */
.stTextInput > div > div > input,
.stTextArea > div > div > textarea,
.stSelectbox > div > div {
    background: var(--bg-card) !important;
    border: 1px solid var(--border) !important;
    border-radius: 10px !important;
    color: var(--text-pri) !important;
    font-family: 'Outfit', sans-serif !important;
    font-size: 14px !important;
}
.stTextInput > div > div > input:focus,
.stTextArea > div > div > textarea:focus {
    border-color: var(--accent) !important;
    box-shadow: 0 0 0 2px rgba(56,189,248,0.15) !important;
}
.stTextInput label, .stTextArea label, .stSelectbox label,
.stMultiSelect label, .stCheckbox label, .stSlider label {
    color: var(--text-sec) !important;
    font-weight: 500 !important;
    font-size: 13px !important;
}

/* ===== FILE UPLOADER ===== */
[data-testid="stFileUploader"] {
    background: var(--bg-card) !important;
    border: 2px dashed var(--border) !important;
    border-radius: 16px !important;
    transition: border-color 0.3s !important;
}
[data-testid="stFileUploader"]:hover {
    border-color: var(--accent) !important;
}
[data-testid="stFileUploader"] p { color: var(--text-sec) !important; }

/* ===== BUTTONS ===== */
.stButton > button {
    background: linear-gradient(135deg, #0ea5e9, #6366f1) !important;
    border: none !important;
    border-radius: 10px !important;
    color: white !important;
    font-family: 'Outfit', sans-serif !important;
    font-weight: 600 !important;
    font-size: 14px !important;
    padding: 10px 20px !important;
    transition: all 0.2s !important;
    letter-spacing: 0.3px !important;
}
.stButton > button:hover {
    transform: translateY(-1px) !important;
    box-shadow: 0 8px 25px rgba(14,165,233,0.4) !important;
}
.stButton > button:active { transform: translateY(0) !important; }

/* Secondary button */
.btn-secondary > button {
    background: var(--bg-card) !important;
    border: 1px solid var(--border) !important;
    color: var(--text-sec) !important;
}

/* ===== RESULT BOXES ===== */
.result-box {
    background: linear-gradient(135deg, rgba(56,189,248,0.05), rgba(129,140,248,0.05));
    border: 1px solid var(--border-hi);
    border-radius: 12px; padding: 20px;
    margin-top: 16px;
}
.result-box h3 { color: var(--accent); font-size: 14px; font-weight: 700; margin-bottom: 12px; letter-spacing: 1px; }

.info-box {
    background: rgba(56,189,248,0.06);
    border: 1px solid rgba(56,189,248,0.2);
    border-radius: 10px; padding: 12px 16px;
    font-size: 13px; color: var(--text-sec);
    margin-top: 12px;
}
.warn-box {
    background: rgba(251,191,36,0.06);
    border: 1px solid rgba(251,191,36,0.2);
    border-radius: 10px; padding: 12px 16px;
    font-size: 13px; color: var(--warn);
    margin-top: 12px;
}
.success-box {
    background: rgba(52,211,153,0.06);
    border: 1px solid rgba(52,211,153,0.2);
    border-radius: 10px; padding: 12px 16px;
    font-size: 13px; color: var(--accent3);
    margin-top: 12px;
}
.danger-box {
    background: rgba(248,113,113,0.06);
    border: 1px solid rgba(248,113,113,0.2);
    border-radius: 10px; padding: 12px 16px;
    font-size: 13px; color: var(--danger);
    margin-top: 12px;
}

/* ===== TABS ===== */
.stTabs [data-baseweb="tab-list"] {
    background: var(--bg-card) !important;
    border-radius: 10px !important;
    padding: 4px !important;
    border: 1px solid var(--border) !important;
    gap: 4px !important;
}
.stTabs [data-baseweb="tab"] {
    background: transparent !important;
    border-radius: 7px !important;
    color: var(--text-muted) !important;
    font-family: 'Outfit', sans-serif !important;
    font-weight: 600 !important;
    font-size: 13px !important;
}
.stTabs [aria-selected="true"] {
    background: linear-gradient(135deg, rgba(56,189,248,0.2), rgba(129,140,248,0.15)) !important;
    color: var(--accent) !important;
}
.stTabs [data-baseweb="tab-border"] { display: none !important; }

/* ===== PROGRESS / SPINNER ===== */
.stSpinner > div { border-top-color: var(--accent) !important; }

/* ===== TABLE ===== */
.stDataFrame { border-radius: 10px; overflow: hidden; }
.stDataFrame table { background: var(--bg-card) !important; }
.stDataFrame th {
    background: rgba(56,189,248,0.1) !important;
    color: var(--accent) !important;
    font-family: 'Outfit', sans-serif !important;
    font-weight: 700 !important;
}
.stDataFrame td {
    color: var(--text-sec) !important;
    font-family: 'JetBrains Mono', monospace !important;
    font-size: 12px !important;
}

/* ===== ALERTS ===== */
.stSuccess, .stInfo, .stWarning, .stError {
    border-radius: 10px !important;
    font-family: 'Outfit', sans-serif !important;
}

/* ===== METRIC ===== */
[data-testid="stMetric"] {
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: 12px; padding: 16px;
}
[data-testid="stMetricLabel"] { color: var(--text-muted) !important; font-size: 12px !important; }
[data-testid="stMetricValue"] { color: var(--accent) !important; font-weight: 800 !important; }

/* ===== MARKDOWN ===== */
.stMarkdown h1, .stMarkdown h2, .stMarkdown h3 { color: var(--text-pri) !important; }
.stMarkdown p, .stMarkdown li { color: var(--text-sec) !important; font-size: 14px !important; }
.stMarkdown strong { color: var(--text-pri) !important; }
.stMarkdown table {
    border-collapse: collapse !important; width: 100% !important;
    font-size: 13px !important; border-radius: 10px !important;
    overflow: hidden !important;
}
.stMarkdown th {
    background: rgba(56,189,248,0.15) !important;
    color: var(--accent) !important; padding: 10px 14px !important;
    text-align: left !important; font-weight: 700 !important;
    border-bottom: 1px solid var(--border) !important;
}
.stMarkdown td {
    padding: 9px 14px !important;
    border-bottom: 1px solid rgba(255,255,255,0.04) !important;
    color: var(--text-sec) !important;
}
.stMarkdown tr:hover td { background: rgba(255,255,255,0.02) !important; }

/* ===== SIDEBAR STATS ===== */
.sidebar-stats {
    padding: 12px 16px;
    border-top: 1px solid var(--border);
    margin-top: auto;
}
.sidebar-stat-row { display: flex; justify-content: space-between; align-items: center; padding: 6px 0; }
.sidebar-stat-label { font-size: 11px; color: var(--text-muted); }
.sidebar-stat-val { font-size: 11px; color: var(--accent3); font-weight: 600; font-family: 'JetBrains Mono', monospace; }

/* ===== SYMPTOM CHECKER SPECIFIC ===== */
.symptom-chip {
    display: inline-flex; align-items: center; gap: 6px;
    background: rgba(56,189,248,0.1); border: 1px solid var(--border-hi);
    border-radius: 20px; padding: 4px 12px;
    font-size: 12px; color: var(--accent); margin: 3px;
    cursor: pointer;
}
.severity-low  { color: var(--accent3) !important; }
.severity-mid  { color: var(--warn) !important; }
.severity-high { color: var(--danger) !important; }

/* ===== HEALTH SCORE RING ===== */
.health-ring-wrapper {
    display: flex; flex-direction: column; align-items: center;
    padding: 20px;
}
.health-score-label { font-size: 13px; color: var(--text-muted); margin-top: 8px; }

/* ===== DIVIDER ===== */
hr { border-color: var(--border) !important; margin: 20px 0 !important; }

/* ===== MULTISELECT ===== */
.stMultiSelect [data-baseweb="tag"] {
    background: rgba(56,189,248,0.15) !important;
    color: var(--accent) !important;
    border-radius: 6px !important;
}

/* ===== CHECKBOX ===== */
.stCheckbox [data-testid="stCheckbox"] svg { color: var(--accent) !important; }

/* ===== SELECT SLIDER ===== */
.stSelectSlider [data-testid="stThumbValue"] { color: var(--accent) !important; }

/* ===== EXPANDER ===== */
.streamlit-expanderHeader {
    background: var(--bg-card) !important;
    border: 1px solid var(--border) !important;
    border-radius: 10px !important;
    color: var(--text-sec) !important;
    font-weight: 600 !important;
}
.streamlit-expanderContent {
    background: var(--bg-card) !important;
    border: 1px solid var(--border) !important;
    border-top: none !important;
}

/* ===== EMERGENCY BANNER ===== */
.emergency-banner {
    background: linear-gradient(135deg, rgba(248,113,113,0.15), rgba(251,191,36,0.1));
    border: 1px solid rgba(248,113,113,0.4);
    border-radius: 12px; padding: 16px 20px;
    display: flex; align-items: center; gap: 12px;
    margin-bottom: 16px;
}
.emergency-icon { font-size: 24px; }
.emergency-text h4 { color: var(--danger); font-size: 14px; font-weight: 700; margin: 0 0 4px; }
.emergency-text p { color: var(--text-sec); font-size: 12px; margin: 0; }

/* ===== ANALYSIS STEP PROGRESS ===== */
.step-progress { display: flex; align-items: center; gap: 8px; margin: 16px 0; }
.step-dot {
    width: 28px; height: 28px; border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-size: 11px; font-weight: 700;
}
.step-dot.done { background: rgba(52,211,153,0.2); border: 1px solid var(--accent3); color: var(--accent3); }
.step-dot.active { background: rgba(56,189,248,0.2); border: 1px solid var(--accent); color: var(--accent); animation: pulse-dot 1.5s infinite; }
.step-dot.pending { background: var(--bg-card); border: 1px solid var(--border); color: var(--text-muted); }
.step-line { flex: 1; height: 1px; background: var(--border); }
.step-line.done { background: var(--accent3); }

/* ===== SHIMMER LOADING ===== */
@keyframes shimmer {
    0% { background-position: -500px 0; }
    100% { background-position: 500px 0; }
}
.shimmer-line {
    height: 14px; border-radius: 7px; margin-bottom: 10px;
    background: linear-gradient(90deg, var(--bg-card) 25%, rgba(56,189,248,0.08) 50%, var(--bg-card) 75%);
    background-size: 500px 100%;
    animation: shimmer 1.5s infinite;
}

/* ===== QUICK QUESTION CHIPS ===== */
.quick-chip {
    display: inline-block;
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: 20px; padding: 5px 14px;
    font-size: 12px; color: var(--text-sec);
    cursor: pointer; margin: 3px;
    transition: all 0.2s;
}
.quick-chip:hover { border-color: var(--accent); color: var(--accent); }

/* ===== FOOTER ===== */
.footer {
    text-align: center;
    padding: 20px;
    border-top: 1px solid var(--border);
    margin-top: 40px;
    font-size: 11px;
    color: var(--text-muted);
    letter-spacing: 0.5px;
}

/* ===== ANIMATION ENTRY ===== */
@keyframes fadeUp {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0); }
}
.fade-up { animation: fadeUp 0.4s ease forwards; }
</style>
""", unsafe_allow_html=True)


# ========== HELPERS ==========
def page_header(icon, title, subtitle, badge=None):
    badge_html = f'<div class="page-badge">⚕️ {badge}</div>' if badge else ''
    st.markdown(f"""
    <div class="page-header fade-up">
        {badge_html}
        <h1>{icon} {title}</h1>
        <p>{subtitle}</p>
    </div>
    """, unsafe_allow_html=True)

def glass_card(content_html):
    st.markdown(f'<div class="glass-card fade-up">{content_html}</div>', unsafe_allow_html=True)

def info_box(text, kind="info"):
    icons = {"info":"ℹ️","warn":"⚠️","success":"✅","danger":"🚨"}
    st.markdown(f'<div class="{kind}-box">{icons.get(kind,"ℹ️")} {text}</div>', unsafe_allow_html=True)

def render_result(text, title="AI Analysis"):
    st.markdown(f"""
    <div class="result-box fade-up">
        <h3>⚕️ {title.upper()}</h3>
    </div>
    """, unsafe_allow_html=True)
    st.markdown(text)

def file_to_base64(uploaded_file):
    """Convert uploaded file to base64 string for API transmission"""
    if uploaded_file is None:
        return None
    return base64.b64encode(uploaded_file.read()).decode()

def extract_text_from_file(uploaded_file):
    """Try to extract readable text from uploaded file"""
    if uploaded_file is None:
        return ""
    try:
        file_type = uploaded_file.type
        content = uploaded_file.read()
        uploaded_file.seek(0)  # reset
        if 'text' in file_type:
            return content.decode('utf-8', errors='ignore')
        return f"[Binary file: {uploaded_file.name}, size: {len(content)} bytes]"
    except:
        return f"[File: {uploaded_file.name}]"

def animated_analyzing(steps):
    """Show animated step-by-step analysis progress"""
    placeholder = st.empty()
    for i, step in enumerate(steps):
        dots_html = ""
        for j, s in enumerate(steps):
            if j < i:
                cls = "done"; label = "✓"
            elif j == i:
                cls = "active"; label = str(j+1)
            else:
                cls = "pending"; label = str(j+1)
            dots_html += f'<div class="step-dot {cls}">{label}</div>'
            if j < len(steps)-1:
                dots_html += f'<div class="step-line {"done" if j < i else ""}"></div>'
        placeholder.markdown(f"""
        <div class="glass-card">
            <div style="font-size:13px;color:var(--text-muted);margin-bottom:12px;font-weight:600;letter-spacing:1px;">ANALYZING</div>
            <div class="step-progress">{dots_html}</div>
            <div style="font-size:13px;color:var(--text-sec);margin-top:12px;">
                🔬 {step}
            </div>
        </div>
        """, unsafe_allow_html=True)
        time.sleep(0.6)
    placeholder.empty()


# ========== SESSION STATE ==========
if 'selected_tool' not in st.session_state:
    st.session_state.selected_tool = "📊 Dashboard"
if 'qna_history' not in st.session_state:
    st.session_state.qna_history = []
if 'total_queries' not in st.session_state:
    st.session_state.total_queries = 0
if 'total_savings' not in st.session_state:
    st.session_state.total_savings = 0

# ========== SIDEBAR ==========
with st.sidebar:
    st.markdown("""
    <div class="sidebar-brand">
        <div class="brand-logo">
            <div class="brand-icon">⚕️</div>
            <div class="brand-name">CHRONOCHECK</div>
        </div>
        <div class="brand-tagline">CHECK • CARE • CLARITY</div>
    </div>
    """, unsafe_allow_html=True)

    page_options = [
        "📊 Dashboard",
        "🧠 Medical Q&A",
        "📄 Report Analyzer",
        "🏥 Hospital Finder",
        "💊 Medicine Explainer",
        "💰 Bill Auditor",
        "🚨 Symptom Checker",
    ]
    selected_page = st.radio("", page_options,
        index=page_options.index(st.session_state.selected_tool)
              if st.session_state.selected_tool in page_options else 0,
        label_visibility="collapsed")
    st.session_state.selected_tool = selected_page

    st.markdown("<hr/>", unsafe_allow_html=True)

    # Sidebar live stats
    now = datetime.now()
    st.markdown(f"""
    <div style="padding:0 8px;">
        <div style="font-size:10px;color:var(--text-muted);letter-spacing:2px;font-weight:700;margin-bottom:10px;">SESSION STATS</div>
        <div class="sidebar-stat-row">
            <span class="sidebar-stat-label">Queries</span>
            <span class="sidebar-stat-val">{st.session_state.total_queries}</span>
        </div>
        <div class="sidebar-stat-row">
            <span class="sidebar-stat-label">Savings Found</span>
            <span class="sidebar-stat-val">₹{st.session_state.total_savings:,}</span>
        </div>
        <div class="sidebar-stat-row">
            <span class="sidebar-stat-label">Session</span>
            <span class="sidebar-stat-val">{now.strftime('%H:%M')}</span>
        </div>
        <div class="sidebar-stat-row">
            <span class="sidebar-stat-label">AI Status</span>
            <span class="sidebar-stat-val" style="color:var(--accent3);">● ONLINE</span>
        </div>
    </div>
    """, unsafe_allow_html=True)

    st.markdown("<hr/>", unsafe_allow_html=True)
    st.markdown("""
    <div style="padding:0 8px;font-size:10px;color:var(--text-muted);line-height:1.8;">
        ⚠️ For informational purposes only.<br/>
        Always consult a licensed physician.
    </div>
    """, unsafe_allow_html=True)


# ================================================================
# ========================= DASHBOARD ============================
# ================================================================
if st.session_state.selected_tool == "📊 Dashboard":
    st.markdown("""
    <div class="fade-up" style="text-align:center;padding:32px 0 16px;">
        <div style="font-size:11px;letter-spacing:4px;color:var(--text-muted);font-weight:700;margin-bottom:12px;">AI-POWERED HEALTHCARE TOOLS</div>
        <h1 style="font-size:42px;font-weight:900;background:linear-gradient(135deg,#e2e8f0,#94a3b8);-webkit-background-clip:text;-webkit-text-fill-color:transparent;margin:0 0 8px;">
            Your Medical AI<br/>Command Center
        </h1>
        <p style="color:var(--text-muted);font-size:15px;max-width:500px;margin:0 auto;">
            Six specialized AI agents for smarter, more informed healthcare decisions.
        </p>
    </div>
    """, unsafe_allow_html=True)

    # Hero stats
    c1, c2, c3, c4 = st.columns(4)
    with c1:
        st.metric("AI Tools", "6", "Specialized agents")
    with c2:
        st.metric("Languages", "10+", "Multilingual support")
    with c3:
        st.metric("File Formats", "5+", "PDF, Images, DOCX")
    with c4:
        st.metric("Response Time", "~5s", "Average")

    st.markdown("<br/>", unsafe_allow_html=True)

    # Tool cards
    tools = [
        {"icon":"🧠","title":"Medical Q&A","desc":"Ask any medical question. Get expert-level answers in 10+ languages at your chosen complexity level.","badge":"AI","page":"🧠 Medical Q&A","color":"#38bdf8"},
        {"icon":"📄","title":"Report Analyzer","desc":"Upload lab reports, scans, or discharge summaries for instant AI-powered analysis.","badge":"OCR","page":"📄 Report Analyzer","color":"#34d399"},
        {"icon":"🏥","title":"Hospital Finder","desc":"Find specialized hospitals by condition, location, and specialty across India.","badge":"LIVE","page":"🏥 Hospital Finder","color":"#818cf8"},
        {"icon":"💊","title":"Medicine Explainer","desc":"Decode your prescription. Understand doses, side effects, and generic alternatives.","badge":"AI","page":"💊 Medicine Explainer","color":"#fbbf24"},
        {"icon":"💰","title":"Bill Auditor","desc":"Detect overcharges, duplicate billing, and inflated costs in your medical bills.","badge":"AUDIT","page":"💰 Bill Auditor","color":"#f87171"},
        {"icon":"🚨","title":"Symptom Checker","desc":"Describe symptoms and get an AI triage assessment with urgency level and next steps.","badge":"NEW","page":"🚨 Symptom Checker","color":"#fb923c"},
    ]

    cols = st.columns(3)
    for i, tool in enumerate(tools):
        with cols[i % 3]:
            st.markdown(f"""
            <div class="tool-card fade-up" style="--delay:{i*0.05}s">
                <div class="tool-badge">{tool['badge']}</div>
                <div class="tool-icon">{tool['icon']}</div>
                <div class="tool-title" style="color:{tool['color']}">{tool['title']}</div>
                <div class="tool-desc">{tool['desc']}</div>
            </div>
            """, unsafe_allow_html=True)
            if st.button(f"Open {tool['title']} →", key=f"db_{i}", use_container_width=True):
                st.session_state.selected_tool = tool['page']
                st.rerun()
            st.markdown("<br/>", unsafe_allow_html=True)

    # How it works
    st.markdown("<hr/>", unsafe_allow_html=True)
    st.markdown('<div style="text-align:center;font-size:11px;letter-spacing:3px;color:var(--text-muted);font-weight:700;margin-bottom:20px;">HOW IT WORKS</div>', unsafe_allow_html=True)
    hc1, hc2, hc3, hc4 = st.columns(4)
    steps_hw = [
        ("01","Select Tool","Choose from 6 AI-powered healthcare modules"),
        ("02","Input Data","Type a question or upload a file"),
        ("03","AI Analyzes","Multi-step AI processing with context-aware prompts"),
        ("04","Get Insights","Receive structured, actionable medical insights"),
    ]
    for col, (num, title, desc) in zip([hc1,hc2,hc3,hc4], steps_hw):
        with col:
            st.markdown(f"""
            <div class="glass-card" style="text-align:center;padding:20px 16px;">
                <div style="font-size:28px;font-weight:900;color:var(--border-hi);margin-bottom:8px;">{num}</div>
                <div style="font-size:14px;font-weight:700;color:var(--text-pri);margin-bottom:6px;">{title}</div>
                <div style="font-size:12px;color:var(--text-muted);">{desc}</div>
            </div>
            """, unsafe_allow_html=True)


# ================================================================
# ========================= MEDICAL Q&A ==========================
# ================================================================
elif st.session_state.selected_tool == "🧠 Medical Q&A":
    page_header("🧠", "Medical Q&A", "Ask medical questions and get AI-powered answers in your language", "MEDICAL Q&A")

    # Quick questions
    st.markdown("""
    <div style="margin-bottom:16px;">
        <div style="font-size:11px;color:var(--text-muted);letter-spacing:2px;font-weight:600;margin-bottom:8px;">QUICK QUESTIONS</div>
        <div>
            <span class="quick-chip">What is HbA1c?</span>
            <span class="quick-chip">Symptoms of diabetes</span>
            <span class="quick-chip">How to read a CBC report?</span>
            <span class="quick-chip">What is creatinine?</span>
            <span class="quick-chip">High blood pressure diet</span>
        </div>
    </div>
    """, unsafe_allow_html=True)

    col1, col2 = st.columns([1,1])
    with col1:
        output_language = st.selectbox("🌍 Response Language", [
            "English","Hindi (हिंदी)","Marathi (मराठी)","Tamil (தமிழ்)",
            "Telugu (తెలుగు)","Bengali (বাংলা)","Gujarati (ગુજરાતી)",
            "Kannada (ಕನ್ನಡ)","Malayalam (മലയാളം)","Punjabi (ਪੰਜਾਬੀ)"
        ])
    with col2:
        expertise = st.selectbox("📚 Explanation Level", [
            "Patient-Friendly — Simple, no jargon",
            "Medical Student — Intermediate terminology",
            "Professional — Full clinical detail"
        ])

    question = st.text_area("💬 Your Medical Question",
        height=130,
        placeholder="Type your question in any language...\n\nExamples:\n• What does elevated ALT mean in a liver function test?\n• मुझे बार-बार सिरदर्द क्यों होता है?\n• What are the side effects of Metformin?")

    # History toggle
    if st.session_state.qna_history:
        with st.expander(f"📜 Question History ({len(st.session_state.qna_history)} queries)"):
            for i, (q, a) in enumerate(reversed(st.session_state.qna_history[-5:]), 1):
                st.markdown(f"**Q{i}:** {q[:80]}...")
                st.markdown(f"*{a[:120]}...*")
                st.markdown("---")

    if st.button("🔍 Get Medical Answer", type="primary", use_container_width=True):
        if not question.strip():
            st.warning("⚠️ Please enter your question.")
        else:
            lang_name = output_language.split(" (")[0]
            level = expertise.split(" —")[0]

            level_prompts = {
                "Patient-Friendly": "Please explain this simply, as if talking to a patient with no medical background. Avoid jargon. Use analogies where helpful.",
                "Medical Student": "Explain at a medical student level. Include relevant pathophysiology, clinical correlations, and medical terminology with brief explanations.",
                "Professional": "Provide a comprehensive, professional clinical analysis. Include mechanisms, differentials, clinical guidelines, evidence-based recommendations, and full medical terminology."
            }

            enhanced_q = question
            enhanced_q += f"\n\n[Instruction: {level_prompts.get(level, '')}]"
            if lang_name != "English":
                enhanced_q += f"\n\n[CRITICAL: Respond ENTIRELY in {lang_name}. All explanations, headings, and content must be in {lang_name}.]"

            with st.spinner(""):
                animated_analyzing([
                    "Parsing medical query...",
                    "Retrieving medical knowledge base...",
                    f"Generating {level} response in {lang_name}...",
                    "Formatting structured answer..."
                ])
                result = api.qna_medical(enhanced_q)

            st.session_state.total_queries += 1

            if result.get("success"):
                answer = result["message"]
                st.session_state.qna_history.append((question, answer))

                st.markdown(f"""
                <div class="glass-card fade-up">
                    <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;">
                        <div style="font-size:11px;color:var(--text-muted);letter-spacing:2px;font-weight:700;">YOUR QUESTION</div>
                        <div style="font-size:10px;color:var(--accent);background:rgba(56,189,248,0.1);padding:3px 10px;border-radius:10px;border:1px solid var(--border-hi);">{level} · {lang_name}</div>
                    </div>
                    <div style="font-size:14px;color:var(--text-sec);font-style:italic;">"{question}"</div>
                </div>
                """, unsafe_allow_html=True)

                st.markdown("<br/>", unsafe_allow_html=True)
                render_result(answer, f"AI Answer — {level}")
                info_box("This is AI-generated information. Always consult a licensed healthcare professional for medical decisions.", "warn")
            else:
                st.error(f"❌ {result.get('error', 'Unknown error')}")


# ================================================================
# ======================== REPORT ANALYZER =======================
# ================================================================
elif st.session_state.selected_tool == "📄 Report Analyzer":
    page_header("📄", "Report Analyzer", "Upload your medical report for comprehensive AI analysis", "REPORT ANALYSIS")

    st.markdown("""
    <div class="info-box">
        📎 Supports: <strong>PDF, Images (JPG/PNG), Text files, DOCX</strong> — Max 200MB
    </div>
    """, unsafe_allow_html=True)
    st.markdown("<br/>", unsafe_allow_html=True)

    uploaded_file = st.file_uploader("📤 Upload Medical Report",
        type=['txt','pdf','docx','jpg','png','jpeg'],
        help="Upload lab report, discharge summary, scan report, or any medical document")

    if uploaded_file:
        file_size = uploaded_file.size / 1024
        st.markdown(f"""
        <div class="success-box">
            ✅ <strong>{uploaded_file.name}</strong> uploaded successfully
            <span style="float:right;font-size:11px;opacity:0.7;">{file_size:.1f} KB · {uploaded_file.type}</span>
        </div>
        """, unsafe_allow_html=True)

        st.markdown("<br/>", unsafe_allow_html=True)
        col1, col2 = st.columns(2)
        with col1:
            analysis_type = st.selectbox("Analysis Focus", [
                "Comprehensive — Full report analysis",
                "Abnormal Values — Flag out-of-range results",
                "Risk Assessment — Evaluate health risks",
                "Quick Summary — Brief overview",
                "Trend Analysis — Changes over time",
                "Diet & Lifestyle Recommendations",
            ])
            output_lang = st.selectbox("Response Language", ["English","Hindi","Marathi","Tamil","Telugu","Bengali"])
        with col2:
            include_normal_range = st.checkbox("Show Normal Ranges", True)
            include_recommendations = st.checkbox("Include Recommendations", True)
            include_risk_flags = st.checkbox("Flag Risk Indicators", True)
            patient_age = st.number_input("Patient Age (for context)", 0, 120, 35, help="Helps AI adjust normal ranges")

        additional_notes = st.text_area("Additional Context (optional)",
            height=80, placeholder="E.g., Patient has Type 2 Diabetes, on Metformin 500mg. Compare with previous CBC from last month...")

        if st.button("🔬 Analyze Report", type="primary", use_container_width=True):
            analysis_focus = analysis_type.split(" —")[0]
            parts = [f"{analysis_focus} analysis", f"Patient age: {patient_age}"]
            if include_normal_range: parts.append("Include normal ranges")
            if include_recommendations: parts.append("Provide actionable recommendations")
            if include_risk_flags: parts.append("Flag any critical/risk values with urgency level")
            if additional_notes: parts.append(f"Additional context: {additional_notes}")
            if output_lang != "English": parts.append(f"Respond in {output_lang}")

            analysis_msg = " | ".join(parts)

            with st.spinner(""):
                animated_analyzing([
                    "Reading uploaded document...",
                    "Extracting medical parameters...",
                    f"Running {analysis_focus} analysis...",
                    "Cross-referencing medical databases...",
                    "Generating structured report..."
                ])
                result = api.analyze_report(analysis_msg, file_uploaded=True, file_name=uploaded_file.name)

            st.session_state.total_queries += 1

            if result.get("success"):
                tab1, tab2 = st.tabs(["📋 Analysis Results", "ℹ️ About This Report"])
                with tab1:
                    render_result(result["message"], "Report Analysis")
                    info_box("Results are AI-generated. Consult your doctor for clinical decisions.", "warn")
                with tab2:
                    st.markdown(f"""
                    **File:** {uploaded_file.name}
                    **Size:** {file_size:.1f} KB
                    **Analysis type:** {analysis_focus}
                    **Patient age context:** {patient_age} years
                    **Language:** {output_lang}
                    """)
            else:
                st.error(f"❌ {result.get('error')}")
    else:
        # Placeholder when no file
        st.markdown("""
        <div class="glass-card" style="text-align:center;padding:48px;opacity:0.6;">
            <div style="font-size:48px;margin-bottom:12px;">📄</div>
            <div style="color:var(--text-muted);font-size:14px;">Upload a medical report to begin analysis</div>
        </div>
        """, unsafe_allow_html=True)


# ================================================================
# ======================== HOSPITAL FINDER =======================
# ================================================================
elif st.session_state.selected_tool == "🏥 Hospital Finder":
    page_header("🏥", "Hospital Finder", "Find the right hospital for your medical needs", "HOSPITAL SEARCH")

    col1, col2 = st.columns([3,2])
    with col1:
        query = st.text_input("🔍 What medical service do you need?",
            placeholder="E.g., Cardiac bypass surgery, NICU, Cancer chemotherapy, Bone marrow transplant...")
        specializations = st.multiselect("Specializations needed:", [
            "Cardiology","Neurology","Orthopedics","Pediatrics","Oncology",
            "Nephrology","Gastroenterology","Pulmonology","General Surgery",
            "Emergency & Trauma","Dermatology","Psychiatry","Ophthalmology",
            "ENT","Gynecology","Urology","Endocrinology","Rheumatology"
        ])
    with col2:
        location = st.selectbox("📍 City / Region", [
            "Pune","Mumbai","Delhi","Chennai","Bangalore","Hyderabad",
            "Kolkata","Ahmedabad","Nagpur","Nashik","Aurangabad",
            "Indore","Bhopal","Lucknow","Jaipur","Chandigarh"
        ])
        preferences = st.multiselect("Preferences:", [
            "NABH Accredited","NABL Lab","Insurance Empanelled",
            "24×7 Emergency","Government Hospital","Private Hospital",
            "Teaching Hospital","Day Care Center"
        ])

    insurance_info = st.text_input("Insurance / TPA (optional):",
        placeholder="E.g., Star Health, Medi Assist, CGHS, Ayushman Bharat...")

    if st.button("🔍 Find Best Hospitals", type="primary", use_container_width=True):
        if not query:
            st.warning("Please describe what you're looking for.")
        else:
            search_q = f"Find hospitals in {location} for: {query}"
            if specializations: search_q += f" | Specializations: {', '.join(specializations)}"
            if preferences: search_q += f" | Preferences: {', '.join(preferences)}"
            if insurance_info: search_q += f" | Insurance: {insurance_info}"
            search_q += " | Provide hospital names, estimated costs, contact info, and recommendation reasoning."

            with st.spinner(""):
                animated_analyzing([
                    f"Searching hospitals in {location}...",
                    "Filtering by specialization...",
                    "Checking accreditation and quality ratings...",
                    "Ranking by relevance to your needs..."
                ])
                result = api.find_hospitals(search_q, location)

            st.session_state.total_queries += 1

            if result.get("success"):
                render_result(result["message"], f"Hospitals in {location}")
                info_box("Always verify hospital details, availability, and costs directly before visiting.", "warn")
            else:
                st.error(f"❌ {result.get('error')}")


# ================================================================
# ====================== MEDICINE EXPLAINER ======================
# ================================================================
elif st.session_state.selected_tool == "💊 Medicine Explainer":
    page_header("💊", "Medicine Explainer", "Understand your prescription — doses, side effects, interactions, and generics", "RX ANALYSIS")

    uploaded_file = st.file_uploader("📤 Upload Prescription or Medicine List",
        type=['txt','pdf','docx','jpg','png','jpeg'])

    if not uploaded_file:
        st.markdown("""
        <div class="info-box">
            💡 Tip: You can also type medicine names directly in the box below without uploading a file.
        </div>
        """, unsafe_allow_html=True)

    if uploaded_file:
        file_size = uploaded_file.size / 1024
        st.markdown(f'<div class="success-box">✅ <strong>{uploaded_file.name}</strong> — {file_size:.1f} KB</div>', unsafe_allow_html=True)

    st.markdown("<br/>", unsafe_allow_html=True)

    # Medicine name input (works even without file)
    medicine_input = st.text_area("💊 Or type medicine names / prescription text:",
        height=100,
        placeholder="E.g., Tab. Metformin 500mg BD, Tab. Atorvastatin 40mg OD, Tab. Amlodipine 5mg OD\n\nOr just: Metformin, Aspirin, Pantoprazole")

    col1, col2, col3 = st.columns(3)
    with col1:
        detail_level = st.select_slider("Detail Level:", options=["Basic","Moderate","Detailed","Expert"])
        patient_conditions = st.text_input("Patient conditions (optional):", placeholder="Diabetes, hypertension, kidney disease...")
    with col2:
        include_generics = st.checkbox("💰 Generic Alternatives", True)
        check_interactions = st.checkbox("⚠️ Drug Interactions", True)
        include_side_effects = st.checkbox("🩺 Side Effects", True)
    with col3:
        include_food = st.checkbox("🍽️ Food Interactions", True)
        include_timing = st.checkbox("⏰ Best Time to Take", True)
        include_missed = st.checkbox("📌 Missed Dose Guidance", False)

    if st.button("🔬 Analyze Medicines", type="primary", use_container_width=True):
        if not uploaded_file and not medicine_input.strip():
            st.warning("Please upload a prescription or enter medicine names.")
        else:
            parts = [f"{detail_level} medicine analysis"]
            if medicine_input.strip(): parts.append(f"Medicines/text: {medicine_input}")
            if patient_conditions: parts.append(f"Patient conditions: {patient_conditions}")
            if include_generics: parts.append("List generic alternatives with cost savings percentage")
            if check_interactions: parts.append("Check all drug-drug interactions with severity levels")
            if include_side_effects: parts.append("List common and serious side effects")
            if include_food: parts.append("List food-drug interactions")
            if include_timing: parts.append("Provide optimal timing for each medicine")
            if include_missed: parts.append("Include missed dose instructions")

            analysis_msg = " | ".join(parts)

            with st.spinner(""):
                animated_analyzing([
                    "Reading prescription...",
                    "Identifying medicines and doses...",
                    "Checking interaction database...",
                    "Finding generic alternatives...",
                    "Compiling medicine guide..."
                ])
                result = api.explain_medicines(analysis_msg,
                    file_uploaded=bool(uploaded_file),
                    file_name=uploaded_file.name if uploaded_file else None)

            st.session_state.total_queries += 1

            if result.get("success"):
                tab1, tab2, tab3 = st.tabs(["💊 Medicine Guide", "💰 Cost Savings", "⚠️ Safety Notes"])
                with tab1:
                    render_result(result["message"], "Prescription Analysis")
                with tab2:
                    if include_generics:
                        st.markdown("""
### 💰 How to Save on Medicines

**Key strategies:**
- Ask your pharmacist specifically for the **generic/salt name** equivalent
- Generic medicines have the **same active ingredient** and efficacy as branded ones
- Jan Aushadhi Kendras (government generic stores) offer 50–90% savings
- Always cross-check with your doctor before switching brands

| Brand Type | Typical Price | Generic Equivalent | Typical Savings |
|------------|---------------|-------------------|-----------------|
| Branded Metformin | ₹80–120/strip | Metformin HCl 500mg | ~85% |
| Branded Atorvastatin | ₹150–200/strip | Atorvastatin 40mg | ~78% |
| Branded Amlodipine | ₹60–100/strip | Amlodipine 5mg | ~80% |
| Branded Pantoprazole | ₹70–90/strip | Pantoprazole 40mg | ~75% |
                        """)
                    else:
                        st.info("Enable 'Generic Alternatives' to see cost savings.")
                with tab3:
                    st.markdown("""
### ⚠️ Medicine Safety Checklist

- ✅ Always take medicines at the prescribed dose and time
- ✅ Complete the full course, especially for antibiotics
- ✅ Store medicines at the recommended temperature
- ⚠️ Never share prescription medicines with others
- 🚨 Seek emergency care for severe allergic reactions (rash, breathing difficulty, swelling)
- 📞 Keep Poison Control helpline handy: **1800-11-9000**
                    """)
                    info_box("This information is educational. Always consult your prescribing doctor before making changes.", "warn")
            else:
                st.error(f"❌ {result.get('error')}")


# ================================================================
# ========================= BILL AUDITOR =========================
# ================================================================
elif st.session_state.selected_tool == "💰 Bill Auditor":
    page_header("💰", "Medical Bill Auditor", "Detect overcharges, duplicate billing, and inflated costs in your hospital bills", "BILL AUDIT")

    # Emergency awareness banner
    st.markdown("""
    <div class="emergency-banner">
        <div class="emergency-icon">💡</div>
        <div class="emergency-text">
            <h4>Know Your Rights</h4>
            <p>You have the right to request an itemized bill. Hospitals must provide this under CGHS/Insurance guidelines. Our AI identifies discrepancies to help you negotiate.</p>
        </div>
    </div>
    """, unsafe_allow_html=True)

    uploaded_file = st.file_uploader("📤 Upload Medical Bill",
        type=['txt','pdf','docx','jpg','png','jpeg'],
        help="Upload your hospital bill, discharge summary with costs, or pharmacy invoice")

    if uploaded_file:
        file_size = uploaded_file.size / 1024
        st.markdown(f'<div class="success-box">✅ <strong>{uploaded_file.name}</strong> — {file_size:.1f} KB</div>', unsafe_allow_html=True)
        st.markdown("<br/>", unsafe_allow_html=True)

        col1, col2 = st.columns(2)
        with col1:
            st.markdown("**🔍 Audit Checks:**")
            check_overcharges  = st.checkbox("Overcharges vs. standard rates", True)
            check_duplicates   = st.checkbox("Duplicate / double-billed items", True)
            check_unbundling   = st.checkbox("Unbundling of procedure charges", True)
            check_upcoding     = st.checkbox("Upcoding / unnecessary upgrades", True)
        with col2:
            hospital_type = st.selectbox("Hospital Type:", ["Private","Government","Trust Hospital","Corporate Chain"])
            insurance_type = st.selectbox("Payment Mode:", ["Self Pay","Health Insurance","CGHS","ECHS","Ayushman Bharat","ESI"])
            city = st.selectbox("City (for local rate comparison):", [
                "Pune","Mumbai","Delhi","Chennai","Bangalore","Hyderabad","Kolkata","Nagpur"
            ])

        additional_notes = st.text_area("Additional context (optional):",
            height=80, placeholder="E.g., Admitted for appendectomy, 3-day stay, semi-private room, covered under Star Health policy...")

        if st.button("🔍 Audit This Bill", type="primary", use_container_width=True):
            parts = ["Comprehensive medical bill audit"]
            if check_overcharges: parts.append("Check all items vs. standard government/NPPA rates")
            if check_duplicates: parts.append("Identify duplicate charges")
            if check_unbundling: parts.append("Flag unbundled procedure items")
            if check_upcoding: parts.append("Flag potential upcoding")
            parts.append(f"Hospital type: {hospital_type} | Payment: {insurance_type} | City: {city}")
            if additional_notes: parts.append(f"Context: {additional_notes}")
            parts.append("Provide an itemized table, total potential overcharge, and specific recommendations to dispute each item.")

            analysis_msg = " | ".join(parts)

            with st.spinner(""):
                animated_analyzing([
                    "Reading bill items...",
                    "Comparing with NPPA / government standard rates...",
                    "Scanning for duplicate and bundled charges...",
                    "Calculating overcharge totals...",
                    "Generating dispute recommendations..."
                ])
                result = api.analyze_bill(analysis_msg, file_uploaded=True, file_name=uploaded_file.name)

            st.session_state.total_queries += 1

            success = result.get("success", False)
            message = result.get("message", "")
            is_demo = result.get("demo_mode", False)

            if is_demo:
                st.markdown("""
                <div class="warn-box">
                    ⚠️ <strong>Demo Mode:</strong> Showing a sample audit report. Connect your Langflow API for real bill analysis.
                </div>
                """, unsafe_allow_html=True)

            if message:
                tab1, tab2 = st.tabs(["📊 Audit Report", "📋 How to Dispute"])
                with tab1:
                    render_result(message, "Bill Audit Report")
                    # Extract potential savings from message if available
                    savings_match = re.search(r'₹([\d,]+)\.00.*[Oo]vercharge', message)
                    if savings_match:
                        savings_num = int(savings_match.group(1).replace(',',''))
                        st.session_state.total_savings += savings_num
                        st.markdown(f"""
                        <div class="success-box" style="text-align:center;padding:20px;">
                            <div style="font-size:28px;font-weight:900;color:var(--accent3);">₹{savings_num:,}</div>
                            <div style="font-size:13px;color:var(--text-sec);">Potential amount you can dispute</div>
                        </div>
                        """, unsafe_allow_html=True)
                with tab2:
                    st.markdown("""
### 📋 Step-by-Step Dispute Guide

**Step 1 — Request Itemized Bill**
Ask the billing department for a fully itemized bill (mandatory under consumer protection rules).

**Step 2 — Contact Hospital TPA/Billing Manager**
Present the audit report. Quote specific items and standard rates. Be calm but firm.

**Step 3 — Insurance Escalation**
If on insurance, contact your TPA/insurer about overcharged items. They have pre-agreed rates.

**Step 4 — Consumer Forum**
For unresolved disputes, file a complaint at:
- **District Consumer Disputes Redressal Forum**
- **National Consumer Helpline:** 1800-11-4000
- **IRDAI (insurance):** 1800-4254-732

**Step 5 — Legal Recourse**
Medical billing fraud can be pursued under Consumer Protection Act 2019.

---
⚖️ *Save all receipts, discharge summaries, and billing communications as evidence.*
                    """)
            elif not success:
                st.error(f"❌ {result.get('error', 'Unknown error')}")
    else:
        st.markdown("""
        <div class="glass-card" style="text-align:center;padding:48px;opacity:0.6;">
            <div style="font-size:48px;margin-bottom:12px;">💰</div>
            <div style="color:var(--text-muted);font-size:14px;">Upload a medical bill to start the audit</div>
        </div>
        """, unsafe_allow_html=True)


# ================================================================
# ====================== SYMPTOM CHECKER =========================
# ================================================================
elif st.session_state.selected_tool == "🚨 Symptom Checker":
    page_header("🚨", "Symptom Checker", "Describe your symptoms for an AI triage assessment with urgency classification", "SYMPTOM TRIAGE")

    st.markdown("""
    <div class="danger-box">
        🚨 <strong>Emergency:</strong> If you have chest pain, difficulty breathing, signs of stroke, or uncontrolled bleeding —
        call <strong>108</strong> (Emergency) immediately. Do NOT use this tool for emergencies.
    </div>
    """, unsafe_allow_html=True)
    st.markdown("<br/>", unsafe_allow_html=True)

    col1, col2, col3 = st.columns(3)
    with col1:
        age = st.number_input("Age", 0, 120, 30)
        gender = st.selectbox("Gender", ["Male","Female","Other"])
    with col2:
        duration = st.selectbox("Symptom Duration", [
            "Just started (minutes-hours)",
            "1–3 days",
            "4–7 days",
            "1–4 weeks",
            "More than 1 month"
        ])
        severity = st.select_slider("Severity (1–10):", options=[str(i) for i in range(1,11)], value="5")
    with col3:
        known_conditions = st.text_area("Known Medical Conditions:", height=80,
            placeholder="Diabetes, Hypertension, Asthma...")
        current_meds = st.text_area("Current Medicines:", height=80,
            placeholder="Metformin, Amlodipine...")

    symptoms = st.text_area("🩺 Describe Your Symptoms in Detail:",
        height=150,
        placeholder="Describe all your symptoms clearly...\n\nExample: I have had a severe headache for 2 days, mostly on the right side, with sensitivity to light and nausea. The pain gets worse with movement. I also have a mild fever of 100.4°F...")

    # Common symptom quick-add chips
    st.markdown("""
    <div style="margin-bottom:16px;">
        <div style="font-size:11px;color:var(--text-muted);letter-spacing:2px;font-weight:600;margin-bottom:8px;">QUICK ADD SYMPTOMS</div>
        <span class="symptom-chip">🤕 Headache</span>
        <span class="symptom-chip">🤒 Fever</span>
        <span class="symptom-chip">😮‍💨 Shortness of breath</span>
        <span class="symptom-chip">🤢 Nausea</span>
        <span class="symptom-chip">💢 Chest pain</span>
        <span class="symptom-chip">🦴 Joint pain</span>
        <span class="symptom-chip">😴 Fatigue</span>
        <span class="symptom-chip">🔴 Rash</span>
    </div>
    """, unsafe_allow_html=True)

    if st.button("🔍 Analyze Symptoms", type="primary", use_container_width=True):
        if not symptoms.strip():
            st.warning("Please describe your symptoms.")
        else:
            prompt = f"""Patient: {age}-year-old {gender}
Symptoms: {symptoms}
Duration: {duration}
Severity: {severity}/10
Known conditions: {known_conditions if known_conditions else 'None'}
Current medications: {current_meds if current_meds else 'None'}

Please provide:
1. TRIAGE LEVEL: (Emergency / Urgent / Semi-Urgent / Non-Urgent) with color coding
2. POSSIBLE CONDITIONS: Top 3-5 differential diagnoses with likelihood
3. RED FLAGS: Any emergency warning signs to watch for
4. IMMEDIATE ACTIONS: What to do right now
5. RECOMMENDED SPECIALIST: Which type of doctor to consult
6. HOME CARE: Safe symptomatic relief while awaiting appointment
7. TIMELINE: When to seek care (immediately / within 24h / within a week / routine)

Format clearly with headings. Include a clear triage classification at the top."""

            with st.spinner(""):
                animated_analyzing([
                    "Parsing symptom profile...",
                    f"Analyzing {age}-year-old {gender} patient data...",
                    "Running differential diagnosis engine...",
                    "Calculating triage urgency...",
                    "Generating care recommendations..."
                ])
                result = api.qna_medical(prompt)

            st.session_state.total_queries += 1

            if result.get("success"):
                answer = result["message"]

                tab1, tab2 = st.tabs(["🩺 Triage Assessment", "📞 Emergency Contacts"])
                with tab1:
                    render_result(answer, "Symptom Triage Report")
                    st.markdown("""
                    <div class="danger-box">
                        🚨 <strong>DISCLAIMER:</strong> This AI triage is NOT a diagnosis. It is for informational guidance only.
                        Always consult a qualified physician. In any emergency, call <strong>108</strong> immediately.
                    </div>
                    """, unsafe_allow_html=True)
                with tab2:
                    st.markdown("""
### 📞 Emergency & Health Helplines (India)

| Service | Number |
|---------|--------|
| 🚨 National Emergency (Ambulance/Police/Fire) | **112** |
| 🏥 Ambulance (MJES) | **108** |
| 👩‍⚕️ Health Helpline (Ministry of Health) | **104** |
| 💊 Poison Control | **1800-11-9000** |
| 🧠 iCall Mental Health | **9152987821** |
| 🏥 AIIMS OPD (Delhi) | **011-26589142** |
| 👴 Senior Citizen Helpline | **14567** |
| 🤰 Janani Suraksha Yojana | **102** |

*Save these in your phone!*
                    """)
            else:
                st.error(f"❌ {result.get('error', 'Unknown error')}")


# ================================================================
# ========================== FOOTER ==============================
# ================================================================
st.markdown(f"""
<div class="footer">
    ⚕️ CHRONOCHECK — CHECK • CARE • CLARITY
    &nbsp;·&nbsp;
    AI-Powered Medical Assistant
    &nbsp;·&nbsp;
    Built for National Hackathon {datetime.now().year}
    <br/><br/>
    <span style="opacity:0.5;">For informational purposes only. Not a substitute for professional medical advice, diagnosis, or treatment.</span>
</div>
""", unsafe_allow_html=True)
