import streamlit as st
import streamlit.components.v1 as components
import plotly.express as px
import folium
from folium.plugins import MarkerCluster
from streamlit_folium import st_folium
import random

st.set_page_config(layout="wide")
st.title("💧 Jordan Wells Dashboard")

# Governorates and sectors
governorates = [
    "Amman", "Irbid", "Zarqa", "Balqa", "Mafraq",
    "Karak", "Tafilah", "Ma’an", "Aqaba", "Ajloun",
    "Jerash", "Madaba"
]

sector_data = {
    "Agriculture": [120, 95, 100, 80, 110, 90, 40, 70, 50, 30, 35, 60],
    "Municipal":   [60, 40, 55, 35, 50, 45, 20, 25, 20, 10, 15, 30],
    "Industrial":  [20, 15, 25, 10, 30, 10, 5, 10, 5, 3, 4, 8],
    "Other":       [10, 5, 8, 6, 7, 5, 3, 4, 2, 2, 1, 3]
}

gov_info = {
    "Amman": {"population": 4200000, "elevation": 900},
    "Irbid": {"population": 1900000, "elevation": 620},
    "Zarqa": {"population": 1500000, "elevation": 605},
    "Balqa": {"population": 500000, "elevation": 790},
    "Mafraq": {"population": 600000, "elevation": 700},
    "Karak": {"population": 320000, "elevation": 930},
    "Tafilah": {"population": 200000, "elevation": 940},
    "Ma’an": {"population": 150000, "elevation": 1000},
    "Aqaba": {"population": 180000, "elevation": 60},
    "Ajloun": {"population": 180000, "elevation": 1100},
    "Jerash": {"population": 250000, "elevation": 650},
    "Madaba": {"population": 270000, "elevation": 740},
}

gov_coords = {
    "Amman": (31.95, 35.91),
    "Irbid": (32.55, 35.85),
    "Zarqa": (32.07, 36.09),
    "Balqa": (32.03, 35.72),
    "Mafraq": (32.34, 36.21),
    "Karak": (31.18, 35.7),
    "Tafilah": (30.83, 35.6),
    "Ma’an": (30.2, 35.73),
    "Aqaba": (29.53, 35.01),
    "Ajloun": (32.33, 35.75),
    "Jerash": (32.28, 35.89),
    "Madaba": (31.72, 35.8)
}

# Initialize stable well points
if "well_points" not in st.session_state:
    st.session_state.well_points = []
    for gov, (lat, lon) in gov_coords.items():
        for _ in range(20):
            offset_lat = lat + random.uniform(-0.03, 0.03)
            offset_lon = lon + random.uniform(-0.03, 0.03)
            st.session_state.well_points.append({
                "gov": gov,
                "lat": offset_lat,
                "lon": offset_lon
            })

# --- Read selected governorate from query param ---
if "gov" in st.query_params:
    selected_gov = st.query_params["gov"]
    if selected_gov not in governorates:
        selected_gov = "Amman"
else:
    selected_gov = "Amman"
    st.query_params["gov"] = "Amman"  # set default if not present

# --- Show Highcharts bar chart ---
st.subheader("📊 Wells per Governorate and Sector")
categories_str = ','.join([f"'{g}'" for g in governorates])
series_str = ','.join([
    f"""{{
        name: '{sector}',
        data: {sector_data[sector]}
    }}""" for sector in sector_data
])

components.html(f"""
<div id="container" style="height: 500px;"></div>
<script src="https://code.highcharts.com/highcharts.js"></script>
<script>
Highcharts.chart('container', {{
    chart: {{
        type: 'column'
    }},
    title: {{
        text: 'Wells in Jordan by Governorate'
    }},
    xAxis: {{
        categories: [{categories_str}],
        crosshair: true
    }},
    yAxis: {{
        min: 0,
        title: {{
            text: 'Number of Wells'
        }}
    }},
    plotOptions: {{
        column: {{
            stacking: 'normal',
            cursor: 'pointer',
            point: {{
                events: {{
                    click: function () {{
                        const gov = this.category;
                        const url = new URL(window.location.href);
                        url.searchParams.set('gov', gov);
                        window.location.replace(url.toString());
                    }}
                }}
            }}
        }}
    }},
    series: [{series_str}]
}});
</script>
""", height=550)

# --- Sidebar content ---
with st.sidebar:
    st.markdown(f"### 📍 {selected_gov}")
    info = gov_info[selected_gov]
    st.markdown(f"- **Population:** {info['population']:,}")
    st.markdown(f"- **Elevation:** {info['elevation']} m")

    pie_data = {
        sector: sector_data[sector][governorates.index(selected_gov)]
        for sector in sector_data
    }
    pie_df = {"Sector": list(pie_data.keys()), "Wells": list(pie_data.values())}
    fig = px.pie(pie_df, names="Sector", values="Wells", title=f"Wells by Sector in {selected_gov}")
    st.plotly_chart(fig, use_container_width=True)

# --- Show folium map ---
st.subheader("🗺️ Cluster Map of Wells in Jordan")
m = folium.Map(location=[31.9, 35.9], zoom_start=7)
marker_cluster = MarkerCluster().add_to(m)
for well in st.session_state.well_points:
    folium.Marker(
        location=[well["lat"], well["lon"]],
        popup=f"{well['gov']} Well",
        icon=folium.Icon(color='blue', icon='tint', prefix='fa')
    ).add_to(marker_cluster)

st_folium(m, width=1200, height=600)
