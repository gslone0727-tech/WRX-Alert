import requests
from bs4 import BeautifulSoup
import smtplib
from email.mime.text import MIMEText
from datetime import datetime
import json
import os

# --- CONFIG ---
SEARCH_ZIP = "33442"
RADIUS = 300
MAX_MILES = 25000
YEARS = [2020, 2021]
RECIPIENT_EMAIL = "gslone0727@gmail.com"
SENDER_EMAIL = "gslone0727@gmail.com"
SENDER_PASS = os.getenv("GMAIL_APP_PASS")  # Store in environment variable

DATA_FILE = "seen_listings.json"

# --- Load previously seen listings ---
if os.path.exists(DATA_FILE):
    with open(DATA_FILE, "r") as f:
        seen_listings = set(json.load(f))
else:
    seen_listings = set()

new_listings = []

def send_email(new_listings):
    body = ""
    for listing in new_listings:
        body += f"{listing['title']} — {listing['price']} — {listing['miles']} miles — {listing['location']}\n{listing['url']}\n\n"
    
    msg = MIMEText(body)
    msg["Subject"] = "🚗 New WRX STI Listing Found!"
    msg["From"] = SENDER_EMAIL
    msg["To"] = RECIPIENT_EMAIL

    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(SENDER_EMAIL, SENDER_PASS)
        server.sendmail(SENDER_EMAIL, RECIPIENT_EMAIL, msg.as_string())

def check_autotrader():
    url = f"https://www.autotrader.com/cars-for-sale/all-cars/subaru/wrx-sti/{SEARCH_ZIP}?dma=&searchRadius={RADIUS}&year={YEARS[0]}&year={YEARS[1]}&mileage={MAX_MILES}"
    soup = BeautifulSoup(requests.get(url).text, "html.parser")
    for listing in soup.select("div.inventory-listing"):
        title = listing.get("data-title", "Subaru WRX STI")
        link = listing.get("data-listing-url")
        if not link:
            continue
        full_link = "https://www.autotrader.com" + link
        price = listing.get("data-price", "N/A")
        miles = listing.get("data-mileage", "N/A")
        location = listing.get("data-location", "N/A")
        if full_link not in seen_listings:
            seen_listings.add(full_link)
            new_listings.append({
                "title": title,
                "price": price,
                "miles": miles,
                "location": location,
                "url": full_link
            })

# Placeholder for Cars.com, CarGurus, Craigslist functions (similar to above)

# Run scrapers
check_autotrader()
# check_carsdotcom()
# check_cargurus()
# check_craigslist()

# Send email if new listings found
if new_listings:
    send_email(new_listings)
    with open(DATA_FILE, "w") as f:
        json.dump(list(seen_listings), f)
