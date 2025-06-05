# 🚘 Evoolv Ride API Documentation

This documentation describes the real-time communication flow between **riders** and **drivers** in the ride-hailing system using WebSockets. Two primary namespaces are involved:

* `/rider`
* `/driver`

---

## 📡 Namespaces

| Namespace | Description            |
| --------- | ---------------------- |
| `/rider`  | Handles rider actions  |
| `/driver` | Handles driver actions |

---

## 🔐 Authentication

> Authentication mechanism (e.g., JWT via query parameters) should be implemented during connection handshake.

---

## 🔁 End-to-End Event Flow

### 1. Rider Requests a Ride

**Client → Server**
**Event:** `ride_request`

```json
{
  "quoteId": "QUOTE-1749135242700-fmkdi3bm",
  "vehicleGrade": "REGULAR",
  "paymentMethod": "CASH",
  "bookingForOthers": false
}
```

### 2. Server Emits Ride Created Event

**Server → Rider**
**Event:** `ride_created`

```json
{
  "tripId": "3feae8d8-c6a5-4aec-bd36-9d3009723ebc",
  "pickup": { "lat": 6.6018, "lng": 3.3515 },
  "dropoff": { "lat": 6.5967, "lng": 3.3375 },
  "estimatedFare": 0,
  "estimatedDistanceKm": 0,
  "estimatedDurationMin": 0,
  "riderId": "16e8701a-8e3e-4034-a19f-7b3c56171966",
  "status": "REQUESTED",
  "carType": "REGULAR",
  "paymentMethod": "CASH",
  "isMoreThan2Passengers": false
}
```

### 3. Server Broadcasts to Drivers

**Server → Driver**
**Event:** `ride_match_started` *(same payload as `ride_created`)*

---

### 4. Driver Accepts or Declines Ride

**Driver → Server**
**Event:** `driver_accept` or `driver_decline`

```json
{ "tripId": "3feae8d8-c6a5-4aec-bd36-9d3009723ebc" }
```

### 5. If Driver Accepts: Notify Rider

**Server → Rider**
**Event:** `driver_assigned`

```json
{
  "tripId": "3feae8d8-c6a5-4aec-bd36-9d3009723ebc",
  "pickup": { "lat": 6.6018, "lng": 3.3515 },
  "dropoff": { "lat": 6.5967, "lng": 3.3375 },
  "estimatedFare": 0,
  "estimatedDistanceKm": 0,
  "estimatedDurationMin": 0,
  "riderId": "16e8701a-8e3e-4034-a19f-7b3c56171966",
  "driverId": "c4a001db-dda9-4927-895b-3f812b291dea",
  "status": "DRIVER_ASSIGNED",
  "carType": "REGULAR",
  "paymentMethod": "CASH",
  "isMoreThan2Passengers": false,
  "driverAssignedAt": "2025-06-05T14:55:58.067Z",
  "cancelledAt": "2025-06-05T14:55:22.961Z"
}
```

### 6. If No Driver Accepts Within 1 Minute

**Server → Rider**
**Event:** `ride_request_timeout` and `ride_match_failed`

```json
{ "tripId": "3feae8d8-c6a5-4aec-bd36-9d3009723ebc" }
```

### 7. Driver Arrives at Pickup

**Driver → Server → Rider**
**Event:** `driver_arrived`

```json
{
  "tripId": "3feae8d8-c6a5-4aec-bd36-9d3009723ebc",
  "pickup": { "lat": 6.6018, "lng": 3.3515 },
  "dropoff": { "lat": 6.5967, "lng": 3.3375 },
  "estimatedFare": 0,
  "estimatedDistanceKm": 0,
  "estimatedDurationMin": 0,
  "riderId": "16e8701a-8e3e-4034-a19f-7b3c56171966",
  "driverId": "c4a001db-dda9-4927-895b-3f812b291dea",
  "status": "DRIVER_ARRIVED",
  "carType": "REGULAR",
  "paymentMethod": "CASH",
  "isMoreThan2Passengers": false,
  "driverAssignedAt": "2025-06-05T14:55:58.067Z",
  "driverArrivedAt": "2025-06-05T15:11:57.599Z",
  "cancelledAt": "2025-06-05T14:55:22.961Z"
}
```

---

## 📍 Real-time Location Sharing

### `location` (Driver → Server)

```json
{
  "userId": "c4a001db-dda9-4927-895b-3f812b291dea",
  "lat": 6.6018,
  "lng": 3.3515,
  "timestamp": "2025-06-27T00:12:16.515Z"
}
```

---

## ❌ Ride Cancellation

### Rider Cancels

**Rider → Server**
**Event:** `ride_cancelled`

```json
{ "tripId": "f13bf01c-61c3-4cd5-953d-b087948f3746" }
```

### Server Notifies Driver

**Server → Driver**
**Event:** `ride_cancelled`

```json
{ "tripId": "c4084970-486d-49e3-a244-084d5e30677b" }
```

---

## 📬 Summary of Events

### Rider Emits

* `ride_request`
* `ride_cancelled`

### Rider Listens To

* `ride_created`
* `ride_request_timeout`
* `ride_match_failed`
* `driver_assigned`
* `driver_arrived`

### Driver Emits

* `location`
* `driver_accept`
* `driver_decline`
* `driver_arrive`
* `driver_start`

### Driver Listens To

* `ride_match_started`
* `ride_cancelled`

---

## 📎 Notes

* All events follow JSON format.
* Timestamps are in ISO 8601 format.
* Fields like `driverAssignedAt`, `driverArrivedAt`, and `cancelledAt` may be omitted if not applicable.
* The system must handle timeouts and fallbacks gracefully.
