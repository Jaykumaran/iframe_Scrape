# iframe_Scrape


# Run these in terminal
# pip install selenium
# pip install bs4

#This is a scrape of a transit/courier website

import os
from selenium import webdriver
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import xml.etree.ElementTree as ET

# Replace 'chromedriver_path' with the actual path to your Chrome WebDriver executable
#Go to chrome options help to find the chromedriver version, dwnld the sepcific version without fail to avoid incompatibilty errors
# Add the Chrome WebDriver directory to the system PATH variable
chrome_driver_path = 'C:/Users/jaikr/Downloads/chromedriver_win32'
os.environ['PATH'] += ';' + chrome_driver_path

# URL to directly open the specific page
url = "url here"
#replace the doc no. for other AWB No.
driver = webdriver.Chrome()
# Open the specific URL directly
driver.get(url)

# Switch to the iframe
iframe_locator = driver.find_element(By.TAG_NAME, 'iframe')
driver.switch_to.frame(iframe_locator)

# Get the HTML content inside the iframe
html_content = driver.page_source

# Switch back to the default content (out of the iframe)
driver.switch_to.default_content()

# Clean up
driver.quit()

# Now you have the HTML content inside the iframe stored in the 'html_content' variable
# You can use BeautifulSoup to parse it as shown in the previous response





# Replace 'html_content' with the actual HTML content inside the iframe


# Parse the iframe content using BeautifulSoup
soup = BeautifulSoup(html_content, 'html.parser')

# Extract the booking details
booking_table = soup.find('table', class_='style11')   #replace class name
booking_details = {}
if booking_table:
    rows = booking_table.find_all('tr')
    for row in rows:
        cols = row.find_all('td')
        if len(cols) >= 2:
            label = cols[0].text.strip()
            value = cols[1].text.strip()
            booking_details[label] = value

# Extract the tracking details
tracking_table = soup.find('table', id='tblTrack')  #replace id
tracking_data = []
if tracking_table:
    rows = tracking_table.find_all('tr')
    for row in rows[1:]:  # Skip the first row with column headers
        cols = row.find_all('td')
        if len(cols) >= 3:
            job = cols[0].text.strip()
            date = cols[1].text.strip()
            transit_route = cols[2].text.strip()
            tracking_data.append((job, date, transit_route))

print("Booking Details:")
for label, value in booking_details.items():
    print(f"{label}: {value}")

print("\nTracking Details:")
for data in tracking_data:
    print(f"Job: {data[0]}, Date: {data[1]}, Transit Route: {data[2]}")


# Create an XML structure for booking details
booking_xml = ET.Element('BookingDetails')
for label, value in booking_details.items():
    entry = ET.SubElement(booking_xml, label.replace(' ', '_'))
    entry.text = value

# Create an XML structure for tracking details
tracking_xml = ET.Element('TrackingDetails')
for data in tracking_data:
    entry = ET.SubElement(tracking_xml, 'Entry')
    ET.SubElement(entry, 'Job').text = data[0]
    ET.SubElement(entry, 'Date').text = data[1]
    ET.SubElement(entry, 'TransitRoute').text = data[2]

# Save the XML data to a file
booking_xml_tree = ET.ElementTree(booking_xml)
tracking_xml_tree = ET.ElementTree(tracking_xml)

booking_xml_tree.write('booking_details.xml', encoding='utf-8', xml_declaration=True)
tracking_xml_tree.write('tracking_details.xml', encoding='utf-8', xml_declaration=True)
