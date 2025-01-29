# AI-Data-Collector---Tech-and-Fashion-Images
assist in collecting personal images of technology gadgets, designer clothing, and regular clothing items priced between $70 and $500. These images will be used to train an AI model, necessitating a diverse and authentic dataset. Each item must have at least three images to ensure comprehensive learning.

Key Responsibilities:

- Image Collection: Search for personal images of tech gadgets (e.g., smartphones, laptops, smartwatches) and clothing items (both designer and everyday apparel) within the specified price range.
- Image Quality: Ensure images are not professional product photos but rather authentic, real-world captures.
- Data Organization: Organize image links, item descriptions, and prices in a Google Sheet. Each item must have at least three images.
- Price Verification: Confirm that all items fall within the $70 to $500 range.
- Data Integrity: Check that images are personal and not from official product pages.

Qualifications:

- Proficiency in internet research and data organization.
- Strong attention to detail to ensure accuracy and relevance.
- Experience working with spreadsheets (Google Sheets preferred).
- Prior experience with AI training data collection is a plus.

Deliverables:

A Google Sheet containing:
- Item/Image links
- Item descriptions
- Prices

Optional notes column for additional details
Each item must have at least three images.
----------
To assist with this task, I can guide you through a Python-based automation approach that combines web scraping and data organization in Google Sheets. Since manually collecting images is labor-intensive, I will break down the code into different steps for automating the process.
Steps to Collect Personal Images of Tech Gadgets & Clothing:

    Web Scraping: Use Python libraries to scrape online marketplaces (e.g., eBay, Poshmark, Craigslist) or image hosting platforms (e.g., Instagram, Pinterest, etc.) for images of tech gadgets and clothing.

    Price Verification: Ensure the collected items are within the price range of $70 to $500.

    Google Sheets Integration: Organize the data into Google Sheets (links to the items, descriptions, and prices).

Tools You’ll Need:

    Selenium: For automating web browsing, especially if the site contains JavaScript-loaded content.
    BeautifulSoup: For parsing HTML content.
    Google Sheets API: To interact with Google Sheets and log the data.
    Requests: For downloading images.
    Regular Expressions: For filtering the desired price range and other criteria.

Step 1: Set Up Environment and Install Libraries

First, set up a virtual environment and install the necessary libraries:

pip install selenium beautifulsoup4 requests google-api-python-client google-auth-httplib2 google-auth-oauthlib pandas

You will also need to set up ChromeDriver for Selenium if you are scraping websites that require JavaScript rendering.
Step 2: Scraping Websites for Images and Prices

This step will focus on scraping data from a website (such as eBay or Poshmark) for tech gadgets and clothing items within the price range. Below is an example code using Selenium and BeautifulSoup to scrape a website.

from selenium import webdriver
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import requests
import time

# Setup the ChromeDriver
driver = webdriver.Chrome(executable_path='path/to/chromedriver')  # Update path to your chromedriver

# Function to scrape product images and prices from an e-commerce page
def get_product_images_and_prices(url):
    driver.get(url)
    time.sleep(5)  # Let the page load for a while
    
    # Parse page with BeautifulSoup
    soup = BeautifulSoup(driver.page_source, 'html.parser')
    
    # Example: Finding price and images on the page (you need to adjust based on page structure)
    products = []
    product_cards = soup.find_all('div', class_='product-card')  # Update selector based on your target website
    
    for card in product_cards:
        name = card.find('h2', class_='product-title').text.strip()  # Update selector
        price = card.find('span', class_='product-price').text.strip()  # Update selector
        
        # Extract price range and validate
        price_value = float(price.replace('$', '').replace(',', ''))
        if 70 <= price_value <= 500:
            images = card.find_all('img', class_='product-image')  # Update selector
            image_urls = [img['src'] for img in images if img['src']]
            
            if len(image_urls) >= 3:  # Ensure at least 3 images
                products.append({
                    'name': name,
                    'price': price,
                    'image_urls': image_urls
                })
    return products

# Example: Scraping products from a URL
url = 'https://www.example.com/products'
product_data = get_product_images_and_prices(url)
print(product_data)

Step 3: Storing the Data in Google Sheets

Once the images, names, and prices are collected, you can store them in a Google Sheet using the Google Sheets API.
Set Up Google Sheets API

    Go to Google Developer Console and create a new project.
    Enable Google Sheets API and Google Drive API.
    Download the credentials as a credentials.json file.

Code to Write Data to Google Sheets

This script authenticates your Google account and writes the scraped data to a Google Sheet.

import os
import pandas as pd
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError

# Authenticate and create the Google Sheets API client
def authenticate_gsheet():
    SCOPES = ['https://www.googleapis.com/auth/spreadsheets']
    creds = None

    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)

        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    
    service = build('sheets', 'v4', credentials=creds)
    return service

# Function to write data to Google Sheets
def write_to_gsheet(service, spreadsheet_id, data):
    try:
        sheet = service.spreadsheets()

        # Data: list of dictionaries (name, price, image_urls)
        values = []
        for entry in data:
            row = [entry['name'], entry['price'], ', '.join(entry['image_urls'])]
            values.append(row)

        body = {
            'values': values
        }

        result = sheet.values().update(
            spreadsheetId=spreadsheet_id,
            range="Sheet1!A2",  # Adjust if you want to start in a different cell
            valueInputOption="RAW",
            body=body
        ).execute()

        print(f"{result.get('updatedCells')} cells updated.")
    
    except HttpError as err:
        print(f"An error occurred: {err}")

# Sample usage
spreadsheet_id = 'your_google_sheet_id'  # Replace with your Google Sheet ID
service = authenticate_gsheet()

# Now you can call this function to write the product data
write_to_gsheet(service, spreadsheet_id, product_data)

Step 4: Full Script to Collect, Validate, and Upload Data

Now, you have:

    The ability to scrape data from a URL.
    A function to upload the data to Google Sheets.
    Automated extraction of price and image validation.

You can automate the process for different websites and run it at scheduled intervals using a task scheduler like cron or Windows Task Scheduler.
Step 5: Handle Errors and Validate Data

Ensure that you handle errors like:

    Invalid URLs or missing data.
    Inconsistent image data or price range errors.
    Duplicate entries in Google Sheets.

You can use a simple check for data integrity, for example:

# Check if data is already in the Google Sheets before uploading
existing_data = service.spreadsheets().values().get(
    spreadsheetId=spreadsheet_id, range="Sheet1!A2:C").execute()
existing_entries = existing_data.get('values', [])

for entry in product_data:
    if entry['name'] not in [e[0] for e in existing_entries]:  # Check for duplicates
        write_to_gsheet(service, spreadsheet_id, [entry])

Conclusion

This workflow allows you to scrape images and price data from web pages, verify that the prices fall within the $70–$500 range, and then store this data in Google Sheets for future use. You can automate this for multiple products and continuously collect new data for your AI model training.

Remember, scraping websites should be done in compliance with their terms of service, so make sure you are allowed to scrape the data and images you need.
