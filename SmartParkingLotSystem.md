# Low-Level Design (LLD) for Smart Parking Lot System

## 1. Overview

The system manages vehicle entry and exit, assigns parking spots based on vehicle type and availability, and calculates parking fees.

## 2. Functional Components

### 2.1 Parking Spot Allocation

- Assigns available spots based on vehicle size (Motorcycle, Car, Bus).
- Maintains a real-time record of occupied and free spots.

### 2.2 Check-In and Check-Out

- Record vehicle entry time.
- Logs exit time and calculates the total duration of stay.

### 2.3 Parking Fee Calculation

- Charges are based on vehicle type and duration of stay.
- Possible fee structure:
  - Motorcycle: \$2/hour
  - Car: \$5/hour
  - Bus: \$10/hour

### 2.4 Real-Time Availability Update

- Updates the status of parking spots when vehicles enter and exit.
- Ensures concurrency handling for multiple vehicles entering/exiting simultaneously.

---

## 3. Data Model

### 3.1 Tables

#### **Vehicle**

| Column      | Type | Description                   |
| ----------- | ---- | ----------------------------- |
| vehicle\_id | UUID | Unique identifier for vehicle |
| type        | ENUM | Motorcycle, Car, Bus          |

#### **ParkingSpot**

| Column   | Type | Description                |
| -------- | ---- | -------------------------- |
| spot\_id | UUID | Unique identifier for spot |
| floor    | INT  | Floor number               |
| type     | ENUM | Motorcycle, Car, Bus       |
| status   | ENUM | Available, Occupied        |

#### **ParkingTransaction**

| Column          | Type     | Description                          |
| --------------- | -------- | ------------------------------------ |
| transaction\_id | UUID     | Unique identifier for transaction    |
| vehicle\_id     | UUID     | Reference to Vehicle                 |
| spot\_id        | UUID     | Reference to ParkingSpot             |
| entry\_time     | DATETIME | Timestamp of vehicle entry           |
| exit\_time      | DATETIME | Timestamp of vehicle exit (nullable) |
| total\_fee      | DECIMAL  | Calculated parking fee               |

---

## 4. Algorithm for Spot Allocation

1. **Vehicle Entry:**

   - Fetch an available spot for the vehicle type.
   - If found, mark it as occupied and create a transaction.
   - If not found, put the vehicle on a waitlist.

2. **Vehicle Exit:**

   - Record exit time and release parking spot.
   - Calculate parking fees based on duration and type.

---

## 5. Fee Calculation Logic

1. Compute the difference between entry and exit times.
2. Convert duration to hours (rounding up partial hours).
3. Multiply by the respective rate for the vehicle type.
4. Store the calculated fee in `ParkingTransaction`.

---

## 6. Concurrency Handling

- **Database Transactions:** Ensure atomic updates to avoid race conditions.
- **Optimistic Locking:** Prevents data conflicts in high-traffic scenarios.
- **Queue System:** Implement a queue for spot assignment to handle simultaneous vehicle arrivals.

---

## 7. API Design

### 7.1 Vehicle Entry

**Endpoint:** `POST /api/entry`
**Request:**

```json
{
  "vehicle_id": "abc-123",
  "type": "Car"
}
```

**Response:**

```json
{
  "spot_id": "xyz-456",
  "floor": 2,
  "message": "Spot assigned"
}
```

### 7.2 Vehicle Exit

**Endpoint:** `POST /api/exit`
**Request:**

```json
{
  "vehicle_id": "abc-123"
}
```

**Response:**

```json
{
  "total_fee": 15.00,
  "message": "Payment successful"
}