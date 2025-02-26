# Scraping HTML Tables with Python

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to scrape HTML tables using Python with Beautiful Soup, pandas, and Requests.

- [Prerequisites](#prerequisites)
- [Understanding the Web Page Structure](#understanding-the-web-page-structure)
- [Sending an HTTP Request to Access the Web Page](#sending-an-http-request-to-access-the-web-page)
- [Parsing the HTML Using Beautiful Soup](#parsing-the-html-using-beautiful-soup)
- [Cleaning and Structuring the Data](#cleaning-and-structuring-the-data)
- [Exporting Cleaned Data to CSV](#exporting-cleaned-data-to-csv)

## Prerequisites

Ensure that you have Python 3.8 or newer installed, create a [virtual environment](https://docs.python.org/3/library/venv.html), and install the following Python packages:

- **[Requests](https://requests.readthedocs.io/en/latest/)**: A library for sending HTTP requests to interact with web services and APIs, enabling data retrieval and submission.
- **[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)**: A tool for parsing HTML documents, allowing structured navigation, searching, and data extraction from web pages.
- **[pandas](https://pandas.pydata.org/)**: A library for analyzing, cleaning, and organizing scraped data, with support for exporting it to formats like CSV or XLSX.

```bash
pip install requests beautifulsoup4 pandas
```

## Understanding the Web Page Structure

In this tutorial, you'll scrape data from the [Worldometer website](https://www.worldometers.info/world-population/population-by-country/), which features up-to-date country population figures for 2024.

![HTML table on the web page](https://brightdata.com/wp-content/uploads/2024/12/image-71-1024x670.png)

To inspect the HTML table, right-click on it and select **Inspect**. This opens the Developer Tools panel, highlighting the corresponding HTML code:

![Inspect element with selected element highlighted](https://brightdata.com/wp-content/uploads/2024/12/image-72-1024x576.png)

The table structure begins with a `<table>` tag (ID `example2`), contains header cells defined by `<th>` tags, and rows defined by `<tr>` tags. Within each row, `<td>` tags create individual cells that hold the data.

> **Note:**
>
> Before scraping, review and comply with the website’s privacy policy and terms of service to ensure you adhere to all data usage and automated access restrictions.

## Sending an HTTP Request to Access the Web Page

To send an HTTP request and access the web page, create a Python file (_eg_ `html_table_scraper.py`) and import the `requests`, `BeautifulSoup`, and `pandas` packages:

```python
# import packages
import requests
from bs4 import BeautifulSoup
import pandas as pd
```

Then, define the URL of the web page you want to scrape and send a GET request to that web page using `https://www.worldometers.info/world-population/population-by-country/`:

```python
# Send a request to the website to get the page content
url = 'https://www.worldometers.info/world-population/population-by-country/'
```

Send a request using the `get()` method from Requests to check whether the response is successful:

```python
# Get the content of the URL
response = requests.get(url)
 
# Check the status of the response.
if response.status_code == 200:
    print("Request was successful!")
else:
    print(f"Error: {response.status_code} - {response.text}")
```

This code sends a GET request to a specified URL and then checks the status of the response. A `200` response indicates the request was successful.

Run the Python script:

```bash
python html_table_scraper.py
```

Your output should look like this:

```
Request was successful!
```

Since the GET request is successful, you now have the HTML content of the entire web page, including the HTML table.

## Parsing the HTML Using Beautiful Soup

Beautiful Soup is built to handle messy or broken HTML, a common issue when scraping web pages. It allows you to:
- Parse HTML to locate the population data table.
- Extract table headers.
- Collect data from each table row.

To begin parsing, create a Beautiful Soup object:

```python
# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(response.content, 'html.parser')
```

Next, locate the table element in the HTML with the `id` attribute `"example2"`. This table contains the population of countries in 2024:

```python
# Find the table containing population data
table = soup.find('table', attrs={'id': 'example2'})
```

### Collecting Table Headers

The table has a header located in the `<thead>` and `<th>` HTML tags. Use the `find()` method from the Beautiful Soup package to extract the data in the `<thead>` tag and the `find_all()` method to collect all the headers:

```python
# Collect the headers from the table
headers = []

# Locate the header row within the <thead> tag
header_row = table.find('thead').find_all('th')

for th in header_row:
    # Add header text to the headers list
    headers.append(th.text.strip())
```

This code creates an empty Python list called `headers`, locates the `<thead>` HTML tag to find all headers within `<th>` HTML tags, and then appends each collected header to the `headers` list.

### Collecting Table Row Data

To collect the data in each row, create an empty Python list called `data` to store the scraped data:

```python
# Initialize an empty list to store our data
data = []
```

Then, extract the data in each row in the table using the `find_all()` method and append them to the Python list:

```python
# Loop through each row in the table (skipping the header row)
for tr in table.find_all('tr')[1:]:
    # Create a list of the current row's data
    row = []
    
# Find all data cells in the current row
    for td in tr.find_all('td'):
        # Get the text content of the cell and remove extra spaces
        cell_data = td.text.strip()
        
        # Add the cleaned cell data to the row list
        row.append(cell_data)
        
    # After getting all cells for this row, add the row to our data list
    data.append(row)
    
# Convert the collected data into a pandas DataFrame for easier handling
df = pd.DataFrame(data, columns=headers)

# Print the DataFrame to see the number of rows and columns
print(df.shape)
```

This code iterates through all `<tr>` HTML tags found within the `table`, starting from the second row (skipping the header row). For each row (`<tr>`), an empty list `row` is created to store the data from that row’s cells. Inside the row, the code finds all `<td>` HTML tags using the `find_all()` method, representing individual data cells in the row.

For each `<td>` HTML tag, the code extracts the text content using the `.text`attribute and applies the `.strip()` method to remove any leading or trailing whitespace from the text. The cleaned cell data is appended to the `row` list. After processing all the cells in the current row, the entire row is appended to the `data` list. Finally, you convert the collected data to a pandas DataFrame with the column names defined by the `headers` list and then show the shape of the data.

The full Python script should look like this:

```python
# Import packages
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Send a request to the website to get the page content
url = 'https://www.worldometers.info/world-population/population-by-country/'

# Get the content of the URL
response = requests.get(url)

# Check if the request was successful
if response.status_code == 200:
    # Parse the HTML content using Beautiful Soup
    soup = BeautifulSoup(response.content, 'html.parser')

    # Find the table containing population data by its ID
    table = soup.find('table', attrs={'id': 'example2'}) 

    # Collect the headers from the table
    headers = []

    # Locate the header row within the <thead> HTML tag
    header_row = table.find('thead').find_all('th')

    for th in header_row:
        # Add header text to the headers list
        headers.append(th.text.strip())

    # Initialize an empty list to store our data
    data = []

    # Loop through each row in the table (skipping the header row)
    for tr in table.find_all('tr')[1:]:

        # Create a list of the current row's data
        row = []

        # Find all data cells in the current row
        for td in tr.find_all('td'):
            # Get the text content of the cell and remove extra spaces
            cell_data = td.text.strip()

            # Add the cleaned cell data to the row list
            row.append(cell_data)

        # After getting all cells for this row, add the row to our data list
        data.append(row)

    # Convert the collected data into a pandas DataFrame for easier handling
    df = pd.DataFrame(data, columns=headers)

    # Print the DataFrame to see the collected data
    print(df.shape)
else:
    print(f"Error: {response.status_code} - {response.text}")
```

Use the following command to run the Python script in your terminal:

```bash
python html_table_scraper.py
```

Your output should look like this:

```
(234,12)
```

Next, use the `head()` method from pandas and `print()` to view the first ten rows of the extracted data:

```python
print(df.head(10))
```

![Top ten rows from the scraped table](https://brightdata.com/wp-content/uploads/2024/12/image-73-1024x333.png)

## Cleaning and Structuring the Data

Cleaning scraped data from an HTML table is crucial for consistency, accuracy, and usability in analysis. Raw data may have missing values, formatting errors, unwanted characters, or incorrect data types, all of which can lead to unreliable results. Proper cleaning standardizes the dataset and aligns it with the intended structure for analysis.

In this section, the following data-cleaning tasks are performed:

- Renaming column names
- Replacing missing values presented in the row data
- Removing commas and convert data types to the correct format
- Removing the percentage sign (%) and convert data types to the correct format
- Changing data types for numerical columns

### Renaming Column Names

Pandas offers the `rename()` method to update column names, making them more descriptive or easier to work with. Simply pass a dictionary to the `columns` parameter, where keys represent current names and values represent the new names. Use this method to update the following column names:

- `#` to `Rank`
- `Yearly change` to `Yearly change %`
- `World Share` to `World Share %`

```python
# Rename columns
df.rename(columns={'#': 'Rank'}, inplace=True)
df.rename(columns={'Yearly Change': 'Yearly Change %'}, inplace=True)
df.rename(columns={'World Share': 'World Share %'}, inplace=True)

# Show the first 5 rows
print(df.head())
```

The columns should now look like this:

![Column names after renaming](https://brightdata.com/wp-content/uploads/2024/12/image-74-1024x186.png)

### Replacing Missing Values

Missing values can skew calculations like averages or sums, leading to inaccurate insights. Remove, replace, or fill these gaps with appropriate values before performing any analysis.

The `Urban Pop %` column currently contains missing values labeled as `N.A.`. Replace `N.A.` with `0%` using the `replace()` method from pandas like this:

```python
# Replace 'N.A.' with '0%' in the 'Urban Pop %' column
df['Urban Pop %'] = df['Urban Pop %'].replace('N.A.', '0%')
```

### Removing Percentage Signs and Convert Data Types

The columns `Yearly Change %`, `Urban Pop %`, and `World Share %` contain numbers with a `%` sign (e.g., `37.0%`), which prevents direct mathematical operations like calculating averages or standard deviations. To fix this, use the `replace()` method to remove the `%` and then convert the values to `float` using `astype()`.

```python
# Remove the '%' sign and convert to float
df['Yearly Change %'] = df['Yearly Change %'].replace('%', '', regex=True).astype(float)
df['Urban Pop %'] = df['Urban Pop %'].replace('%', '', regex=True).astype(float)
df['World Share %'] = df['World Share %'].replace('%', '', regex=True).astype(float)

# Show the first 5 rows
df.head()
```

This code removes the `%` sign from the values in the columns `Yearly Change %`, `Urban Pop %`, and `World Share %` using the `replace()` method with a regular expression. Then, it converts the cleaned values to a `float` data type using `astype(float)`. Finally, it displays the first five rows of the DataFrame with `df.head()`.

Your output should look like this:

![Top five rows](https://brightdata.com/wp-content/uploads/2024/12/image-75-1024x186.png)

### Removing Commas and Convert Data Types

Currently, the columns `Population (2024)`, `Net Change`, `Density (P/Km²)`, `Land Area (Km²)`, and `Migrants (net)` contain numerical values with commas (_eg_ 1,949,236). This makes it impossible to perform mathematical operations for analysis.

To fix this, you can apply the `replace()` and `astype()` to remove commas and convert the numbers to the integers data type:

```python
# Remove commas and convert to integers
columns_to_convert = [
    'Population (2024)', 'Net Change', 'Density (P/Km²)', 'Land Area (Km²)',
    'Migrants (net)'
]

for column in columns_to_convert:
    # Ensure the column is treated as a string first
    df[column] = df[column].astype(str)

    # Remove commas
    df[column] = df[column].str.replace(',', '')

    # Convert to integers
    df[column] = df[column].astype(int)
```

This code creates a list called `columns_to_convert` for the columns to process. For each column, it converts values to strings with `astype(str)`, removes commas using `str.replace(',', '')`, and then converts them to integers with `astype(int)`, preparing the data for mathematical operations.

### Changing Data Types for Numerical Columns

The `Rank`, `Med. Age`, and `Fert. Rate` columns are stored as objects even though they contain numbers. Convert these columns to integers or floats to enable mathematical operations:

```python
# Convert to integer or float data types and integers

df['Rank'] = df['Rank'].astype(int)
df['Med. Age'] = df['Med. Age'].astype(int)
df['Fert. Rate'] = df['Fert. Rate'].astype(float)
```

This code converts the values in the `Rank` and `Med. Age` columns into an integer data type and the values in the `Fert. Rate` into a float data type.

Finally, check to make sure the cleaned data has the correct data types using the `head()` method:

```python
print(df.head(10))
```

Your output should look like this:

![Top ten rows of the cleaned data](https://brightdata.com/wp-content/uploads/2024/12/image-76-1024x333.png)

## Exporting Cleaned Data to CSV

After cleaning your data, save it for future analysis and sharing by exporting it to a CSV file. Use pandas' `to_csv()` method to export your DataFrame to a file named `world_population_by_country.csv`:

```python
# Save the data to a file
filename = 'world_population_by_country.csv'
df.to_csv(filename, index=False)
```

## Conclusion

Extracting data from complex websites can be difficult and time-consuming. To save you time and make things easier, consider using the [Bright Data Web Scraper API](https://brightdata.com/products/web-scraper). This powerful tool offers a prebuilt scraping solution, allowing you to extract data from complex websites with minimal technical knowledge.

Sign up and start your free Web Scraper API trial!