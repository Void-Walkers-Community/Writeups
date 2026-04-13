Gateway to the Turnpike
Flag - 15533
writeup -
First run this python file locally with the problem image in the samefolder
```
extract.py
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS
from geopy.geocoders import Nominatim
def get_exif_data(image_path):
 image = Image.open(image_path)
 info = image._getexif()
 if not info:
 return None
 exif = {}
 for tag, value in info.items():
 decoded = TAGS.get(tag, tag)
 if decoded == "GPSInfo":
 gps_data = {}
 for t in value:
 sub_decoded = GPSTAGS.get(t, t)
 gps_data[sub_decoded] = value[t]
 exif[decoded] = gps_data
 else:
 exif[decoded] = value
 return exif
def get_decimal_coordinates(info):
 for key in ["Latitude", "Longitude"]:
 if "GPSInfo" in info and f"GPS{key}" in info["GPSInfo"]:
 gps_coord = info["GPSInfo"][f"GPS{key}"]
 ref = info["GPSInfo"][f"GPS{key}Ref"]
 degrees = float(gps_coord[0])
 minutes = float(gps_coord[1])
 seconds = float(gps_coord[2])
 decimal = degrees + (minutes / 60.0) + (seconds / 3600.0)
 if ref in ["S", "W"]:
 decimal = -decimal
 if key == "Latitude":
 lat = decimal
 else:
 lon = decimal
 return lat, lon
def find_zip_code(image_path):
 try:
 data = get_exif_data(image_path)
 if data and "GPSInfo" in data:
 lat, lon = get_decimal_coordinates(data)
 geolocator = Nominatim(user_agent="geo_locator")
 location = geolocator.reverse(f"{lat}, {lon}")
 address = location.raw.get("address", {})
 zip_code = address.get("postcode", "ZIP not found")
 print(f"Location Found: {location.address}")
 print(f"Extracted ZIP Code: {zip_code}")
 else:
 print("No GPS metadata found in this image.")
 print(
 "To find the ZIP Code, the script would need a Vision API
or manual input."
 )
 except Exception as e:
 print(f"Error: {e}")
find_zip_code("gateway.png")
```
---

Then it will give the output - 15533


* [🔙 Back to OSINT Directory](../OSINT)
* [🔙 Back to OSINT Index Directory](../OSINT/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
