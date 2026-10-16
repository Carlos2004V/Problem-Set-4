# Problem-Set-4
import requests
import csv

# Your ipstack API key
# Note: For your final public GitHub repository, it is best practice to load this 
# from an environment variable or keep it in a separate file (e.g., config.py)!
API_KEY = "0ba17b3e2823c7d269acc62440fb42da"

# A curated list of interesting IP addresses to query
targets = {
    "University of Florida": "128.227.19.86",
    "Google Public DNS": "8.8.8.8",
    "Cloudflare DNS": "1.1.1.1",
    "Apple Server": "17.253.144.10"
}

# List to hold processed data for saving later
processed_data = []

print("=" * 60)
print("       IPSTACK API GEOLOCATION & ANALYSIS REPORT")
print("=" * 60 + "\n")

for name, ip_address in targets.items():
    url = f"http://api.ipstack.com/{ip_address}?access_key={API_KEY}"
    
    try:
        response = requests.get(url)
        response.raise_for_status()
        data = response.json()
        
        # Check if the API returned an error
        if "success" in data and data["success"] == False:
            print(f"[{name}] API Error: {data['error']['info']}")
        else:
            # Extract specific parts of the returned data (Beyond just raw print)
            ip = data.get("ip")
            city = data.get("city", "Unknown")
            region = data.get("region_name", "Unknown")
            country = data.get("country_name", "Unknown")
            latitude = data.get("latitude")
            longitude = data.get("longitude")
            
            # Display the processed data nicely in the console
            print(f"Target Name : {name}")
            print(f"IP Address  : {ip}")
            print(f"Location    : {city}, {region}, {country}")
            print(f"Coordinates : Latitude {latitude}, Longitude {longitude}")
            print("-" * 60)
            
            # Append the structured data for our CSV export
            processed_data.append({
                "Target Name": name,
                "IP Address": ip,
                "City": city,
                "Region": region,
                "Country": country,
                "Latitude": latitude,
                "Longitude": longitude
            })
            
    except requests.exceptions.RequestException as e:
        print(f"Connection error for {name}: {e}")

# Save the extracted and processed data to a CSV file (Fulfills the "save" requirement)
csv_filename = "ip_geolocation_report.csv"
csv_columns = ["Target Name", "IP Address", "City", "Region", "Country", "Latitude", "Longitude"]

try:
    with open(csv_filename, mode="w", newline="", encoding="utf-8") as csv_file:
        writer = csv.DictWriter(csv_file, fieldnames=csv_columns)
        writer.writeheader()
        for row in processed_data:
            writer.writerow(row)
    print(f"\n[Success] Processed data successfully saved to {csv_filename}!")
except IOError:
    print("\n[Error] Failed to save CSV file.")
