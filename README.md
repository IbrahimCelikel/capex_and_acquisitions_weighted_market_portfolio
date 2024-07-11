# Project Description.
Goal: Compare the market value-weighted market portfolio with the net capex and acquisitions weighted market portfolio.
<br>
<br>Processes: Used only free sources(to learn web scraping, data cleaning, etc.) Web scraped for financials, S&P500 list and historical changes, and year-ending market values of companies. Then, built and compared the portfolios.
<br>
<br>Results: Since the data is limited, there is no solid evidence to choose which is better. However, capex and acquisitions-based portfolios won within the investment horizon. Therefore, the project and common logic suggest that the recent investments-based market portfolio adapts to changes faster and selects companies better when the economy is in recession (the investment horizon is from 2019 ending to 2022 ending). I will re-do this project with more data later.

# Create S&P500 components lists for recent years with Python.
On the Wikipedia, there is the current list and historical changes.
```
import pandas as pd

url = "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"
data = pd.read_html(url)
```
Collect and save to a CSV file the current SP500 list.
```
sp500_2023_8 = data[0].iloc[:,[0,1]]
sp500_2023_8.columns = ['Ticker', 'Security']

sp500_2023_8.to_csv(r'C:\depo\repositories\sp500 r&d based market portfolio\sp500_2023_8.csv')
```
Collect historical changes to prepare year-ending lists for previous years.
```
changes = data[1].iloc[:,[0,1,2,3,4]]
changes.columns = ['date', 'ticker_added', 'security_added', 'ticker_deleted', 'security_deleted']
changes['date'] = changes['date'].str[-4:]
changes = changes.set_index('date')
changes = changes.replace(np.nan, "-----")
```
Delete companies added in 2023 and add the companies deleted in 2023 to reach the 2022 ending SP500 list. Then, save as a CSV file.
```
sp2023_added = changes.loc['2023']
sp2023_added = sp2023_added.iloc[:,[0,1]]
sp2023_added.columns = ['Ticker', 'Security']

sp500_2022end = pd.merge(sp500_2023_8, sp2023_added, indicator=True, how='outer').query('_merge=="left_only"').drop('_merge',axis=1)

sp2023_deleted = changes.loc['2023']
sp2023_deleted = sp2023_deleted.iloc[:,[2,3]]
sp2023_deleted.columns = ['Ticker', 'Security']

sp500_2022end = pd.concat([sp500_2022end, sp2023_deleted])

sp500_2022end.to_csv(r'C:\depo\repositories\sp500 r&d based market portfolio\sp500_2022end.csv')
```
Delete companies added in 2022 and add the companies deleted in 2022 to reach the 2021 ending SP500 list. Then, save as a CSV file.
```
sp2022_added = changes.loc['2022']
sp2022_added = sp2022_added.iloc[:,[0,1]]
sp2022_added.columns = ['Ticker', 'Security']

sp500_2021end = pd.merge(sp500_2022end, sp2022_added, indicator=True, how='outer').query('_merge=="left_only"').drop('_merge',axis=1)

sp2022_deleted = changes.loc['2022']
sp2022_deleted = sp2022_deleted.iloc[:,[2,3]]
sp2022_deleted.columns = ['Ticker', 'Security']

sp500_2021end = pd.concat([sp500_2021end, sp2022_deleted])

sp500_2021end.to_csv(r'C:\depo\repositories\sp500 r&d based market portfolio\sp500_2021end.csv')
```
Delete companies added in 2021 and add the companies deleted in 2021 to reach the 2020 ending SP500 list. Then, save as a CSV file.
```
sp2021_added = changes.loc['2021']
sp2021_added = sp2021_added.iloc[:,[0,1]]
sp2021_added.columns = ['Ticker', 'Security']

sp500_2020end = pd.merge(sp500_2021end, sp2021_added, indicator=True, how='outer').query('_merge=="left_only"').drop('_merge',axis=1)

sp2021_deleted = changes.loc['2021']
sp2021_deleted = sp2021_deleted.iloc[:,[2,3]]
sp2021_deleted.columns = ['Ticker', 'Security']

sp500_2020end = pd.concat([sp500_2020end, sp2021_deleted])

sp500_2020end.to_csv(r'C:\depo\repositories\sp500 r&d based market portfolio\sp500_2020end.csv')
```
Delete companies added in 2020 and add the companies deleted in 2020 to reach the 2019 ending SP500 list. Then, save as a CSV file.
```
sp2020_added = changes.loc['2020']
sp2020_added = sp2020_added.iloc[:,[0,1]]
sp2020_added.columns = ['Ticker', 'Security']

sp500_2019end = pd.merge(sp500_2020end, sp2020_added, indicator=True, how='outer').query('_merge=="left_only"').drop('_merge',axis=1)

sp2020_deleted = changes.loc['2020']
sp2020_deleted = sp2020_deleted.iloc[:,[2,3]]
sp2020_deleted.columns = ['Ticker', 'Security']

sp500_2019end = pd.concat([sp500_2019end, sp2020_deleted])

sp500_2019end.to_csv(r'C:\depo\repositories\sp500 r&d based market portfolio\sp500_2019end.csv')
```
Added every year's list to the main Excel file.

# Data collection(web scraping) for the required financials.
Prepared a list for tickers with Excel.
```
import pandas as pd
import csv

tickers = pd.read_csv('C:/depo/repositories/sp500 r&d based market portfolio/tickers_for_prices_and_financials.csv')
tickers['tickers']
```
Scrape the entire cash flow statement. Used try statement because there are missing companies.
```
for ticker in tickers['tickers']:
    try:
        url = "https://www.marketwatch.com/investing/stock/" + ticker + "/financials/cash-flow"
        data = pd.read_html(url)
        df = data[5]
        file = open("C:/depo/repositories/sp500 r&d based market portfolio/scraped_data.csv", "a", newline='')
        writer = csv.writer(file)
        writer.writerow([ticker,df])
        file.close()
    except:
        pass
```
There are missing companies(the portfolio will not be an exact S&P500 portfolio).

# Found stock prices and dividends data in Kaggle which is scraped from the yahoo.finance (prices are adjusted for splits).

# Data cleaning for cash flows with MySQL.
```
CREATE DATABASE sp500_r_and_d_based_market_portfolio
```
Imported data
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN capex text,
ADD COLUMN capex2 text,
ADD COLUMN n text
```
Cleaning, splitting and merging data for the capex.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex = SUBSTRING_INDEX(SUBSTRING_INDEX(scraped,"\n0",2),"\n1",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capes2 = SUBSTRING_INDEX(SUBSTRING_INDEX(scraped,"\n0",-1),"NaN",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex = SUBSTRING_INDEX(capex,"Capital Expenditures  Capital Expenditures",-1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex = CONCAT(capex, " ", capex2)

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN capex2
```
Applied many times to decrease the number of spaces to 1.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex = REPLACE(capex, "  ", " ")
```
Cleaning, splitting and merging data for the acquisitions.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN net_assets_from_acquisitions text,
ADD COLUMN net_assets_from_acquisitions2 text

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = SUBSTRING_INDEX(SUBSTRING_INDEX(scraped,"Sale of Fixed Assets & Businesses  Sale of Fix",1),"\n",-2)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET n = SUBSTRING_INDEX(net_assets_from_acquisitions," ",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET n = CONCAT("\n", n)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2 = SUBSTRING_INDEX(SUBSTRING_INDEX(scraped,n,-1),"NaN",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = SUBSTRING_INDEX(net_assets_from_acquisitions,"Net Assets from Acquisitions  Net Assets from",-1)
```
Applied many times to decrease the number of spaces to 1.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = REPLACE(net_assets_from_acquisitions, "  ", " ")
```
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = SUBSTRING_INDEX(net_assets_from_acquisitions,"\n",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = CONCAT(net_assets_from_acquisitions, " ", net_assets_from_acquisitions2)
```
Applied many times to decrease the number of spaces to 1.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = REPLACE(net_assets_from_acquisitions, "  ", " ")
```
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN net_assets_from_acquisitions2
```
Cleaning, splitting and merging data for the sale of fixed assets and businesses.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN sale_of_fixed_assets_businesses text

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN sale_of_fixed_assets_businesses2 text

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses = SUBSTRING_INDEX(SUBSTRING_INDEX(scraped,"Purchase/Sale of Investments  Purchase/Sale of",1),"\n",-2)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET n = SUBSTRING_INDEX(sale_of_fixed_assets_businesses," ",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET n = CONCAT("\n",n)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2 = SUBSTRING_INDEX(SUBSTRING_INDEX(scraped,n,-1),"NaN",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses = SUBSTRING_INDEX(SUBSTRING_INDEX(sale_of_fixed_assets_businesses,"Sale of Fixed Assets & Businesses  Sale of Fix",-1),"\n",1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses = CONCAT(sale_of_fixed_assets_businesses, " ", sale_of_fixed_assets_businesses2)

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN sale_of_fixed_assets_businesses2
```
Applied many times to decrease the number of spaces to 1.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses = REPLACE(sale_of_fixed_assets_businesses, "  ", " ")
```
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN n
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN scraped
```
```
Create capex columns for the years and append data by splitting the capex field.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN capex2018 text

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN capex2019 text,
ADD COLUMN capex2020 text,
ADD COLUMN capex2021 text,
ADD COLUMN capex2022 text

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2018 = SUBSTRING_INDEX(capex," ",2)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2019 = SUBSTRING_INDEX(SUBSTRING_INDEX(capex," ",3)," ",-1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2020 = SUBSTRING_INDEX(SUBSTRING_INDEX(capex," ",4)," ",-1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2021 = SUBSTRING_INDEX(SUBSTRING_INDEX(capex," ",5)," ",-1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2022 = SUBSTRING_INDEX(capex," ",-2)
```
Drop capex.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN capex
```
Create acquisitions columns for the years and append data by splitting the acquisitions field.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN net_assets_from_acquisitions2018 text

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN net_assets_from_acquisitions2019 text,
ADD COLUMN net_assets_from_acquisitions2020 text,
ADD COLUMN net_assets_from_acquisitions2021 text,
ADD COLUMN net_assets_from_acquisitions2022 text

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = REPLACE(net_assets_from_acquisitions,"...","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions = REPLACE(net_assets_from_acquisitions,"  "," ")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2018 = SUBSTRING_INDEX(net_assets_from_acquisitions," ",2)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2019 = SUBSTRING_INDEX(SUBSTRING_INDEX(net_assets_from_acquisitions," ", 3), " ", -1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2020 = SUBSTRING_INDEX(SUBSTRING_INDEX(net_assets_from_acquisitions," ", 4), " ", -1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2021 = SUBSTRING_INDEX(SUBSTRING_INDEX(net_assets_from_acquisitions," ", 5), " ", -1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2022 = SUBSTRING_INDEX(net_assets_from_acquisitions," ",-2)
```
Drop acquisitions.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN net_assets_from_acquisitions
```
Create sale of fixed assets and businesses columns for the years and append data by splitting the SOFAB field.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses = REPLACE(sale_of_fixed_assets_businesses,"...","")
```
Applied many times to decrease the number of spaces to 1.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses = REPLACE(sale_of_fixed_assets_businesses,"  "," ")

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN sale_of_fixed_assets_businesses2018 text

ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
ADD COLUMN sale_of_fixed_assets_businesses2019 text,
ADD COLUMN sale_of_fixed_assets_businesses2020 text,
ADD COLUMN sale_of_fixed_assets_businesses2021 text,
ADD COLUMN sale_of_fixed_assets_businesses2022 text

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2018 = SUBSTRING_INDEX(sale_of_fixed_assets_businesses," ", 2)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2019 = SUBSTRING_INDEX(SUBSTRING_INDEX(sale_of_fixed_assets_businesses," ", 3), " ", -1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2020 = SUBSTRING_INDEX(SUBSTRING_INDEX(sale_of_fixed_assets_businesses," ", 4), " ", -1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2021 = SUBSTRING_INDEX(SUBSTRING_INDEX(sale_of_fixed_assets_businesses," ", 5), " ", -1)

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2022 = SUBSTRING_INDEX(sale_of_fixed_assets_businesses," ", -2)
```
Drop sale_of_fixed_assets_businesses.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.scraped_data
DROP COLUMN sale_of_fixed_assets_businesses
```
Cleaning.
```
UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2018 = REPLACE(capex2018," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2019 = REPLACE(capex2019," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2020 = REPLACE(capex2020," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2021 = REPLACE(capex2021," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET capex2022 = REPLACE(capex2022," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2018 = REPLACE(net_assets_from_acquisitions2018," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2019 = REPLACE(net_assets_from_acquisitions2019," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2020 = REPLACE(net_assets_from_acquisitions2020," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2021 = REPLACE(net_assets_from_acquisitions2021," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET net_assets_from_acquisitions2022 = REPLACE(net_assets_from_acquisitions2022," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2018 = REPLACE(sale_of_fixed_assets_businesses2018," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2019 = REPLACE(sale_of_fixed_assets_businesses2019," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2020 = REPLACE(sale_of_fixed_assets_businesses2020," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2021 = REPLACE(sale_of_fixed_assets_businesses2021," ","")

UPDATE sp500_r_and_d_based_market_portfolio.scraped_data
SET sale_of_fixed_assets_businesses2022 = REPLACE(sale_of_fixed_assets_businesses2022," ","")
```
Exported as a CSV file.

# Excel
Prepared 2019, 2020, 2021 and 2022 portfolio pages(added tickers, stock names, and headers of the required info)
<br>Prepared financial_data sheet to hold all the data in one sheet.
<br>For financials_cleaned
```
	added to main file
	text to columns
	replace all "-" with "0"
	replace all "(" with "-"
	replace all ")" with ""
	replace all "K" with " K"
	replace all "M" with " M"
	replace all "B" with " B"
	text to column all columns with space seperator
	replace all " " with ""
	replace all "K" with "1000"
	replace all "M" with "1000000"
	replace all "B" with "1000000000"
	created columns for yearly financials by multiplying seperated columns
	moved to financial_data
```
Calculated investments (capex + net assets from acquisitions + sale of fixed assets&businesses).
<br>Deleted financials_cleaned from the main file.

# Collected required prices and dividends from +500 different CSV files with Python.
```
import pandas as pd
import datetime as dt
import csv
```
Read tickers again.
```
df = pd.read_csv('C:/depo/repositories/sp500 r&d based market portfolio/tickers_for_prices_and_dividends.csv')
```
Filtered data with the date column and dates. Then, appended to a CSV file.
```
tickers = df['tickers']
for ticker in tickers:
    try:
        file = "C:/depo/repositories/sp500 r&d based market portfolio/stock prices/" + ticker + ".csv"
        ticker_file = pd.read_csv(file)
        last_day_2018 = ticker_file[(ticker_file['Date'] > '2018-12-01 00:00:00') & (ticker_file['Date'] < '2018-12-31 23:59:59')]
        last_day_2018 = last_day_2018.iloc[-1]
        last_day_2019 = ticker_file[(ticker_file['Date'] > '2019-12-01 00:00:00') & (ticker_file['Date'] < '2019-12-31 23:59:59')]
        last_day_2019 = last_day_2019.iloc[-1]
        last_day_2020 = ticker_file[(ticker_file['Date'] > '2020-12-01 00:00:00') & (ticker_file['Date'] < '2020-12-31 23:59:59')]
        last_day_2020 = last_day_2020.iloc[-1]
        last_day_2021 = ticker_file[(ticker_file['Date'] > '2021-12-01 00:00:00') & (ticker_file['Date'] < '2021-12-31 23:59:59')]
        last_day_2021 = last_day_2021.iloc[-1]
        last_day_2022 = ticker_file[(ticker_file['Date'] > '2022-12-01 00:00:00') & (ticker_file['Date'] < '2022-12-31 23:59:59')]
        last_day_2022 = last_day_2022.iloc[-1]

        days2018 = ticker_file[(ticker_file['Date'] > '2018-01-01 00:00:00') & (ticker_file['Date'] < '2018-12-31 23:59:59')]
        dividends2018 = days2018['Dividends'].sum()
        days2019 = ticker_file[(ticker_file['Date'] > '2019-01-01 00:00:00') & (ticker_file['Date'] < '2019-12-31 23:59:59')]
        dividends2019 = days2019['Dividends'].sum()
        days2020 = ticker_file[(ticker_file['Date'] > '2020-01-01 00:00:00') & (ticker_file['Date'] < '2020-12-31 23:59:59')]
        dividends2020 = days2020['Dividends'].sum()
        days2021 = ticker_file[(ticker_file['Date'] > '2021-01-01 00:00:00') & (ticker_file['Date'] < '2021-12-31 23:59:59')]
        dividends2021 = days2021['Dividends'].sum()
        days2022 = ticker_file[(ticker_file['Date'] > '2022-01-01 00:00:00') & (ticker_file['Date'] < '2022-12-31 23:59:59')]
        dividends2022 = days2022['Dividends'].sum()

        prices_mined = open("C:/depo/repositories/sp500 r&d based market portfolio/prices_mined.csv", "a", newline='')
        writer = csv.writer(prices_mined)
        
        writer.writerow([
            ticker,
            last_day_2018, 
            last_day_2019, 
            last_day_2020, 
            last_day_2021, 
            last_day_2022, 
            dividends2018, 
            dividends2019, 
            dividends2020, 
            dividends2021, 
            dividends2022,
            ])
        
        file.close()
        
    except:
        pass
```
# Excel
For the prices and dividends.
```
	replace all *Close with ""
	replace all Volume* with ""
	replace all " " with ""
	removed newlines
		replace all "ctrl+j" with ""
```
Ddded dividends from prices_mined to financial_data.
```
	=XLOOKUP(A490,prices_mined!$A$2:$A$486,prices_mined!$G$2:$G$486)
	=XLOOKUP(A490,prices_mined!$A$2:$A$486,prices_mined!$H$2:$H$486)
	=XLOOKUP(A490,prices_mined!$A$2:$A$486,prices_mined!$I$2:$I$486)
	=XLOOKUP(A490,prices_mined!$A$2:$A$486,prices_mined!$J$2:$J$486)
	=XLOOKUP(A490,prices_mined!$A$2:$A$486,prices_mined!$K$2:$K$486)
```
Added prices with xlookup.
<br>Deleted prices_mined.

# Web scraping for market caps.
Company page URLs do not contain tickers. Get URLs from the dropdown list after typing the ticker to search bar with the selenium.
```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
import pandas as pd
import csv

driver = webdriver.Chrome()

driver.get("https://companiesmarketcap.com/")
search_box = driver.find_element(By.XPATH, '//*[@id="search-input"]')

tickers = pd.read_csv('C:/depo/repositories/sp500 r&d based market portfolio/tickers.csv')
for ticker in tickers['tickers']:
    try:
        search_box.clear()
        driver.implicitly_wait(5)
        search_box.send_keys(ticker)
        driver.implicitly_wait(5)
        
        company_url = driver.find_element(By.XPATH,'//*[@id="typeahead-search-results"]/a')
        company_url = company_url.get_attribute('href')

        file = open("C:/depo/repositories/sp500 r&d based market portfolio/market_caps/urls.csv", "a", newline='')
        writer = csv.writer(file)
        writer.writerow([ticker, company_url])
        file.close()

    except:
        pass
```
Found duplicates with conditional formatting and corrected wrong URLs.

# Web scraping every company's market caps to different CSV files.
```
import scrapy
import pandas as pd
import csv

class MarketcapspiderSpider(scrapy.Spider):
    name = "marketcapspider"
    allowed_domains = ["companiesmarketcap.com"]
    start_urls = ["https://companiesmarketcap.com/"]



    def parse(self, response):
        urls = pd.read_csv("C:/depo/repositories/sp500 r&d based market portfolio/market_caps/urls.csv")
        for url in urls['url']:
            yield response.follow(url, callback= self.parse_company_page)
            
    def parse_company_page(self, response):
        ticker = response.css('div[class="company-title-container"] div[class="company-code"]::text').getall()
        
        file_path = "C:/depo/repositories/sp500 r&d based market portfolio/market_caps/" + ticker[0] + ".csv"
        
        trs = response.css('*[class="table"] tbody tr')
        
        for tr in trs:
            row = tr.css('td::text').getall()

            file = open(file_path, "a", newline='')
            writer = csv.writer(file)
            writer.writerow([row])
            file.close()
```
# Prepare market caps (find relevant information from +500 CSV files) with Python.
```
import pandas as pd
import csv
import re

market_values = []
df = pd.DataFrame(market_values)

tickers = pd.read_csv("C:/depo/repositories/sp500 r&d based market portfolio/market_caps/tickers_urls.csv")
for ticker in tickers['ticker']:    
    try :
        file_path = "C:/depo/repositories/sp500 r&d based market portfolio/market_caps/" + ticker + ".csv"
        df2 = pd.read_csv(file_path, header=None)
        
        df2.columns = ['data']
        df2[['year', 'market_cap', 'change']] = df2['data'].str.split(',', expand=True)
        df2 = df2.drop(['data'], axis=1)
        df2 = df2.drop(['change'], axis=1)
        df2['year'] = df2['year'].replace("[\',)]", '',  regex = True)
        df2['year'] = df2['year'].replace("[\[,)]", '',  regex = True)
        df2['year'] = df2['year'].replace("[\ ,)]", '',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace('[\$,)]', '',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("[\',)]", '',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("[\],)]", '',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("[\ ,)]", '',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("K", ' K',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("M", ' M',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("B", ' B',  regex = True)
        df2['market_cap'] = df2['market_cap'].replace("T", ' T',  regex = True)
        df2[['a', 'b']] = df2['market_cap'].str.split(' ', expand=True)
        df2['b'] = df2['b'].replace("K", '1000',  regex = True)
        df2['b'] = df2['b'].replace("M", '1000000',  regex = True)
        df2['b'] = df2['b'].replace("B", '1000000000',  regex = True)
        df2['b'] = df2['b'].replace("T", '1000000000000',  regex = True)
        df2['market_cap'] = (df2['a'].astype(float)) * (df2['b'].astype(float))
        df2 = df2.drop(['a'], axis=1)
        df2 = df2.drop(['b'], axis=1)

        mv2018_ = df2[df2['year'] == '2018']
        mv2018 = mv2018_.drop(['year'], axis=1)

        mv2019_ = df2[df2['year'] == '2019']
        mv2019 = mv2019_.drop(['year'], axis=1)

        mv2020_ = df2[df2['year'] == '2020']
        mv2020 = mv2020_.drop(['year'], axis=1)

        mv2021_ = df2[df2['year'] == '2021']
        mv2021 = mv2021_.drop(['year'], axis=1)

        mv2022_ = df2[df2['year'] == '2022']
        mv2022 = mv2022_.drop(['year'], axis=1)

        mv2023_ = df2[df2['year'] == '2023']
        mv2023 = mv2023_.drop(['year'], axis=1)

        file = open("C:/depo/repositories/sp500 r&d based market portfolio/market_values.csv", "a", newline='')
        writer = csv.writer(file)
        writer.writerow([ticker,mv2018,mv2019,mv2020,mv2021,mv2022,mv2023])
        file.close()
    except:
        pass
```

# Clean the market caps with MySQL.
First row was header, add header as a row
```
INSERT INTO sp500_r_and_d_based_market_portfolio.market_values
(MMM, `market_cap
5  1.098600e+11`, `market_cap
4  1.014700e+11`, `market_cap
3  1.008200e+11`, `market_cap
2  1.015700e+11`, `market_cap
1  6.628000e+10`, `market_cap
0  5.903000e+10`)
VALUES("MMM", "market_cap
5  1.098600e+11", "market_cap
4  1.014700e+11", "market_cap
3  1.008200e+11", "market_cap
2  1.015700e+11", "market_cap
1  6.628000e+10", "market_cap
0  5.903000e+10")
```
Rename the columns.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.market_values
RENAME COLUMN `MMM` to ticker

ALTER TABLE sp500_r_and_d_based_market_portfolio.market_values
RENAME COLUMN `market_cap
5  1.098600e+11` to `2018`,
RENAME COLUMN `market_cap
4  1.014700e+11` to `2019`,
RENAME COLUMN `market_cap
3  1.008200e+11` to `2020`,
RENAME COLUMN  `market_cap
2  1.015700e+11` to `2021`,
RENAME COLUMN  `market_cap
1  6.628000e+10` to `2022`,
RENAME COLUMN  `market_cap
0  5.903000e+10` to `2023`
```
Delete spaces.
```
UPDATE sp500_r_and_d_based_market_portfolio.market_values
SET 
	`2018`=  REPLACE(`2018`, ' ', ''), 
    `2019`=  REPLACE(`2019`, ' ', ''), 
    `2020`=  REPLACE(`2020`, ' ', ''), 
    `2021`=  REPLACE(`2021`, ' ', ''), 
    `2022`=  REPLACE(`2022`, ' ', '')
```
Delete new lines.
```
UPDATE sp500_r_and_d_based_market_portfolio.market_values
SET 
	`2018`=  REPLACE(`2018`, '\n', ''), 
    `2019`=  REPLACE(`2019`, '\n', ''), 
    `2020`=  REPLACE(`2020`, '\n', ''), 
    `2021`=  REPLACE(`2021`, '\n', ''), 
    `2022`=  REPLACE(`2022`, '\n', '')    
```
Delete 'market_cap'.
```
update sp500_r_and_d_based_market_portfolio.market_values
set 
	`2018`=  REPLACE(`2018`, 'market_cap', ''), 
    `2019`=  REPLACE(`2019`, 'market_cap', ''), 
    `2020`=  REPLACE(`2020`, 'market_cap', ''), 
    `2021`=  REPLACE(`2021`, 'market_cap', ''), 
    `2022`=  REPLACE(`2022`, 'market_cap', '')
```
For 2023 column.
```
UPDATE sp500_r_and_d_based_market_portfolio.market_values
SET `2023`=  REPLACE(`2023`, 'market_cap', '')  

UPDATE sp500_r_and_d_based_market_portfolio.market_values
SET `2023`=  REPLACE(`2023`, '\n', '') 

UPDATE sp500_r_and_d_based_market_portfolio.market_values
SET `2023`=  REPLACE(`2023`, ' ', '')
```
For empty cells that is not empty.
```
UPDATE sp500_r_and_d_based_market_portfolio.market_values
SET
	`2018`=  replace(`2018`, 'EmptyDataFrameColumns:[]Index:[]', '0'), 
    `2019`=  replace(`2019`, 'EmptyDataFrameColumns:[]Index:[]', '0'), 
    `2020`=  replace(`2020`, 'EmptyDataFrameColumns:[]Index:[]', '0'), 
    `2021`=  replace(`2021`, 'EmptyDataFrameColumns:[]Index:[]', '0'), 
    `2022`=  replace(`2022`, 'EmptyDataFrameColumns:[]Index:[]', '0'),
    `2023`=  replace(`2023`, 'EmptyDataFrameColumns:[]Index:[]', '0')
```
Change data type to integer.
```
ALTER TABLE sp500_r_and_d_based_market_portfolio.market_values
MODIFY COLUMN `2018` bigint,
MODIFY COLUMN `2019` bigint,
MODIFY COLUMN `2020` bigint,
MODIFY COLUMN `2021` bigint,
MODIFY COLUMN `2022` bigint,
MODIFY COLUMN `2023` bigint
```
Exported as a CSV file.

# Finish the model.
Text to column market_caps_cleaned
<br>Added to financial_data sheet
```
	=XLOOKUP($A3,market_caps_cleaned!$A$2:$A$545,market_caps_cleaned!G$2:G$545)
```
market_caps_cleaned deleted
<br>For 2019, 2020 and 2021.
```
	Retrieved inputs from financial_data sheet with xlookup.
	Removed formulas with copy pasting as value.
	Removed companies with missing data.
	Calculated returns.
```

# Analysis.
Created tables for yearly returns of each portfolio.
<br>Calculated correlations between stock returns and portfolio weights.
<br>Created a scatter graph.
<br>Analyzed regression.
# Results, presentations etc.
Created a graph for the yearly portfolio returns.
<br>Graph for correlations.
<br>Powerpoint slide.
<br>PDF with powerpoint.
<br>Expectations
```
    Both are passive market portfolios that aims for low risk and low effort.
    Managers have private information. By investing based on capex etc., we invest based on publicly available information + every company's own private information.
    Returns should be higher in the long-term.
    It should perform better in downturns(we will invest in companies that have opportunities in bad times)
    Returns of investments will be different for every company. We did not take this into account.
    Companies must earn or find money to be able to invest in new projects. Therefore, investors will have an effect on managers's decisions and capex vs. based portfolio.
```
Results
```
    In the worst year, capex vs based(2018-2021) performed better
    Both previous capex vs. and problematic year's opportunistic investments performed well
    The market may needs time to value companies' investments
    Firms with opportunistic investments perform better in problematic years
    Capex vs based(2018-2021) performed better in the investment horizon
```
END
```
    Companies's investments bring returns. Returns bring higher market cap(The market waits for good signs).
    Capex vs. based portfolios act faster than market cap-based portfolios.
    If the level of risk is not higher, capex vs. based portfolio is better than the market cap-based portfolio. (We can look for deleted companies from the yearly lists to understand whether survivor bias is present.)
```
