Lead Verification Project - README

This project automates the process of verifying and scoring lead data collected from a Facebook lead generation campaign. The goal is to identify high-quality business leads by comparing submitted addresses and names to verified data from the Google Places API.

Data Cleaning
1) Load input Data File: Leads.csv
   
2) HTML Special Characters Removal
Fields like "restaurant_name" are cleaned using "clean_text()" to replace HTML entities like "&amp;", "&#x27;", and "&#x2F;" with readable characters.

3) Postal Code Normalization
Zip/postal codes are normalized to ensure they are 5-digit strings using "normalize_postal_code()". Short codes are padded with zeros (e.g. "9201" becomes "09201"), and longer ones are truncated after 5 digits.

Google Places API Queries
1) API Call Functionality
call_places_api() makes HTTP POST requests to the Google Places API
Queries are sent using textQuery field, with optional field masks to limit returned data

2) Two queries are performed for each lead
"Address Query": Searches by address only to gather all business listings at a location
"Full Query": Searches by business name + address to validate exact identity

Data Processing Logic
1) Google API Return Address Normalization
Google-formatted addresses are normalized using "refine_address_with_unit()":
Converted to lowercase
Removes unit identifiers like "suite", "apt", "unit", "#", "floor", etc.
Collapses extra whitespace and commas
Ensures consistency for comparison with lead-provided addresses

2) Full Address Construction from the lead's file: 
A "full_address" field is constructed from components (address, city, state, zip) to match the formatted addresses returned by Google.

Matching Logic
1) Address Match
The address from the full query must match one of the addresses from the address-only query (after normalization).

2) Name Match
The system compares the lead’s business name with the Google result using "broad_name_match()"
Uses token-based comparison
Filters out generic words like "restaurant", "cuisine", etc.
Considers a match if one name is a subset of the other or if fuzzy similarity exceeds a threshold (default: 0.65)

3) Cuisine Match
The "cuisine_match()"  function checks if the Google "primaryType" includes target terms like "pizza" or "italian"

Match Scoring
1) Each lead is evaluated on five factors:

| Factor           | Weight | Description                                            |
|------------------|--------|--------------------------------------------------------|
| Status           | 30%    | Whether the business is currently operational          |
| Cuisine Match    | 15%    | Business type includes "pizza" or "italian"            |
| Address Match    | 30%    | Lead address matches one returned by Google            |
| Name Match       | 15%    | Lead business name closely resembles Google result     |
| Website Presence | 10%    | Google entry has a website listed                      |

2) The final lead score is calculated as:
lead_score = (status * 0.30 + cuisine_match * 0.15 + address_match * 0.30 + name_match * 0.15 + website * 0.10) * 100

Output
The final output is a CSV file named "LEADS_with_PlacesAPI_WeightedScore.csv", which includes the original lead data along with:
Cleaned addresses
Matched Google business data
Match flags
Final lead score as a percentage

Notes:
API calls are rate-limited by Google; caching and deduplication strategies can help reduce costs
The fuzzy match threshold and scoring weights can be configured via the "CONFIG" dictionary
Normalization functions are crucial for ensuring consistent, meaningful comparisons across systems
