# 🌌 ISS Overhead Notifier
This Python script checks if the International Space Station (ISS) is overhead at your location during nighttime and sends an email notification when both conditions are met.

## 🛠️ Requirements
- Python Libraries: Make sure to install the following libraries:
```bash
pip install requests python-dotenv
```
- **Environment Variables**: Sensitive information like your email and password are loaded from a .env file for security.

## 📂 Project Structure
- **isISSOverhead**: Checks if the ISS's current location is within ±5 degrees of latitude and longitude of your specified location.
- **isNight**: Determines if it's currently nighttime based on sunrise and sunset times for your location.
- **Email Notification**: Sends an email to your address when the ISS is overhead and it’s nighttime.
---
## 🔧 Code Explanation
### 1. Setup and Imports
```python
import requests
from datetime import datetime
import smtplib
import os
from dotenv import load_dotenv
import time

load_dotenv()

myEmail = os.getenv("YOUR_EMAIL")
password = os.getenv("YOUR_PASSWORD")

MY_LAT = 51.507351 # Your latitude
MY_LONG = -0.127758 # Your longitude
```
- load_dotenv: Loads environment variables from a .env file.
- MY_LAT and MY_LONG: Replace with your latitude and longitude.

### 2. Check if ISS is Overhead
```python
def isISSOverhead():
    response = requests.get(url="http://api.open-notify.org/iss-now.json")
    response.raise_for_status()
    data = response.json()

    iss_latitude = float(data["iss_position"]["latitude"])
    iss_longitude = float(data["iss_position"]["longitude"])

    if MY_LAT-5 <= iss_latitude <= MY_LAT + 5 and MY_LONG - 5 <= iss_latitude <= MY_LONG + 5:
        return True
```
- isISSOverhead: Fetches the current ISS position and returns True if it is within ±5 degrees of your location.

### 3. Check if it’s Nighttime
```python
def isNight():
    parameters = {
        "lat": MY_LAT,
        "lng": MY_LONG,
        "formatted": 0,
    }

    response = requests.get("https://api.sunrise-sunset.org/json", params=parameters)
    response.raise_for_status()
    data = response.json()
    sunrise = int(data["results"]["sunrise"].split("T")[1].split(":")[0])
    sunset = int(data["results"]["sunset"].split("T")[1].split(":")[0])

    time_now = datetime.now().hour
    if time_now >= sunset or time_now <= sunrise:
        return True
```
- isNight: Checks if the current hour is past sunset or before sunrise to determine if it’s night.

### 4. Main Loop and Notification
```python
while True:
    time.sleep(60)
    if isISSOverhead() and isNight():
        connection = smtplib.SMTP("smtp.gmail.com")
        connection.starttls()
        connection.login(myEmail, password)
        connection.sendmail(
            from_addr=myEmail, 
            to_addrs=myEmail, 
            msg="Subject:Look Up👆\n\nThe ISS is above you in the sky."
        )
```
- Every 60 seconds, the program checks if the ISS is overhead and it’s nighttime. If both conditions are met, an email notification is sent.
---
## 📄 .env File Setup
Create a .env file in the same directory as the script with the following content:
```bash
YOUR_EMAIL=your_email@example.com
YOUR_PASSWORD=your_email_password
```
⚠️ Note: Make sure your email provider allows SMTP access, and enable "less secure app access" if required (for testing purposes only).
---
## 🚀 How to Run
1. Install required libraries:
```bash
pip install requests python-dotenv
```
2. Set your latitude, longitude, and email information in the script and .env file.
3. Run the script:
```bash
python script_name.py
```
4. The script will continuously run, checking every 60 seconds. If the ISS is overhead and it’s nighttime, you’ll receive an email notification.
---
## 📝 Additional Notes
- The ISS position API (http://api.open-notify.org/iss-now.json) provides real-time location data.
- The https://api.sunrise-sunset.org/json API gives sunrise and sunset times based on your location.
- Use responsibly to avoid spamming the ISS API.
  ---
