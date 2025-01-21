import pandas as pd
import requests
import json
import tkinter as tk
from tkinter import simpledialog
from pandasgui import show
import configparser

# Load configuration from config.ini
def load_config():
    config = configparser.ConfigParser()
    config.read('config.ini')
    
    api_token = config.get('API', 'token', fallback=None)
    base_url = config.get('API', 'base_url', fallback="https://app.allfunds.com/api/v1/funds/")
    
    if not api_token:
        raise ValueError("API token not found in config.ini")
    
    return api_token, base_url

# Function to fetch API data
def fetch_api_data(isin, api_token, base_url):
    endpoints = {
        "share_class": f"{base_url}{isin}/share_class",
        "overview": f"{base_url}{isin}/overview",
        "regulatory": f"{base_url}{isin}/regulatory?kind=emt"
    }
    
    headers = {"Authorization": f"Token {api_token}"}
    responses = {}

    for key, url in endpoints.items():
        try:
            response = requests.get(url, headers=headers)
            response.raise_for_status()  # Raise an exception for HTTP errors
            responses[key] = response.json()
        except (requests.exceptions.RequestException, json.JSONDecodeError) as e:
            print(f"Error fetching {key} data: {e}")
            return None
        print(responses)   
    return responses

# Function to process the API response data
def process_data(responses):
    try:
        share_class_df = pd.json_normalize(responses["share_class"]["share_class"])
        overview_df = pd.json_normalize(responses["overview"])
        regulatory_df = pd.json_normalize(responses["regulatory"])

        regulatory_df.rename(columns={'emt.00010_Financial_Instrument_Identifying_Data': 'isin'}, inplace=True)
        
        final_df = share_class_df.merge(overview_df, on="isin").merge(regulatory_df, on="isin")
        
        return final_df
    except KeyError as e:
        print(f"Data processing error: {e}")
        return None

# Function to filter funds based on criteria
def filter_funds(df):
    df['Fund is Valid ?'] = df.apply(lambda row: 'YES' if row["premium"] and 
                                     row["available_for_dealing"] and 
                                     "CHE" in row["countries_available_for_sale"] and 
                                     row.get('emt.01010_Investor_Type_Retail', '') == "Y"
                                     else 'NO', axis=1)
    return df

# Main function to run the script
def main():
    api_token, base_url = load_config()

    while True:
        ROOT = tk.Tk()
        ROOT.withdraw()
        isin = simpledialog.askstring(title="Search Fund", prompt="Enter ISIN:")
        ROOT.destroy()
        
        if not isin:
            print("No ISIN entered. Exiting.")
            break
        
        responses = fetch_api_data(isin, api_token, base_url)
        
        if responses:
            final_df = process_data(responses)
            if final_df is not None:
                filtered_df = filter_funds(final_df)
                #filtered_df.to_csv("request_filtered.csv", index=False)
                show(filtered_df)
            else:
                print("Error in processing data.")
        else:
            print("API request failed.")

if __name__ == "__main__":
    main()
