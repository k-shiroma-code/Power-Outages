# ⚡ Weather & Energy Dashboard

A full-stack web application featuring **global weather data** and **California grid load forecasting** using machine learning.

![Dashboard Preview](https://img.shields.io/badge/Status-Active-brightgreen) ![React](https://img.shields.io/badge/React-18-blue) ![Astro](https://img.shields.io/badge/Astro-4-purple) ![Python](https://img.shields.io/badge/Python-3.10-yellow)

---

## 🌟 Features

### 🌍 Global Weather Dashboard
- Real-time weather for any city worldwide
- 7-day forecasts with temperature trends
- Toggle between °C and °F
- Interactive charts and data tables

### ⚡ Grid Load Forecasting
- 14-day electricity demand predictions
- **4 California service areas:** SCE, PG&E, SDG&E, VEA
- **3 ML models:** Ridge Regression, Random Forest, Gradient Boosting
- Cross-validation results and feature importance analysis

---

## 📊 Model Performance

| Model | MAE (MW) | RMSE (MW) | MAPE |
|-------|----------|-----------|------|
| Ridge + Weather | 840.0 | 1,023.0 | 3.41% |
| Random Forest | 576.9 | 751.1 | 2.29% |
| **Gradient Boosting** | **573.2** | **724.8** | **2.26%** |

*Validated using 5-fold expanding window cross-validation on 315,648 observations*

---

## 🛠️ Tech Stack

**Frontend:**
- React 18
- Astro 4
- Recharts (data visualization)

**Backend:**
- FastAPI
- Python 3.10
- scikit-learn

**APIs:**
- OpenWeather API
- Open-Meteo API

---

## 📁 Project Structure

```
├── src/
│   ├── components/
│   │   ├── Header.jsx         # Navigation bar
│   │   ├── Forecast.jsx       # Grid load forecasting
│   │   └── Table_Charts.jsx   # Weather dashboard
│   └── pages/
│       ├── index.astro        # Home page
│       ├── Forecast.astro     # Forecast page
│       └── table_charts.astro # Weather page
├── app.py                     # FastAPI backend
├── package.json
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- Python 3.10+
- OpenWeather API key

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/k-shiroma-code/Weather-API-Project.git
   cd Weather-API-Project
   ```

2. **Install frontend dependencies**
   ```bash
   npm install
   ```

3. **Install backend dependencies**
   ```bash
   pip install fastapi uvicorn requests python-dotenv
   ```

4. **Set up environment variables**
   ```bash
   echo "OPENWEATHER_API_KEY=your_api_key_here" > .env
   ```

5. **Run the backend**
   ```bash
   uvicorn app:app --reload --port 8000
   ```

6. **Run the frontend** (in a new terminal)
   ```bash
   npm run dev
   ```

7. **Open** http://localhost:4321

---

## 📖 Glossary

| Term | Definition |
|------|------------|
| **MW (Megawatt)** | Unit of power. 1 MW powers ~750-1,000 homes |
| **MAE** | Mean Absolute Error - average prediction error in MW |
| **MAPE** | Mean Absolute Percentage Error - error as a percentage |
| **Grid Load** | Total electricity demand at any moment |
| **Gradient Boosting** | ML technique combining multiple models iteratively |
| **Cross-Validation** | Method to test model performance on unseen data |

---

## 🏢 Service Areas

| Area | Region | Population | Avg Load |
|------|--------|------------|----------|
| **SCE** | Southern California | 15 million | ~24,000 MW |
| **PG&E** | Northern & Central CA | 16 million | ~18,000 MW |
| **SDG&E** | San Diego Area | 3.7 million | ~4,500 MW |
| **VEA** | Nevada/CA Border | 45,000 | ~200 MW |

---

## 📈 Data Source

- **Grid Data:** [Kaggle - California ISO](https://www.kaggle.com/) (315,648 observations, 2019-2021)
- **Weather Data:** OpenWeather API, Open-Meteo API

---

## 🤝 Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request.

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 👤 Author

**K. Shiroma**
- GitHub: [@k-shiroma-code](https://github.com/k-shiroma-code)

---

<p align="center">
  Built with ☕ and ⚡
</p>
