# UAE Real Estate Data Guide

A practical guide to accessing UAE and Dubai real estate data programmatically - covering available data sources, APIs, and how to use them for research, investment analysis, and application development.

Maintained by [Happy Endpoint](https://happyendpoint.com).

---

## The problem with UAE property data

If you have ever tried to get structured property data from the UAE, you know how frustrating it is.

Bayut and PropertyFinder are the two dominant portals. Between them they have hundreds of thousands of listings, years of transaction history, and detailed agent and agency data. But neither offers a public API. The only way to get the data is to either scrape it (which gets you blocked fast and violates their terms) or pay for expensive enterprise data feeds.

This guide covers the practical options available in 2026 for developers, researchers, and investors who need UAE property data.

---

## Data sources overview

| Source | Type | Coverage | Access |
|---|---|---|---|
| Bayut API (Happy Endpoint) | Real-time API | UAE-wide listings, agents, transactions | RapidAPI - free tier available |
| PropertyFinder API (Happy Endpoint) | Real-time API | UAE-wide listings | RapidAPI - free tier available |
| UAE Real Estate API (Happy Endpoint) | Real-time API | UAE property data | RapidAPI |
| Dubai Land Department (DLD) | Official data | Transaction records | Government portal (limited) |
| Bayut Bulk Dataset (Happy Endpoint) | Static dataset | 100K+ records | Purchase - happyendpointhq@gmail.com |
| Web scraping | DIY | Anything | High maintenance, legal risk |

For most use cases, the Bayut API is the best starting point. It covers the most data, has the best documentation, and has a free tier.

---

## Bayut API

The Bayut API is the most comprehensive option for UAE property data. It is built and maintained by Happy Endpoint and available on RapidAPI.

- API: https://rapidapi.com/happyendpoint/api/bayut14/
- Docs: https://bayutapi.dev
- Base URL: `https://bayut14.p.rapidapi.com`

### What data is available

**Property listings**
- Properties for sale and rent across all UAE
- Filters: location, property type, bedrooms, bathrooms, price range, area, amenities, completion status, furnishing
- Returns: title, price, area, photos, agent, agency, amenities, coordinates, reference number

**Off-plan projects**
- New development projects under construction or recently launched
- Filters: developer, completion percentage, pre-handover payment, location
- Returns: project details, developer info, starting price, completion status

**Agents**
- Real estate agents across UAE
- Search by location, name, or specialisation
- Returns: profile, languages, listing count, TruBroker badge status, service areas

**Agencies**
- Real estate agencies
- Search by location or name
- Returns: agency profile, listing count, agent count, contact info

**Developers**
- Property developers (Emaar, Damac, Sobha, Nakheel, etc.)
- Search by name
- Returns: developer ID for use in off-plan project filtering

**Transaction history**
- Historical sale and rental transactions
- Up to 24 months of data
- Filters: location, property type, price range, time period
- Useful for price trend analysis and market research

**Locations**
- Autocomplete search for UAE areas
- Returns location IDs used in all other endpoints

### Quick start

```python
import requests

headers = {
    "x-rapidapi-host": "bayut14.p.rapidapi.com",
    "x-rapidapi-key": "YOUR_KEY"
}

# Step 1 - get location ID
r = requests.get(
    "https://bayut14.p.rapidapi.com/autocomplete",
    params={"query": "dubai marina"},
    headers=headers
)
location_id = r.json()["data"]["locations"][0]["externalID"]
# location_id = "5003"

# Step 2 - search properties
r = requests.get(
    "https://bayut14.p.rapidapi.com/search-property",
    params={
        "purpose": "for-sale",
        "location_ids": location_id,
        "property_type": "apartments",
        "rooms": "1",
        "page": 1
    },
    headers=headers
)
data = r.json()["data"]
print(f"{data['total']} properties found")
```

---

## Dubai property market - key facts

Understanding the market context helps when working with the data.

### Market size

- Dubai recorded over AED 761 billion in real estate transactions in 2024
- More than 180,000 individual transactions in 2024 - a record at the time
- Over 500,000 active listings on Bayut at any given time
- Investors from 100+ nationalities active in the market

### Property types

**Residential**
- Apartments (studios, 1-bed, 2-bed, 3-bed, 4-bed+)
- Villas
- Townhouses
- Penthouses
- Hotel apartments

**Commercial**
- Offices
- Retail shops
- Warehouses
- Showrooms

### Key areas and typical price ranges (AED per sqft, 2025-2026)

| Area | For Sale (AED/sqft) | Gross Rental Yield |
|---|---|---|
| Palm Jumeirah | 3,200 - 5,000+ | 4-5% |
| Downtown Dubai | 2,600 - 3,200 | 5-6% |
| Dubai Marina | 2,200 - 2,800 | 6-7% |
| Business Bay | 1,700 - 2,200 | 6-7% |
| Dubai Hills Estate | 1,800 - 2,400 | 5-6% |
| JLT | 1,200 - 1,700 | 7-8% |
| JVC | 900 - 1,300 | 7-8% |

Note: these are approximate ranges. Use the API's `/transactions` endpoint to get current data for specific areas.

### Off-plan market

Off-plan (properties sold before completion) accounts for 40%+ of Dubai's residential sales by volume. Key characteristics:

- Typical payment plan: 10-20% down, instalments during construction, 40-50% on handover
- Construction timelines: 2-5 years from launch to handover
- Developers are required by law to hold buyer payments in DLD escrow accounts
- Major developers: Emaar, Damac, Sobha, Nakheel, Meraas, Aldar (Abu Dhabi)

### Rental market

- Rental contracts are typically annual, paid upfront by cheque (1-4 cheques)
- Monthly and short-term rentals exist but are less common
- Rental yields are among the highest of any major global city
- No rent control in Dubai (unlike Abu Dhabi which has a rent cap system)

---

## Common use cases

### 1. Property search application

Build a search portal for Dubai properties. The core flow:

1. User types an area name - use `/autocomplete` to get location ID
2. User sets filters (type, bedrooms, price) - pass to `/search-property`
3. User clicks a listing - use `/property-details` for full info
4. Show off-plan section - use `/search-new-projects`

### 2. Investment analysis

Calculate rental yields and compare areas:

```python
def rental_yield(location_id, rooms="1"):
    # get sale prices
    sale = requests.get(
        "https://bayut14.p.rapidapi.com/search-property",
        params={"purpose": "for-sale", "location_ids": location_id,
                "property_type": "apartments", "rooms": rooms},
        headers=headers
    ).json()["data"]["properties"]

    # get rental prices
    rent = requests.get(
        "https://bayut14.p.rapidapi.com/search-property",
        params={"purpose": "for-rent", "location_ids": location_id,
                "property_type": "apartments", "rooms": rooms},
        headers=headers
    ).json()["data"]["properties"]

    avg_sale = sum(p["price"] for p in sale if p.get("price")) / len(sale)
    avg_rent = sum(p["price"] for p in rent if p.get("price")) / len(rent)

    return round((avg_rent / avg_sale) * 100, 2)
```

### 3. Market research and data analysis

Use the `/transactions` endpoint to pull historical data and analyse price trends:

```python
# Get 24 months of transaction data for an area
r = requests.get(
    "https://bayut14.p.rapidapi.com/transactions",
    params={
        "purpose": "for-sale",
        "location_ids": "5003",  # Dubai Marina
        "time_period": "24m",
        "category_ids": "apartments",
        "sort": "date_asc"
    },
    headers=headers
)
transactions = r.json()["data"]
```

### 4. Agent and agency directory

Build a searchable directory of Dubai real estate agents:

```python
# Get all agents in Dubai Marina
r = requests.get(
    "https://bayut14.p.rapidapi.com/agent-search",
    params={
        "location_ids": "5003",
        "purpose": "for-sale",
        "category": "residential",
        "page": 1
    },
    headers=headers
)
```

### 5. Price alert system

Monitor listings and alert users when prices drop or new listings appear matching their criteria. See the [dubai-property-price-tracker](https://github.com/yourusername/dubai-property-price-tracker) repo for a full implementation.

---

## Working with the data

### Area in sqm vs sqft

The API returns area in square metres. Dubai's market typically quotes prices in square feet. Convert:

```python
area_sqft = area_sqm * 10.764
price_per_sqft = price / area_sqft
```

### Pagination

All list endpoints return paginated results (typically 24 per page for properties, 40 for agents). Always check `totalPages` and loop if you need all results:

```python
all_results = []
page = 1

while True:
    data = fetch_page(page)
    all_results.extend(data["properties"])
    if page >= data["totalPages"]:
        break
    page += 1
```

### Language support

Most text fields support multiple languages. Pass `langs=en,ar` to get both English and Arabic names in the response. Available: `en`, `ar`, `ru`, `zh`.

### Location hierarchy

Locations are hierarchical. Level 0 is UAE, level 1 is emirate (Dubai, Abu Dhabi, etc.), level 2 is neighbourhood. When filtering, you can use any level - using a level 1 ID (e.g., Dubai) returns all properties in that emirate.

---

## Bulk data

If you need large-scale historical data that goes beyond what the real-time API provides, Happy Endpoint sells pre-compiled datasets:

- 100,000+ Bayut records (listings + transactions)
- CSV and JSON formats
- Suitable for ML model training, econometric analysis, and large-scale research

Contact: happyendpointhq@gmail.com
Website: https://happyendpoint.com

---

## Other Happy Endpoint APIs

Happy Endpoint also provides APIs for other UAE and international property portals:

- **PropertyFinder UAE API**: https://rapidapi.com/happyendpoint/api/propertyfinder-uae-data
- **UAE Real Estate API**: https://rapidapi.com/happyendpoint/api/uae-real-estate-api/

Full list: https://rapidapi.com/user/happyendpoint

---

## Links

- Bayut API: https://rapidapi.com/happyendpoint/api/bayut14/
- Bayut API docs: https://bayutapi.dev
- Happy Endpoint: https://happyendpoint.com
- Twitter: https://x.com/happyendpointhq
- Email: happyendpointhq@gmail.com

---

## Contributing

Found something outdated or missing? Open an issue or PR. Market data changes fast and this guide is a living document.

---

## License

MIT
