# DS3000-Project

Member Contribution Plan:  
Clean data – Darsheen  
Graphs and visualizations – Hayli  
Machine learning – Ethan  
Final report and presentation – Eric  

import requests
import pandas as pd
api_key = '7a802e204a08789569034837ff203fb7'
url_base = 'https://api.stlouisfed.org/fred/series/observations'

def fetch_and_clean_data(series_ids, start_date='2000-01-01', end_date='2024-12-31'):
    """ pulls macro economic data from the given series ID with specified start and 
        end date and cleans the data
    
    Args:
        series_ids (list): list of macro economic series id to get from the API
        start_date (str) : start date for collecting data in YY-MM-DD format
        end_date (str) : end date for collecting data in YY-MM-DD format
    Returns:
        merged_df (DataFrame): a merged dataframe with numerical and categorical data from
                                the series requested. Each column corrosponds to a series ID 
                                with its values
    """
    # creating an empty list to store all values
    all_data = []
     
    # using for loop to iterate through each series ID to get data from API key  
    for series_id in series_ids:
        params = {
            'series_id': series_id,
            'api_key': api_key,
            'file_type': 'json',
            'observation_start': start_date,
            'observation_end': end_date
        }
        
        # sending a request to API
        response = requests.get(url_base, params=params)
        data = response.json()['observations']
        
        # Creating the dataframe
        df = pd.DataFrame(data)
        df = df[['date', 'value']]
        
        # converting the values to numeric and date in datetime format
        df['value'] = pd.to_numeric(df['value'], errors='coerce')
        df['date'] = pd.to_datetime(df['date'])
        
        # renaming the value columns to series ID
        df.rename(columns={'value': series_id}, inplace=True)
        
        # Append dataFrame to the list
        all_data.append(df)

    # Adding categorical feature: Inflation Level
    if 'CPIAUCSL' in series_ids:
        merged_df['Inflation_Level'] = pd.cut(merged_df['CPIAUCSL'], 
                                              bins=[-float('inf'), 2, 3, float('inf')], 
                                              labels=['Low', 'Medium', 'High'])
    
    # calculating GDP growth rate and categorizing it as recession, stagnation or growth
    if 'GDP' in series_ids:
        merged_df['GDP_Growth_Rate'] = merged_df['GDP'].pct_change() * 100
        merged_df['GDP_Growth_Stage'] = pd.cut(merged_df['GDP_Growth_Rate'], 
                                               bins=[-float('inf'), 0, 2, float('inf')], 
                                               labels=['Recession', 'Stagnation', 'Growth'])

    # Saving the merged DataFrame to a CSV file
    merged_df.to_csv('macro_data.csv')

    return merged_df
# list of series ID
series_ids = ['GDP', 'UNRATE', 'CPIAUCSL', 'RSAFS', 'INDPRO']
merged_df = fetch_and_clean_data(series_ids)

merged_df.head()
