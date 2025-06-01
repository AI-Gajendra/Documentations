## API Documentation

### Authentication

All protected routes require a valid JSON Web Token (JWT) in the `Authorization` header:
`Authorization: Bearer <your_jwt_token>`

---

### 1. User Authentication & Account Management

#### 1.1. Register User
*   **Method:** `POST`
*   **Route:** `/api/auth/register`
*   **Description:** Registers a new user (initial step, might be followed by profile completion).
*   **Request Body:**
    ```json
    {
      "email": "user@example.com", // or phone
      "password": "securepassword123",
      "accountType": "Patient" // Enum: "Patient", "Doctor", "Business", "Researcher", "Laboratory", "Hospital"
    }
    ```
*   **Response Body (Success 201):**
    ```json
    {
      "message": "OTP sent to your email/phone. Please verify.",
      "userId": "60d5f1b5f9d7a40015e9b0a1" // Temporary or actual user ID
    }
    ```
*   **Status Codes:**
    *   `201 Created`: Registration initiated, OTP sent.
    *   `400 Bad Request`: Invalid input (e.g., email format, weak password, missing fields).
    *   `409 Conflict`: Email/phone already exists.

#### 1.2. Verify OTP
*   **Method:** `POST`
*   **Route:** `/api/auth/verify-otp`
*   **Description:** Verifies the OTP sent during registration or password reset.
*   **Request Body:**
    ```json
    {
      "userId": "60d5f1b5f9d7a40015e9b0a1", // or email/phone
      "otp": "123456",
      "purpose": "registration" // Enum: "registration", "password_reset", "login"
    }
    ```
*   **Response Body (Success 200 for registration/reset, or token for login):**
    ```json
    // For registration/reset completion
    {
      "message": "OTP verified successfully. Please complete your profile / set new password."
    }
    // For OTP-based login
    {
      "token": "jwt.token.here",
      "user": {
        "id": "60d5f1b5f9d7a40015e9b0a1",
        "email": "user@example.com",
        "accountType": "Patient",
        "isProfileComplete": false
      }
    }
    ```
*   **Status Codes:**
    *   `200 OK`: OTP verified.
    *   `400 Bad Request`: Invalid OTP, expired OTP, missing fields.
    *   `404 Not Found`: User not found.

#### 1.3. Resend OTP
*   **Method:** `POST`
*   **Route:** `/api/auth/resend-otp`
*   **Description:** Resends an OTP.
*   **Request Body:**
    ```json
    {
      "email": "user@example.com", // or phone or userId
      "purpose": "registration" // Enum: "registration", "password_reset", "login"
    }
    ```
*   **Response Body (Success 200):**
    ```json
    {
      "message": "OTP resent successfully."
    }
    ```
*   **Status Codes:**
    *   `200 OK`: OTP resent.
    *   `400 Bad Request`: Missing fields.
    *   `404 Not Found`: User not found.
    *   `429 Too Many Requests`: Rate limit exceeded.

#### 1.4. Login User
*   **Method:** `POST`
*   **Route:** `/api/auth/login`
*   **Description:** Logs in an existing user.
*   **Request Body:**
    ```json
    {
      "email": "user@example.com", // or phone
      "password": "securepassword123"
    }
    ```
*   **Response Body (Success 200):**
    ```json
    {
      "token": "jwt.token.here",
      "user": {
        "id": "60d5f1b5f9d7a40015e9b0a1",
        "email": "user@example.com",
        "name": "John Doe",
        "accountType": "Patient",
        "isProfileComplete": true,
        "profilePictureUrl": "url/to/image.jpg"
      }
    }
    ```
*   **Status Codes:**
    *   `200 OK`: Login successful.
    *   `400 Bad Request`: Missing fields.
    *   `401 Unauthorized`: Invalid credentials.
    *   `403 Forbidden`: Account pending approval or blocked.

#### 1.5. Social Login/Signup
*   **Method:** `POST`
*   **Route:** `/api/auth/social/{provider}` (e.g., `/api/auth/social/google`, `/api/auth/social/apple`)
*   **Description:** Handles login/signup via social providers.
*   **Request Body:**
    ```json
    {
      "token": "provider_access_token_or_id_token",
      "accountType": "Patient" // Required if it's a new signup
    }
    ```
*   **Response Body (Success 200):** (Same as regular login)
*   **Status Codes:**
    *   `200 OK`: Login/Signup successful.
    *   `400 Bad Request`: Invalid provider token, missing accountType for new user.
    *   `401 Unauthorized`: Provider token validation failed.

#### 1.6. Forgot Password
*   **Method:** `POST`
*   **Route:** `/api/auth/forgot-password`
*   **Description:** Initiates the password reset process.
*   **Request Body:**
    ```json
    {
      "email": "user@example.com" // or phone
    }
    ```
*   **Response Body (Success 200):**
    ```json
    {
      "message": "Password reset OTP sent to your email/phone."
    }
    ```
*   **Status Codes:**
    *   `200 OK`.
    *   `400 Bad Request`: Missing fields.
    *   `404 Not Found`: Email/phone not found.

#### 1.7. Reset Password
*   **Method:** `POST`
*   **Route:** `/api/auth/reset-password`
*   **Description:** Sets a new password after OTP verification.
*   **Request Body:**
    ```json
    {
      "email": "user@example.com", // or phone or resetToken from email link
      "otp": "123456", // If OTP based
      "newPassword": "newSecurePassword456"
    }
    ```
*   **Response Body (Success 200):**
    ```json
    {
      "message": "Password reset successfully."
    }
    ```
*   **Status Codes:**
    *   `200 OK`.
    *   `400 Bad Request`: Invalid OTP/token, weak password.
    *   `404 Not Found`: User not found.

#### 1.8. Logout User (Client-side token removal usually, but can have a server endpoint)
*   **Method:** `POST`
*   **Route:** `/api/auth/logout`
*   **Description:** Logs out the user (e.g., invalidates token if using a refresh token strategy or server-side session).
*   **Request Body:** (Empty or may contain refresh token)
*   **Response Body (Success 200):**
    ```json
    {
      "message": "Logged out successfully."
    }
    ```
*   **Status Codes:**
    *   `200 OK`.
    *   `401 Unauthorized`: Not authenticated.

---

### 2. User Profiles (Patient, Doctor, Business)

#### 2.1. Get My Profile
*   **Method:** `GET`
*   **Route:** `/api/users/me/profile`
*   **Description:** Retrieves the profile details of the authenticated user. Response structure varies by `accountType`.
*   **Request Body:** (None)
*   **Response Body (Success 200 - Patient Example):**
    ```json
    {
      "id": "60d5f1b5f9d7a40015e9b0a1",
      "fullName": "Johnsan Watson",
      "email": "johnsan@example.com",
      "phone": "+919876543210",
      "address": "123 Main St, Anytown",
      "age": 30,
      "gender": "Male",
      "maritalStatus": "Single",
      "profilePictureUrl": "url/to/image.jpg",
      "accountType": "Patient",
      "aadharNumber": "xxxx-xxxx-1234", // Masked or full based on context
      // ... other patient-specific fields from "Create Your Account" screen
    }
    ```
*   **Status Codes:**
    *   `200 OK`.
    *   `401 Unauthorized`.
    *   `404 Not Found`: Profile not yet created (if separate from registration).

#### 2.2. Create/Update My Profile
*   **Method:** `PUT` (or `POST` if profile creation is a distinct step after registration)
*   **Route:** `/api/users/me/profile`
*   **Description:** Creates or updates the profile of the authenticated user.
*   **Request Body (Patient Example from "Create Your Account"):**
    ```json
    {
      "fullName": "Johnsan Watson",
      "phone": "+919876543210",
      "address": { // Could be a nested object or reference to an Address ID
        "addressLine1": "123 Main St",
        "addressLine2": "Apt 4B",
        "area": "Downtown",
        "village": "",
        "landmark": "Near City Hall",
        "city": "Anytown",
        "state": "Anystate",
        "postalCode": "12345"
      },
      "age": 30,
      "gender": "Male",
      "maritalStatus": "Single",
      "aadharNumber": "1234-5678-9012",
      "ethnicCategory": "Asian", // from "Which category describes you?"
      // ... other fields
    }
    ```
    **Request Body (Doctor Example - from "Doctor Profile Creation"):**
    ```json
    {
      "fullName": "Dr. Jane Smith",
      "country": "India",
      "phone": "+919876543211",
      "email": "drjane@example.com", // May be pre-filled
      "address": "Clinic Address...",
      "gender": "Female",
      "panNumber": "ABCDE1234F",
      "dateOfRegistration": "2010-05-15",
      "medicalSpecialty": ["Cardiologist", "General Practitioner"],
      "medicalCouncilCertificateUrl": "url/to/certificate.pdf", // Link after upload
      "idCardUrl": "url/to/idcard.jpg" // Link after upload
    }
    ```
    **Request Body (Business Example - from "Business Profile Creation"):**
    ```json
    {
        "businessName": "John Medical Store",
        "businessType": "Pharmacy", // Wellness, Medical Devices
        "businessRegistrationNumber": "U12345XYZ",
        "businessPhone": "+919000000000",
        "email": "store@example.com",
        "country": "India",
        "postalCode": "500001",
        "address": "Store Address...",
        "businessDescription": "Your friendly neighborhood pharmacy.",
        "businessLogoUrl": "url/to/logo.png", // Link after upload
        "idCardUrl": "url/to/owner_id.jpg", // Link after upload (Live Picture related)
        "businessDocuments": ["url/to/doc1.pdf"] // Link after upload
    }
    ```
*   **Response Body (Success 200):** Updated profile object (same as GET).
*   **Status Codes:**
    *   `200 OK`: Profile updated.
    *   `201 Created`: Profile created (if POST).
    *   `400 Bad Request`: Invalid input.
    *   `401 Unauthorized`.

#### 2.3. Upload Profile Picture / Document
*   **Method:** `POST`
*   **Route:** `/api/users/me/documents`
*   **Description:** Uploads a document (profile picture, ID card, medical certificate).
*   **Request Body:** `multipart/form-data` with a field like `file` and `documentType`.
    ```
    // Form data fields
    file: (binary_file_data)
    documentType: "profilePicture" // "idCard", "medicalCouncilCertificate", "businessLogo", "businessDocument"
    ```
*   **Response Body (Success 200):**
    ```json
    {
      "message": "File uploaded successfully.",
      "fileUrl": "https://cdn.example.com/path/to/uploaded_file.jpg",
      "documentType": "profilePicture"
    }
    ```
*   **Status Codes:**
    *   `200 OK`.
    *   `400 Bad Request`: No file, invalid file type, missing documentType.
    *   `401 Unauthorized`.

#### 2.4. Get User Registration Status (For pending approvals)
*   **Method:** `GET`
*   **Route:** `/api/users/me/registration-status`
*   **Description:** Checks if the user's account (especially for Doctors/Businesses) is approved.
*   **Response Body (Success 200):**
    ```json
    {
      "status": "Pending" // "Approved", "Rejected"
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

---

### 3. Addresses

#### 3.1. List My Addresses
*   **Method:** `GET`
*   **Route:** `/api/users/me/addresses`
*   **Description:** Retrieves all addresses for the authenticated user.
*   **Response Body (Success 200):**
    ```json
    [
      {
        "id": "60d5f1c3f9d7a40015e9b0a2",
        "name": "Home", // User-defined label for the address
        "fullName": "Johnsan Watson", // Recipient name
        "phoneNumber": "(603) 555-0123",
        "addressLine1": "123 Main St",
        "addressLine2": "Apt 4B",
        "area": "Downtown",
        "village": "",
        "landmark": "Near City Hall",
        "city": "Anytown",
        "state": "Anystate",
        "isDefault": true
      }
    ]
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 3.2. Add New Address
*   **Method:** `POST`
*   **Route:** `/api/users/me/addresses`
*   **Description:** Adds a new address for the authenticated user.
*   **Request Body (from "Add New Address" screen):**
    ```json
    {
      "name": "Work", // User-defined label
      "fullName": "Johnsan Watson",
      "phoneNumber": "(603) 555-0123",
      "addressLine1": "456 Office Park",
      "addressLine2": "Suite 100",
      "area": "Business District",
      "village": "",
      "landmark": "Next to Cafe",
      "city": "Anytown",
      "state": "Anystate"
    }
    ```
*   **Response Body (Success 201):** The newly created address object.
*   **Status Codes:** `201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 3.3. Update Address
*   **Method:** `PUT`
*   **Route:** `/api/users/me/addresses/{addressId}`
*   **Description:** Updates an existing address.
*   **Request Body:** Same as Add New Address.
*   **Response Body (Success 200):** The updated address object.
*   **Status Codes:** `200 OK`, `400 Bad Request`, `401 Unauthorized`, `404 Not Found`.

#### 3.4. Delete Address
*   **Method:** `DELETE`
*   **Route:** `/api/users/me/addresses/{addressId}`
*   **Description:** Deletes an address.
*   **Response Body (Success 204):** (No content)
*   **Status Codes:** `204 No Content`, `401 Unauthorized`, `404 Not Found`.

#### 3.5. Set Default Address
*   **Method:** `PUT`
*   **Route:** `/api/users/me/addresses/{addressId}/default`
*   **Description:** Sets a specific address as the default.
*   **Response Body (Success 200):**
    ```json
    { "message": "Default address updated." }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `404 Not Found`.

---

### 4. Payment Methods

#### 4.1. List My Payment Methods
*   **Method:** `GET`
*   **Route:** `/api/users/me/payment-methods`
*   **Description:** Retrieves saved payment methods (cards) for the user.
*   **Response Body (Success 200):**
    ```json
    [
      {
        "id": "card_1J...",
        "type": "Credit/Debit Card",
        "brand": "MasterCard", // "Visa"
        "last4": "9876",
        "expMonth": 8,
        "expYear": 2028,
        "cardHolderName": "Johnsan Watson",
        "isDefault": true
      }
    ]
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 4.2. Add New Card (Payment Method)
*   **Method:** `POST`
*   **Route:** `/api/users/me/payment-methods`
*   **Description:** Adds a new card. This would typically involve tokenizing the card details with a payment gateway (e.g., Stripe, Razorpay) on the client-side, and then sending the payment method token to your backend.
*   **Request Body (from "Add New Card" screen, after tokenization):**
    ```json
    {
      "paymentMethodToken": "pm_xxxxxxxxxxxxxx", // Token from Stripe.js/Razorpay SDK
      "cardHolderName": "Vikash", // Or "Johnsan Watson"
      "saveCard": true // From "Save Card Details" or "Use as default payment method"
    }
    ```
*   **Response Body (Success 201):** The newly added payment method object.
*   **Status Codes:** `201 Created`, `400 Bad Request` (invalid token, gateway error), `401 Unauthorized`.

#### 4.3. Set Default Payment Method
*   **Method:** `PUT`
*   **Route:** `/api/users/me/payment-methods/{cardId}/default`
*   **Description:** Sets a card as the default payment method.
*   **Response Body (Success 200):**
    ```json
    { "message": "Default payment method updated." }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `404 Not Found`.

#### 4.4. Delete Payment Method
*   **Method:** `DELETE`
*   **Route:** `/api/users/me/payment-methods/{cardId}`
*   **Description:** Deletes a saved payment method.
*   **Response Body (Success 204):** (No content)
*   **Status Codes:** `204 No Content`, `401 Unauthorized`, `404 Not Found`.

---

### 5. Health Records (Patient-specific)

#### 5.1. Add/Update Genetic Information
*   **Method:** `POST` or `PUT`
*   **Route:** `/api/users/me/health-records/genetic`
*   **Description:** Adds or updates patient's genetic information (from "Add your genetic information" screen).
*   **Request Body:**
    ```json
    {
      "fillerName": "Self",
      "patientWorkDescription": "Software Engineer",
      "relationshipToPatient": "Self",
      "accompanyPerson": "None",
      "referringPhysicianName": "Dr. Emily Carter",
      "practiceName": "City General Hospital",
      "pcpAddress": "123 Health St",
      "pcpPhoneNumber": "+919998887776",
      "referredByHealthCareProvider": true, // Yes/No
      "referringHealthCareProvider": "Dr. Emily Carter",
      "specialty": "Genetics",
      "reasonForGeneticEvaluation": "Family history of heart disease."
    }
    ```
*   **Response Body (Success 200/201):** Updated/created genetic info object.
*   **Status Codes:** `200 OK`/`201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 5.2. Get Genetic Results
*   **Method:** `GET`
*   **Route:** `/api/users/me/health-records/genetic-results`
*   **Description:** Retrieves patient's genetic test results (from "Your Genetic Result" screen).
*   **Response Body (Success 200):**
    ```json
    {
      "patientName": "Parkash Singanya",
      "city": "Delhi",
      "gender": "Male",
      "reportDate": "2023-10-26T10:50:00Z",
      "sampleId": "MHSKSJHUSS",
      "conditions": [
        {
          "category": "Cardiovascular Diseases",
          "iconUrl": "url/to/cardio_icon.png",
          "results": [
            { "conditionName": "Atrial Fibrillation", "riskLevel": "Increased", "details": "Increased risk for Atrial Fibrillation" },
            // ... more results
          ]
        },
        {
          "category": "Endocrine",
          "iconUrl": "url/to/endocrine_icon.png",
          "results": [
            // ...
          ]
        }
        // ... more categories
      ]
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `404 Not Found`.

#### 5.3. Add/Update Menstrual History
*   **Method:** `POST` or `PUT`
*   **Route:** `/api/users/me/health-records/menstrual-history`
*   **Description:** Adds or updates menstrual history (from "Find Doctor" screen).
*   **Request Body:**
    ```json
    {
      "cyclePattern": ["Light periods"], // Array of selected patterns
      "daysBetweenPeriods": "25 Days Gap",
      "daysOfBleeding": "7 Days"
      // ... other fields
    }
    ```
*   **Response Body (Success 200/201):** Updated/created menstrual history object.
*   **Status Codes:** `200 OK`/`201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 5.4. Add/Update Health Insurance
*   **Method:** `POST` or `PUT`
*   **Route:** `/api/users/me/health-records/insurance`
*   **Description:** Adds or updates health insurance details.
*   **Request Body (from "Health Insurance" screen):**
    ```json
    {
      "primaryInsurance": {
        "companyName": "HealthGuard Inc.",
        "policyHolderName": "Johnsan Watson",
        "policyNumber": "HG123456789",
        "typeOfInsurancePlan": "Family Floater Health Insurance",
        "claimMailingAddress": "Claims Dept, HealthGuard Inc., PO Box 123"
      },
      "secondaryInsurance": null // or similar object
    }
    ```
*   **Response Body (Success 200/201):** Updated/created insurance object.
*   **Status Codes:** `200 OK`/`201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 5.5. Add/Update Social History (Alcohol, Smoking, etc.)
*   **Method:** `POST` or `PUT`
*   **Route:** `/api/users/me/health-records/social-history`
*   **Description:** Adds or updates social history (alcohol, smoking, drugs, exercise).
*   **Request Body (from "Alcohol Frequency" & "Social History" screens):**
    ```json
    {
      "alcohol": {
        "unitsConsumedPerSession": "Select", // Or specific value
        "frequency6OrMoreUnitsFemale8OrMoreMale": "Daily 4 Or Almost Daily",
        // ... other alcohol questions
      },
      "smoking": {
        "caffeinatedBeveragesPerDay": "1 cup of coffee",
        "smokesCigarettes": true,
        "howManyPerDay": "10",
        "howManyYears": "5",
        "quitYear": null,
        "exposedToSecondHandSmoke": false
      },
      "drugUse": {
        "usesMarijuanaCocaineEtc": false,
        "description": ""
      },
      "exercise": {
        "exercisesRegularly": true,
        "hoursPerWeek": " Vigorous (i.e. running)", // Or Moderate
        "feelsSafeAtHome": true,
        "explanation": ""
      }
      // ... Health Habits and Personal Safety
    }
    ```
*   **Response Body (Success 200/201):** Updated/created social history object.
*   **Status Codes:** `200 OK`/`201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 5.6. Add/Update Drug History
*   **Method:** `POST` or `PUT`
*   **Route:** `/api/users/me/health-records/drug-history`
*   **Description:** Adds or updates current medications (from "Drug History" screen).
*   **Request Body:**
    ```json
    {
      "medications": [
        { "category": "Cardiology", "subCategory": "Beta Blocker", "drugName": "Carvedilol", "isTaking": true },
        { "category": "Cardiology", "subCategory": "Beta Blocker", "drugName": "Metoprolol", "isTaking": false },
        { "category": "Cardiology", "subCategory": "Statins", "drugName": "Rosuvastatin", "isTaking": true }
        // ... more drugs
      ]
    }
    ```
*   **Response Body (Success 200/201):** Updated/created drug history object.
*   **Status Codes:** `200 OK`/`201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 5.7. Get Drug Interaction Report
*   **Method:** `GET`
*   **Route:** `/api/users/me/health-records/drug-interactions`
*   **Description:** Generates a report based on the user's drug history (from "Drug Report" screen).
*   **Response Body (Success 200):**
    ```json
    [
      {
        "category": "Cardiology",
        "subCategory": "Beta Blocker",
        "drugs": [
          { "genericDrug": "Carvedilol", "actionIcon": "caution", "actionText": "Use with caution" },
          // ...
        ]
      },
      // ... more categories
    ]
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 5.8. Add/Update Vaccination History
*   **Method:** `POST` or `PUT`
*   **Route:** `/api/users/me/health-records/vaccinations`
*   **Description:** Adds or updates vaccination history.
*   **Request Body (from "Vaccination History" screen):**
    ```json
    {
      "vaccines": [
        { "name": "Influenza (flu) vaccine", "date": "2023-10-01", "group": "All Patients" },
        { "name": "Tetanus vaccine", "date": "2020-01-15", "group": "All Patients" }
        // ... more vaccines
      ],
      "otherPhysicians": [
        { "physicianName": "Dr. House", "specialty": "Cardiologist" }
      ],
      "upToDateOn": [
        { "name": "Tdap", "date": "2023-01-01"}
      ],
      "screenings": [
        { "name": "Colonoscopy", "date": "2022-05-10"}
      ]
    }
    ```
*   **Response Body (Success 200/201):** Updated/created vaccination object.
*   **Status Codes:** `200 OK`/`201 Created`, `400 Bad Request`, `401 Unauthorized`.

#### 5.9. Manage Perceptions (Prescriptions - Patient Uploaded)
*   **Method:** `POST`
*   **Route:** `/api/users/me/perceptions`
*   **Description:** Uploads a new perception/prescription document (from "Add New Perceptions" screen).
*   **Request Body:** `multipart/form-data`
    ```
    // Form data fields
    file: (binary_file_data) // PNG, PDF, JPG
    title: "Xyz Problem Medicine"
    additionalDetails: "Notes about this perception"
    ```
*   **Response Body (Success 201):**
    ```json
    {
      "id": "perception_id_123",
      "title": "Xyz Problem Medicine",
      "fileUrl": "url/to/perception.jpg",
      "uploadedAt": "2023-10-26T09:41:00Z",
      "additionalDetails": "Notes about this perception"
    }
    ```
*   **Status Codes:** `201 Created`, `400 Bad Request`, `401 Unauthorized`.

*   **Method:** `GET`
*   **Route:** `/api/users/me/perceptions`
*   **Description:** Lists all uploaded perceptions for the patient.
*   **Response Body (Success 200):** Array of perception objects (like above).
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 5.10. Medical Consent
*   **Method:** `POST`
*   **Route:** `/api/users/me/consent`
*   **Description:** Submits the medical consent form.
*   **Request Body (from "Medical Consent Form" screen):**
    ```json
    {
      "authorizePersonName": "John Doe Sr.", // If minor
      "yourId": "121717717171", // Aadhar or other ID
      "consentGiven": true
    }
    ```
*   **Response Body (Success 200):**
    ```json
    { "message": "Consent recorded successfully." }
    ```
*   **Status Codes:** `200 OK`, `400 Bad Request`, `401 Unauthorized`.

#### 5.11. Patient Health Data Overview (For Doctor viewing Patient)
*   **Method:** `GET`
*   **Route:** `/api/doctors/patients/{patientId}/health-data`
*   **Description:** Retrieves a summary of patient's health data (vitals like Heart Rate, Blood Pressure, Glucose, Weight, Cholesterol).
*   **Request Body:** (None)
*   **Response Body (Success 200):**
    ```json
    {
      "heartRate": { "current": "70 bpm", "date": "01 August, 2024", "status": "Normal", "history": [/* ... */] },
      "bloodPressure": { "current": "120/80 mmHg", "systolic": 120, "diastolic": 80, "status": "Normal", "history": [/* ... */] },
      "bloodGlucose": { "current": "90 mg/dL (Fasting)", "date": "01 August, 2024", "history": [/* ... */] },
      "weight": { "current": "72 kg", "date": "01 August, 2024", "history": [/* ... */] },
      "cholesterol": { "current": "110 mg/dL", "hdl": 50, "ldl": 60, "date": "01 August, 2024", "history": [/* ... */] }
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized` (Doctor not authenticated), `403 Forbidden` (Doctor not authorized for this patient), `404 Not Found` (Patient not found).

#### 5.12. Patient Reports (For Doctor viewing Patient)
*   **Method:** `GET`
*   **Route:** `/api/doctors/patients/{patientId}/reports`
*   **Description:** Lists test reports and clinical records for a patient.
*   **Response Body (Success 200):**
    ```json
    {
      "tests": [
        { "id": "report1", "name": "Blood Glucose Test", "date": "01 August, 2024", "viewUrl": "/api/reports/report1/view" },
        // ...
      ],
      "clinicalRecords": [
        { "id": "record1", "name": "Operation Name", "date": "01 August, 2024", "viewUrl": "/api/reports/record1/view" }
        // ...
      ]
    }
    ```
*   **Status Codes:** Same as Health Data Overview.

#### 5.13. Add New Report / Clinical Record (By Doctor for Patient)
*   **Method:** `POST`
*   **Route:** `/api/doctors/patients/{patientId}/reports` (or `/clinical-records`)
*   **Description:** Doctor adds a new report or clinical record for a patient.
*   **Request Body:** `multipart/form-data`
    ```
    // Form data fields
    file: (binary_file_data)
    title: "Blood Test Results"
    reportType: "Blood Glucose Test" // Or a category
    date: "2024-08-01"
    description: "Fasting blood sugar levels."
    // ... other fields from "Add New Report" screen
    ```
*   **Response Body (Success 201):** The created report object.
*   **Status Codes:** `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

---

### 6. Doctors & Appointments

#### 6.1. List Specialties
*   **Method:** `GET`
*   **Route:** `/api/specialties`
*   **Description:** Retrieves a list of medical specialties.
*   **Response Body (Success 200):**
    ```json
    [
      { "id": "spec1", "name": "Gastroenterologist (stomach and stomach-related issues)" },
      { "id": "spec2", "name": "Endocrine/Hormonal (Diabetes, hair loss, thyroid gland problems, rapid weight gain or loss)" }
      // ...
    ]
    ```
*   **Status Codes:** `200 OK`.

#### 6.2. Search/List Doctors
*   **Method:** `GET`
*   **Route:** `/api/doctors`
*   **Description:** Searches for doctors based on criteria (specialty, symptoms, name, location).
*   **Query Parameters:** `specialtyId`, `symptom`, `name`, `location`, `page`, `limit`.
*   **Response Body (Success 200):**
    ```json
    {
      "doctors": [
        {
          "id": "doc123",
          "name": "Doctor Name",
          "specialty": "Cardiologist",
          "rating": 4.8,
          "consultationFee": 162.92,
          "imageUrl": "url/to/doctor_image.jpg",
          "description": "Dedicated and experienced...",
          "distance": "1.0 km away" // If location search
        }
      ],
      "total": 25, "page": 1, "limit": 10
    }
    ```
*   **Status Codes:** `200 OK`.

#### 6.3. Get Doctor Details
*   **Method:** `GET`
*   **Route:** `/api/doctors/{doctorId}`
*   **Description:** Retrieves detailed information about a specific doctor.
*   **Response Body (Success 200):** (Similar to list item but more details, including availability schedule).
    ```json
    {
      "id": "doc123",
      "name": "Doctor Name",
      "specialty": "Cardiologist",
      "rating": 4.8,
      "consultationFee": 162.92,
      "imageUrl": "url/to/doctor_image.jpg",
      "description": "Dedicated and experienced...",
      "availability": [ // From "Setup Availability" for this doctor
        { "date": "2024-07-16", "slots": ["09:00 AM", "10:00 AM", "11:00 AM", "02:00 PM", "03:00 PM", "08:00 PM"] },
        // ... more dates
      ],
      "clinicInfo": { /* ... */ }
    }
    ```
*   **Status Codes:** `200 OK`, `404 Not Found`.

#### 6.4. Get Doctor Availability (for booking)
*   **Method:** `GET`
*   **Route:** `/api/doctors/{doctorId}/availability`
*   **Query Parameters:** `date` (e.g., YYYY-MM-DD)
*   **Description:** Retrieves available time slots for a doctor on a specific date or date range.
*   **Response Body (Success 200):**
    ```json
    [ // Array of available dates with slots
      {
        "date": "2024-07-16",
        "day": "Thursday",
        "shifts": [
             {"label": "Shift A: 8am To 8pm", "slots": ["09:00 AM", "10:00 AM", ...]},
             {"label": "Shift B: 8am To 8pm", "slots": [...]}
        ]
      }
      // ... for multiple dates if no specific date query param
    ]
    ```
*   **Status Codes:** `200 OK`, `404 Not Found` (Doctor not found).

#### 6.5. Setup Doctor Availability (Doctor's own endpoint)
*   **Method:** `PUT`
*   **Route:** `/api/doctors/me/availability`
*   **Description:** Doctor sets up their working hours, irregular off days, appointment fee, and patient capacity.
*   **Request Body (from "Setup Availability" screen):**
    ```json
    {
      "workingHours": [ // For recurring weekly schedule
        { "dayOfWeek": "Mon", "slots": ["09:00 AM", "10:00 AM", "..."] },
        { "dayOfWeek": "Thu", "slots": ["10:00 PM", "11:00 AM", "02:00 PM", "03:00 PM", "08:00 PM"] }
        // ...
      ],
      "irregularOff": [
        { "date": "2024-07-16", "reason": "Conference" }
      ],
      "appointmentFee": 17.00,
      "maxPatientsPerDay": 4
    }
    ```
*   **Response Body (Success 200):**
    ```json
    { "message": "Availability updated successfully." }
    ```
*   **Status Codes:** `200 OK`, `400 Bad Request`, `401 Unauthorized`.

#### 6.6. Book an Appointment
*   **Method:** `POST`
*   **Route:** `/api/appointments`
*   **Description:** Patient books an appointment with a doctor.
*   **Request Body (from "My Booking" screen, after selecting doctor, slot, and before payment):**
    ```json
    {
      "doctorId": "doc123",
      "patientId": "user_self_id", // Can be inferred from auth token if patient is booking for self
      "appointmentDateTime": "2024-07-16T10:00:00Z",
      "appointmentType": "Physical", // "Virtual"
      "symptoms": "Severe stomach pain", // Optional
      "notes": "Previous history of gastritis.", // Optional
      "documentUrls": ["url/to/uploaded_doc1.pdf"], // Optional, from "Upload Documents"
      "couponCode": "SAVE10" // Optional
    }
    ```
*   **Response Body (Success 201 - before payment):**
    ```json
    {
      "appointmentId": "appt_xyz",
      "doctorId": "doc123",
      "patientId": "user_self_id",
      "appointmentDateTime": "2024-07-16T10:00:00Z",
      "status": "PendingPayment", // or "Confirmed" if no payment or paid upfront
      "appointmentCost": 156.00,
      "adminFee": 10.00,
      "discount": 0.00, // if coupon applied
      "totalAmount": 166.00,
      "paymentIntentId": "pi_xxxx" // If payment is required, for client to complete payment
    }
    ```
*   **Status Codes:** `201 Created`, `400 Bad Request` (slot unavailable, invalid doctorId), `401 Unauthorized`.

#### 6.7. Confirm Appointment Payment (Webhook or explicit call)
*   **Method:** `POST`
*   **Route:** `/api/appointments/{appointmentId}/payment-confirmation`
*   **Description:** Confirms that payment for an appointment was successful. Often a webhook from payment gateway.
*   **Request Body (from payment gateway or client after successful payment):**
    ```json
    {
      "transactionId": "txn_12345",
      "paymentGatewayResponse": { /* ... gateway specific data ... */ }
    }
    ```
*   **Response Body (Success 200):**
    ```json
    {
      "appointmentId": "appt_xyz",
      "status": "Confirmed",
      "message": "Appointment confirmed."
    }
    ```
*   **Status Codes:** `200 OK`, `400 Bad Request` (invalid appointmentId, payment verification failed).

#### 6.8. List My Appointments (Patient or Doctor)
*   **Method:** `GET`
*   **Route:** `/api/users/me/appointments` (for patient) OR `/api/doctors/me/appointments` (for doctor)
*   **Query Parameters:** `status=Upcoming` or `status=Past`, `page`, `limit`.
*   **Description:** Lists appointments for the authenticated user.
*   **Response Body (Success 200 - "My Bookings" / "Appointments" screen):**
    ```json
    {
      "appointments": [
        {
          "id": "appt_xyz",
          "orderId": "58967895", // Or same as id
          "doctorName": "Doctor Name", // For patient view
          "patientName": "Jane Cooper", // For doctor view
          "patientAgeGender": "56 Years Old (Male)", // For doctor view
          "specialty": "Cardiology",
          "imageUrl": "url/to/doctor_or_patient_image.jpg",
          "appointmentDateTime": "2022-06-25T10:00:00Z", // "June 25, 2022, 10:00 PM - 03:00 PM" - duration might be fixed
          "status": "Upcoming", // "Past", "Cancelled"
          "consultationType": "Physical", // "Virtual"
          "location": "1.0 km away" // If physical
        }
      ],
      "total": 15, "page": 1, "limit": 10
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 6.9. Get Appointment Details
*   **Method:** `GET`
*   **Route:** `/api/appointments/{appointmentId}`
*   **Description:** Retrieves details of a specific appointment.
*   **Response Body (Success 200):** (Similar to list item, but with more details like notes, uploaded documents).
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden` (if user not part of appointment), `404 Not Found`.

#### 6.10. Cancel Appointment
*   **Method:** `POST` (or `PUT`)
*   **Route:** `/api/appointments/{appointmentId}/cancel`
*   **Description:** Cancels an appointment (by patient or doctor).
*   **Request Body:**
    ```json
    {
      "reason": "Patient unavailable" // Optional
    }
    ```
*   **Response Body (Success 200):**
    ```json
    { "message": "Appointment cancelled successfully." }
    ```
*   **Status Codes:** `200 OK`, `400 Bad Request` (cannot cancel, e.g., too close to appointment time), `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 6.11. Start Consultation (For Virtual Appointments)
*   **Method:** `POST`
*   **Route:** `/api/appointments/{appointmentId}/start-consultation`
*   **Description:** Marks a virtual consultation as started, potentially generates video call tokens/links.
*   **Response Body (Success 200):**
    ```json
    {
      "message": "Consultation started.",
      "videoCallDetails": {
        "service": "Agora/Twilio/Jitsi",
        "roomId": "secure_room_id",
        "token": "session_token_for_user"
      }
    }
    ```
*   **Status Codes:** `200 OK`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 6.12. List Doctor's Patients (Doctor View)
*   **Method:** `GET`
*   **Route:** `/api/doctors/me/patients`
*   **Description:** Lists all patients associated with the authenticated doctor.
*   **Query Parameters:** `search`, `page`, `limit`.
*   **Response Body (Success 200 - "My Patients" screen):**
    ```json
    {
      "patients": [
        {
          "id": "patient_id_1",
          "name": "Patient Name",
          "imageUrl": "url/to/patient_image.jpg",
          "lastAppointmentDate": "09-Sept-2024",
          "totalAppointments": 15
        }
      ],
      "total": 50, "page": 1, "limit": 10
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

---

### 7. Subscriptions & Plans

#### 7.1. List Subscription Plans
*   **Method:** `GET`
*   **Route:** `/api/subscriptions/plans`
*   **Description:** Retrieves available subscription plans (Lite, Dr.AI, PROGNOZ.AI).
*   **Response Body (Success 200 - from "Profiles" screen):**
    ```json
    [
      {
        "id": "plan_lite",
        "name": "Lite Profile",
        "price": 9.99,
        "currency": "USD", // Or "INR"
        "billingCycle": "month",
        "features": [
          "144/CONSULTATION FOR 15 MINS WITH MBBS DOCTORS",
          "243/CONSULATION FOR 15 MINS"
        ]
      },
      // ... other plans
    ]
    ```
*   **Status Codes:** `200 OK`.

#### 7.2. Subscribe to a Plan
*   **Method:** `POST`
*   **Route:** `/api/subscriptions`
*   **Description:** User subscribes to a plan. Involves payment.
*   **Request Body:**
    ```json
    {
      "planId": "plan_prognoz_ai",
      "paymentMethodId": "card_1J..." // From user's saved payment methods or a new one
    }
    ```
*   **Response Body (Success 201):**
    ```json
    {
      "subscriptionId": "sub_xxxx",
      "planId": "plan_prognoz_ai",
      "status": "Active", // or "PendingPayment"
      "nextBillingDate": "2024-08-26",
      "message": "Subscription successful."
    }
    ```
*   **Status Codes:** `201 Created`, `400 Bad Request` (invalid plan, payment failed), `401 Unauthorized`, `402 Payment Required` (if payment step fails but intent created).

---

### 8. Reviews

#### 8.1. Add a Review
*   **Method:** `POST`
*   **Route:** `/api/reviews`
*   **Description:** User adds a review for a doctor or product.
*   **Request Body (from "Write a Review" screen):**
    ```json
    {
      "targetType": "Doctor", // "Product"
      "targetId": "doc123", // Doctor ID or Product ID
      "bookingId": "appt_xyz", // Optional, to link review to a specific consultation/order
      "rating": 5, // 1-5
      "comment": "Great care!"
    }
    ```
*   **Response Body (Success 201):** The created review object.
*   **Status Codes:** `201 Created`, `400 Bad Request` (already reviewed, invalid rating), `401 Unauthorized`.

#### 8.2. Get Reviews for Target
*   **Method:** `GET`
*   **Route:** `/api/reviews/target/{targetType}/{targetId}` (e.g., `/api/reviews/target/Doctor/doc123`)
*   **Description:** Lists reviews for a specific doctor or product.
*   **Query Parameters:** `page`, `limit`.
*   **Response Body (Success 200):**
    ```json
    {
      "reviews": [
        {
          "id": "review1",
          "userName": "Person Name 2",
          "userImageUrl": "url/to/user_image.jpg",
          "rating": 4.5,
          "comment": "Top-notch service!",
          "createdAt": "2023-10-20T10:00:00Z"
        }
      ],
      "averageRating": 4.7,
      "totalReviews": 150,
      "ratingDistribution": { "5": 75, "4": 16, "3": 5, "2": 1, "1": 3 } // Percentages or counts
    }
    ```
*   **Status Codes:** `200 OK`, `404 Not Found` (Target not found).

---

### 9. Chat / Messaging

#### 9.1. List My Chats
*   **Method:** `GET`
*   **Route:** `/api/chats`
*   **Description:** Retrieves a list of chat conversations for the authenticated user.
*   **Response Body (Success 200 - from "Chat" list screen):**
    ```json
    [
      {
        "chatId": "chat_1",
        "otherParticipant": {
          "id": "user_abc",
          "name": "Darlene Ro",
          "imageUrl": "url/to/darlene.jpg",
          "isBlockedByMe": false,
          "amIBlocked": false
        },
        "lastMessage": {
          "text": "You: Hi john w...",
          "timestamp": "2020-06-12T00:00:00Z",
          "isRead": true
        },
        "unreadCount": 0
      }
    ]
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 9.2. Get Messages in a Chat
*   **Method:** `GET`
*   **Route:** `/api/chats/{chatId}/messages`
*   **Query Parameters:** `beforeTimestamp` (for pagination), `limit`.
*   **Description:** Retrieves messages for a specific chat.
*   **Response Body (Success 200 - from chat interface screen):**
    ```json
    [
      {
        "messageId": "msg_1",
        "senderId": "user_abc", // Darlene
        "receiverId": "me",
        "text": "Man images for free download...",
        "imageUrl": null, // Or "url/to/image_in_chat.jpg"
        "timestamp": "2023-02-09T09:00:00Z",
        "isRead": true
      },
      {
        "messageId": "msg_2",
        "senderId": "me",
        "receiverId": "user_abc",
        "text": "Thanks!",
        "imageUrl": null,
        "timestamp": "2023-02-09T09:01:00Z",
        "isRead": false // Read by Darlene
      }
    ]
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden` (not part of chat), `404 Not Found` (chat not found).

#### 9.3. Send Message
*   **Method:** `POST`
*   **Route:** `/api/chats/{chatId}/messages` (or `/api/chats/send` if chatId is unknown and needs to be created)
*   **Description:** Sends a message in a chat. If `chatId` is not provided or doesn't exist for participants, a new chat might be initiated.
*   **Request Body:**
    ```json
    {
      // If new chat, provide participant IDs
      // "participants": ["userId1", "userId2"],
      "text": "Hello there!",
      "imageUrl": null // Or "url/to/uploaded_image.jpg" if image attached
    }
    ```
*   **Response Body (Success 201):** The sent message object (similar to GET response item).
*   **Status Codes:** `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`.

#### 9.4. Upload File in Chat
*   **Method:** `POST`
*   **Route:** `/api/chats/{chatId}/messages/files`
*   **Description:** Uploads a file to be sent in a chat.
*   **Request Body:** `multipart/form-data` with `file` field.
*   **Response Body (Success 200):**
    ```json
    {
      "fileUrl": "url/to/chat_file.jpg",
      "fileType": "image/jpeg" // Mime type
    }
    ```
*   **Status Codes:** `200 OK`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`. (The client would then send a message with this `fileUrl`).

#### 9.5. Block/Unblock User in Chat
*   **Method:** `POST`
*   **Route:** `/api/chats/users/{otherUserId}/block` or `/unblock`
*   **Description:** Blocks or unblocks another user from chatting.
*   **Response Body (Success 200):**
    ```json
    { "message": "User blocked/unblocked successfully." }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `404 Not Found` (other user not found).

---

### 10. E-commerce / Marketplace (Basic)

#### 10.1. List Products
*   **Method:** `GET`
*   **Route:** `/api/products`
*   **Description:** Lists products available in the marketplace.
*   **Query Parameters:** `category`, `search`, `sortBy`, `page`, `limit`.
*   **Response Body (Success 200 - from "E-commerce Home Screen"):**
    ```json
    {
      "products": [
        {
          "id": "prod_1",
          "name": "Ergonomic Seat",
          "price": 150.00,
          "currency": "INR",
          "imageUrl": "url/to/ergonomic_seat.jpg",
          "discount": "10% OFF", // Or numeric value
          "rating": 4.8
        }
      ],
      "total": 100, "page": 1, "limit": 10
    }
    ```
*   **Status Codes:** `200 OK`.

#### 10.2. Get Product Details
*   **Method:** `GET`
*   **Route:** `/api/products/{productId}`
*   **Description:** Retrieves details of a specific product.
*   **Response Body (Success 200 - from "Product Name" screen):**
    ```json
    {
      "id": "prod_1",
      "name": "Ergonomic Seat",
      "images": ["url1.jpg", "url2.jpg"],
      "price": 150.00,
      "currency": "INR",
      "stock": 70,
      "sizes": ["S", "M", "L", "XL", "XXL"], // If applicable
      "description": "Detailed product description...",
      "specifications": { /* ... */ },
      "shippingInfo": "Orders placed before 3pm...",
      "careDetails": "Do not wash...",
      "businessName": "Medicine Store Inc.",
      "reviewsSummary": { /* see reviews endpoint */ }
    }
    ```
*   **Status Codes:** `200 OK`, `404 Not Found`.

#### 10.3. List Product Categories
*   **Method:** `GET`
*   **Route:** `/api/products/categories`
*   **Description:** Get categories for marketplace (Thermometers, Ortho Foams etc.)
*   **Response Body:**
    ```json
    [
        {"id": "cat1", "name": "Thermometers"},
        {"id": "cat2", "name": "Ortho Foams"}
    ]
    ```
*   **Status Codes:** `200 OK`.

#### 10.4. List Banners/Promotions
*   **Method:** `GET`
*   **Route:** `/api/promotions/banners`
*   **Description:** Get promotional banners for the marketplace home.
*   **Response Body:**
    ```json
    [
        {"id": "banner1", "imageUrl": "url/to/banner1.jpg", "linkUrl": "/products/prod_blood_pressure_kit", "title": "Get Your Blood Pressure Kit", "discountText": "50% OFF"}
    ]
    ```
*   **Status Codes:** `200 OK`.

---

### 11. General Content & Utilities

#### 11.1. Get General Categories (Health Topics)
*   **Method:** `GET`
*   **Route:** `/api/categories/health` (or just `/api/categories`)
*   **Description:** Retrieves general health categories (Diet, Health, Physical, etc. from "Categories" screen).
*   **Response Body (Success 200):**
    ```json
    [
      { "id": "cat_diet", "name": "Diet", "iconUrl": "url/to/diet_icon.png" },
      { "id": "cat_health", "name": "Health", "iconUrl": "url/to/health_icon.png" }
      // ...
    ]
    ```
*   **Status Codes:** `200 OK`.

#### 11.2. Get Static Content (Privacy Policy, T&C, FAQs)
*   **Method:** `GET`
*   **Route:** `/api/content/{contentType}` (e.g., `/api/content/privacy-policy`, `/api/content/terms`, `/api/content/faq`)
*   **Description:** Retrieves static content.
*   **Response Body (Success 200):**
    ```json
    {
      "title": "Privacy Policy",
      "htmlContent": "<p>Your privacy is important to us...</p>" // Or structured content
    }
    ```
*   **Status Codes:** `200 OK`, `404 Not Found`.

#### 11.3. List Notifications
*   **Method:** `GET`
*   **Route:** `/api/notifications`
*   **Description:** Retrieves notifications for the authenticated user.
*   **Query Parameters:** `page`, `limit`, `status=unread`.
*   **Response Body (Success 200 - from "Notification" screen):**
    ```json
    {
      "notifications": [
        {
          "id": "notif1",
          "title": "Chest Pain Reported",
          "message": "John Doe has reported chest pain and shortness of breath",
          "type": "Alert", // "Reminder", "Update"
          "imageUrl": "url/to/john_doe_image.jpg",
          "timestamp": "2023-10-26T18:00:00Z",
          "isRead": false,
          "link": "/appointments/appt_john_doe" // Optional deep link
        }
      ],
      "unreadCount": 5
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 11.4. Mark Notification as Read
*   **Method:** `PUT`
*   **Route:** `/api/notifications/{notificationId}/read` (or `/api/notifications/read-all`)
*   **Description:** Marks one or all notifications as read.
*   **Response Body (Success 200):**
    ```json
    { "message": "Notification(s) marked as read." }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `404 Not Found`.

#### 11.5. Update Notification Settings
*   **Method:** `PUT`
*   **Route:** `/api/users/me/settings/notifications`
*   **Description:** Updates user's notification preferences.
*   **Request Body (from "Settings" screen):**
    ```json
    {
      "appointmentRequests": true,
      "messengerNotifications": true,
      "reminders10MinutesBefore": false
    }
    ```
*   **Response Body (Success 200):** Updated settings object.
*   **Status Codes:** `200 OK`, `401 Unauthorized`.

#### 11.6. List Pharmacies
*   **Method:** `GET`
*   **Route:** `/api/pharmacies`
*   **Description:** Lists pharmacies, potentially filterable by location.
*   **Query Parameters:** `latitude`, `longitude`, `radius`, `search`.
*   **Response Body (Success 200 - from "All Pharmacies" screen):**
    ```json
    [
      {
        "id": "pharm1",
        "name": "Pharmacy Name",
        "bannerUrl": "url/to/pharmacy_banner.jpg",
        "logoUrl": "url/to/pharmacy_logo.png",
        "rating": 4.5,
        "distance": "24 km away",
        "callLink": "tel:+1234567890" // If "Call Us" button is direct
      }
    ]
    ```
*   **Status Codes:** `200 OK`.

---

### 12. Admin Panel Endpoints

These would be under a separate prefix like `/api/admin/...` and require admin role.

#### 12.1. Get Admin Dashboard Overview
*   **Method:** `GET`
*   **Route:** `/api/admin/dashboard/overview`
*   **Description:** Retrieves statistics for the admin dashboard.
*   **Query Parameters:** `period=ThisMonth` or `year=2024`.
*   **Response Body (Success 200 - from "Overview" screens):**
    ```json
    {
      "totalUsers": 2345,
      "patients": 1600,
      "doctors": 400,
      "businesses": 200,
      "researchers": 45,
      "usersCurrentlyOnline": 150,
      "activeSessions": 28000,
      "diagnosAiProfileUpdates": 13,
      "economyProfileUpdates": 6,
      "newSignUps": {
        "period": "This Month",
        "patients": 120, "doctors": 25, "businesses": 67, "researchers": 5
      },
      "genderDistribution": { "year": 2024, "male": 34, "female": 64, "others": 20 }, // Percentages
      "transactions": {
        "period": "This Month",
        "consultationPayments": 18000,
        "ecommerceSales": 7000,
        "diagnosProfilePayments": 11000,
        "economyProfilePayments": 2000
      }
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`.

#### 12.2. List Users (Admin)
*   **Method:** `GET`
*   **Route:** `/api/admin/users`
*   **Description:** Lists users with filtering options for admin.
*   **Query Parameters:** `role` (Doctor, Patient, etc.), `status` (Pending, Active), `search`, `page`, `limit`.
*   **Response Body (Success 200 - from "User Management" screen):**
    ```json
    {
      "users": [
        {
          "id": "user_xyz",
          "name": "John Doe",
          "email": "john.doe@example.com",
          "imageUrl": "url/to/user_image.jpg",
          "status": "Active", // "Pending"
          "role": "Patient", // "Doctor"
          "profileType": "Basic Profile" // "Economy Profile", "Diagnoz.AI Profile"
        }
      ],
      "total": 100, "page": 1, "limit": 10
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`.

#### 12.3. Get User Details (Admin)
*   **Method:** `GET`
*   **Route:** `/api/admin/users/{userId}`
*   **Description:** Admin retrieves full details of a user, including documents for approval.
*   **Response Body (Success 200 - from "Business/Person Name" approval screen):**
    ```json
    {
      "id": "user_abc",
      "name": "Business Name / Person Name",
      "profilePictureUrl": "url/to/profile.jpg",
      "bannerUrl": "url/to/business_banner.jpg", // If business
      "category": "Medical Devices (Ecommerce)",
      "phone": "+912828282822",
      "email": "john.doe@example.com",
      "profileType": "Basic",
      "platformJoiningDate": "2024-09-29",
      "status": "PendingApproval", // "Active", "Rejected", "Blocked"
      "documents": [
        { "type": "ID Card", "url": "url/to/id_card.jpg", "verified": false },
        { "type": "Medical Council Certificate", "url": "url/to/medical_cert.pdf", "verified": false }
      ],
      // ... other profile fields
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 12.4. Approve/Reject User Registration
*   **Method:** `POST`
*   **Route:** `/api/admin/users/{userId}/approve` or `/api/admin/users/{userId}/reject`
*   **Description:** Admin approves or rejects a user's registration (e.g., for doctors, businesses).
*   **Request Body (for reject, optional):**
    ```json
    {
      "reason": "Documents unclear."
    }
    ```
*   **Response Body (Success 200):**
    ```json
    { "message": "User approved/rejected successfully." }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 12.5. Block/Unblock User (Platform-wide)
*   **Method:** `POST`
*   **Route:** `/api/admin/users/{userId}/block` or `/api/admin/users/{userId}/unblock`
*   **Description:** Admin blocks or unblocks a user from the platform.
*   **Request Body:**
    ```json
    {
      "reason": "Violation of terms of service." // For blocking
    }
    ```
*   **Response Body (Success 200):**
    ```json
    { "message": "User blocked/unblocked successfully." }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 12.6. List Employees (Admin)
*   **Method:** `GET`
*   **Route:** `/api/admin/employees`
*   **Description:** Lists internal employees/staff.
*   **Response Body (Success 200 - from "Employees" screen):**
    ```json
    [
      {
        "id": "emp1",
        "name": "Bessie Cooper",
        "imageUrl": "url/to/bessie.jpg",
        "status": "Active",
        "role": "Super Admin" // "Admin", "Support Staff"
      }
    ]
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`.

#### 12.7. Add New Employee (Admin)
*   **Method:** `POST`
*   **Route:** `/api/admin/employees`
*   **Description:** Admin adds a new employee.
*   **Request Body (from "Add New Employ" screen):**
    ```json
    {
      "fullName": "John Medical Store", // Placeholder, actual name
      "email": "new.employee@example.com",
      "phone": "+919876500000",
      "password": "initialPassword123",
      "role": "Support Staff", // "Admin"
      "permissions": [
        "Can access dashboard Xyz option", // List of specific permissions
        // ...
      ],
      "profilePictureUrl": "url/to/employee_photo.jpg" // After upload
    }
    ```
*   **Response Body (Success 201):** The created employee object.
*   **Status Codes:** `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`.

#### 12.8. Get Employee Details (Admin)
*   **Method:** `GET`
*   **Route:** `/api/admin/employees/{employeeId}`
*   **Description:** Admin views details of an employee.
*   **Response Body (Success 200 - from "Employ Name" screen):**
    ```json
    {
      "id": "emp1",
      "name": "Employ Name",
      "profilePictureUrl": "url/to/employee_photo.jpg",
      "role": "Support Staff",
      "phone": "+912828282822",
      "email": "john.doe@example.com",
      "passwordLastSet": "2023-01-01T10:00:00Z", // For security, not actual password
      "memberSince": "2024-09-29",
      "permissions": [
        { "id": "perm1", "description": "Can access dashboard Xyz option", "hasAccess": true },
        { "id": "perm2", "description": "Can access dashboard Abc option", "hasAccess": false }
      ]
    }
    ```
*   **Status Codes:** `200 OK`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 12.9. Update Employee (Admin)
*   **Method:** `PUT`
*   **Route:** `/api/admin/employees/{employeeId}`
*   **Description:** Admin updates employee details or permissions.
*   **Request Body:** Similar to Add New Employee or Get Employee Details (for updatable fields).
*   **Response Body (Success 200):** The updated employee object.
*   **Status Codes:** `200 OK`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

#### 12.10. Remove Employee (Admin)
*   **Method:** `DELETE`
*   **Route:** `/api/admin/employees/{employeeId}`
*   **Description:** Admin removes an employee (deactivates or deletes).
*   **Response Body (Success 204):** (No content)
*   **Status Codes:** `204 No Content`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`.

---
