
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



* * *

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
	    "accomodations":{},
	    "hospitals":{},
	    "policeStations":{},
	    "fuelStations":{},
	    "vehicleService":{}
	 },
    "estimatedCost": {
      "car": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 },
      "bike": { "fuel": 0, "tolls": 0, "accommodation": 0, "food": 0, "parking": 0, "total": 0 }
    },
    "journeyKit": [ {"item":"string", "necesssity":"string"} ],
    "tollGates": [{
	    "name":"string",
	    "location":{
		    "type":"string",
			"coordinates":["number"]	
		},
		"cost":"number"
	    }],
    "precautions": [ /* array */ ],
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

**Request Body:**

| Field         | Type                  | Required |
|---------------|------------------------|----------|
| title         | string                | yes      |
| description   | string                | no       |
| startPoint    | object or JSON-string | yes      |
| endPoint      | object or JSON-string | yes      |
| checkpoints   | array or JSON-string  | no       |
| roadInfo      | object or JSON-string | no       |
| journeyKit    | array or JSON-string  | no       |
| precautions   | array or JSON-string  | no       |
| estimatedCost | object or JSON-string | no       |
| images        | files (key `images`)  | no       |

```
**Response: 201**


### PUT `/trips/:id`
Update trip (Admin only)
**Headers:** Authorization required
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

* * *

## 5\. Reviews APIs

### GET `/trips/:tripId/reviews`
Get reviews for a trip
**Query Parameters:**
*   `checkpointId`: string (optional, filter by checkpoint)
*   `page`: number
*   `limit`: number

**Response:**

```json
{
  "reviews": [
    {
      "id": "string",
      "userId": "string",
      "userName": "string",
      "userAvatar": "string",
      "checkpointId": "string",
      "checkpointName": "string",
      "rating": "number",
      "comment": "string",
      "images": ["string"],
      "upvotes": "number",
      "downvotes": "number",
      "userVote": "up|down|null",
      "createdAt": "date"
    }
  ]
}
```

### POST `/trips/:tripId/reviews`
Add review for a checkpoint
**Headers:** Authorization required
**Request Body:**

```json
{
  "checkpointId": "string",
  "rating": "number",
  "comment": "string",
  "images": ["file"]
}
```

### PUT `/reviews/:reviewId/vote`
Vote on a review
**Headers:** Authorization required
**Request Body:**

```json
{
  "vote": "up|down"
}
```

### DELETE `/reviews/:reviewId/vote`
Remove vote from a review
**Headers:** Authorization required
* * *

## 6\. User Journey APIs

### POST `/journeys/start`
Start a new journey
**Headers:** Authorization required
**Request Body:**

```json
{
  "tripId": "string",
  "startLocation": {
    "coordinates": [lat, lng],
    "address": "string"
  }
}
```

### PUT `/journeys/:journeyId/checkpoint`
Complete a checkpoint
**Headers:** Authorization required
**Request Body:**

```json
{
  "checkpointId": "string",
  "completedAt": "date",
  "notes": "string"
}
```

### PUT `/journeys/:journeyId/end`
End a journey
**Headers:** Authorization required
**Request Body:**

```json
{
  "endLocation": {
    "coordinates": [lat, lng],
    "address": "string"
  },
  "totalDistance": "number",
  "totalDuration": "number"
}
```

### GET `/journeys/active`
Get active journey
**Headers:** Authorization required

### GET `/journeys/history`
Get journey history
**Headers:** Authorization required
**Query Parameters:**
*   `page`: number
*   `limit`: number
* * *

## 7\. Saved Itineraries APIs

### GET `/users/saved-trips`
Get user's saved trips
**Headers:** Authorization required

### POST `/users/saved-trips/:tripId`
Save a trip
**Headers:** Authorization required

### DELETE `/users/saved-trips/:tripId`
Remove saved trip
**Headers:** Authorization required
* * *

## 8\. Recommendations APIs

### GET `/recommendations`
Get personalized trip recommendations
**Headers:** Authorization required (optional)
**Query Parameters:**
*   `lat`: number (current location)
*   `lng`: number (current location)
*   `budget`: number
*   `duration`: number (days)

**Response:**

```json
{
  "recommendations": [
    {
      "tripId": "string",
      "title": "string",
      "distance": "number",
      "estimatedCost": "number",
      "rating": "number",
      "matchScore": "number",
      "reasons": ["string"]
    }
  ]
}
```

* * *

## 9\. Admin APIs

### GET `/admin/users`
Get all users (Admin only)
**Headers:** Authorization required
**Query Parameters:**
*   `page`: number
*   `limit`: number
*   `status`: string (active|banned|inactive)

### PUT `/admin/users/:userId/status`
Update user status (Admin only)
**Headers:** Authorization required
**Request Body:**

```json
{
  "status": "active|banned|inactive",
  "reason": "string"
}
```

### GET `/admin/trips`
Get all trips for admin review
**Headers:** Authorization required
**Query Parameters:**
*   `status`: string (pending|approved|rejected)

### PUT `/admin/trips/:tripId/approve`
Approve/Reject trip
**Headers:** Authorization required
**Request Body:**

```json
{
  "status": "approved|rejected",
  "comments": "string"
}
```

### GET `/admin/reports`
Get user reports
**Headers:** Authorization required

### PUT `/admin/reports/:reportId`
Handle user report
**Headers:** Authorization required
**Request Body:**

```json
{
  "action": "dismiss|warning|ban",
  "comments": "string"
}
```

* * *

## 10\. Notifications APIs

### GET `/notifications`
Get user notifications
**Headers:** Authorization required

### PUT `/notifications/:notificationId/read`
Mark notification as read
**Headers:** Authorization required

### POST `/notifications/subscribe`
Subscribe to push notifications
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

* * *

## 11\. App Feedback APIs

### POST `/feedback`
Submit app feedback
**Headers:** Authorization required
**Request Body:**

```json
{
  "type": "bug|ui|feature",
  "category": "string",
  "description": "string",
  "screenshots": ["file"],
  "deviceInfo": {
    "platform": "string",
    "version": "string",
    "model": "string"
  }
}
```

* * *
