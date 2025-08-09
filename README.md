# Bayut_WA_Ag
For jlt Cl D E F
import requests
from bs4 import BeautifulSoup
import json
import os
from datetime import datetime

# Your UltraMsg credentials
INSTANCE_ID = os.getenv("ULTRA_INSTANCE")
TOKEN = os.getenv("ULTRA_TOKEN")
PHONE = os.getenv("ALERT_PHONE")  # Your phone in full international format

# Bayut URLs for JLT Clusters D, E, F
URLS = [
    "https://www.bayut.com/to-rent/property/dubai/jumeirah-lake-towers/d-cluster/",
    "https://www.bayut.com/to-rent/property/dubai/jumeirah-lake-towers/e-cluster/",
    "https://www.bayut.com/to-rent/property/dubai/jumeirah-lake-towers/f-cluster/",
    "https://www.bayut.com/for-sale/property/dubai/jumeirah-lake-towers/d-cluster/",
    "https://www.bayut.com/for-sale/property/dubai/jumeirah-lake-towers/e-cluster/",
    "https://www.bayut.com/for-sale/property/dubai/jumeirah-lake-towers/f-cluster/"
]

DATA_FILE = "data.json"

def load_previous():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    return {}

def save_current(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f)

def scrape(url):
    listings = []
    r = requests.get(url, headers={"User-Agent": "Mozilla/5.0"})
    soup = BeautifulSoup(r.text, "html.parser")
    for card in soup.select("article"):
        title = card.get_text(strip=True)
        link_tag = card.find("a", href=True)
        link = "https://www.bayut.com" + link_tag["href"] if link_tag else ""
        listings.append({"title": title, "link": link})
    return listings

def send_whatsapp(message):
    url = f"https://api.ultramsg.com/{INSTANCE_ID}/messages/chat"
    payload = {"token": TOKEN, "to": PHONE, "body": message}
    requests.post(url, data=payload)

def main():
    previous = load_previous()
    current = {}
    new_items = []

    for url in URLS:
        listings = scrape(url)
        current[url] = listings

        old_listings = previous.get(url, [])
        old_links = {l["link"] for l in old_listings}
        for l in listings:
            if l["link"] not in old_links:
                new_items.append(l)

    if new_items:
        for item in new_items:
            send_whatsapp(f"New Bayut Listing: {item['title']}\n{item['link']}")
        print(f"Sent {len(new_items)} alerts.")

    save_current(current)

if __name__ == "__main__":
    main()
