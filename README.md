# Problem-Set-4
import os
import sys
import requests
import csv

# Read API key from environment variable (do NOT hardcode in source)
API_KEY = os.getenv("IPSTACK_API_KEY")
if not API_KEY:
    print("Error: set the IPSTACK_API_KEY environment variable (do NOT commit your API key).")
    sys.exit(1)

targets = {
    "University of Florida": "128.227.19.86",
    "Google Public DNS": "8.8.8.8",
    "Cloudflare DNS": "1.1.1.1",
    "Apple Server": "17.253.144.10"
}

processed_data = []

print("=" * 60)
print("       IPSTACK API GEOLOCATION & ANALYSIS REPORT")
print("=" * 60 + "\n")

for name, ip_address in targets.items():
    url = f"https://api.ipstack.com/{ip_address}?access_key={API_KEY}"
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        data = response.json()
    except requests.exceptions.RequestException as e:
        print(f"Connection error for {name} ({ip_address}): {e}")
        continue
    except ValueError:
        print(f"Invalid JSON returned for {name} ({ip_address}).")
        continue

    # Handle API-level error (ipstack returns {"success": false, "error": {...}} on failure)
    if data.get("success") is False:
        info = data.get("error", {}).get("info", "Unknown API error")
        print(f"[{name}] API Error: {info}")
        continue

    ip = data.get("ip", ip_address)
    city = data.get("city") or "Unknown"
    region = data.get("region_name") or "Unknown"
    country = data.get("country_name") or "Unknown"
    latitude = data.get("latitude")
    longitude = data.get("longitude")

    print(f"Target Name : {name}")
    print(f"IP Address  : {ip}")
    print(f"Location    : {city}, {region}, {country}")
    print(f"Coordinates : Latitude {latitude}, Longitude {longitude}")
    print("-" * 60)

    processed_data.append({
        "Target Name": name,
        "IP Address": ip,
        "City": city,
        "Region": region,
        "Country": country,
        "Latitude": latitude,
        "Longitude": longitude
    })

# Write CSV once after processing all targets
csv_filename = "ip_geolocation_report.csv"
csv_columns = ["Target Name", "IP Address", "City", "Region", "Country", "Latitude", "Longitude"]

try:
    with open(csv_filename, mode="w", newline="", encoding="utf-8") as csv_file:
        writer = csv.DictWriter(csv_file, fieldnames=csv_columns)
        writer.writeheader()
        writer.writerows(processed_data)
    print(f"\n[Success] Processed data successfully saved to {csv_filename}!")
except IOError as e:
    print(f"\n[Error] Failed to save CSV file: {e}")

**University of Florida IP Address : 128.227.19.86 Location : Gainesville, Florida, United States Coordinates : Latitude 29.6499, Longitude -82.3455**

**Google Public DNS IP Address : 8.8.8.8 Location : Mountain View, California, United States Coordinates : Latitude 37.3860, Longitude -122.0840**

**Cloudflare DNS IP Address : 1.1.1.1 Location : Sydney, New South Wales, Australia Coordinates : Latitude -33.4940, Longitude 143.2104**

**Apple Server IP Address : 17.253.144.10 Location : Cupertino, California, United States Coordinates : Latitude 37.3220, Longitude -122.0322**
