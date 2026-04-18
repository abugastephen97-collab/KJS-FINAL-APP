"""
KJS Junior School Exam Management System
Streamlit App — Online Version
Compatible data structure with Excel workbook
"""

import streamlit as st
import pandas as pd
import json
import io
from datetime import datetime

# ─── PAGE CONFIG ──────────────────────────────────────────────────────────────
st.set_page_config(
    page_title="KJS Exam System",
    page_icon="🏫",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ─── CUSTOM CSS ───────────────────────────────────────────────────────────────
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
html, body, [class*="css"] { font-family: 'Inter', sans-serif; }
.main-header {
    background: linear-gradient(135deg, #C0392B, #922B21);
    color: white; padding: 20px 30px; border-radius: 12px;
    margin-bottom: 20px; text-align: center;
}
.main-header h1 { margin: 0; font-size: 1.8rem; font-weight: 700; }
.main-header p { margin: 4px 0 0; opacity: 0.85; font-size: 0.9rem; }
.metric-card {
    background: #f8f9fa; border-left: 4px solid #C0392B;
    padding: 12px 16px; border-radius: 8px; margin: 4px 0;
}
.rubric-EE1{background:#D6EAF8;color:#1A5276;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-EE2{background:#D6EAF8;color:#2E86C1;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-ME1{background:#D5F5E3;color:#1E8449;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-ME2{background:#D5F5E3;color:#239B56;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-AE1{background:#FDEBD0;color:#E67E22;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-AE2{background:#FDEBD0;color:#CA6F1E;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-BE1{background:#FADBD8;color:#E74C3C;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.rubric-BE2{background:#FADBD8;color:#C0392B;font-weight:700;padding:3px 10px;border-radius:6px;display:inline-block}
.stButton>button {
    border-radius: 8px; font-weight: 600; transition: all 0.2s;
}
div[data-testid="stExpander"] { border: 1px solid #e0e0e0; border-radius: 10px; }
</style>
""", unsafe_allow_html=True)

# ─── DEFAULT DATA ─────────────────────────────────────────────────────────────
DEFAULT_RUBRICS = [
    {"code":"EE1","desc":"Exceeding Expectation (High)","low":90,"high":100,"pts":8},
    {"code":"EE2","desc":"Exceeding Expectation",       "low":80,"high":89, "pts":7},
    {"code":"ME1","desc":"Meeting Expectation (High)",  "low":70,"high":79, "pts":6},
    {"code":"ME2","desc":"Meeting Expectation",         "low":60,"high":69, "pts":5},
    {"code":"AE1","desc":"Approaching Expectation (High)","low":50,"high":59,"pts":4},
    {"code":"AE2","desc":"Approaching Expectation",     "low":40,"high":49, "pts":3},
    {"code":"BE1","desc":"Below Expectation (High)",    "low":20,"high":39, "pts":2},
    {"code":"BE2","desc":"Below Expectation",           "low":0, "high":19, "pts":1},
]
DEFAULT_SUBJECTS = [
    {"code":"901","name":"English"},{"code":"902","name":"Kiswahili"},
    {"code":"903","name":"Mathematics"},{"code":"907","name":"Social Studies"},
    {"code":"908","name":"CRE"},{"code":"912","name":"Pre-Technical Studies"},
    {"code":"906","name":"Agriculture"},{"code":"905","name":"Integrated Science"},
    {"code":"911","name":"Creative Arts & Sports"},
]
DEFAULT_REMARKS_T = {
    "EE1":"Outstanding performance! You have exceeded all expectations. Keep it up.",
    "EE2":"Excellent work! You consistently exceed expectations.",
    "ME1":"Very good performance. You are meeting expectations at a high level.",
    "ME2":"Good progress. You are meeting expectations well. More revision will help.",
    "AE1":"Fair performance. You are approaching the expected level. More effort needed.",
    "AE2":"Some progress noted but more work is needed to meet expectations.",
    "BE1":"Performance is below expectations. Dedicate more time to studies.",
    "BE2":"Very poor performance. Urgent improvement required.",
}
DEFAULT_REMARKS_P = {
    "EE1":"Exceptional results. You are a role model. We are proud of your achievement.",
    "EE2":"Excellent results. Aim to maintain and surpass this next time.",
    "ME1":"Very satisfactory results. You are on the right track.",
    "ME2":"Satisfactory results. Aim for exceeding expectations next time.",
    "AE1":"Fair results. There is room for improvement. Work harder next term.",
    "AE2":"Below average results. Focus more on studies and attend all lessons.",
    "BE1":"Poor performance. Consult your teachers and parents/guardians.",
    "BE2":"Very unsatisfactory. Immediate intervention required.",
}

# ─── SESSION STATE INIT ───────────────────────────────────────────────────────
def init_state():
    if "settings" not in st.session_state:
        st.session_state.settings = {
            "school_name": "KIABIRAA DEB JUNIOR SCHOOL",
            "address": "P.O BOX 33-40221 OMOGONCHORO",
            "year": "2026",
            "teachers": {7: "", 8: "", 9: ""},
            "term_dates": {
                1: {"open": "10th January 2026", "close": "28th March 2026"},
                2: {"open": "4th May 2026",      "close": "8th August 2026"},
                3: {"open": "7th September 2026","close": "28th November 2026"},
            },
            "rubrics": DEFAULT_RUBRICS,
            "subjects": DEFAULT_SUBJECTS,
            "remarks_teacher": DEFAULT_REMARKS_T,
            "remarks_principal": DEFAULT_REMARKS_P,
        }
    if "students" not in st.session_state:
        st.session_state.students = {7: [], 8: [], 9: []}
    if "marks" not in st.session_state:
        st.session_state.marks = {}  # key: (grade,term,exam,adm_no,subj_code) → mark

init_state()

# ─── HELPERS ──────────────────────────────────────────────────────────────────
def get_rubric(score, rubrics=None):
    if rubrics is None: rubrics = st.session_state.settings["rubrics"]
    if score is None or score == "": return ""
    try:
        s = float(score)
        for r in rubrics:
            if r["low"] <= s <= r["high"]: return r["code"]
    except: pass
    return "?"

def get_points(score, rubrics=None):
    if rubrics is None: rubrics = st.session_state.settings["rubrics"]
    code = get_rubric(score, rubrics)
    for r in rubrics:
        if r["code"] == code: return r["pts"]
    return 0

def rubric_badge(code):
    if not code: return ""
    return f'<span class="rubric-{code}">{code}</span>'

def overall_rubric(total, n_subjects, rubrics=None):
    if rubrics is None: rubrics = st.session_state.settings["rubrics"]
    if not total: return ""
    pct = (total / (n_subjects * 100)) * 100
    return get_rubric(pct, rubrics)

def get_mark(grade, term, exam, adm, subj_code):
    key = (grade, term, exam, adm, subj_code)
    return st.session_state.marks.get(key, "")

def set_mark(grade, term, exam, adm, subj_code, val):
    key = (grade, term, exam, adm, subj_code)
    st.session_state.marks[key] = val

# ─── SIDEBAR ──────────────────────────────────────────────────────────────────
with st.sidebar:
    st.markdown("""
    <div style="background:#C0392B;color:white;padding:16px;border-radius:10px;text-align:center;margin-bottom:16px">
    <b style="font-size:1.1rem">🏫 KJS Exam System</b><br>
    <small>Junior School Management</small>
    </div>
    """, unsafe_allow_html=True)
    
    page = st.radio("📋 Navigation", [
        "🏠 Dashboard",
        "⚙️ Settings",
        "👥 Student Register",
        "📝 Enter Marks",
        "📊 Master Marksheet",
        "📄 Report Cards",
        "🔍 Data Checker",
    ], label_visibility="collapsed")

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: DASHBOARD
# ═══════════════════════════════════════════════════════════════════════════════
if page == "🏠 Dashboard":
    s = st.session_state.settings
    st.markdown(f"""
    <div class="main-header">
    <h1>🏫 {s['school_name']}</h1>
    <p>{s['address']} &nbsp;|&nbsp; Academic Year {s['year']}</p>
    </div>
    """, unsafe_allow_html=True)

    col1, col2, col3 = st.columns(3)
    for ci, grade in zip([col1,col2,col3], [7,8,9]):
        students = st.session_state.students[grade]
        col = {7:"#C0392B",8:"#1A5276",9:"#1E8449"}[grade]
        with ci:
            st.markdown(f"""
            <div style="background:{col};color:white;padding:20px;border-radius:12px;text-align:center">
            <div style="font-size:2rem;font-weight:700">Grade {grade}</div>
            <div style="font-size:1.4rem">{len(students)}</div>
            <div style="opacity:0.8;font-size:0.85rem">students registered</div>
            </div>
            """, unsafe_allow_html=True)

    st.markdown("---")
    st.subheader("📅 Term Dates")
    cols = st.columns(3)
    for ti, term in enumerate([1,2,3]):
        d = s["term_dates"][term]
        with cols[ti]:
            st.markdown(f"""
            <div class="metric-card">
            <b>Term {term}</b><br>
            📂 Opens: {d['open']}<br>
            📁 Closes: {d['close']}
            </div>
            """, unsafe_allow_html=True)

    st.markdown("---")
    st.subheader("📏 Rubric Scale")
    rubric_cols = st.columns(4)
    for ri, rub in enumerate(s["rubrics"]):
        with rubric_cols[ri % 4]:
            st.markdown(f"""<div style="text-align:center;padding:8px;margin:3px;background:#f0f2f6;border-radius:8px">
            {rubric_badge(rub['code'])} <br><small>{rub['low']}–{rub['high']} • {rub['pts']} pts</small>
            </div>""", unsafe_allow_html=True)

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: SETTINGS
# ═══════════════════════════════════════════════════════════════════════════════
elif page == "⚙️ Settings":
    st.title("⚙️ System Settings")
    s = st.session_state.settings

    with st.expander("🏫 School Information", expanded=True):
        c1,c2,c3 = st.columns(3)
        s["school_name"] = c1.text_input("School Name", s["school_name"])
        s["address"]     = c2.text_input("Address", s["address"])
        s["year"]        = c3.text_input("Academic Year", s["year"])
        c4,c5,c6 = st.columns(3)
        s["teachers"][7] = c4.text_input("Grade 7 Class Teacher", s["teachers"][7])
        s["teachers"][8] = c5.text_input("Grade 8 Class Teacher", s["teachers"][8])
        s["teachers"][9] = c6.text_input("Grade 9 Class Teacher", s["teachers"][9])

    with st.expander("📅 Term Dates", expanded=True):
        st.info("Type dates exactly as you want them to appear on report cards (e.g. '10th January 2026')")
        for term in [1,2,3]:
            c1,c2 = st.columns(2)
            s["term_dates"][term]["open"]  = c1.text_input(f"Term {term} Opening Date", s["term_dates"][term]["open"])
            s["term_dates"][term]["close"] = c2.text_input(f"Term {term} Closing Date", s["term_dates"][term]["close"])

    with st.expander("📏 Rubric Settings", expanded=False):
        st.info("Edit mark ranges and points. Changes apply to all marksheets and report cards immediately.")
        df_rub = pd.DataFrame(s["rubrics"])
        edited = st.data_editor(df_rub, num_rows="fixed", use_container_width=True,
                                column_config={
                                    "code":{"disabled":True},
                                    "low": st.column_config.NumberColumn("Mark From",min_value=0,max_value=100),
                                    "high":st.column_config.NumberColumn("Mark To",  min_value=0,max_value=100),
                                    "pts": st.column_config.NumberColumn("Points",   min_value=0,max_value=10),
                                })
        s["rubrics"] = edited.to_dict("records")

    with st.expander("📚 Subjects / Learning Areas", expanded=False):
        st.info("Edit subjects. Changes apply to all marksheets.")
        df_subj = pd.DataFrame(s["subjects"])
        edited_subj = st.data_editor(df_subj, num_rows="dynamic", use_container_width=True)
        s["subjects"] = edited_subj.to_dict("records")

    with st.expander("💬 Auto-Remarks", expanded=False):
        st.info("Edit teacher and principal remarks per rubric. These auto-fill all report cards.")
        for code in [r["code"] for r in s["rubrics"]]:
            st.markdown(f"**{code}**")
            c1,c2 = st.columns(2)
            s["remarks_teacher"][code]   = c1.text_area(f"Teacher Remark ({code})",   s["remarks_teacher"].get(code,""),   height=80, key=f"tr_{code}")
            s["remarks_principal"][code] = c2.text_area(f"Principal Remark ({code})", s["remarks_principal"].get(code,""), height=80, key=f"pr_{code}")

    if st.button("💾 Save Settings", type="primary"):
        st.success("✅ Settings saved! All sheets updated.")

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: STUDENT REGISTER
# ═══════════════════════════════════════════════════════════════════════════════
elif page == "👥 Student Register":
    st.title("👥 Student Register")
    
    grade = st.selectbox("Select Grade", [7,8,9], format_func=lambda x: f"Grade {x}")
    students = st.session_state.students[grade]
    
    # Display current students
    if students:
        df = pd.DataFrame(students)
        # Check for duplicate adm nos
        if df["adm"].duplicated().any():
            st.error("⚠️ Duplicate Admission Numbers detected! Please fix before proceeding.")
        st.dataframe(df.rename(columns={"adm":"Adm No","name":"Student Name","gender":"Gender"}),
                    use_container_width=True, height=min(400, len(students)*36+40))
    else:
        st.info("No students registered yet. Add students below.")

    st.markdown("---")
    st.subheader("➕ Add / Edit Students")
    st.info("Add students one by one OR paste a list. Admission numbers must be unique.")
    
    method = st.radio("Method", ["Add individual student", "Bulk paste (CSV format)"], horizontal=True)
    
    if method == "Add individual student":
        c1,c2,c3,c4 = st.columns([2,3,1.5,1.5])
        new_adm  = c1.text_input("Admission No", key="new_adm")
        new_name = c2.text_input("Student Name (Surname First)", key="new_name")
        new_gen  = c3.selectbox("Gender", ["M","F"], key="new_gen")
        
        if c4.button("Add Student", type="primary"):
            if new_adm and new_name:
                # Check duplicate
                existing_adms = [s["adm"] for s in students]
                if new_adm in existing_adms:
                    st.error(f"❌ Admission No '{new_adm}' already exists!")
                else:
                    st.session_state.students[grade].append({"adm":new_adm,"name":new_name,"gender":new_gen})
                    st.success(f"✅ Added: {new_name}")
                    st.rerun()
            else:
                st.warning("Please fill in both Admission No and Name.")
    
    else:
        st.markdown("Paste CSV format: `AdmNo,Name,Gender` (one student per line)")
        bulk_text = st.text_area("Paste here", height=150, placeholder="119,ROSEMARY AMENYA,F\n120,JOHN KAMAU,M")
        if st.button("Import Students", type="primary"):
            added = 0; errors = []
            existing_adms = [s["adm"] for s in students]
            for line in bulk_text.strip().split("\n"):
                parts = [p.strip() for p in line.split(",")]
                if len(parts) >= 2:
                    adm = parts[0]; name = parts[1]
                    gender = parts[2].upper() if len(parts) > 2 else "M"
                    if adm in existing_adms:
                        errors.append(f"Duplicate: {adm}")
                    else:
                        st.session_state.students[grade].append({"adm":adm,"name":name,"gender":gender})
                        existing_adms.append(adm); added += 1
            if added: st.success(f"✅ Added {added} students.")
            if errors: st.error("Duplicates skipped: " + ", ".join(errors))
            if added: st.rerun()

    # Delete student
    if students:
        st.markdown("---")
        st.subheader("🗑️ Remove Student")
        del_adm = st.selectbox("Select student to remove", 
                               [f"{s['adm']} — {s['name']}" for s in students])
        if st.button("Remove Selected Student", type="secondary"):
            adm_to_del = del_adm.split(" — ")[0]
            st.session_state.students[grade] = [s for s in students if s["adm"] != adm_to_del]
            st.success("Student removed."); st.rerun()

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: ENTER MARKS
# ═══════════════════════════════════════════════════════════════════════════════
elif page == "📝 Enter Marks":
    st.title("📝 Enter Marks")
    
    c1,c2,c3 = st.columns(3)
    grade = c1.selectbox("Grade", [7,8,9], format_func=lambda x: f"Grade {x}", key="em_grade")
    term  = c2.selectbox("Term",  [1,2,3], format_func=lambda x: f"Term {x}",  key="em_term")
    exam  = c3.selectbox("Exam",  ["Opener","Midterm","Endterm"],                key="em_exam")
    
    students = st.session_state.students[grade]
    subjects = st.session_state.settings["subjects"]
    
    if not students:
        st.warning(f"No students registered in Grade {grade}. Please add students first.")
        st.stop()
    
    st.markdown(f"### Grade {grade} | Term {term} | {exam}")
    st.info("Enter marks (0–100). Rubric and points calculate automatically. 🟡 = needs entry")
    
    # Build dataframe of current marks
    rows = []
    for stu in students:
        row = {"Adm No": stu["adm"], "Name": stu["name"], "Gender": stu["gender"]}
        for subj in subjects:
            m = get_mark(grade, term, exam, stu["adm"], subj["code"])
            row[subj["code"]] = m if m != "" else None
        rows.append(row)
    df = pd.DataFrame(rows)
    
    # Editable data entry
    col_config = {
        "Adm No": st.column_config.TextColumn("Adm No", disabled=True, width=80),
        "Name":   st.column_config.TextColumn("Name", disabled=True, width=200),
        "Gender": st.column_config.TextColumn("G", disabled=True, width=40),
    }
    for subj in subjects:
        col_config[subj["code"]] = st.column_config.NumberColumn(
            subj["name"][:14], min_value=0, max_value=100, step=1,
            help=f"{subj['code']} {subj['name']} — max 100"
        )
    
    edited_df = st.data_editor(df, column_config=col_config, use_container_width=True,
                                num_rows="fixed", key=f"marks_editor_{grade}_{term}_{exam}")
    
    if st.button("💾 Save All Marks", type="primary", use_container_width=True):
        saved = 0; errors = []
        for _, row in edited_df.iterrows():
            adm = str(row["Adm No"])
            for subj in subjects:
                val = row.get(subj["code"])
                if pd.notna(val) and val != "":
                    try:
                        v = float(val)
                        if v > 100:
                            errors.append(f"{row['Name']}/{subj['name']}: {v} > 100")
                            continue
                        set_mark(grade, term, exam, adm, subj["code"], int(v))
                        saved += 1
                    except: pass
                else:
                    set_mark(grade, term, exam, adm, subj["code"], "")
        if errors:
            st.error("❌ Marks exceeding 100 rejected:\n" + "\n".join(errors))
        st.success(f"✅ {saved} marks saved!")

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: MASTER MARKSHEET
# ═══════════════════════════════════════════════════════════════════════════════
elif page == "📊 Master Marksheet":
    st.title("📊 Master Marksheet")
    
    c1,c2,c3 = st.columns(3)
    grade = c1.selectbox("Grade", [7,8,9], format_func=lambda x: f"Grade {x}", key="ms_grade")
    term  = c2.selectbox("Term",  [1,2,3], format_func=lambda x: f"Term {x}",  key="ms_term")
    exam  = c3.selectbox("Exam",  ["Opener","Midterm","Endterm"],                key="ms_exam")
    
    students = st.session_state.students[grade]
    subjects = st.session_state.settings["subjects"]
    rubrics  = st.session_state.settings["rubrics"]
    
    if not students:
        st.warning(f"No students in Grade {grade}."); st.stop()
    
    teacher = st.session_state.settings["teachers"].get(grade, "")
    st.markdown(f"**Class Teacher:** {teacher or '(not set)'}  &nbsp;|&nbsp; **Grade {grade} | Term {term} | {exam}**")
    
    # Build marksheet
    rows = []
    for si, stu in enumerate(students):
        row = {"#":si+1, "Adm No":stu["adm"], "Name":stu["name"], "Gender":stu["gender"]}
        total_marks = 0; total_pts = 0; has_data = False
        for subj in subjects:
            m = get_mark(grade, term, exam, stu["adm"], subj["code"])
            row[f"{subj['code']}_mk"] = m
            row[f"{subj['code']}_rb"] = get_rubric(m, rubrics) if m != "" else ""
            if m != "":
                total_marks += int(m); total_pts += get_points(m, rubrics); has_data=True
        row["Total"] = total_marks if has_data else ""
        row["Rubric"] = overall_rubric(total_marks if has_data else None, len(subjects), rubrics)
        row["Points"] = total_pts if has_data else ""
        rows.append(row)
    
    df = pd.DataFrame(rows)
    
    # Assign rank
    if "Total" in df.columns:
        df_with_total = df[df["Total"] != ""].copy()
        df_with_total["Rank"] = df_with_total["Total"].rank(ascending=False, method="min").astype(int)
        df["Rank"] = df_with_total["Rank"]
        df["Rank"] = df["Rank"].fillna("").astype(str).replace("nan","").replace(".0","")
        # Clean rank display
        df["Rank"] = df["Rank"].apply(lambda x: str(int(float(x))) if x not in ["","nan"] else "")
    
    # Display
    display_cols = ["#","Adm No","Name","Gender"]
    for subj in subjects:
        display_cols += [f"{subj['code']}_mk", f"{subj['code']}_rb"]
    display_cols += ["Total","Rubric","Points","Rank"]
    
    col_rename = {"#":"#","Adm No":"Adm No","Name":"Student Name","Gender":"G",
                  "Total":"Total","Rubric":"Rubric","Points":"Pts","Rank":"Rank"}
    for subj in subjects:
        col_rename[f"{subj['code']}_mk"] = f"{subj['name'][:10]}\nMks"
        col_rename[f"{subj['code']}_rb"] = f"{subj['code']}\nRub"
    
    df_display = df[display_cols].rename(columns=col_rename)
    st.dataframe(df_display, use_container_width=True, height=min(800, len(students)*36+80))
    
    # Statistics
    st.markdown("---")
    col1, col2, col3, col4 = st.columns(4)
    totals_with_data = [r["Total"] for r in rows if r["Total"] != ""]
    if totals_with_data:
        col1.metric("Class Average", f"{sum(totals_with_data)/len(totals_with_data):.1f}")
        col2.metric("Highest Score", max(totals_with_data))
        col3.metric("Lowest Score",  min(totals_with_data))
    col4.metric("Students", len(students))
    
    # Gender analysis
    m_totals = [r["Total"] for r in rows if r["Total"] != "" and st.session_state.students[grade][[s["adm"] for s in st.session_state.students[grade]].index(r["Adm No"])]["gender"] == "M"] if students else []
    f_totals = [r["Total"] for r in rows if r["Total"] != "" and st.session_state.students[grade][[s["adm"] for s in st.session_state.students[grade]].index(r["Adm No"])]["gender"] == "F"] if students else []
    
    gcols = st.columns(2)
    if m_totals: gcols[0].metric("Boys Average", f"{sum(m_totals)/len(m_totals):.1f}")
    if f_totals: gcols[1].metric("Girls Average", f"{sum(f_totals)/len(f_totals):.1f}")
    
    # Export to Excel
    st.markdown("---")
    if st.button("📥 Export Marksheet to Excel", type="secondary"):
        output = io.BytesIO()
        with pd.ExcelWriter(output, engine="openpyxl") as writer:
            df_display.to_excel(writer, index=False, sheet_name=f"G{grade}T{term}_{exam}")
        st.download_button("⬇️ Download Excel", output.getvalue(),
                           f"KJS_G{grade}T{term}_{exam}_Marksheet.xlsx",
                           "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: REPORT CARDS
# ═══════════════════════════════════════════════════════════════════════════════
elif page == "📄 Report Cards":
    st.title("📄 Report Cards")
    
    c1,c2,c3 = st.columns(3)
    grade = c1.selectbox("Grade", [7,8,9], format_func=lambda x: f"Grade {x}", key="rc_grade")
    term  = c2.selectbox("Term",  [1,2,3], format_func=lambda x: f"Term {x}",  key="rc_term")
    exam  = c3.selectbox("Exam",  ["Opener","Midterm","Endterm"],                key="rc_exam")
    
    students = st.session_state.students[grade]
    subjects = st.session_state.settings["subjects"]
    rubrics  = st.session_state.settings["rubrics"]
    s        = st.session_state.settings
    
    if not students:
        st.warning(f"No students in Grade {grade}."); st.stop()
    
    student_opts = [f"{st['adm']} — {st['name']}" for st in students]
    sel = st.selectbox("Select Student", ["All Students (for PDF export)"] + student_opts)
    
    def render_rc(stu):
        adm = stu["adm"]; name = stu["name"]; gender = stu["gender"]
        subj_rows = []
        total_marks = 0; total_pts = 0; has_data = False
        for subj in subjects:
            m = get_mark(grade, term, exam, adm, subj["code"])
            rb = get_rubric(m, rubrics) if m != "" else ""
            pts = get_points(m, rubrics) if m != "" else ""
            subj_rows.append({"CODE":subj["code"],"LEARNING AREA":subj["name"],
                               "MARKS":m,"PERFORMANCE LEVEL":rb,"POINTS":pts})
            if m != "": total_marks += int(m); total_pts += pts; has_data=True
        
        ov_rb = overall_rubric(total_marks if has_data else None, len(subjects), rubrics)
        
        # Rank
        all_totals = {}
        for st2 in students:
            t2 = sum(int(get_mark(grade,term,exam,st2["adm"],sj["code"])) for sj in subjects 
                     if get_mark(grade,term,exam,st2["adm"],sj["code"]) != "")
            if any(get_mark(grade,term,exam,st2["adm"],sj["code"]) != "" for sj in subjects):
                all_totals[st2["adm"]] = t2
        sorted_totals = sorted(all_totals.values(), reverse=True)
        rank = sorted_totals.index(total_marks)+1 if has_data and total_marks in sorted_totals else "—"
        
        teacher_remark   = s["remarks_teacher"].get(ov_rb,"")
        principal_remark = s["remarks_principal"].get(ov_rb,"")
        td = s["term_dates"][term]
        
        html = f"""
        <div style="border:2px solid #333;padding:24px 28px;max-width:620px;margin:0 auto 32px;font-family:Arial,sans-serif;background:white;font-size:13px">
          <div style="text-align:center;margin-bottom:12px">
            <div style="font-size:22px;font-weight:700;color:#C0392B;letter-spacing:1px">{s['school_name']}</div>
            <div style="font-size:12px;color:#333">{s['address']}</div>
            <div style="font-size:16px;font-weight:700;margin-top:8px">PROGRESSIVE REPORT</div>
            <div style="font-size:15px;font-weight:700">{exam.upper()}</div>
          </div>
          <table style="width:100%;border-collapse:collapse;margin-bottom:10px">
            <tr>
              <td style="padding:4px 6px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700;width:80px">ADMNO:</td>
              <td style="padding:4px 6px;border:1px solid #333;font-weight:700">{adm}</td>
              <td style="padding:4px 6px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700;width:80px">GRADE:</td>
              <td style="padding:4px 6px;border:1px solid #333;font-weight:700">{grade}</td>
            </tr>
            <tr>
              <td style="padding:4px 6px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700">NAME:</td>
              <td style="padding:4px 6px;border:1px solid #333;font-weight:700">{name}</td>
              <td style="padding:4px 6px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700">YEAR:</td>
              <td style="padding:4px 6px;border:1px solid #333;font-weight:700">{s['year']}</td>
            </tr>
            <tr>
              <td style="padding:4px 6px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700">GENDER:</td>
              <td style="padding:4px 6px;border:1px solid #333;font-weight:700">{gender}</td>
              <td style="padding:4px 6px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700">TERM:</td>
              <td style="padding:4px 6px;border:1px solid #333;font-weight:700">{term}</td>
            </tr>
          </table>
          <table style="width:100%;border-collapse:collapse;margin-bottom:10px">
            <tr style="background:#1A1A2E;color:white">
              <th style="padding:5px 6px;border:1px solid #333;text-align:center;width:50px">CODE</th>
              <th style="padding:5px 6px;border:1px solid #333;text-align:left">LEARNING AREA</th>
              <th style="padding:5px 6px;border:1px solid #333;text-align:center;width:55px">MARKS</th>
              <th style="padding:5px 6px;border:1px solid #333;text-align:center;width:110px">PERFORMANCE LEVEL</th>
              <th style="padding:5px 6px;border:1px solid #333;text-align:center;width:55px">POINTS</th>
            </tr>
        """
        for i,sr in enumerate(subj_rows):
            bg = "#F2F3F4" if i%2==0 else "white"
            html += f"""
            <tr style="background:{bg}">
              <td style="padding:4px 6px;border:1px solid #ddd;text-align:center">{sr['CODE']}</td>
              <td style="padding:4px 6px;border:1px solid #ddd">{sr['LEARNING AREA']}</td>
              <td style="padding:4px 6px;border:1px solid #ddd;text-align:center;font-weight:700">{sr['MARKS']}</td>
              <td style="padding:4px 6px;border:1px solid #ddd;text-align:center;font-weight:700;color:{'#1A5276' if 'EE' in str(sr['PERFORMANCE LEVEL']) else '#1E8449' if 'ME' in str(sr['PERFORMANCE LEVEL']) else '#E67E22' if 'AE' in str(sr['PERFORMANCE LEVEL']) else '#C0392B'}">{sr['PERFORMANCE LEVEL']}</td>
              <td style="padding:4px 6px;border:1px solid #ddd;text-align:center;font-weight:700">{sr['POINTS']}</td>
            </tr>"""
        
        html += f"""
          </table>
          <table style="width:100%;border-collapse:collapse;margin-bottom:10px">
            <tr style="background:#F2F3F4"><td style="padding:5px 8px;border:1px solid #ddd;width:160px">Total marks:</td><td style="padding:5px 8px;border:1px solid #ddd;font-weight:700">{total_marks if has_data else '—'}</td></tr>
            <tr><td style="padding:5px 8px;border:1px solid #ddd">Total points:</td><td style="padding:5px 8px;border:1px solid #ddd;font-weight:700">{total_pts if has_data else '—'}</td></tr>
            <tr style="background:#F2F3F4"><td style="padding:5px 8px;border:1px solid #ddd">Performance Level:</td><td style="padding:5px 8px;border:1px solid #ddd;font-weight:700;font-size:15px">{ov_rb}</td></tr>
            <tr><td style="padding:5px 8px;border:1px solid #ddd">Position / Rank:</td><td style="padding:5px 8px;border:1px solid #ddd;font-weight:700">{rank}</td></tr>
          </table>
          <div style="border:1px solid #ddd;padding:10px 12px;margin-bottom:8px">
            <div style="font-weight:700;margin-bottom:4px">Class Teacher's Remarks</div>
            <div style="font-style:italic;color:#333">{teacher_remark}</div>
            <div style="font-size:11px;color:#777;margin-top:8px">Signature:..................................   Date:…………………………………………</div>
          </div>
          <div style="border:1px solid #ddd;padding:10px 12px;margin-bottom:8px">
            <div style="font-weight:700;margin-bottom:4px">Principal's Remarks</div>
            <div style="font-style:italic;color:#333">{principal_remark}</div>
            <div style="font-size:11px;color:#777;margin-top:8px">Signature:..................................   Date:…………………………………………</div>
            <div style="font-size:11px;color:#777;margin-top:4px">Parents/Guardian Signature:............   Date:…………………………………………</div>
          </div>
          <table style="width:100%;border-collapse:collapse">
            <tr>
              <td style="padding:6px 10px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700;width:120px">Closing Date:</td>
              <td style="padding:6px 10px;border:1px solid #333;font-weight:700">{td['close']}</td>
              <td style="padding:6px 10px;border:1px solid #333;background:#1A1A2E;color:white;font-weight:700;width:120px">Opening Date:</td>
              <td style="padding:6px 10px;border:1px solid #333;font-weight:700">{td['open']}</td>
            </tr>
          </table>
          <div style="font-size:10px;color:#aaa;margin-top:10px">Generated: {datetime.now().strftime('%d/%m/%Y %H:%M')}</div>
        </div>
        """
        return html
    
    if sel == "All Students (for PDF export)":
        st.info("💡 To print/save all report cards as PDF: use your browser's Print function (Ctrl+P) and select 'Save as PDF'. All cards will be on separate pages.")
        show_all = st.button("🖨️ Render All Report Cards", type="primary")
        if show_all:
            for stu in students:
                st.markdown(render_rc(stu), unsafe_allow_html=True)
    else:
        adm_sel = sel.split(" — ")[0]
        stu_sel = next((s for s in students if s["adm"]==adm_sel), None)
        if stu_sel:
            st.markdown(render_rc(stu_sel), unsafe_allow_html=True)

# ═══════════════════════════════════════════════════════════════════════════════
# PAGE: DATA CHECKER
# ═══════════════════════════════════════════════════════════════════════════════
elif page == "🔍 Data Checker":
    st.title("🔍 Data Checker — Gatekeeper")
    st.info("Runs validation across all grades, terms and exams.")
    
    if st.button("▶️ Run Full Data Check", type="primary", use_container_width=True):
        issues = []
        
        # 1. Duplicate Adm Numbers per grade
        for grade in [7,8,9]:
            adms = [s["adm"] for s in st.session_state.students[grade]]
            seen = {}
            for adm in adms:
                if adm in seen: issues.append({"Type":"⚠️ Duplicate Adm No","Grade":grade,"Detail":f"Adm '{adm}' appears more than once"})
                seen[adm]=True
        
        # 2. Marks >100 or negative
        for grade in [7,8,9]:
            for term in [1,2,3]:
                for exam in ["Opener","Midterm","Endterm"]:
                    for stu in st.session_state.students[grade]:
                        for subj in st.session_state.settings["subjects"]:
                            m = get_mark(grade,term,exam,stu["adm"],subj["code"])
                            if m != "":
                                try:
                                    v=float(m)
                                    if v>100: issues.append({"Type":"❌ Mark > 100","Grade":grade,"Detail":f"T{term} {exam}: {stu['name']} / {subj['name']} = {v}"})
                                    if v<0:   issues.append({"Type":"❌ Negative Mark","Grade":grade,"Detail":f"T{term} {exam}: {stu['name']} / {subj['name']} = {v}"})
                                except: issues.append({"Type":"❌ Invalid Mark","Grade":grade,"Detail":f"T{term} {exam}: {stu['name']} / {subj['name']} = '{m}'"})
        
        # 3. Missing student names or adm
        for grade in [7,8,9]:
            for i,stu in enumerate(st.session_state.students[grade]):
                if not stu.get("name","").strip(): issues.append({"Type":"⚠️ Missing Name","Grade":grade,"Detail":f"Row {i+1}: Adm {stu['adm']} has no name"})
                if not stu.get("adm","").strip():  issues.append({"Type":"⚠️ Missing Adm No","Grade":grade,"Detail":f"Row {i+1} has no Admission Number"})
        
        if not issues:
            st.success("✅ ALL CHECKS PASSED! No issues found. Your data is clean and ready for report generation.")
        else:
            st.error(f"❌ Found {len(issues)} issue(s):")
            df_issues=pd.DataFrame(issues)
            st.dataframe(df_issues,use_container_width=True)
        
        # Summary stats
        st.markdown("---")
        st.subheader("📊 Data Summary")
        for grade in [7,8,9]:
            n = len(st.session_state.students[grade])
            filled = sum(1 for t in [1,2,3] for e in ["Opener","Midterm","Endterm"] 
                        for stu in st.session_state.students[grade]
                        for subj in st.session_state.settings["subjects"]
                        if get_mark(grade,t,e,stu["adm"],subj["code"]) != "")
            st.write(f"**Grade {grade}:** {n} students, {filled} marks entered")

# ─── FOOTER ───────────────────────────────────────────────────────────────────
st.markdown("---")
st.markdown("""
<div style="text-align:center;color:#aaa;font-size:11px">
KJS Exam Management System &nbsp;|&nbsp; Kiabiraa DEB Junior School &nbsp;|&nbsp; 
Built for Speed & Accuracy
</div>
""", unsafe_allow_html=True)
