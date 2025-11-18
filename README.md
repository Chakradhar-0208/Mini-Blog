
# Travel App - Complete API Documentation

## Project Overview
A comprehensive travel planning app with user-generated reviews, trip recommendations, and real-time location-based services for road trips and travel planning.

## Technology Stack Recommendations
*   **Backend**: Node.js with Express.js
*   **Database**: MongoDB
*    **Authentication**: JWT tokens + Google OAuth
*   **Maps Integration**: Leaflet.js
*   **File Storage**: Cloudinary for images
*   **Push Notifications**: Firebase Cloud Messaging
* * *

## API Documentation

### Base URL

```cpp
Production: https://api.travelapp.com/v1
Development: http://localhost:3000/api/v1
```

### Authentication
All protected endpoints require Bearer token in Authorization header:

```gherkin
Authorization: Bearer <jwt_token>
```
Admin endpoints require `role: "Admin"` in request payload
```gherkin
Middleware: requireRole("admin")
```


## 1\. Authentication APIs

### POST `/api/v1/auth/register`
Register a new user
Frontend must send a `profileImage` (user-uploaded or default avatar)

**Request Body:**

```json
{
  "name": "string",
  "email": "string",
  "password": "string",
  "phone": "string",
  "profileImage": "string"
}
```

**Response: 201**
```json
{
  "token": "JWT string",
  "user": {
    "id": "string",
    "name": "string",
    "email": "string",
    "phone": "string",
    "profileImage": "string"
  }
}
```

### POST `/api/v1/auth/google`
Login or register a user through Google OAuth

**Request Body:**

```json
{
  "googleToken": "string",
  "deviceInfo": {
    "platform": "string",
    "deviceId": "string"
  }
}
```

**Response: 200/201**
```json
{
  "token": "JWT string",
  "user": {
    "_id": "string",
    "name": "string",
    "email": "string",
    "profileImage": "string",
    "googleId": "string",
    "phone": "string"
  }
}
```
### POST `/api/v1/auth/login`
Authenticate an existing user

**Request Body:**

```json
{
  "email": "string",
  "password": "string"
}
```

**Response: 200**
```json
{
  "authToken": "JWT string",
  "user": {
    "_id": "string",
    "name": "string",
    "email": "string",
    "phone": "string",
    "profileImage": "string",
    "role": "string"
  }
}
```

### POST `/api/v1/auth/forgot-password`
Send password reset email

**Request Body:**

```json
{
  "email": "string"
}
```
**Response: 200**
```json
{
  "message": "If that email exists, a password reset link has been sent."
}
```

### POST `/api/v1/auth/reset-password`
Reset password using a valid JWT token
**Request Body:**

```json
{
  "token": "string",
  "password": "string"
}
```
**Response: 200**
```json
{
  "message": "Password reset successful"
}

```
### POST `api/v1/auth/logout`
Logout user, Frontend should delete token
**Headers:** 
```gherkin
Authorization: Bearer <jwt_token>
```

**Response: 200**
```json
{
  "message": "Logged out successfully"
}
```

## 2\. User Management APIs

### GET  `/api/v1/users`
Health check route

**Response: 200**
```
User Management Route Active
```

### GET `/api/v1/users/getUser`
Fetch user details (basic or detailed)

**Headers:** Authorization required

**Query Params:**
| Key      | Type   | Required | Description                                   |
|----------|--------|----------|-----------------------------------------------|
| email    | string | yes      | Email of user to fetch                        |
| detailed | string | no       | If `"true"`, includes longestTrip data        |

**Note:** User can fetch their own data, Admin can fetch any user

**Response: 200** 
```json
{
    "user":{
	"name":"string",
	"email":"string",
	"age":"number",
	"gender":"string",
	"role":"string",
	"tripCount":"number",
	"totalDistance":"number",
	"totalJourneyTime":"number",
	"longestTrip":{
		"byDistance":{
			"title":"string",
			"distance":"number",
			"duration":"number",
		},
		"byDuration":{
			"title":"string",
			"distance":"number",
			"duration":"number",
		}
	}
    },
    "source":"string"
}
```

### GET  `api/v1/users/savedTrips`
Gets all trips saved by current user

**Headers:** Authorization required

**Response: 200**

```json
{
  "savedTrips": 
    [{
      "_id": "string",
      "title": "string",
      "imageURLs": ["string"]
    }]
}
```

### PUT `/api/v1/users/updateUser/:id`
Update user information

**Headers:** Authorization required

**URL Params:** ``` id (required) ```

**Request Body:**
```json
{
    "name": "string",
    "email": "string",
    "phone": "string",
    "age": "number",
    "gender": "string",
    "interests":["string"],
    "travelType": "string",
    "preferences":{
        "tripDifficulty":"string",
        "budgetRange":"string",
        "altitudeSickness":"boolean",
        "tripSuggesions":"boolean",
        "checkpointAlerts":"boolean",
        "systemUpdates":"boolean"
    }
    "tripCount":"number"
}
```
**Response: 200**

```json
{
    "message": "User updated successfully",
    "user": {
        "name": "string",
        "email": "string",
        "phone": "string",
        "age": "number",
        "gender": "string",
        "role": "string"
    }
}
```
### GET `api/v1/users/getProfileImage`
Retrieve a user's profile image

**Query Params:**
| Key   | Type   | Required |
|-------|--------|----------|
| email | string | yes      |

**Response: 200**
```json
{
    "profileImage":"string",
    "source":"string"
}
```

### PUT `/api/v1/users/updateProfileImage`
Update user's profile image

**Request Type:** `multipart/form-data`

**Headers:** Authorization required

**Note:** User can update their own image, Admin can update any user

**Fields:**

| Field        | Type  | Required |
|--------------|--------|----------|
| profileImage | file  | yes      |
| email        | string| yes      |

**Response: 202**
```json
{
  "message": "Profile image update queued successfully (background job)"
}
```
### DELETE `api/v1/users/deleteUser`
Delete a user by email

**Headers:** Authorization required

**Request Body:**
```json
{
    "email":"string"
}
```
**Response: 200**
```json
{
    "message": "User deleted successfully"
}
```

## 3\. Trip Management APIs


**Router-level rate limit:** 
`100 Requests per minute` per IP, Post `message: "Too many requests from this IP, please try again later."`

### GET `api/v1/trips`
Get paginated list of trips with filtering

**Query Params:**

| Key        | Type   | Required | Description / Notes                      |
|------------|--------|----------|-------------------------------------------|
| page       | number | no       | Default `1`                               |
| limit      | number | no       | Default `10`                              |
| sort       | string | no       | Field to sort by (default `"title"`)      |
| difficulty | string | no       | Allowed: `easy`, `moderate`, `hard`       |
| minRating  | number | no       | Must be between 1 and 5                   |
| status     | string | no       | Allowed: `active`, `inactive`, `deleted`  |

**Response: 200**

```json
{
  "response":{
    "currentPage": "number",
    "totalPages":"number",
    "tripCount": "number",
    "data":{
      _id": "string",
      "title": "string",
      "description": "string",
      "distance": "number",
      "duration": "number",
      "rating": "number",
      "reviewCount": "number",
      "imageURLs": ["string"],
      "difficulty": "string",
      "status": "string",
      "startPoint": {
        "name":"string",
        "location":{
          "type":"string",
          "coordinates":["number"]	
        }
      },
      "endPoint": {
        "name":"string",
        "location":{
          "type":"string",
          "coordinates":["number"]	
        }
      },
      "estimatedCost": {
        "car":{
          "fuel":"number",
          "tolls":"number",
          "accommodation":"number",
          "food":"number",
          "parking":"number",
          "total":"number"
        },
        "bike":{
          "fuel":"number",
          "tolls":"number",
          "accommodation":"number",
          "food":"number",
          "parking":"number",
          "total":"number"
        }
      }
    }
  },
  "source":"string"
}
```

### GET `/api/v1/trips/:id`
Get detailed trip view by ID

**URL Params**

|KEY |  TYPE  |REQUIRED|
|----|--------|--------|
| id | string |  yes   |

**Response: 200**

```json
{
  "trip": {
    "_id": "string",
    "title": "string",
    "description": "string",
    "startPoint": {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      }
    },
    "endPoint": {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      }
    },
    "distance": "number",
    "duration": "number",
    "roadInfo": {
      "highways":["string"],
      "ghats":["string"],
      "roadCondition":"string",
      "traffic":"string",
    },
    "checkPoints": [ {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]
      },
      "description":"string",
      "type":"string",
      "estimatedStopTime":"number"
    } ],
    "informativePlaces": {
      "restaurants":{},
      "accommodations":{},
      "hospitals":{},
      "policeStations":{},
      "fuelStations":{},
      "vehicleService":{}
    },
    "estimatedCost": {
      "car": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 },
      "bike": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 }
    },
    "journeyKit": [ {"item":"string", "necessity":"string"} ],
    "tollGates": [{
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      },
      "cost":"number"
    }],
    "precautions": ["string"],
    "rating": "number",
    "reviewCount": "number",
    "imageURLs": ["string"],
    "createdBy": { "_id": "string", "name": "string", "email": "string" },
    "createdAt": "date",
    "status": "string"
  },
  "source": "string"
}

```
### POST `/api/v1/trips/:id/save`
Save a trip to current user's saved list

**Headers:** Authorization required

**URL Params:**
|KEY |TYPE |REQUIRED|
|---|---|--|
|id	|string|yes|

**Response Body: 200**
```json
{
    "message":"Trip saved successfully"
}
```

### POST `/api/v1/trips`
Create a new trip

**Headers:** Authorization required

**Request Type:** `multipart/form-data`

**Note:** 
* New trips are created with `status:"inactive"`,, admin approval required to activated
* Uploaded images are sent to Cloudinary; `trip.imageURLs` will contain the `secure_url` values

**Fields:**

| Field         | Type                  | Required |
|---------------|---------------|----------|
| title         | string                | yes      |
| description   | string                | no       |
| startPoint    | object                | yes      |
| endPoint      | object                | yes      |
|distance|number|yes|
|duration|number|yes|
|keywords|array|no|
|difficulty|string|no|
| checkpoints   | array                 | no       |
| roadInfo      | object                | no       |
|informativePlaces|array|no|
| journeyKit    | array                 | no       |
|tollGates|array|no|
| precautions   | array                 | no       |
| estimatedCost | object                | no       |
| images      | files                 | no       |


**Fields Expanded:**
```json
{
  "title": "string",
  "description": "string",
  "distance": "number",
  "duration": "number",
  "imageURLs": ["string"],
  "difficulty": "string",
  "startPoint": {
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]	
    }
  },
  "endPoint": {
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]	
    }
  },
  "keywords":"string",
  "checkPoints": [ {
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]
    },
    "description":"string",
    "type":"string",
    "estimatedStopTime":"number"
  } ], 
  "roadInfo": {
    "highways":["string"],
    "ghats":["string"],
    "roadCondition":"string",
    "traffic":"string",
  },
  "informativePlaces": {
    "restaurants":{},
    "accommodations":{},
    "hospitals":{},
    "policeStations":{},
    "fuelStations":{},
    "vehicleService":{}
  },
  "tollGates": [{
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]	
    },
    "cost":"number"
  }],
  "precautions": ["string"],
  "estimatedCost": {
    "car":{
      "fuel":"number",
      "tolls":"number",
      "accomodation":"number",
      "food":"number",
      "parking":"number",
      "total":"number"
    },
    "bike":{
      "fuel":"number",
      "tolls":"number",
      "accomodation":"number",
      "food":"number",
      "parking":"number",
      "total":"number"
    }
  },
  "journeyKit": [ {"item":"string", "necessity":"string"} ],
  "imageURLs":"string"

}
```

**Response: 201**
```json
{
  "message":"string",
  "trip":{
    "title": "string",
    "description": "string",
    "distance": "number",
    "duration": "number",
    "imageURLs": ["string"],
    "difficulty": "string",
    "startPoint": {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      }
    },
    "endPoint": {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      }
    },
    "keywords":"string",
    "checkPoints": [ {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]
      },
      "description":"string",
      "type":"string",
      "estimatedStopTime":"number"
    } ], 
    "roadInfo": {
      "highways":["string"],
      "ghats":["string"],
      "roadCondition":"string",
      "traffic":"string",
    },
    "informativePlaces": {
      "restaurants":{},
      "accommodations":{},
      "hospitals":{},
      "policeStations":{},
      "fuelStations":{},
      "vehicleService":{}
    },
    "tollGates": [{
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      },
      "cost":"number"
    }],
    "precautions": ["string"],
    "estimatedCost": {
      "car":{
        "fuel":"number",
        "tolls":"number",
        "accomodation":"number",
        "food":"number",
        "parking":"number",
        "total":"number"
      },
      "bike":{
        "fuel":"number",
        "tolls":"number",
        "accomodation":"number",
        "food":"number",
        "parking":"number",
        "total":"number"
      }
    },
    "journeyKit": [ {"item":"string", "necessity":"string"} ],
    "imageURLs":"string"
  }
}
```

### PUT `/api/v1/trips/:id`
Update an existing trip

**Headers:** Authorization required

**Request Type:** `multipart/form-data`


**Fields:**

| Field         | Type                  | Required |
|---------------|---------------|----------|
| title         | string                | yes      |
| description   | string                | no       |
| startPoint    | object                | yes      |
| endPoint      | object                | yes      |
|distance|number|yes|
|duration|number|yes|
|keywords|array|no|
|difficulty|string|no|
| checkpoints   | array                 | no       |
| roadInfo      | object                | no       |
|informativePlaces|array|no|
| journeyKit    | array                 | no       |
|tollGates|array|no|
| precautions   | array                 | no       |
| estimatedCost | object                | no       |
| images        | files                 | no       |


**Fields Expanded:**
```json
{
  "title": "string",
  "description": "string",
  "distance": "number",
  "duration": "number",
  "imageURLs": ["string"],
  "difficulty": "string",
  "startPoint": {
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]	
    }
  },
  "endPoint": {
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]	
    }
  },
  "keywords":"string",
  "checkPoints": [ {
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]
    },
    "description":"string",
    "type":"string",
    "estimatedStopTime":"number"
  } ], 
  "roadInfo": {
    "highways":["string"],
    "ghats":["string"],
    "roadCondition":"string",
    "traffic":"string",
  },
  "informativePlaces": {
    "restaurants":{},
    "accommodations":{},
    "hospitals":{},
    "policeStations":{},
    "fuelStations":{},
    "vehicleService":{}
  },
  "tollGates": [{
    "name":"string",
    "location":{
      "type":"string",
      "coordinates":["number"]	
    },
    "cost":"number"
  }],
  "precautions": ["string"],
  "estimatedCost": {
    "car":{
      "fuel":"number",
      "tolls":"number",
      "accomodation":"number",
      "food":"number",
      "parking":"number",
      "total":"number"
    },
    "bike":{
      "fuel":"number",
      "tolls":"number",
      "accomodation":"number",
      "food":"number",
      "parking":"number",
      "total":"number"
    }
  },
  "journeyKit": [ {"item":"string", "necessity":"string"} ],
  "imageURLs":"string"

}
```

**Response: 200**
```json
{
  "message":"string",
  "updatedTrip":{
    "title": "string",
    "description": "string",
    "distance": "number",
    "duration": "number",
    "imageURLs": ["string"],
    "difficulty": "string",
    "startPoint": {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      }
    },
    "endPoint": {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      }
    },
    "keywords":"string",
    "checkPoints": [ {
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]
      },
      "description":"string",
      "type":"string",
      "estimatedStopTime":"number"
    } ], 
    "roadInfo": {
      "highways":["string"],
      "ghats":["string"],
      "roadCondition":"string",
      "traffic":"string",
    },
    "informativePlaces": {
      "restaurants":{},
      "accommodations":{},
      "hospitals":{},
      "policeStations":{},
      "fuelStations":{},
      "vehicleService":{}
    },
    "tollGates": [{
      "name":"string",
      "location":{
        "type":"string",
        "coordinates":["number"]	
      },
      "cost":"number"
    }],
    "precautions": ["string"],
    "estimatedCost": {
      "car":{
        "fuel":"number",
        "tolls":"number",
        "accomodation":"number",
        "food":"number",
        "parking":"number",
        "total":"number"
      },
      "bike":{
        "fuel":"number",
        "tolls":"number",
        "accomodation":"number",
        "food":"number",
        "parking":"number",
        "total":"number"
      }
    },
    "journeyKit": [ {"item":"string", "necessity":"string"} ],
    "imageURLs":"string"
  }
}
```

### DELETE `/api/v1/trips/:id`
Delete a trip 

**Headers:** Authorization required

**Note:** 
* User can update their own image, Admin can update any user
* If trip has images, server deletes Cloudinary resources by prefix `trips/<tripId>` before deleting the trip document.


**URL Params:**
|KEY|TYPE|REQUIRED|
|--|--|--|
|id|string|yes|

**Response: 200**
```json
{
	"message":"Trip deleted successfully"
}
```

### DELETE `api/v1/trips/saved-trips/:tripId`
Remove a trip from the current user's saved list
**Headers:** Authorization required

**URL Params:**

|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|

**Response: 200**
```json
{
	"message":"success"
}
```


## 4\. Maps & Location APIs

### GET `/maps/nearby`
Get nearby places (restaurants, accommodations, etc.)
**Query Parameters:**
*   `lat`: number
*   `lng`: number
*   `type`: string (restaurant | accommodation | hospital | fuel | service)
*   `radius`: number (in km, default: 5)

**Response:**

```json
{
  "places": [
    {
      "id": "string",
      "name": "string",
      "type": "string",
      "coordinates": [lat, lng],
      "rating": "number",
      "priceRange": "string",
      "distance": "number",
      "openNow": "boolean",
      "contact": "string"
    }
  ]
}
```

### GET `/maps/route`
Get route information between two points
**Query Parameters:**
*   `start`: string (lat,lng)
*   `end`: string (lat,lng)
*   `vehicle`: string (car|bike)

**Response:**

```json
{
  "route": {
    "distance": "number",
    "duration": "number",
    "coordinates": [[lat, lng]],
    "instructions": ["string"],
    "tollGates": [
      {
        "name": "string",
        "coordinates": [lat, lng],
        "cost": "number"
      }
    ]
  }
}
```

## 5\. Reviews APIs


### GET `/api/v1/reviews`

Review Health check route

**Response: 200**

```
Review Management Route Active
```
### GET `/api/v1/reviews/:tripId`
Get reviews for a trip
**URL Params:**

|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|

**Response: 200**

```json
{
  "reviews": [
    {
      "_id": "string",
      "trip": "string",
      "user": {
        "_id": "string",
        "name": "string",
        "email": "string",
        "profileImage": "string"
      },
      "rating": "number",
      "comment": "string",
      "upVotes": "number",
      "downVotes": "number",
      "createdAt": "date"
    }
  ],
  "total": "number",
  "source": "string"
}
```
### GET `/api/v1/reviews/:tripId/:reviewId`

Fetch a specific detailed review

**URL Params:**

|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|
|reviewId|string|yes|

**Response Body:**

```json
{
  "review": {
    "_id": "string",
    "trip": "string",
    "user": {
      "_id": "string",
      "name": "string",
      "email": "string",
      "profileImage": "string"
    },
    "rating": "number",
    "comment": "string",
    "images": ["string"],
    "checkpoints": ["object"],
    "upVotes": "number",
    "downVotes": "number",
    "votes": [{
      "userId": "string",
      "vote": "string"
    }],
    "createdAt": "date",
    "updatedAt": "date"
  },
  "source": "string"
}

```

### POST `api/v1/reviews/:tripId`

Create a new review for a trip

**Headers:** Authorization required

**Request Type:** `multipart/form-data`

**URL Params:**
|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|

**Fields:**

|FIELD|TYPE|REQUIRED|DESCRIPTION|
|--|--|--|--|
|rating|number|yes|Between 0 - 5
|comment|string|no| |
|checkpoints|array|no|JSON String of data|
|images|fiels|no|Mutilple files allowed|

**Response:201**

```json
{
  "message": "Review created successfully",
  "review": {
      "_id": "string",
      "trip": "string",
      "user": "string",
      "rating": "number",
      "comment": "string",
      "checkpoints": ["object"],
      "images": ["string"], 
      "upVotes": 0,
      "downVotes": 0
  }
}
```

### PUT `/api/v1/reviews/:tripId/:reviewId/update`
Update an existing review

**Headers:** Authorization required

**Request Type:** `multipart/form-data`

**URL Params:**
|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|
|reviewId|string|yes|


**Fields:**
|FIELD|TYPE|REQUIRED|DESCRIPTION|
|--|--|--|--|
|rating|number|no|Between 0 - 5
|comment|string|no| |
|checkpoints|array|no|JSON String of data|
|images|fiels|no|Mutilple files allowed|

**Response: 200**
```json
{
  "message": "Review updated successfully",
  "review": {
      "_id": "string",
      "rating": "number",
      "comment": "string",
      "images": ["string"]
  }
}
```

### PUT `/api/v1/reviews/:tripId/:reviewId/voting`
Vote on a review (Up,Down or Remove Vote)

**Headers:** Authorization Required

**URL Params:** 

|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|
|reviewId|string|yes|

**Request Body: 200**
```json
{
  "userVote":"string"
}
```
* **Note:** `userVote`allowed values: `"up", "down", null (to remove vote/default)` 

**Response: 200** 
```json
{
  "message": "Vote updated successfully",
  "review": {
    "_id": "string",
    "upVotes": "number",
    "downVotes": "number",
    "votes": [{
      "userId": "string",
      "vote": "string"
    }]
  }
}
```

### DELETE `api/v1/reviews/:tripId/:reviewId`
Delete a review

**Headers:** Authorization Required

**URL Params:**
|KEY|TYPE|REQUIRED|
|--|--|--|
|tripId|string|yes|
|reviewId|string|yes|

**Response: 200**
```json
{
  "message":"Review deleted successsfully"
}
```
## 6\. User Journey APIs

### GET `/api/v1/journeys`
```
Journey Health check route
```

**Response: 200**
```json
{
  "message":"journey Management Route Active"
}
```

### POST `/api/v1/journeys/start`
Start a new Journey

**Request Body:**
```json
{
  "tripId": "string",
  "userId": "string",
  "startLocation": {
    "coordinates": ["number"],
    "address": "string"
  },
  "journeyType": "string", 
  "notes": "string"
}
```
* **Note:** JourneyType accepts `solo(default), group, family` only


**Response: 201**
```json
{
  "_id": "string",
  "tripId": "string",
  "userId": "string",
  "status": "active",
  "startLocation": {
    "coordinates": [0, 0],
    "address": "string"
  },
  "startedOn": "date",
  "totalDistance": 0,
  "totalDuration": 0,
  "checkpoints": []
}
```

### PUT `/api/v1/journeys/:id/checkpoint`
Record a completed checkpoint and update journey totals

**URL Params:** 

| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Request Body:**

```JSON
{
  "checkpoint": {
    "name": "string",
    "distance": "number",
    "duration": "number",
    "coordinates": ["number"]
  }
}
```
* **Note:** `distance` (km) and `duration` (hours) are added to the journey totals. `coordinates` requires exactly 2 numbers

**Response: 200**
```json
{
  "_id": "string",
  "status": "active",
  "totalDistance": "number",
  "totalDuration": "number",
  "checkpoints": [{
    "name": "string",
    "distance": "number",
    "duration": "number",
    "coordinates": [0, 0],
    "completedAt": "date"
    }]
}
```

### PUT `/api/v1/journeys/:id/end`
End an active journey

**URL Params:**
| KEY | TYPE | REQUIRED | 
|-----|--------|----------|
| id | string | yes |

**Request Body:**

```JSON
{
  "endLocation": {
    "coordinates": ["number"],
    "address": "string"
  }
}
```
**Response: 200**

```JSON
{
  "_id": "string",
  "status": "completed",
  "endLocation": {
    "coordinates": [0, 0],
    "address": "string"
  },
  "completedOn": "date",
  "totalDistance": "number",
  "totalDuration": "number"
}
```

### GET `/api/v1/journeys/active`
Get all currently active journeys

**Response: 200**

```JSON
[
  {
    "_id": "string",
    "tripId": "string",
    "userId": "string",
    "status": "active",
    "startLocation": "object",
    "startedOn": "date"
  }
]
```

### GET `/api/v1/journeys/history`
Get history of completed or cancelled journeys

**Response: 200**

```JSON
[
  {
    "_id": "string",
    "tripId": "string",
    "userId": "string",
    "status": "completed",
    "startedOn": "date",
    "completedOn": "date",
    "totalDistance": "number"
  }
]
```


### DELETE `/api/v1/journeys/:id/delete`
Delete an entire journey record

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Response: 200**


```JSON
{
  "message": "Journey deleted successfully"
}
```

### DELETE `/api/v1/journeys/:journeyId/:checkpointId/delete`
Delete a specific checkpoint from a journey and recalculate totals

**URL Params:**
| KEY | TYPE | REQUIRED | 
|--------------|--------|----------| 
| journeyId | string | yes | 
| checkpointId | string | yes |

**Response: 200**

```JSON
{
  "message": "Checkpoint delted successfully"
}
```

## 7\. Recommendations APIs

### GET `/recommendations`
Get personalized trip recommendations based on user preferences (profile data) and current context (location/budget)

**Headers:** Authorization required

**Query Parameters:**

|KEY|TYPE|REQUIRED|DESCRIPTION
|--|--|--|--|
|lat|number|yes| User's current Latitude|
|lng|number|yes|User's current Longitude|
|budget|number|no| Maximum Budget|
|duration|number|no| Preferred maximum Duration|

**Response:  200**

```json
{
  "recommendations": [
    {
      "_id": "string",
      "title": "string",
      "description": "string",
      "distance": "number",
      "duration": "number",
      "difficulty": "string",
      "keywords": ["string"],
      "rating": "number",
      "startPoint": {
        "name": "string",
        "location": {
          "type": "string",
          "coordinates": ["number"]
        }
      },
      "estimatedCost": {
        "car": {
          "total": "number"
        }
      },
      "recommendationScore": "number",
      "scoreBreakdown": {
        "altitudeSickness": "number",
        "difficulty": "number",
        "interests": "number",
        "distance": "number",
        "rating": "number",
        "budget": "number",
        "duration": "number"
      }
    }
  ]
}
```


## 8\. Admin APIs


**Note:** All routes require `Authorization: Bearer <token>` and `role: "admin"`


### GET `/admin`

Admin Health check route

**Response: 200**

```JSON
{
  "message": "Admin Route Active"
}
```

### GET `/admin/users`

Get a list of all users. 

* **Note:** Returns specific fields (name, email, role, profileImage, tripCount, etc.) sorted by role and creation date.

**Response: 200**

```JSON
{
  "users": [
    {
      "_id": "string",
      "name": "string",
      "email": "string",
      "role": "string",
      "profileImage": "string",
      "tripCount": "number",
      "travelType": "string",
      "interests": ["string"],
      "status": "string"
    }
  ],
  "source":"string"
}
```

### GET `/admin/users/:id`
Get details of a specific user.

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Response: 200**

```JSON
{
  "user": {
    "_id": "string",
    "name": "string",
    "phone": "string",
    "email": "string",
    "age": "number",
    "gender": "string",
    "profileImage": "string",
    "interests": ["string"],
    "tripCount": "number",
    "totalDistance": "number",
    "totalJourneyTime": "number",
    "travelType": "string",
    "role": "string",
    "status": "string",
    "savedTrips": ["string"],
    "preferences": {
      "tripDifficulty": "string", 
      "budgetRange": "string",
      "altitudeSickness": "boolean",
      "tripSuggestions": "boolean",
      "checkpointAlerts": "boolean",
      "systemUpdates": "boolean"
    },
    "googleId": "string",
    "createdAt": "date",
    "updatedAt": "date"
  }
}
```

### PUT `/admin/users/:id`

Update user details (Status, Role, or Profile Info).

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Request Body:**

```JSON
{
  "status": "active|banned|inactive",
  "role": "user|admin",
  "email": "string",
  "name": "string",
  "profileImage": "string",
  "tripCount": "number",
  "travelType": "string",
  "totalDistance": "number",
  "totalJourneyTime": "number",
  "gender": "string",
  "interests": ["string"],
  "preferences": {
      "tripDifficulty": "string",
      "budgetRange": "string",
      "altitudeSickness": "boolean",
      "tripSuggestions": "boolean",
      "checkpointAlerts": "boolean",
      "systemUpdates": "boolean"
  }
}
```

**Response: 200**

```JSON
{
  "user": {
    "status": "active|banned|inactive",
    "role": "user|admin",
    "email": "string",
    "name": "string",
    "profileImage": "string",
    "tripCount": "number",
    "travelType": "string",
    "totalDistance": "number",
    "totalJourneyTime": "number",
    "gender": "string",
    "interests": ["string"],
    "preferences": {
      "tripDifficulty": "string",
      "budgetRange": "string",
      "altitudeSickness": "boolean",
      "tripSuggestions": "boolean",
      "checkpointAlerts": "boolean",
      "systemUpdates": "boolean"
  }
}
}
```

### DELETE `/admin/users/:id`
Delete a user permanently.

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Response: 200**

```JSON
{
  "message": "User deleted successfully"
}
```

### GET `/admin/trips`
Get all Active (Approved) trips.

**Response: 200**

```JSON
{
  "trips": [
    {
      "_id": "string",
      "title": "string",
      "description": "string",
      "distance": "number",
      "duration": "number",
      "rating": "number",
      "reviewCount": "number",
      "difficulty": "string",
      "altitudeSickness": "boolean",
      "imageURLs": ["string"],
      "estimatedCost": {
        "car": { "total": "number" },
        "bike": { "total": "number" }
      },
      "createdBy": {
        "_id": "string",
        "name": "string",
        "email": "string"
      }
    }
  ],
  "source": "string"
}
```

### GET `/admin/trips/inactive`
Get all Inactive (Pending/New) trips for review.

**Response: 200**

```JSON
{
  "trips": [
    {
      "_id": "string",
      "title": "string",
      "description": "string",
      "distance": "number",
      "duration": "number",
      "rating": "number",
      "reviewCount": "number",
      "difficulty": "string",
      "altitudeSickness": "boolean",
      "imageURLs": ["string"],
      "estimatedCost": {
        "car": { "total": "number" },
        "bike": { "total": "number" }
      },
      "createdBy": {
        "_id": "string",
        "name": "string",
        "email": "string"
      }
    }
  ],
  "source": "string"
}
```

### GET `/admin/trips/:id`
Get full details of a specific trip.

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Response: 200**

```JSON
{
  "trip": {
    "_id": "string",
    "title": "string",
    "description": "string",
    "startPoint": {
      "name": "string",
      "location": { "type": "Point", "coordinates": [lng, lat] }
    },
    "endPoint": {
      "name": "string",
      "location": { "type": "Point", "coordinates": [lng, lat] }
    },
    "distance": "number",
    "duration": "number",
    "keywords": ["string"],
    "altitudeSickness": "boolean",
    "estimatedCost": {
      "car": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 },
      "bike": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 }
    },
    "rating": "number",
    "difficulty": "string",
    "imageURLs": ["string"],
    "status": "string",
    "roadInfo": {
      "highways": ["string"],
      "ghats": ["string"],
      "roadCondition": "string",
      "traffic": "string"
    },
    "checkPoints": [
      {
        "name": "string",
        "location": { "type": "Point", "coordinates": [lng, lat] },
        "description": "string",
        "type": "string",
        "estimatedStopTime": "number"
      }
    ],
    "informativePlaces": {
      "restaurants": [],
      "accommodations": [],
      "hospitals": [],
      "policeStations": [],
      "fuelStations": [],
      "vehicleService": []
    },
    "journeyKit": [
      { "item": "string", "necessity": "essential|recommended|optional" }
    ],
    "tollGates": [
      {
        "name": "string",
        "location": { "type": "Point", "coordinates": [lng, lat] },
        "cost": "number"
      }
    ],
    "precautions": ["string"],
    "createdBy": {
      "_id": "string",
      "name": "string",
      "email": "string"
    },
    "createdAt": "date",
    "updatedAt": "date"
  },
  "source":"string"
}
```


### PUT `/admin/trips/:id`
Update trip content (Title, Description, Route info, etc.).

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Request Body:**

```JSON
{
  "_id": "string",
  "title": "string",
  "description": "string",
  "startPoint": {
    "name": "string",
      "location": { 
        "type": "Point", 
        "coordinates": [lng, lat] 
      }
  },
  "endPoint": {
    "name": "string",
    "location": {
      "type": "Point", 
      "coordinates": [lng, lat] 
    }
  },
  "distance": "number",
  "duration": "number",
  "keywords": ["string"],
  "altitudeSickness": "boolean",
  "estimatedCost": {
    "car": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 },
    "bike": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 }
  },
  "difficulty": "string",
  "imageURLs": ["string"],
  "roadInfo": {
    "highways": ["string"],
    "ghats": ["string"],
    "roadCondition": "string",
    "traffic": "string"
  },
  "checkPoints": [{
    "name": "string",
    "location": { "type": "Point", "coordinates": [lng, lat] },
    "description": "string",
    "type": "string",
    "estimatedStopTime": "number"
  }],
  "informativePlaces": {
    "restaurants": [],
    "accommodations": [],
    "hospitals": [],
    "policeStations": [],
    "fuelStations": [],
    "vehicleService": []
  },
  "journeyKit": [
    { "item": "string", "necessity": "essential|recommended|optional" }
  ],
  "tollGates": [{
        "name": "string",
        "location": { "type": "Point", "coordinates": [lng, lat] },
        "cost": "number"
    }],
  "precautions": ["string"],
}
```

**Response: 200**

```JSON
{
  "trip": {
    "_id": "string",
    "title": "string",
    "description": "string",
    "startPoint": {
      "name": "string",
      "location": { 
        "type": "Point", 
        "coordinates": [lng, lat] 
      }
    },
    "endPoint": {
      "name": "string",
      "location": {
        "type": "Point", 
        "coordinates": [lng, lat] 
      }
    },
    "distance": "number",
    "duration": "number",
    "keywords": ["string"],
    "altitudeSickness": "boolean",
    "estimatedCost": {
      "car": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 },
      "bike": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 }
    },
    "difficulty": "string",
    "imageURLs": ["string"],
    "roadInfo": {
      "highways": ["string"],
      "ghats": ["string"],
      "roadCondition": "string",
      "traffic": "string"
    },
    "checkPoints": [{
      "name": "string",
      "location": { "type": "Point", "coordinates": [lng, lat] },
      "description": "string",
      "type": "string",
      "estimatedStopTime": "number"
    }],
    "informativePlaces": {
      "restaurants": [],
      "accommodations": [],
      "hospitals": [],
      "policeStations": [],
      "fuelStations": [],
      "vehicleService": []
    },
    "journeyKit": [
      { "item": "string", "necessity": "essential|recommended|optional" }
    ],
    "tollGates": [{
        "name": "string",
        "location": { "type": "Point", "coordinates": [lng, lat] },
        "cost": "number"
    }],
    "precautions": ["string"],
  },
  "source":"string"
}
```

### PUT `/admin/trips/:id/status`
Approve (activate) or Reject (deactivate/delete) a trip.

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Request Body:**

```JSON
{
  "status": "active|inactive|deleted"
}
```

**Response: 200**

```JSON
{
  "trip": {
    "_id": "string",
    "title": "string",
    "description": "string",
    "startPoint": {
      "name": "string",
      "location": { 
        "type": "Point", 
        "coordinates": [lng, lat] 
      }
    },
    "endPoint": {
      "name": "string",
      "location": {
        "type": "Point", 
        "coordinates": [lng, lat] 
      }
    },
    "distance": "number",
    "duration": "number",
    "keywords": ["string"],
    "altitudeSickness": "boolean",
    "estimatedCost": {
      "car": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 },
      "bike": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 }
    },
    "difficulty": "string",
    "imageURLs": ["string"],
    "roadInfo": {
      "highways": ["string"],
      "ghats": ["string"],
      "roadCondition": "string",
      "traffic": "string"
    },
    "checkPoints": [{
      "name": "string",
      "location": { "type": "Point", "coordinates": [lng, lat] },
      "description": "string",
      "type": "string",
      "estimatedStopTime": "number"
    }],
    "informativePlaces": {
      "restaurants": [],
      "accommodations": [],
      "hospitals": [],
      "policeStations": [],
      "fuelStations": [],
      "vehicleService": []
    },
    "journeyKit": [
      { "item": "string", "necessity": "essential|recommended|optional" }
    ],
    "tollGates": [{
        "name": "string",
        "location": { "type": "Point", "coordinates": [lng, lat] },
        "cost": "number"
    }],
    "precautions": ["string"],
  },
  "message": "Trip status updated to ${status}"
}
```

### DELETE `/admin/trips/:id`
Permanently delete a trip and its associated images.

**URL Params:** 
| KEY | TYPE | REQUIRED | 
|-----|--------|----------| 
| id | string | yes |

**Response: 200**

```JSON
{
  "message": "Trip deleted successfully"
}
```

### GET `/admin/reports`
Get all user reports.

**Response: 200**

```JSON
{
  "reports": [
    {
      "_id": "string",
      "reportedUser": { "name": "string", "email": "string" },
      "reportedBy": { "name": "string", "email": "string" },
      "reason": "string", 
      "createdAt": "date"
    }
  ]
}
```




## 9\. Report APIs

### GET `/api/v1/report`
Report Health check Route

**Response Body: 200**
```
Report Management Route Active
```

### POST `api/v1/report/user`
Report a user for inappropriate behaviour or profile content.

**Request Body:**

```json
{
  "target":"string",
  "reason":"string",
  "description":"string"
}
```
**Response: 201**
```json
{
  "message":"User report created"
}
```

### POST `api/v1/reports/trip`
Report a trip (eg., for misleading information or saftey concerns)

**Request Body:**
```json
{
  "target":"string",
  "reason":"string",
  "description":"string"
}
```

**Response: 200**
```json
{
  "message":"Trip report created"
}
```

### POST `api/v1/reports/review`
Report a specific review (eg., for spam or abuse)

**Request Body:**
```json
{
  "target":"string",
  "reason":"string",
  "description":"string"
}
```

**Response: 201**
```json
{
  "message":"Review report created"
}
```


## 10\. Notifications APIs

### GET `/api/v1/notifications`
Get user notifications

**Headers:** Authorization required

**Response: 200**
```json
{
  "count":"number",
  "notifications":[{
    "_id":"string",
    "userId":"string",
    "title":"string",
    "description":"string",
    "type":"string",
    "read":"boolean",
    "createdAt":"date",
    "updatedAt";"date"
  }]
}
```

### POST `/api/v1/notifications/subscribe`
Register or update the user's fcmToken and update their notification preferences.

**Headers:** Authorization required

**Request Body:**

```json
{
  "fcmToken": "string",
  "preferences": {
    "tripSuggestions": "boolean",
    "checkpointAlerts": "boolean",
    "systemUpdates": "boolean"
  }
}
```

**Response: 200**

```json
{
  "message":"Subscribed Successfully"
}
```

### POST `api/v1/notifications/send`
Send notifications to user

**Headers: ** Authorization required

**Request Body:**
```json
{
  "title":"string",
  "description":"string",
  "type":"string"
}
```
* **Note:** `type` allowed fields : `"tripSuggestions", "checkpointAlerts","systemUpdates"`


**Response: 200**
```json
{
  "message":"Message sent Successfully"
}
```




## 11\. App Feedback APIs

### POST `/api/v1/feedback`
Submit app feedback

**Headers:** Authorization required

**Request Type:** `multipart/form-data`

**Fields:**

|FIELD|TYPE|REQUIRED|DESCRIPTION|
|--|--|--|--|
|type|string|yes| Must be `"bug"/"ui"/"feature"`|
|category|string|yes| eg., "Maps Display","Auth Flow"|
|description|string|yes|Detailed explanation of feedback|
|deviceInfo|stringified JSON|yes| User's device info|
|screenshots|file|no|Multiple files  supported|

**Field `deviceInfo` Expanded:**
```json
{
  "platform":"string",
  "version":"string",
  "model":"string"
}
```

**Request: 200**

```json
{
  "message":"Feedback received successsfully"
}
```

