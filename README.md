import requests
from bs4 import BeautifulSoup
import re
import csv
from datetime import datetime  # Import for date formatting

# URLs for each platform (replace placeholders with actual cities, dates, and class)
platforms = {
    "Google Flights": "https://www.google.com/flights?hl=en#flt=/m/{FROM_CITY}.{TO_CITY}.{DEPARTURE_DATE}*..;c:{CLASS}",
    "Expedia": "https://www.expedia.com/Flights-Search?trip=oneway&leg1=from:{FROM_CITY},to:{TO_CITY},departure:{DEPARTURE_DATE}TANYT&passengers=adults:1&cabinclass={CLASS}",
    "Kayak": "https://www.kayak.com/flights/{FROM_CITY}-{TO_CITY}/{DEPARTURE_DATE}?class={CLASS}"
}

# Mapping user input class to acceptable format for each platform
class_map = {
    "economy": "e",
    "business": "b",
    "first": "f"
}

# CSV file to store results
output_file = "flight_prices_to_africa.csv"

def fetch_content(url):
    """Fetch the content of a web page."""
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return response.text
        else:
            print(f"Failed to retrieve content from {url}")
            return None
    except Exception as e:
        print(f"Error fetching content from {url}: {e}")
        return None

def extract_flight_prices(content):
    """Extract flight prices from the page content."""
    soup = BeautifulSoup(content, 'html.parser')
    prices = []
    
    # Placeholder for price extraction logic
    for price_div in soup.find_all('div', class_=re.compile('price')):
        price_text = price_div.get_text(strip=True)
        prices.append(price_text)
    
    if prices:
        return min(prices, key=lambda p: float(re.sub(r'[^\d.]', '', p)))  # Find the lowest price
    
    return None

def search_flights(platforms, from_city, to_city, departure_date, travel_class):
    """Search for flights to a user-provided African city."""
    results = []

    for platform_name, url_template in platforms.items():
        # Construct the search URL
        search_url = url_template.format(
            FROM_CITY=from_city,
            TO_CITY=to_city,
            DEPARTURE_DATE=departure_date,
            CLASS=class_map[travel_class.lower()]  # Map to class code
        )
        print(f"Searching {platform_name} for flights from {from_city} to {to_city} in {travel_class} class...")
        
        content = fetch_content(search_url)
        if content:
            price = extract_flight_prices(content)
            if price:
                results.append([platform_name, from_city, to_city, travel_class, price])
    
    return results

def save_results_to_csv(results, output_file):
    """Save the search results to a CSV file."""
    with open(output_file, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["Platform", "From City", "To City", "Class", "Price"])
        for result in results:
            writer.writerow(result)

def highlight_cheapest(results):
    """Find and highlight the cheapest flight."""
    if results:
        prices = [(platform, float(re.sub(r'[^\d.]', '', price))) for platform, from_city, to_city, travel_class, price in results]
        cheapest_platform, cheapest_price = min(prices, key=lambda x: x[1])
        
        print("\nFlight Comparison:")
        for platform, price in prices:
            if platform == cheapest_platform:
                print(f"-> {platform}: ${price:.2f} (Cheapest)")
            else:
                print(f"{platform}: ${price:.2f}")

def convert_date_format(date_str):
    """Convert date from DD/MM/YYYY to YYYY-MM-DD for URL usage."""
    return datetime.strptime(date_str, "%d/%m/%Y").strftime("%Y-%m-%d")

# Prompt the user for the departure airport (abbreviation)
from_city = input("Please enter the departure airport (e.g., JFK for New York): ").strip().upper()

# Prompt the user for the destination city
to_city = input("Please enter the African city you want to fly to: ").strip()

# Prompt the user for the departure date (DD/MM/YYYY format)
departure_date_input = input("Please enter the departure date (DD/MM/YYYY): ").strip()

# Convert the date to the format required for the URLs (YYYY-MM-DD)
departure_date = convert_date_format(departure_date_input)

# Prompt the user for the class they want to travel in
travel_class = input("Please enter the class you'd like to travel in (Economy, Business, First): ").strip().lower()

# Execute the search and save results
results = search_flights(platforms, from_city, to_city, departure_date, travel_class)

if results:
    save_results_to_csv(results, output_file)
    highlight_cheapest(results)
else:
    print("No matches found.")
