import streamlit as st

# ---------------- ML MODEL ---------------- #

BASE = 0.16

def predict_attrition(emp):
    score = BASE

    if emp["age"] < 28:
        score += 0.18
    elif emp["age"] < 35:
        score += 0.05
    elif emp["age"] > 50:
        score -= 0.12

    if emp["overtime"] == "Yes":
        score += 0.31
    else:
        score -= 0.08

    score += (5 - emp["jobSatisfaction"]) * 0.09
    score += (5 - emp["workLifeBalance"]) * 0.08

    if emp["yearsAtCompany"] < 2:
        score += 0.22
    elif emp["yearsAtCompany"] < 5:
        score += 0.08
    elif emp["yearsAtCompany"] > 10:
        score -= 0.10

    if emp["monthlyIncome"] < 3000:
        score += 0.15
    elif emp["monthlyIncome"] < 6000:
        score += 0.05
    elif emp["monthlyIncome"] > 12000:
        score -= 0.10

    if emp["distanceFromHome"] > 20:
        score += 0.12
    elif emp["distanceFromHome"] > 10:
        score += 0.04

    if emp["jobLevel"] == 1:
        score += 0.14
    elif emp["jobLevel"] == 2:
        score += 0.04
    elif emp["jobLevel"] >= 4:
        score -= 0.10

    prob = max(0.03, min(0.97, score))
    prob = round(prob * 100, 1)

    if prob >= 65:
        risk = "HIGH"
    elif prob >= 38:
        risk = "MEDIUM"
    else:
        risk = "LOW"

    return prob, risk


# ---------------- STREAMLIT UI ---------------- #

st.title("⚡ AttritionIQ - Employee Attrition Predictor")

st.subheader("Employee Profile")

col1, col2 = st.columns(2)

with col1:
    name = st.text_input("Employee Name")
    department = st.selectbox(
        "Department",
        ["Sales", "R&D", "HR", "Finance", "Marketing", "IT"]
    )
    age = st.number_input("Age", 18, 65, 30)
    overtime = st.selectbox("Overtime", ["Yes", "No"])
    income = st.number_input("Monthly Income", 1000, 20000, 5000)
    job_level = st.slider("Job Level", 1, 5, 2)

with col2:
    job_sat = st.slider("Job Satisfaction", 1, 4, 3)
    work_life = st.slider("Work Life Balance", 1, 4, 3)
    years_company = st.number_input("Years at Company", 0, 40, 3)
    total_years = st.number_input("Total Working Years", 0, 40, 6)
    distance = st.number_input("Distance From Home", 0, 50, 10)

emp = {
    "age": age,
    "overtime": overtime,
    "jobSatisfaction": job_sat,
    "workLifeBalance": work_life,
    "yearsAtCompany": years_company,
    "monthlyIncome": income,
    "distanceFromHome": distance,
    "jobLevel": job_level,
}

if st.button("Run Prediction"):

    prob, risk = predict_attrition(emp)

    st.subheader("Prediction Result")

    st.metric("Attrition Probability", f"{prob}%")

    if risk == "HIGH":
        st.error(f"{risk} RISK")
    elif risk == "MEDIUM":
        st.warning(f"{risk} RISK")
    else:
        st.success(f"{risk} RISK")
