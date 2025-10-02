# Task-1import requests
import pandas as pd
import matplotlib.pyplot as plt
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Image
from reportlab.lib.styles import getSampleStyleSheet

# -------------------------
# CONFIG
# -------------------------
API_KEY = "your_api_key_here"   # <-- Replace with your OpenWeather API key
CITY = "London"
OUTPUT_CSV = "weather_forecast.csv"
OUTPUT_PDF = "weather_report.pdf"

# -------------------------
# FETCH WEATHER DATA
# -------------------------
url = f"http://api.openweathermap.org/data/2.5/forecast?q={CITY}&appid={API_KEY}&units=metric"
response = requests.get(url)
data = response.json()

forecast_list = data["list"]

# -------------------------
# CREATE DATAFRAME
# -------------------------
df = pd.DataFrame([{
    "datetime": item["dt_txt"],
    "temperature": item["main"]["temp"],
    "humidity": item["main"]["humidity"],
    "wind_speed": item["wind"]["speed"]
} for item in forecast_list])

df.to_csv(OUTPUT_CSV, index=False)

# -------------------------
# PLOTS
# -------------------------
plt.figure(figsize=(10,5))
plt.plot(df["datetime"], df["temperature"], marker="o")
plt.xticks(rotation=45)
plt.title("Temperature Forecast")
plt.xlabel("Datetime")
plt.ylabel("Temperature (°C)")
plt.tight_layout()
plt.savefig("temp_forecast.png")
plt.close()

plt.figure(figsize=(8,5))
plt.hist(df["temperature"], bins=10, edgecolor="black")
plt.title("Temperature Distribution")
plt.xlabel("Temperature (°C)")
plt.ylabel("Frequency")
plt.savefig("temp_hist.png")
plt.close()

# -------------------------
# PDF REPORT
# -------------------------
doc = SimpleDocTemplate(OUTPUT_PDF)
styles = getSampleStyleSheet()
story = []

story.append(Paragraph(f"Weather Forecast Report for {CITY}", styles['Title']))
story.append(Spacer(1, 12))
story.append(Paragraph(f"Data Source: OpenWeather API", styles['Normal']))
story.append(Spacer(1, 12))

story.append(Paragraph("Temperature Forecast:", styles['Heading2']))
story.append(Image("temp_forecast.png", width=400, height=200))
story.append(Spacer(1, 12))

story.append(Paragraph("Temperature Distribution:", styles['Heading2']))
story.append(Image("temp_hist.png", width=400, height=200))
story.append(Spacer(1, 12))

doc.build(story)

print("✅ Report and CSV generated successfully!")
