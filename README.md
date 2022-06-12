# GCP Endpoints

Documentations for each endpoints deployed in cloud. **NOTE** that this is by any means **valid only when the service is considered up** and you should not assume that the link is static as the link can be changed.

## Table of Contents
- [GCP Endpoints](#gcp-endpoints)
  - [Table of Contents](#table-of-contents)
  - [Cloud Run](#cloud-run)
    - [Safe Route Sandbox](#safe-route-sandbox)
    - [JWT Auth](#jwt-auth)
      - [Usage](#usage)
        - [Endpoints](#endpoints)
      - [Requests](#requests)
    - [Return All Subdistrict Coordinates](#return-all-subdistrict-coordinates)
      - [Usage](#usage-1)
    - [Public Website](#public-website)
    - [Get Subdistrict Stat](#get-subdistrict-stat)
      - [Usage](#usage-2)
  - [Cloud Functions](#cloud-functions)
    - [Add Row Jakarta Crime History](#add-row-jakarta-crime-history)
      - [Usage](#usage-3)
    - [BigQuery Clustering](#bigquery-clustering)
    - [Calculate Then Send Email](#calculate-then-send-email)
      - [Usage](#usage-4)
    - [Calculate Trigger](#calculate-trigger)
      - [Usage](#usage-5)
    - [CSV Clustering](#csv-clustering)
    - [JSON to Firestore](#json-to-firestore)
    - [Send Email](#send-email)
      - [Usage](#usage-6)
  - [Compute Engine](#compute-engine)
    - [Habit Tracker Alpha](#habit-tracker-alpha)
    - [Valhalla Jakarta Server](#valhalla-jakarta-server)
      - [Usage](#usage-7)

## Cloud Run

### Safe Route Sandbox

Is a flask web app not a backend container, visit the website by visiting https://safe-route-sandbox-ck44nnq7hq-an.a.run.app

### JWT Auth

Is a auth backend for saferoute, meant both as a fallback plan, connected to MongoDB Atlas that is deployed on GCP https://safe-route-jwt-authentication-ck44nnq7hq-an.a.run.app

#### Usage

##### Endpoints

- https://safe-route-jwt-authentication-ck44nnq7hq-an.a.run.app/signup form encoded, required payload with POST method is username and password
- https://safe-route-jwt-authentication-ck44nnq7hq-an.a.run.app/login form encoded, required payload is the same with signup with POST method, username and password
- https://safe-route-jwt-authentication-ck44nnq7hq-an.a.run.app/user/profile serving as a protected route simulation, can only be accessed if you have the right token GET method with Bearer token.

#### Requests
Using HTTPie

```bash
$ http POST :3000/signup username=<username> password=<password>

<HTTPie details>

{
    "message": "Signup successful",
    "user": {
        "_id": "[id]",
        "password": "[encrypted password]",
    }
}

$ http POST :3000/login username=<username> password=<password>

<HTTPie details>

{
    "token": "ey..."
}


$ export JWT="ey..."


$ http GET :5000/user/profile Authorization:"Bearer $JWT"

<HTTPie details>

{
    "message": "You made it to the secure route",
    "user": {
        "_id": "[id]"
    },
    "token": "[token]"
}
```

### Return All Subdistrict Coordinates

Is a JSON query endpoint, https://return-all-subdistrict-coord-ck44nnq7hq-et.a.run.app

#### Usage

```bash
curl -X POST https://return-all-subdistrict-coord-ck44nnq7hq-et.a.run.app -H "Content-Type:application/json"  -d '{"sub
district":"All"}'
```

return

```bash
...
{"latitude":-6.243231,"longitude":106.8436975},{"latitude":-6.2431953,"longitude":106.843367},{"latitude":-6.2431469,"longitude":106.8429592},{"latitude":-6.2430653,"longitude":106.8425272},{"latitude":-6.2429319,"longitude":106.8418121},{"latitude":-6.2427776,"longitude":106.8411866},{"latitude":-6.2424452,"longitude":106.8399455},{"latitude":-6.2417947,"longitude":106.837787},{"latitude":-6.2413665,"longitude":106.8362533},{"latitude":-6.2410689,"longitude":106.8352331}
...
```

### Public Website

Is a flask web application for marketing purposes https://public-website-ck44nnq7hq-an.a.run.app

### Get Subdistrict Stat

Is a node js backend JSON query server, https://get-subdistrict-stat-ck44nnq7hq-et.a.run.app

#### Usage

```bash
curl -X POST http://localhost:8080 -H "Content-Type:application/json" -d '{"sub
district":"Tanjung Priok"}'
```

to get a response like:

```bash
{
  "subdistrict": "Tanjung Priok",
  "total_crime": 80,
  "crime_info": {
    "Aggravated Assault": 2,
    "Aiding and Abetting / Accessory": 1,
    "Arson": 2,
    "Assault / Battery": 2,
    "Attempt": 6,
    "Child Abandonment": 2,
    "Conspiracy": 1,
    "Cyberbullying": 2,
    "Disorderly Conduct": 4,
    "Disturbing the Peace": 1,
    "Forgery": 2,
    "Fraud": 1,
    "Harassment": 5,
    "Homicide": 3,
    "Kidnapping": 4,
    "Manslaughter: Involuntary": 2,
    "Manslaughter: Voluntary": 5,
    "Murder: First-degree": 4,
    "Murder: Second-degree": 3,
    "Open Container (of alcohol)": 1,
    "Pyramid Schemes": 1,
    "Rape": 4,
    "Robbery": 5,
    "Securities Fraud": 1,
    "Sexual Assault": 3,
    "Solicitation": 1,
    "Stalking": 6,
    "Statutory Rape": 3,
    "Theft": 3
  }
}
```

## Cloud Functions

### Add Row Jakarta Crime History

Is a function for adding row of json object array to bigquery. https://asia-southeast2-safe-route-351803.cloudfunctions.net/add-row-jakarta_crime_history

#### Usage

```bash
curl -m 70 -X POST https://asia-southeast2-safe-route-351803.cloudfunctions.net/add-row-jakarta_crime_history \
-H "Authorization:bearer $(gcloud auth print-identity-token)" \
-H "Content-Type:application/json" \
-d '{
    "date": "2022-01-02",
    "time": 15,
    "latitude": -6.12597108,
    "longitude": 106.9203174
}'
```

Should return

```bash
"Insert success
```

### BigQuery Clustering

Is a pub-sub BigQuery Job done triggered function

### Calculate Then Send Email

Is a function for triggering model prediction result then send an email.https://asia-southeast2-safe-route-351803.cloudfunctions.net/calculate_then_send_email

#### Usage

```bash
curl --request POST \
  --url http://localhost:8080/ \
  --header 'Content-Type: application/json' \
  --data '{
	"prediction":[0.0, 0.0],
	"cur_position":[0.0101, 0.1001],
    "recipient":"test@example.com",
    "username":"turin_turambar",
}'
```

Should return

```bash
Success
```

### Calculate Trigger

Is a model helper functions for determining if forecasted coordinate is more than 1km. https://asia-southeast2-safe-route-351803.cloudfunctions.net/calculate_trigger

#### Usage

```bash
curl --request POST \
  --url http://localhost:8080/ \
  --header 'Content-Type: application/json' \
  --data '{
	"prediction":[0.0, 0.0],
	"cur_position":[0.0101, 0.1001]
}'
```

to get a response like

```bash
{
	"too_far?": "True" / "False"
}
```

### CSV Clustering

Is a function for doing clustering job from csv blob, bucket triggered function

### JSON to Firestore

Doing clustering job from BigQuery to Firestore, Bucket triggered function.

### Send Email

Is a standalone function (that has alreadby been fused so it is **deprecated**)https://asia-southeast2-safe-route-351803.cloudfunctions.net/send_email

#### Usage

```bash
curl --request POST \
  --url https://asia-southeast2-safe-route-351803.cloudfunctions.net/send_email \
  --header 'Content-Type: application/json' \
  --data '{
	"bool_prediction":"True",
	"recipient":"test@example.com",
    "location":"0.0, 0.0"
}'
```

Should return

```bash
Success
```

## Compute Engine

### Habit Tracker Alpha

No endpoint, Vertex AI Workbench, for ML development purposes.

### Valhalla Jakarta Server

Is a valhalla server running on Container OS, 35.187.197.248:8002/route

#### Usage

```bash
curl http://35.187.197.248:8002/route --data '{"locations":[{"lat":-6.189626,"l
on":106.825811},{"lat":-6.148926,"lon":106.727605}],"costing":"auto","directions_o
ptions":{"units":"miles"}}' | jq '.'
```

should response with

```bash
{
  "trip": {
    "language": "en-US",
    "status": 0,
    "units": "miles",
    "status_message": "Found route between points",
    "legs": [
      {
        "shape": "fhwxJ__dwjEt~Bw_CtEcFjJyMn@iBCiCSkA]}@aBiBoqA}@qR?kD?a@M}E?uf@}@gKMyX_@qr@iu@gCyCeCkAuD{@eBOiC]@dE?hCMpGKfNW`SM~HKvCEhCq@vCFxLQn^BfCObQWbRAz@QbQEhCIjKMbGQ|IWpRIfDUdPGbFChCXxBRxBCpGBdE?tE?dE?l@AxWC`\\W`SGlJEvXKtEGtDQhCGfDMtDSnIC|@QjKIvCe@nr@JjVN~Ih@bPb@zLhCdx@n@b[NnS?l@TrQ\\b[NfXBlJDnIHrPMxWA`HC`SCvMIvD]fDy@vCoAfCgCfDkCxCwC?eI?c\\L}IN_ENiEj@aD|@uEvCsNlKwPhLiCzAwBzAaCxBwDz@wAl@eA|@yAz@gCjAaB^_D\\kWnIc\\lJiOdEyPdFof@vMy_@jK{LfDyHzA{HzAoKjAeUfDq]tEc|@rPcF|@iQvCkc@`Hy`@`Hs~AxWog@|IcANge@~H}kApQqXtEcANyh@nH{Cl@sNjBq[tE]?cD\\uGl@qBLu[zAmCNpCdYpAxMlCfXRjBhHvv@l@tE\\zAb@fCbAxCnC~H~I~RnOjW|_@pp@xo@beAxNzUnd@`q@pAfCdBfD|ElKrXro@hT`g@t@zA|@hBfBfDzBtElE`HjAhBx\\pf@zFlJlCvDf@|@zBdD|KdPbd@|s@nAhCfAxAv@lAhAhBrM~SlCdEnIvMtHhM|L`SpJbP|IhM|LhXtElJx@hBIxB@ND\\bBbFjBtETl@lQtd@dFzKjQ|^zLr[zAvC~EzKdF`HtHlJpHpGdGhChEfDvOzKtT|Jl}@pq@dTtOjM|IrFvDlM|InMnIdMfNpFpG|DbG~ElJfElKt@xBt@xBnCnHvj@fvBjE~RhDzUfB|Tp@jLj@zj@`BplD`@jUlAhb@jG|s@f@fD|BdO`G|^hO~r@l`@`dBrvB~sHhQbp@tZpeAlIb[~Mfm@xKfn@pCnR|Hxv@pAdOf@vNvCro@zEviDjHlaG~Up_GvAp\\vMfhDdIplDrId_Ed@lTvEjsBj@jUrC||AlI~kF}@naFHb[`CtEZxBJhCf@li@AfM@z`@d@nIdDpR|Jtn@bBbPr@xWaBtYyFdy@eAlKE`HJlItArF`BvDnE`HrHbGlItDzKzAxHOdI{@~GwCxFuE|EaHnCcGdAqG^kLwA}JcDoH{FoI_GwD_HyBoO{@mTz@aSfDuUfDqOxBmG|@gG?}k@|Im~@bQqDl@euBb[sn@|JyqBr[{ZbFwdChb@g`BhWso@nIip@`GkhApH_tArF__@zAiZjAk~@xBex@hC}M\\k~BrFg[z@qq@hC_hBhMqRxByjAjL}sAnHupBvDkT\\_aANqdCkB}@|@}@l@cA?aB?_e@LoVm@i~D_IgI\\{MrGkSm@}M?uI?{J?eUNoGOyDOoC?qVNkA?iP?cd@Ni[LyM?wTNi`AuE{JyBsUwC}]yCm`@uDgI{A",
        "summary": {
          "has_time_restrictions": false,
          "max_lon": 106.830025,
          "max_lat": -6.148922,
          "time": 1545,
          "length": 11.921,
          "min_lat": -6.193448,
          "min_lon": 106.727104
        },
        "maneuvers": [
          {
            "travel_mode": "drive",
            "begin_shape_index": 0,
            "length": 0.236,
            "time": 37,
            "type": 2,
            "end_shape_index": 4,
            "instruction": "Drive southeast on Jalan Gereja Theresia.",
            "verbal_pre_transition_instruction": "Drive southeast on Jalan Gereja Theresia for 2 tenths of a mile.",
            "travel_type": "car",
            "street_names": [
              "Jalan Gereja Theresia"
            ]
          },
          {
            "travel_type": "car",
            "travel_mode": "drive",
            "verbal_pre_transition_instruction": "Turn left onto Jalan HOS Cokroaminoto.",
            "verbal_transition_alert_instruction": "Turn left onto Jalan HOS Cokroaminoto.",
            "length": 0.339,
            "instruction": "Turn left onto Jalan HOS Cokroaminoto.",
            "end_shape_index": 22,
            "type": 15,
            "time": 97,
            "verbal_post_transition_instruction": "Continue for 3 tenths of a mile.",
            "street_names": [
              "Jalan HOS Cokroaminoto"
            ],
            "begin_shape_index": 4
          },
          ...
```