# Society Management App(Gateway) - API Design

**Version:** 1.0.0
**Base URL:** `/api/v1`

This document provides a specification for the REST API. It's the single source of truth for both frontend and backend development.

---

## 1. Enroll your society (`/leads`)

Manages when the customer comes to our website for enrolling their society or for taking a demo.

### **POST** `/leads`

- **Description:** Form for enrolling the society for taking a demo or as a existing user
- **Authorization:** None(public).
- **Request Body:**
  ```json
  {
    "name": "Aditya Mishra",
    "mobile": "9876543210",
    "societyName": "Prestige Lakeside Habitation",
    "city": "Bhopal",
    "Reason": "Request a Demo" // or existing user
  }
  ```

## 2.  Enroll a New Society (Internal API)

This is the API for the form your team will use to onboard a new society after a sales lead is converted.

### **POST** `/enrollment`

- **Description:** Registers a new society and creates the primary admin account. This endpoint should be protected and only accessible by your internal team.
- **Authorization:**  Internal Admin Token required.
- **Request Body:**
  ```json
  {
    "societyName": "Prestige Lakeside Habitation",
    "registrationNumber": "KA-BNG-12345",
    "address": "Whitefield Main Road",
    "adminName": "Aditya Mishra",
    "adminEmail": "aditya.m@example.com",
    "adminPhone": "9876543210",
    "password": "a_very_strong_password"
  }
  ```
- **Successful Response (200 OK):**
  ```json
  {
    "message": "Society enrolled successfully!",
    "societyId": "a-new-society-uuid",
    "adminId": "a-new-admin-user-uuid"
  }
  ```
- **Error Response (409 Conflict - User already exists):**
  ```json
  {
    "message": "Society enrolled successfully!",
    "societyId": "a-new-society-uuid",
    "adminId": "a-new-admin-user-uuid"
  }
  ```

---

## 3. Authentication (`/auth`)

Handles user registration, login, and session management.

### **POST** `api/v1/auth/login`

Logs in a user and returns a session token.

- **Description:** A user provides their email and password to receive a JWT (JSON Web Token) for authenticating future requests.
- **Request Body:**
  ```json
  {
    "email": "email@email.com",
    "password": "user_password",
    "role": "Admin" // Can be "Admin", "Resident_Owner", "Resident_Tenant", or "Guard"
  }
  ```
- **Successful Response (200 OK):**
  ```json
  {
    "token": "a_long_encrypted_jwt_string",
    "user": {
      "id": "a_uuid_string",
      "fullName": "Rohan Sharma",
      "role": "Admin",
      "societyId": "society_uuid_string"
    }
  }
  ```
- **Error Response: Invalid Credentials (401 Unauthorized):**
  ```json
  {
    "error": "Invalid email or password."
  }
  ```
- **Error Response: Role Mismatch (403 Forbidden):**
  ```json
  {
    "error": "Access denied. You do not have permission to access this portal."
  }
  ```
- **Error Response: Inactive Account (403 Forbidden):**
  ```json
  {
    "error": "Your account has been deactivated. Please contact support."
  }
  ```

### **GET** `/auth/me`

Retrieves the profile of the currently logged-in user.

- **Description:** Uses the JWT from the `Authorization` header to identify and return the user's details.
- **Authorization:** Bearer Token required.
- **Successful Response (200 OK):**
  ```json
  {
    "id": "a_uuid_string",
    "fullName": "Rohan Sharma",
    "email": "rohan@example.com",
    "role": "Admin",
    "societyId": "society_uuid_string",
    "flatId": null
  }
  ```
- **Error Response (401 Unauthorized):**
  ```json
  {
    "error": "Authentication token is missing or invalid."
  }
  ```

---


## 4. Society & People Mangement

This module handles the core entities of the application: the society itself, its physical structure (flats), and all the people associated with it.

**Authorization**: All endpoints in this module require a Bearer Token in the Authorization header. The backend will use the user's id, role, and societyId from this token to authorize actions.


## 4.1. Get Society Details

### **GET** `/society`


- **Description:** Retrieves the details of the society that the currently authenticated user belongs to. The backend uses the societyId from the user's token to fetch the correct record.
- **Authorization:** Bearer Token required (Any role: Admin, Resident, etc., can view their own society's details).
- **Successful Response (200 OK):**
  ```json
  {
    "id": "a-society-uuid",
    "name": "Prestige Lakeside Habitation",
    "registrationNumber": "KA-BNG-12345",
    "address": "Whitefield Main Road",
    "city": "Bengaluru",
    "pincode": "560066"
  }
  ```
- **Error Response (404 Not Found):**
  ```json
  {
    "error": "Society not found."
  }
  ```


### **PUT** `/society`

- **Description:** Allows an admin to update the core details of their society, such as its name or address.
- **Authorization:**  Bearer Token required (Admin role only).

- **Request Body:**
  ```json
  {
    "name": "Prestige Lakeside Habitation (Updated)",
    "address": "New Address, Whitefield, Bengaluru"
  }
  ```
- **Successful Response (200 OK):**
  ```json
  {
    "message": "Society details updated successfully.",
    "society": {
      "id": "a-society-uuid",
      "name": "Prestige Lakeside Habitation (Updated)",
      "address": "New Address, Whitefield, Bengaluru"
    }
  }
  ```
- **Error Response (403 Forbidden):**
  ```json
  {
    "error": "You do not have permission to perform this action."
  }
  ```

## 5. Members & Flats (`/members`)

Manages the residents, owners, tenants, and staff of a society.

### **GET** `/members`

Retrieves a paginated list of all members in the society.

- **Description:** Fetches a list of all users (residents, owners, etc.) for the admin's society. Supports searching and filtering.
- **Authorization:** Bearer Token required (Admin role).
- **Query Parameters (Optional):**
  - `?search=Rohan`: Filters members by name, flat, or contact.
  - `?role=Tenant`: Filters members by a specific role.
  - `?page=1&limit=10`: For pagination.
- **Successful Response (200 OK):**
  ```json
  {
    "data": [
      {
        "id": "user_uuid_1",
        "flatNo": "A-101",
        "status": "Occupied",
        "residentName": "Rohan Sharma",
        "role": "Owner",
        "contact": "9876543210"
      },
      {
        "id": "user_uuid_2",
        "flatNo": "B-204",
        "status": "Occupied",
        "residentName": "Anita Desai",
        "role": "Tenant",
        "contact": "9123456789"
      }
    ],
    "pagination": {
      "total": 150,
      "page": 1,
      "limit": 10,
      "totalPages": 15
    }
  }
  ```

### **POST** `/members`

Creates a new member in the society.

- **Description:** Adds a new resident, owner, or staff member to the database.
- **Authorization:** Bearer Token required (Admin role).
- **Request Body:**
  ```json
  {
    "fullName": "New Member Name",
    "phoneNumber": "9998887776",
    "email": "new.member@example.com",
    "role": "Resident_Tenant",
    "flatId": "flat_uuid_string_for_their_apartment"
  }
  ```
- **Successful Response (201 Created):**
  ```json
  {
    "id": "new_user_uuid",
    "message": "Member created successfully."
  }
  ```
- **Error Response (400 Bad Request):**
  ```json
  {
    "error": "A user with this phone number already exists."
  }
  ```
