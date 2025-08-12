# To develop a dashboard for a company where staff must visit anytime, they need to buy one way ticket from Lagos to London.  TO ensure cheapert ticket is available always

#Step 1: Define Problem and Objective
#Problem ID: Cheapest One-Way Ticket Finder (Lagos to London)
#Goal: Advise Panama Equity Strategy Plc on the best/cheapest airline ticket by simultaneously comparing current prices from#least 5 airlines and recommending the lowest.
#Step 2: Explore Data Sources (API Endpoints)
#Find airline APIs: Research official developer documentation or travel aggregator APIs (e.g., Skyscanner, Amadeus, Kiwi, British #Airways, Qatar Airways, #Air France)
#API requirements: Most airline APIs require authentication (API key/token), valid endpoints, and return results in JSON.
#Step 3: Distributed Python Implementation (Ray)
#Here is code to fetch and visualize the cheapest tickets using Ray for parallel/distributed computing in Python:

import ray
import requests
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Ray for distributed execution
ray.init()  # for local machine; use ray.init(address="auto") for cluster

# Example API endpoints (replace with real endpoints and authentication details)
API_ENDPOINTS = {
    "British Airways": "https://api.britishairways.com/flights/los-lon/oneway",
    "Virgin Atlantic": "https://api.virginatlantic.com/flights/los-lon/oneway",
    "Qatar Airways": "https://api.qatarairways.com/flights/los-lon/oneway",
    "Air France": "https://api.airfrance.com/flights/los-lon/oneway",
    "RwandAir": "https://api.rwandair.com/flights/los-lon/oneway"
}

@ray.remote
def fetch_ticket(airline, url):
    try:
        # Use headers/auth as required by each API
        resp = requests.get(url, timeout=10)
        data = resp.json()  # {'price': ..., 'details': ...}
        return {"airline": airline, "price": data["price"], "details": data["details"]}
    except Exception as e:
        return {"airline": airline, "error": str(e)}

# Fetch tickets in parallel across endpoints
results = ray.get([
    fetch_ticket.remote(airline, url) for airline, url in API_ENDPOINTS.items()
])

# Filter successful results
valid_tickets = [r for r in results if "price" in r]
if not valid_tickets:
    print("No valid ticket data found. Check API connectivity and authentication.")
else:
    # Sort tickets by price
    valid_tickets.sort(key=lambda x: x['price'])
    best_ticket = valid_tickets[0]

    # Visualization (using matplotlib)
    df = pd.DataFrame(valid_tickets)
    plt.bar(df['airline'], df['price'], color='skyblue')
    plt.title("One-Way Ticket Prices to London (Lagos-London)")
    plt.ylabel("Price (USD)")
    plt.xlabel("Airline")
    plt.tight_layout()
    plt.show()

    # Print and recommend
    print("All offers:")
    for ticket in valid_tickets:
        print(f"{ticket['airline']}: ${ticket['price']} - {ticket['details']}")
    print(f"\nRecommended: {best_ticket['airline']} - ${best_ticket['price']} ({best_ticket['details']})")

#####################################################################################################Notes:
#Replace endpoints above with real API URLs and include required authentication (API key, headers, etc.) 
# Most commercial APIs####will not work without this.
#Visualization: You can use Dash or Streamlit to upgrade from matplotlib to an interactive web dashboard.
###############################################################################################################
