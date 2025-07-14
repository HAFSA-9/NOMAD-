import streamlit as st
import requests

# Streamlit app config
st.set_page_config(page_title="🔬 NOMAD Material Explorer", layout="centered")
st.title("🔬 NOMAD Material Explorer")
st.markdown("Enter a material formula (e.g., Li, Mg, CrTe2) to fetch metadata from the NOMAD database.")

# Input field
material = st.text_input("Material formula or element symbol:")

# Function to fetch NOMAD entries
def fetch_nomad_data(material):
    url = "https://nomad-lab.eu/prod/rae/api/v1/entries"
    params = {
        "elements": material,
        "per_page": 1  # Limit to 1 result for simplicity
    }
    response = requests.get(url, params=params)
    if response.status_code == 200:
        data = response.json()
        return data.get("data", [None])[0]
    else:
        return None

# Function to fetch archive data (e.g., band gap)
def fetch_nomad_archive(entry_id):
    url = f"https://nomad-lab.eu/prod/rae/api/v1/entries/{entry_id}/archive"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        return {}

# When user inputs a material
if material:
    try:
        entry = fetch_nomad_data(material)

        if entry:
            st.subheader(f"🔍 NOMAD Entry Found: {entry.get('formula', 'Unknown')}")
            st.markdown(f"**Entry ID:** `{entry.get('entry_id')}`")
            st.markdown(f"**DFT Code:** `{entry.get('dft', {}).get('code_name', 'N/A')}`")
            st.markdown(f"**XC Functional:** `{entry.get('dft', {}).get('xc_functional', 'N/A')}`")
            st.markdown(f"**Number of Atoms:** `{entry.get('atoms', {}).get('n_atoms', 'N/A')}`")
            st.markdown(f"**Elements:** `{', '.join(entry.get('atoms', {}).get('elements', []))}`")
            st.markdown(f"**System Type:** `{entry.get('atoms', {}).get('structure_type', 'N/A')}`")
            st.markdown(f"**Upload Time:** `{entry.get('upload_time', 'N/A')}`")

            # Try to get band gap from archive data
            archive = fetch_nomad_archive(entry["entry_id"])
            band_gap = archive.get("results", {}).get("properties", {}).get("electronic", {}).get("band_structure_electronic", {}).get("band_gap", None)
            if band_gap is not None:
                st.markdown(f"**Band Gap:** `{band_gap:.3f} eV`")
            else:
                st.warning("⚠️ Band gap not available in archive.")

        else:
            st.warning("⚠️ No results found for the entered formula.")

    except Exception as e:
        st.error(f"❌ Error: {e}")
