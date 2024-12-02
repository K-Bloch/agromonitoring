# Agromonitoring: Crop Field Monitoring with Power Automate and Power Apps

This is a personal learning project of mine, though it could easily be adapted into a real-world application. While exploring open API data to experiment with, I came across the Agromonitoring API through the Open Weather platform. The Agromonitoring website allows users to create polygons representing their agricultural fields of interest. Using the API, one can access real-time weather and soil data, along with satellite imagery for specific fields, enabling efficient field monitoring. I thought this was an amazing idea and decided to use it for something fun and practical.

## Project Overview

The goal of this project is to create an app for farmers that allows them to monitor the conditions of their fields in real-time. While the API provides free access to current data, historical data requires a paid plan. To make the project cost-effective, I designed a system that collects and stores daily data in Dataverse tables, which are then connected to a Power Apps interface.

For testing, I randomly selected two locations in Slovakia, naming the polygons after the villages that lay nearby.

---

## Step 1: Inspecting the APIs and Creating Polygons

You can explore the Agromonitoring API documentation here: [Agromonitoring API](https://agromonitoring.com/api).

Initially, I considered using a simple weather API, but I wanted a project that would challenge me a bit. A farmer with multiple fields could benefit greatly from an app like this. On the Agromonitoring website, I used their polygon tool to create two sample fields. Each polygon is assigned a unique ID, which can be managed using the Polygon API.

For this project, I utilized the following APIs:
- **Polygon API**
- **Satellite Imagery API**
- **Current Weather API**
- **Current Soil API**

After signing up and obtaining an API key, I was ready to dive in.

---

## Step 2: Automating Data Collection with Power Automate

To ensure scalability, the data collection process follows this workflow:
1. A farmer creates polygons using the map on the dashboard.
2. Three Power Automate flows are triggered to save data into Dataverse tables.
3. The tables are connected to a Power Apps app, allowing the farmer to view real-time conditions and satellite imagery.

### Flow 1: Daily Retrieve Polygons
This flow, triggered daily, retrieves all polygons associated with a user (identified by their unique API key). It parses the API's JSON output and checks the 'terrain' table to avoid saving duplicate polygons. Using a loop, the flow iterates through each polygon ID:
- If the ID already exists, the iteration ends.
- If the ID is new, the data is saved into the table.

To handle failures, I used a **Try-Catch** block. However, the current implementation only logs that the Try block failed without pinpointing the specific action. Improving this error handling is on my to-do list.

### Flow 2: Daily Retrieve Conditions
This flow collects weather and soil data for each polygon listed in the 'terrain' table. It uses two parallel branches: 
- One retrieves soil data using the polygon ID.
- The other retrieves weather data using latitude and longitude.

Data from both branches is merged into a single variable and saved into the 'conditions' table after each iteration. This approach avoids the complications of updating rows later due to Dataverse's rigid column and key constraints.

### Flow 3: Weekly Retrieve Images
Satellite imagery is updated less frequently than weather data, so this flow runs weekly. It retrieves images taken in the past 7 days for each polygon. The process involves:
1. Requesting an API call for each polygon.
2. Using the API call to fetch the actual image as a `.png` file.
3. Saving the image, satellite type, and cloud coverage percentage in the 'photos' table.

---

## Step 3: Building the Power Apps Interface

The app provides a simple interface where users can select a field and view either weather data or the latest satellite image. Here's how it works:
1. **Field Selection**: Users choose their field from a list.
2. **Navigation**: Buttons direct users to either the weather or photo details screen.
3. **Display Data**: The app dynamically retrieves and displays the latest conditions or imagery for the selected field.

### Observations
- **Image Quality**: The satellite images are low resolution—great for monitoring overall field conditions but not detailed enough for granular analysis.
- **Technical Challenges**: Power Automate proved easier to work with than Power Apps, which can feel overwhelming. Issues like rigid table key settings added to the complexity, but these were manageable with persistence.

---

## Reflections

This project was a nice challenge, combining API integrations, automated workflows, and app development. While it’s far from perfect, the concept demonstrates significant potential for real-world use. The most frustrating part? Dataverse's insistence on rigidity with table keys and not being able to change column types—an area I hope they’ll improve in the future.
