# Dashboard Analisis Kualitas Udara

Dashboard ini berfungsi untuk menganalisis kualitas udara di wilayah Guanyuan, Beijing. Data yang digunakan mencakup konsentrasi PM2.5, PM10, suhu, tekanan, hujan, dan kecepatan angin.

# I ran this code in Google Colab
%%writefile dashboard_air_quality.py

import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.preprocessing import StandardScaler
import folium
from streamlit_folium import folium_static
from datetime import datetime

st.set_page_config(
    page_title="Dashboard Analisis Kualitas Udara",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Load dan bersihkan data
data = pd.read_csv('air_quality_data/PRSA_Data_20130301-20170228/PRSA_Data_Guanyuan_20130301-20170228.csv')
data_clean = data.fillna(data.select_dtypes(include=np.number).median())
data_clean['wd'] = data['wd'].fillna(data['wd'].mode()[0])
data_clean['station'] = data['station'].fillna(data['station'].mode()[0])
data_clean['date'] = pd.to_datetime(data_clean[['year', 'month', 'day', 'hour']])

# Sidebar - Filter Data
st.sidebar.header("Filter Data")
filter_year = st.sidebar.slider('Pilih Tahun', 2013, 2017, (2013, 2017))
filter_month = st.sidebar.selectbox('Pilih Bulan', range(1, 13))

data_clean['year'] = data_clean['date'].dt.year
data_clean['month'] = data_clean['date'].dt.month
filtered_data = data_clean[(data_clean['year'] >= filter_year[0]) &
                           (data_clean['year'] <= filter_year[1]) &
                           (data_clean['month'] == filter_month)]

# Ringkasan Data
st.title("📊 Dashboard Analisis Kualitas Udara di Guanyuan")
st.markdown("Dashboard ini bertujuan untuk menganalisis kualitas udara di Guanyuan berdasarkan data PM2.5 dan variabel lainnya.")
st.markdown("## Ringkasan Data")
st.write(filtered_data.describe())

# Visualisasi
st.markdown("## Distribusi PM2.5")
fig, ax = plt.subplots()
sns.histplot(filtered_data['PM2.5'], bins=30, kde=True, color="skyblue", ax=ax)
ax.set_title('Distribusi PM2.5 di Guanyuan')
st.pyplot(fig)

# Hubungan PM10 dan PM2.5
st.markdown("## Hubungan antara PM10 dan PM2.5")
fig, ax = plt.subplots()
sns.scatterplot(x='PM10', y='PM2.5', data=filtered_data, color="purple", alpha=0.6, ax=ax)
ax.set_title('Hubungan antara PM10 dan PM2.5')
st.pyplot(fig)

# Rata-rata PM2.5 per Bulan
st.markdown("## Rata-rata PM2.5 per Bulan")
pm25_monthly_avg = data_clean.set_index('date').resample('M')['PM2.5'].mean()
fig, ax = plt.subplots()
pm25_monthly_avg.plot(ax=ax, color="orange", linewidth=2)
ax.set_title('Rata-rata PM2.5 per Bulan')
ax.set_ylabel("PM2.5 (µg/m³)")
st.pyplot(fig)

# Peta Lokasi
st.markdown("## Peta Lokasi Guanyuan")
guanyuan_location = [39.929, 116.365]  # Koordinat Guanyuan, Beijing
m = folium.Map(location=guanyuan_location, zoom_start=12)
folium.Marker(
    location=guanyuan_location,
    popup="Lokasi Pengukuran Guanyuan",
    icon=folium.Icon(color='green')
).add_to(m)
folium_static(m)

# Footer
st.markdown("---")
st.markdown("### Catatan")
st.markdown("Data ini diambil dari dataset PRSA dan digunakan untuk analisis kualitas udara.")
st.markdown("Dikembangkan dengan ❤️ menggunakan Streamlit.")

# Then I added this code to open my Dashboard using Streamlit
!streamlit run dashboard_air_quality.py & npx localtunnel --port 8501

# Before running the code, I installed this in Google Colab
!pip install streamlit_folium
!pip install pyngrok

# Note
I am just learning and entering the world of data analysis, I don't understand why when I work on my project in Google Colab (using the available templates) if I want to see my project in Google Colab all the zip or the file is lost, in the end if I want to run it again I have to import the zip Air Quality Dataset into Google Colab and change it to file form.
From the bottom of my heart, I apologize profusely
