import altair as alt
import numpy as np
import pandas as pd
import streamlit as st

"""
# Welcome to My Travel!

This app is designed to provide personalized recommendations for hotels, flights,
and restaurants, ensuring a tailored and delightful travel experience tailored to
your preferences

"""

# Function to make API requests
import requests
from datetime import datetime


#Initialize Streamlit App
st.title('Streamlit Travel App')
import json

# Variables to get from the user
initial_budget = st.number_input("Enter your initial budget: $")
preferred_city = st.text_input("Enter your favorite city:")
departure_city = st.text_input("Enter your departure city:")
arrival_date = st.date_input("Enter the arrival date:")
departure_date = st.date_input("Enter the departure date:")
adult_number = st.number_input("Enter the number of adults traveling:", min_value=1, step=1)
senior_number = st.number_input("Enter the number of seniors traveling:", min_value=0, step=1)
children_number = st.number_input("Enter the number of children traveling:", min_value=1, step=1)
children_age = st.text_input("Enter the age of children:")

number_people = adult_number + children_number

# Hotels
# First API: Get Destination ID
headers = {
    "X-RapidAPI-Key": "1cb0a66893msh213a4c5c6e5fcebp12b90bjsnae7b7c96396e",
    "X-RapidAPI-Host": "booking-com15.p.rapidapi.com"
}

url = "https://booking-com15.p.rapidapi.com/api/v1/hotels/searchDestination"

querystring = {"query": preferred_city}

response = requests.get(url, headers=headers, params=querystring)

data = response.json()
dest_id = data["data"][0]["dest_id"]

# Second API: Get Hotels based on Destination ID
url = "https://booking-com15.p.rapidapi.com/api/v1/hotels/searchHotels"
querystring = {
    "dest_id": dest_id,
    "search_type": "city",
    "arrival_date": arrival_date.strftime("%Y-%m-%d"),
    "departure_date": departure_date.strftime("%Y-%m-%d"),
    "adults": adult_number,
    "children_age": children_age,
    "room_qty": "1",
    "page_number": "1",
    "languagecode": "en-us",
    "currency_code": "AED"
}
response = requests.get(url, headers=headers, params=querystring)
data_1 = response.json()

st.title("Hotel IDs")


# Retrieve the hotel id 
hotels = data_1["data"]["hotels"]

hotel_ids = []

for hotel in hotels:
    hotel_id = hotel.get("hotel_id")
    if hotel_id is not None:
        hotel_ids.append(hotel_id)

# create a suggestion based on a random hotel id
import random

random_hotel_id = random.choice(hotel_ids)
if st.button("Show Suggestion"):
    random_hotel_id = random.choice(hotel_ids)


# Get hotel details

#def get_hotel_details(random_hotel_id, arrival_date, departure_date, adult_number, children_age):
url = "https://booking-com15.p.rapidapi.com/api/v1/hotels/getHotelDetails"
querystring = {"hotel_id": random_hotel_id,
               "arrival_date": arrival_date,
               "departure_date": departure_date,
               "adults": adult_number,
               "children_age": children_age,
               "room_qty": "1",
               "languagecode": "en-us",
               "currency_code": "EUR"}
headers = {

    "X-RapidAPI-Key": "1cb0a66893msh213a4c5c6e5fcebp12b90bjsnae7b7c96396e",
    "X-RapidAPI-Host": "booking-com15.p.rapidapi.com",
    }
    
response = requests.get(url, headers=headers, params=querystring)
hotel_data = response.json()

st.title("Hotel Details")

# hotel info

hotel_name = hotel_data["data"]["hotel_name"]
gross_amount = hotel_data["data"]["composite_price_breakdown"]["gross_amount"]["amount_rounded"]
breakfast_review = hotel_data["data"]["breakfast_review_score"]["review_score"]

facilities_block = hotel_data["data"]["facilities_block"]["facilities"]
facility_names = [facilities.get("name") for facilities in facilities_block if facilities.get("name") is not None]

st.write("Here is some information about this hotel:")
st.write(f"Hotel's name: {hotel_name}")
st.write(f"Total stay amount: {gross_amount}")

if breakfast_review == 0:
    st.write("This hotel has no breakfast reviews")
else:
    st.write(f"Breakfast review score: {breakfast_review}")

st.write("Hotel facilities:")
for facility in facility_names:
    st.write(f"- {facility}")

# Recommendations of popular places

url = "https://booking-com15.p.rapidapi.com/api/v1/hotels/getPopularAttractionNearBy"
querystring = {"hotel_id": hotel_id, "languagecode":"en-us"}
headers = {"X-RapidAPI-Key": "1cb0a66893msh213a4c5c6e5fcebp12b90bjsnae7b7c96396e",
           "X-RapidAPI-Host": "booking-com15.p.rapidapi.com"}

response = requests.get(url, headers=headers, params=querystring)
recommendation = response.json()

# Popular places recommendation

popular_places = recommendation["data"]["popular_landmarks"]
places_names = []
for popular_landmarks in popular_places:
    popular_place = popular_landmarks.get("tag")
    if popular_place is not None:
        places_names.append(popular_place)

if places_names == []:
    st.write ("There is no recommendation for popular places")
else:
    popular_places_random = random.choice(places_names, k=3)
    st.write(f"Popular places to visit in the city: {popular_places_random}")

# Flights

st.title("Flights Info")

# Search preferred city airport
url = "https://tripadvisor16.p.rapidapi.com/api/v1/flights/searchAirport"
querystring = {"query": preferred_city}
headers = {
	"X-RapidAPI-Key": "1cb0a66893msh213a4c5c6e5fcebp12b90bjsnae7b7c96396e",
	"X-RapidAPI-Host": "tripadvisor16.p.rapidapi.com"
}

response = requests.get(url, headers=headers, params=querystring)
search_airport_1 = response.json()

# Extract airport names
airports = search_airport_1["data"]
airport_names_1 = []

for data in airports:
    airport_name = data.get("name")
    if airport_name is not None:
        airport_names_1.append(airport_name)

destination_airport = st.selectbox("Select destination airport:", airport_names_1)

# Search departure city airport
url = "https://tripadvisor16.p.rapidapi.com/api/v1/flights/searchAirport"
querystring = {"query": departure_city}

headers = {
	"X-RapidAPI-Key": "1cb0a66893msh213a4c5c6e5fcebp12b90bjsnae7b7c96396e",
	"X-RapidAPI-Host": "tripadvisor16.p.rapidapi.com"
}

response = requests.get(url, headers=headers, params=querystring)
search_airport_2 = response.json()

# Extract airport names
airports = search_airport_2["data"]
airport_names_2 = []

for data in airports:
    airport_name = data.get("name")
    if airport_name is not None:
        airport_names_2.append(airport_name)

departure_airport = st.selectbox("Select destination airport:", airport_names_2)

# Itinerary Type
itinerary_options = {"1": "ONE_WAY", "2": "ROUND_TRIP"}
selected_itinerary = st.radio("Choose an itinerary type:", list(itinerary_options.values()))

# Cabin Class
cabin_class_options = {"1": "ECONOMY", "2": "PREMIUM_ECONOMY", "3": "BUSINESS", "4": "FIRST"}
selected_cabin_class = st.radio("Choose a cabin class:", list(cabin_class_options.values()))

# Sort Order
sort_options = {
    "1": "DURATION",
    "2": "PRICE",
    "3": "EARLIEST_OUTBOUND_ARRIVAL",
    "4": "EARLIEST_OUTBOUND_DEPARTURE",
    "5": "LATEST_OUTBOUND_ARRIVAL",
    "6": "LATEST_OUTBOUND_DEPARTURE",
}
selected_sort_option = st.radio("Choose a way to sort the flights:", list(sort_options.values()))

# Search flights

url = "https://tripadvisor16.p.rapidapi.com/api/v1/flights/searchFlights"

querystring = {"sourceAirportCode":departure_airport,
               "destinationAirportCode": destination_airport,
               "date": arrival_date,
               "itineraryType": selected_itinerary,
               "sortOrder": selected_sort_option,
               "numAdults": number_people,
               "numSeniors": senior_number,
               "classOfService": selected_cabin_class,
               "returnDate": departure_date,
               "pageNumber": "1",
               "nonstop": "yes",}

headers = {
	"X-RapidAPI-Key": "1cb0a66893msh213a4c5c6e5fcebp12b90bjsnae7b7c96396e",
	"X-RapidAPI-Host": "tripadvisor16.p.rapidapi.com"
}

response = requests.get(url, headers=headers, params=querystring)
flights_data = response.json()

flight_info = flights_data[0]["segments"][0]["legs"][0]

# Departure date time 
date_time_departure = flight_info["departureDateTime"]
date_time_1 = datetime.strptime(date_time_departure, "%Y-%m-%dT%H:%M:%SZ")
formatted_date_time_1 = date_time_1.strftime("%A, %B %d, %Y %I:%M %p")
st.write("Departure date time:", formatted_date_time_1)

# Arrival date time
date_time_arrival = flight_info["arrivalDateTime"]
date_time_2 = datetime.strptime(date_time_arrival, "%Y-%m-%dT%H:%M:%S%z")
formatted_date_time_2 = date_time_2.strftime("%A, %B %d, %Y %I:%M %p %Z")
st.write("Arrival date time:", formatted_date_time_2)

# Flight duration
departure_dt = date_time_1
arrival_dt = date_time_2
diff_seconds = (arrival_dt - departure_dt).total_seconds()
hours = diff_seconds // 3600
minutes = (diff_seconds % 3600) // 60
st.write(f"The flight lasts for {int(hours)} hours and {int(minutes)} minutes.")

# Flight number
flight_number = flight_info["flightNumber"]
st.write("Flight Number:", flight_number)

# Airplane model
airplane_model = flight_info["equipmentId"]
st.write("Airplane Model:", airplane_model)

# Price per person
flight_price = flights_data[0]["purchaseLinks"][0]["totalPricePerPassenger"]
st.write("Price per Person:", flight_price)

# Purchase link
flight_link = flights_data[0]["purchaseLinks"][0]["url"]
st.write("Purchase Link:", flight_link)

# Operating company logo
airline_logo_url = flight_info["operatingCarrier"]["logoUrl"]
st.image(airline_logo_url, caption="Operating Company Logo", use_column_width=True)


class BudgetTracker:
    def __init__(self, initial_budget):
        self.budget = initial_budget

    def subtract_expense(self, expense):
        if self.budget >= expense:
            self.budget -= expense
            return True
        else:
            return False


st.title('Travel Budget Tracker')

# Create a BudgetTracker instance
budget_tracker = BudgetTracker(initial_budget)

flight_price_total = flight_price * (number_people + senior_number)

# Subtract hotel expenses from the budget
if budget_tracker.subtract_expense(gross_amount):
    st.write(f"Hotel booked! Remaining budget after hotel: ${budget_tracker.budget}")
    if budget_tracker.subtract_expense(flight_price_total):
        st.write(f"Flight booked! Remaining budget after flights: ${budget_tracker.budget}")
    else:
        st.warning("Sorry, it's out of budget for flights. Please choose another flight.")
else:
    st.warning("Sorry, it's out of budget for the hotel. Please choose another hotel.")
