The Temple of Doom
Flag - DawgCTF{The_Ziggurat_Building}
writeup -
First run this python file locally with the problem image in the samefolder
```
filelocation.py
from exif import Image
def get_image_location(img_path):
 with open(img_path, 'rb') as src:
 img = Image(src)

 if not img.has_exif:
 return "No EXIF metadata found."

 try:
 lat = img.gps_latitude
 lat_ref = img.gps_latitude_ref
 lon = img.gps_longitude
 lon_ref = img.gps_longitude_ref

 decimal_lat = lat[0] + lat[1]/60 + lat[2]/3600
 decimal_lon = lon[0] + lon[1]/60 + lon[2]/3600

 if lat_ref == 'S': decimal_lat = -decimal_lat
 if lon_ref == 'W': decimal_lon = -decimal_lon

 return decimal_lat, decimal_lon
 except AttributeError:
 return "No GPS coordinates found in metadata."
print(get_image_location('image_5e5045.png'))
```
After this Script is run we will get the latitude and longitude :-
33.5610671 - 117.7133452
Then Put this [33.5610671 - 117.7133452] in goole map and we get out
exact location and then I search on google that what was the nickname
of this building and found out an article about this building on
Wikipedia
Link -
https://en.wikipedia.org/wiki/Chet_Holifield_Federal_Building
From this article in the Wikipedia, I found out that the nickname of
this building was - The Ziggurat Building

---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)



