# flight-navlog-estimator
# Desktop Flight Plan Navigation Log & Fuel Burn Estimator

Educational Spring Boot backend that generates a multi-leg navigation log from user-entered waypoints, estimates wind correction, groundspeed, headings, ETE and fuel, persists the plan to MySQL, exposes REST endpoints, and provides a SOAP lookup endpoint.

## Requirements
- Java 17+
- Eclipse IDE / Spring Tools (STS)
- MySQL 8+
- Maven (included through Maven Wrapper only if you add it; otherwise Eclipse's Maven integration)
- Postman or SoapUI

## Database
Create the database:
```sql
CREATE DATABASE flight_navlog_db;
```
Edit `src/main/resources/application.properties` and set your MySQL password.

## Eclipse import
1. Download/extract the project ZIP.
2. In Eclipse: File > Import > Maven > Existing Maven Projects.
3. Select the extracted `flight-navlog-estimator` folder.
4. Right-click project > Maven > Update Project.
5. Run `FlightNavlogApplication.java` as Java Application or Spring Boot App.

## REST API
- `POST http://localhost:8080/api/flight-plans` creates/calculates and saves a plan.
- `GET http://localhost:8080/api/flight-plans` lists saved plans.
- `GET http://localhost:8080/api/flight-plans/{id}` retrieves a plan.

Example POST JSON:
```json
{
  "pilotName": "Student Pilot",
  "departure": "VAAH",
  "destination": "VABO",
  "cruiseTasKt": 105,
  "windFromDeg": 270,
  "windSpeedKt": 12,
  "magneticVariationDeg": 1.5,
  "fuelBurnGph": 8.5,
  "reserveMinutes": 30,
  "waypoints": [
    {"name":"VAAH", "latitude":23.077, "longitude":72.634},
    {"name":"Waypoint A", "latitude":22.8, "longitude":72.9},
    {"name":"VABO", "latitude":22.336, "longitude":73.226}
  ]
}
```

## SOAP
WSDL: `http://localhost:8080/ws/flightPlan.wsdl`
POST to `http://localhost:8080/ws` with `Content-Type: text/xml; charset=utf-8` and:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:fl="http://krish.com/flightnavlog/soap">
  <soapenv:Header/>
  <soapenv:Body>
    <fl:getFlightPlanRequest><fl:id>1</fl:id></fl:getFlightPlanRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

## Formula overview
- Great-circle distance: haversine, Earth radius 3440.065 nautical miles.
- Initial true course: spherical initial-bearing formula.
- Wind correction angle: estimated from crosswind component and TAS.
- Groundspeed: TAS corrected for along-track wind and heading correction.
- Magnetic heading: true heading minus magnetic variation (east variation positive).
- ETE: distance / groundspeed * 60 minutes.
- Trip fuel: leg ETE hours * fuel burn in gallons/hour.
- Reserve fuel: fuel burn * reserve minutes / 60.
- Total fuel: trip fuel + configured reserve fuel.

## Dataset integration note
This demo accepts waypoint coordinates as input. To fulfil a requirement for open navigation datasets, an extension point is to import a licensed/public airport and waypoint CSV (for example, OurAirports data) into a separate `waypoints` table and add a search endpoint. This starter project does not yet fetch live airway/VOR datasets.

## Aviation safety
This is an educational prototype, not an operational flight-planning tool. It does not include certified weather, pressure altitude/temperature TAS conversion, official magnetic variation models, terrain/obstacle checks, NOTAMs, airspace, performance charts, alternate requirements, or regulatory-specific reserve rules. Verify all calculations and information against current official sources and an instructor before any real flight. Reserve minutes must be set according to the applicable jurisdiction/operation; 30 minutes here is only a sample value.

