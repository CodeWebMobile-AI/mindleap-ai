```markdown
# MindLeap AI: Architecture Document

**Version:** 1.0
**Date:** October 26, 2023
**Author:** AI Assistant (Simulating CTO/Principal Engineer)

## 1. Overview

### 1.1 Problem Statement

Mental health support faces significant challenges, including high costs, limited accessibility, and persistent social stigma. Many individuals experiencing stress, anxiety, or other emotional difficulties lack readily available resources, especially outside of traditional business hours.  This leaves a gap in providing immediate and continuous support, hindering timely intervention and proactive mental wellness management.

### 1.2 Target Audience

MindLeap AI aims to address this problem by targeting the following user groups:

*   Individuals seeking readily available mental wellness support.
*   People experiencing mild to moderate anxiety and stress.
*   Individuals considering traditional therapy but seeking an initial exploration.
*   Users who desire 24/7 accessible emotional support and coping mechanisms.

### 1.3 Goals

*   Provide 24/7 accessible AI-powered mental health support.
*   Offer personalized coping strategies tailored to individual needs.
*   Enable users to track their mood and identify emotional patterns.
*   Facilitate crisis intervention by connecting users with relevant helplines.
*   Provide guided meditation and exercises for stress reduction.
*   Monitor user progress and provide data-driven insights.
*   Offer therapist matching and referral services for professional support.

## 2. Design System

### 2.1 Typography

*   **Font Family:** Open Sans (Primary), Roboto (Secondary for code snippets)
    *   Open Sans provides excellent readability and a friendly feel.
    *   Roboto ensures clear differentiation for code examples.
*   **Headings (H1-H6):** Open Sans, Bold
    *   H1: 36px
    *   H2: 30px
    *   H3: 24px
    *   H4: 20px
    *   H5: 16px
    *   H6: 14px
*   **Body Text:** Open Sans, Regular, 16px
*   **Code Snippets:** Roboto, Regular, 14px

### 2.2 Color Scheme

*   **Primary Color:** `#66BB6A` (Green – Promotes calmness and well-being)
*   **Secondary Color:** `#29B6F6` (Light Blue – Represents clarity and trust)
*   **Accent Color:** `#FFCA28` (Yellow – Symbolizes hope and optimism)
*   **Background Color:** `#F5F5F5` (Light Gray – Neutral and unobtrusive)
*   **Text Color (Primary):** `#212121` (Dark Gray – Excellent contrast)
*   **Text Color (Secondary):** `#757575` (Medium Gray – Subdued information)
*   **Error Color:** `#E53935` (Red – Alerts and warnings)
*   **Success Color:** `#43A047` (Dark Green – Positive feedback)
*   **Warning Color:** `#FB8C00` (Orange – Cautionary messages)

## 3. Database Schema

We will use PostgreSQL as our relational database due to its robustness, reliability, and extensive support for various data types.

```sql
-- Users Table
CREATE TABLE Users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    date_of_birth DATE,
    gender VARCHAR(50),
    registration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);

-- Sessions Table
CREATE TABLE Sessions (
    session_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    end_time TIMESTAMP,
    session_type VARCHAR(255),  -- e.g., "Chatbot", "Meditation", "Exercise"
    session_details TEXT        -- JSON blob for session-specific data
);

-- Mood Table
CREATE TABLE Mood (
    mood_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    mood_level INTEGER NOT NULL CHECK (mood_level >= 1 AND mood_level <= 10), -- Scale of 1-10
    mood_description TEXT,
    date_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    trigger VARCHAR(255),
    notes TEXT
);

-- Exercises Table
CREATE TABLE Exercises (
    exercise_id SERIAL PRIMARY KEY,
    exercise_name VARCHAR(255) NOT NULL,
    exercise_type VARCHAR(255), -- e.g., "Breathing", "Mindfulness", "Yoga"
    description TEXT,
    instructions TEXT,
    duration_minutes INTEGER,
    audio_url VARCHAR(255),
    video_url VARCHAR(255)
);

-- UserExercises (Many-to-Many relationship between Users and Exercises)
CREATE TABLE UserExercises (
    user_exercise_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    exercise_id INTEGER REFERENCES Exercises(exercise_id),
    completion_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    notes TEXT,
    rating INTEGER CHECK (rating >= 1 AND rating <= 5) -- Optional rating for the exercise
);

-- CrisisHelplines Table
CREATE TABLE CrisisHelplines (
    helpline_id SERIAL PRIMARY KEY,
    helpline_name VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20) NOT NULL,
    website_url VARCHAR(255),
    description TEXT,
    keywords TEXT -- Comma-separated keywords for searching
);

-- Referrals Table
CREATE TABLE Referrals (
    referral_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    therapist_name VARCHAR(255),
    therapist_specialty VARCHAR(255),
    therapist_contact VARCHAR(255),
    referral_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    notes TEXT
);

-- Progress Table
CREATE TABLE Progress (
    progress_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    metric_name VARCHAR(255) NOT NULL, -- e.g., "Anxiety Level", "Sleep Quality", "Mood Stability"
    metric_value DECIMAL,
    date_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    notes TEXT
);

-- Strategies Table
CREATE TABLE Strategies (
    strategy_id SERIAL PRIMARY KEY,
    strategy_name VARCHAR(255) NOT NULL,
    strategy_type VARCHAR(255), -- e.g., "Cognitive", "Behavioral", "Relaxation"
    description TEXT,
    instructions TEXT,
    audio_url VARCHAR(255),
    video_url VARCHAR(255)
);

-- UserStrategies (Many-to-Many relationship between Users and Strategies)
CREATE TABLE UserStrategies (
    user_strategy_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    strategy_id INTEGER REFERENCES Strategies(strategy_id),
    date_assigned TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_completed BOOLEAN DEFAULT FALSE,
    completion_date TIMESTAMP,
    notes TEXT
);

-- Chatbot Responses Table
CREATE TABLE ChatbotResponses (
    response_id SERIAL PRIMARY KEY,
    intent VARCHAR(255) NOT NULL,
    response_text TEXT NOT NULL,
    action VARCHAR(255),       -- Optional action to trigger (e.g., "suggest_exercise", "connect_helpline")
    context VARCHAR(255)        -- Context for the response (e.g., "greeting", "anxiety", "stress")
);

-- Chatbot Training Data
CREATE TABLE ChatbotTrainingData (
    training_data_id SERIAL PRIMARY KEY,
    user_input TEXT NOT NULL,
    intent VARCHAR(255) NOT NULL
);

-- Therapist Profiles (For therapist matching and referrals)
CREATE TABLE TherapistProfiles (
    therapist_id SERIAL PRIMARY KEY,
    therapist_name VARCHAR(255) NOT NULL,
    specialty VARCHAR(255),
    license VARCHAR(255),
    contact_email VARCHAR(255),
    contact_phone VARCHAR(20),
    address VARCHAR(255),
    bio TEXT,
    insurance_accepted TEXT, -- JSON array of insurance providers
    languages_spoken TEXT    -- JSON array of languages
);

-- Events (e.g., scheduled check-ins, reminders for exercises)
CREATE TABLE Events (
    event_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES Users(user_id),
    event_type VARCHAR(255) NOT NULL, -- e.g., "Check-in Reminder", "Exercise Reminder", "Therapy Session"
    event_time TIMESTAMP NOT NULL,
    event_details TEXT, -- JSON blob with event-specific details
    is_recurring BOOLEAN DEFAULT FALSE,
    recurrence_rule TEXT  -- iCalendar RRULE string
);
```

## 4. API Endpoints

We will use a RESTful API architecture, utilizing JSON for data exchange. All endpoints will be authenticated using JWT (JSON Web Tokens).  We will use Node.js with Express.js for the backend.

**Base URL:** `/api/v1`

### 4.1 User Authentication

*   **`POST /api/v1/auth/register`**
    *   Description: Registers a new user.
    *   Request Body:
        ```json
        {
            "username": "newuser",
            "email": "newuser@example.com",
            "password": "SecurePassword123",
            "firstName": "John",
            "lastName": "Doe"
        }
        ```
    *   Response (Success - 201 Created):
        ```json
        {
            "message": "User registered successfully",
            "userId": 123
        }
        ```
    *   Response (Error - 400 Bad Request):
        ```json
        {
            "error": "Username already exists"
        }
        ```

*   **`POST /api/v1/auth/login`**
    *   Description: Logs in an existing user.
    *   Request Body:
        ```json
        {
            "username": "existinguser",
            "password": "SecurePassword123"
        }
        ```
    *   Response (Success - 200 OK):
        ```json
        {
            "message": "Login successful",
            "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywidXNlcm5hbWUiOiJleGlzdGluZ3VzZXIiLCJpYXQiOjE2OTgyOTIzODEsImV4cCI6MTY5ODI5NTk4MX0.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        }
        ```
    *   Response (Error - 401 Unauthorized):
        ```json
        {
            "error": "Invalid credentials"
        }
        ```

*   **`POST /api/v1/auth/logout`**
    *   Description: Logs out the current user (invalidates the JWT).  Frontend should delete the stored token.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Response (Success - 200 OK):
        ```json
        {
            "message": "Logout successful"
        }
        ```

### 4.2 User Profile

*   **`GET /api/v1/users/me`**
    *   Description: Retrieves the current user's profile.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Response (Success - 200 OK):
        ```json
        {
            "userId": 123,
            "username": "existinguser",
            "email": "existinguser@example.com",
            "firstName": "Jane",
            "lastName": "Smith",
            "dateOfBirth": "1990-01-01",
            "gender": "Female",
            "registrationDate": "2023-10-26T00:00:00.000Z"
        }
        ```

*   **`PUT /api/v1/users/me`**
    *   Description: Updates the current user's profile.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Request Body:
        ```json
        {
            "firstName": "UpdatedJane",
            "lastName": "UpdatedSmith"
        }
        ```
    *   Response (Success - 200 OK):
        ```json
        {
            "message": "Profile updated successfully"
        }
        ```

### 4.3 Mood Tracking

*   **`POST /api/v1/moods`**
    *   Description: Creates a new mood entry.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Request Body:
        ```json
        {
            "moodLevel": 7,
            "moodDescription": "Feeling pretty good today",
            "trigger": "Finished a project at work",
            "notes": "Had a good day overall"
        }
        ```
    *   Response (Success - 201 Created):
        ```json
        {
            "message": "Mood entry created successfully",
            "moodId": 456
        }
        ```

*   **`GET /api/v1/moods`**
    *   Description: Retrieves mood entries for the current user.  Supports pagination and filtering by date range.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Query Parameters:
        *   `startDate`: (optional) Start date for filtering (YYYY-MM-DD)
        *   `endDate`: (optional) End date for filtering (YYYY-MM-DD)
        *   `page`: (optional) Page number (default: 1)
        *   `limit`: (optional) Number of entries per page (default: 10)
    *   Response (Success - 200 OK):
        ```json
        {
            "moods": [
                {
                    "moodId": 456,
                    "userId": 123,
                    "moodLevel": 7,
                    "moodDescription": "Feeling pretty good today",
                    "dateTime": "2023-10-26T12:00:00.000Z",
                    "trigger": "Finished a project at work",
                    "notes": "Had a good day overall"
                },
                {
                    "moodId": 457,
                    "userId": 123,
                    "moodLevel": 3,
                    "moodDescription": "Feeling anxious",
                    "dateTime": "2023-10-25T18:00:00.000Z",
                    "trigger": "Upcoming presentation",
                    "notes": "Need to practice more"
                }
            ],
            "totalCount": 2,
            "currentPage": 1,
            "totalPages": 1
        }
        ```

*   **`GET /api/v1/moods/:moodId`**
    *   Description: Retrieves a specific mood entry.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Response (Success - 200 OK):
        ```json
        {
           "moodId": 456,
           "userId": 123,
           "moodLevel": 7,
           "moodDescription": "Feeling pretty good today",
           "dateTime": "2023-10-26T12:00:00.000Z",
           "trigger": "Finished a project at work",
           "notes": "Had a good day overall"
       }
        ```
    *   Response (Error - 404 Not Found):
         ```json
         {
            "error": "Mood entry not found"
         }
         ```

*   **`DELETE /api/v1/moods/:moodId`**
    *    Description: Deletes a specific mood entry.
    *    Request Headers: `Authorization: Bearer <token>`
    *    Response (Success - 200 OK):
         ```json
         {
            "message": "Mood entry deleted successfully"
         }
         ```
    *    Response (Error - 404 Not Found):
         ```json
         {
            "error": "Mood entry not found"
         }
         ```

### 4.4 Exercises

*   **`GET /api/v1/exercises`**
    *   Description: Retrieves a list of exercises.  Supports filtering by exercise type.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Query Parameters:
        *   `type`: (optional) Exercise type (e.g., "Breathing", "Mindfulness")
    *   Response (Success - 200 OK):
        ```json
        {
            "exercises": [
                {
                    "exerciseId": 1,
                    "exerciseName": "Deep Breathing",
                    "exerciseType": "Breathing",
                    "description": "A simple deep breathing exercise",
                    "instructions": "Inhale deeply through your nose...",
                    "durationMinutes": 5,
                    "audioUrl": "https://example.com/audio/deep_breathing.mp3",
                    "videoUrl": null
                },
                {
                    "exerciseId": 2,
                    "exerciseName": "Body Scan Meditation",
                    "exerciseType": "Mindfulness",
                    "description": "A guided body scan meditation",
                    "instructions": "Lie down comfortably...",
                    "durationMinutes": 15,
                    "audioUrl": "https://example.com/audio/body_scan.mp3",
                    "videoUrl": null
                }
            ]
        }
        ```

*   **`GET /api/v1/exercises/:exerciseId`**
    * Description: Retrieves a specific exercise.
    * Request Headers: `Authorization: Bearer <token>`
    * Response (Success - 200 OK):
        ```json
        {
            "exerciseId": 1,
            "exerciseName": "Deep Breathing",
            "exerciseType": "Breathing",
            "description": "A simple deep breathing exercise",
            "instructions": "Inhale deeply through your nose...",
            "durationMinutes": 5,
            "audioUrl": "https://example.com/audio/deep_breathing.mp3",
            "videoUrl": null
        }
        ```

*   **`POST /api/v1/user-exercises`**
    * Description: Records that a user completed an exercise.
    * Request Headers: `Authorization: Bearer <token>`
    * Request Body:
        ```json
        {
            "exerciseId": 1,
            "notes": "Felt very relaxed after this exercise.",
            "rating": 5
        }
        ```
     *   Response (Success - 201 Created):
        ```json
        {
            "message": "User exercise recorded successfully",
            "userExerciseId": 789
        }
        ```

*  **`GET /api/v1/user-exercises`**
     * Description: Retrieves a list of exercises completed by user. Supports filtering by date range.
     * Request Headers: `Authorization: Bearer <token>`
     *  Query Parameters:
        *   `startDate`: (optional) Start date for filtering (YYYY-MM-DD)
        *   `endDate`: (optional) End date for filtering (YYYY-MM-DD)

     * Response (Success - 200 OK):
         ```json
         {
            "userExercises": [
               {
                  "userExerciseId": 789,
                  "userId": 123,
                  "exerciseId": 1,
                  "completionDate": "2023-10-26T15:00:00.000Z",
                  "notes": "Felt very relaxed after this exercise.",
                  "rating": 5
               }
            ]
         }
         ```

### 4.5 Crisis Helplines

*   **`GET /api/v1/helplines`**
    *   Description: Retrieves a list of crisis helplines. Supports searching by keyword.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Query Parameters:
        *   `keyword`: (optional) Search keyword (e.g., "suicide", "anxiety")
    *   Response (Success - 200 OK):
        ```json
        {
            "helplines": [
                {
                    "helplineId": 1,
                    "helplineName": "National Suicide Prevention Lifeline",
                    "phoneNumber": "1-800-273-8255",
                    "websiteUrl": "https://suicidepreventionlifeline.org/",
                    "description": "Provides 24/7, free and confidential support for people in distress.",
                    "keywords": "suicide, depression, crisis"
                }
            ]
        }
        ```

### 4.6 Referrals

*   **`POST /api/v1/referrals`**
    *   Description: Creates a referral for the user.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Request Body:
        ```json
        {
            "therapistName": "Dr. Jane Smith",
            "therapistSpecialty": "Anxiety and Depression",
            "therapistContact": "dr.jane.smith@example.com",
            "notes": "Recommended by a friend"
        }
        ```
    *   Response (Success - 201 Created):
        ```json
        {
            "message": "Referral created successfully",
            "referralId": 901
        }
        ```

*   **`GET /api/v1/referrals`**
    *   Description: Retrieves the user's referrals.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Response (Success - 200 OK):
        ```json
        {
            "referrals": [
                {
                    "referralId": 901,
                    "userId": 123,
                    "therapistName": "Dr. Jane Smith",
                    "therapistSpecialty": "Anxiety and Depression",
                    "therapistContact": "dr.jane.smith@example.com",
                    "referralDate": "2023-10-26T16:00:00.000Z",
                    "notes": "Recommended by a friend"
                }
            ]
        }
        ```

### 4.7 Strategies

*   **`GET /api/v1/strategies`**
    *   Description: Retrieves a list of coping strategies.  Supports filtering by strategy type.
    *   Request Headers: `Authorization: Bearer <token>`
     *    Query Parameters:
        *   `type`: (optional) Strategy type (e.g., "Cognitive", "Behavioral")

    *   Response (Success - 200 OK):
        ```json
        {
            "strategies": [
                {
                    "strategyId": 1,
                    "strategyName": "Cognitive Restructuring",
                    "strategyType": "Cognitive",
                    "description": "Identify and challenge negative thoughts",
                    "instructions": "Write down your negative thoughts...",
                    "audioUrl": null,
                    "videoUrl": null
                }
            ]
        }
        ```

*   **`POST /api/v1/user-strategies`**
    * Description: Assigns a coping strategy to the user.
    * Request Headers: `Authorization: Bearer <token>`
    * Request Body:
        ```json
        {
            "strategyId": 1,
            "notes": "Trying this strategy to manage anxiety."
        }
        ```
     *   Response (Success - 201 Created):
        ```json
        {
            "message": "User strategy assigned successfully",
            "userStrategyId": 345
        }
        ```

### 4.8 Chatbot

*   **`POST /api/v1/chatbot/message`**
    *   Description: Sends a message to the AI chatbot and receives a response.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Request Body:
        ```json
        {
            "message": "I'm feeling really stressed today."
        }
        ```
    *   Response (Success - 200 OK):
        ```json
        {
            "response": "I'm sorry to hear you're feeling stressed. Have you tried any breathing exercises today? I can suggest some if you'd like.",
            "action": "suggest_exercise",
            "context": "stress"
        }
        ```

### 4.9 Therapist Profiles (For Therapist Matching)

*   **`GET /api/v1/therapists`**
    *   Description: Retrieves a list of therapist profiles.  Supports filtering by specialty and insurance accepted.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Query Parameters:
        *   `specialty`: (optional) Therapist specialty (e.g., "Anxiety", "Depression")
        *   `insurance`: (optional) Insurance provider accepted (e.g., "Aetna", "Blue Cross")
    *   Response (Success - 200 OK):  Returns an array of therapist profile objects.

### 4.10 Events (Reminders, Scheduled Check-ins)

*   **`POST /api/v1/events`**
    *   Description: Creates a new event (e.g., reminder for an exercise).
    *   Request Headers: `Authorization: Bearer <token>`
    *   Request Body:
        ```json
        {
            "eventType": "Exercise Reminder",
            "eventTime": "2023-10-27T10:00:00.000Z",
            "eventDetails": {
                "exerciseId": 1,
                "exerciseName": "Deep Breathing"
            },
            "isRecurring": false
        }
        ```
    *   Response (Success - 201 Created):
        ```json
        {
            "message": "Event created successfully",
            "eventId": 1001
        }
        ```

*   **`GET /api/v1/events`**
    *   Description: Retrieves the user's events.  Supports filtering by date range.
    *   Request Headers: `Authorization: Bearer <token>`
    *   Query Parameters:
        *   `startDate`: (optional) Start date for filtering (YYYY-MM-DD)
        *   `endDate`: (optional) End date for filtering (YYYY-MM-DD)
    *   Response (Success - 200 OK): Returns an array of event objects.

## 5. Frontend Component Structure

We will use React for the frontend, utilizing a component-based architecture.

```
mindleap-ai-frontend/
├── src/
│   ├── components/
│   │   ├── Auth/
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   ├── Chatbot/
│   │   │   ├── ChatbotInterface.jsx
│   │   │   ├── Message.jsx
│   │   ├── Dashboard/
│   │   │   ├── Dashboard.jsx
│   │   │   ├── MoodTracker.jsx
│   │   │   ├── ExerciseList.jsx
│   │   │   ├── HelplineList.jsx
│   │   │   ├── ReferralList.jsx
│   │   │   ├── StrategyList.jsx
│   │   │   ├── ProgressChart.jsx
│   │   ├── Exercises/
│   │   │   ├── ExerciseCard.jsx
│   │   │   ├── ExerciseDetails.jsx
│   │   ├── Mood/
│   │   │   ├── MoodEntryForm.jsx
│   │   │   ├── MoodEntryList.jsx
│   │   │   ├── MoodEntry.jsx
│   │   ├── Referrals/
│   │   │   ├── ReferralCard.jsx
│   │   ├── Strategies/
│   │   │   ├── StrategyCard.jsx
│   │   ├── Events/
│   │   │   ├── EventList.jsx
│   │   │   ├── EventItem.jsx
│   │   ├── Layout/
│   │   │   ├── Navbar.jsx
│   │   │   ├── Footer.jsx
│   │   ├── Common/
│   │   │   ├── Button.jsx
│   │   │   ├── Input.jsx
│   │   │   ├── LoadingSpinner.jsx
│   │   │   ├── ErrorMessage.jsx
│   ├── pages/
│   │   ├── Home.jsx
│   │   ├── About.jsx
│   │   ├── Contact.jsx
│   ├── services/
│   │   ├── authService.js
│   │   ├── moodService.js
│   │   ├── exerciseService.js
│   │   ├── helplineService.js
│   │   ├── referralService.js
│   │   ├── strategyService.js
│   │   ├── eventService.js
│   │   ├── chatbotService.js
│   ├── context/
│   │   ├── AuthContext.jsx
│   ├── utils/
│   │   ├── api.js  (Handles API requests with JWT)
│   │   ├── dateUtils.js
│   ├── App.jsx
│   ├── index.js
│   ├── App.css
│   ├── index.css
├── public/
│   ├── index.html
│   ├── ...
├── package.json
├── README.md
```

**Component Details:**

*   **`Auth/Login.jsx`**:  Login form component.  Handles user authentication.  Uses `authService.js` to make API calls.
*   **`Auth/Register.jsx`**: Registration form component. Handles user registration. Uses `authService.js`.
*   **`Chatbot/ChatbotInterface.jsx`**:  Main chatbot interface. Sends user messages to the backend and displays chatbot responses.  Uses `chatbotService.js`.
*   **`Chatbot/Message.jsx`**:  Displays a single chat message (either user or chatbot).
*   **`Dashboard/Dashboard.jsx`**:  Main dashboard component, displaying a summary of user data (mood, exercises completed, etc.). Orchestrates other dashboard components.
*   **`Dashboard/MoodTracker.jsx`**:  Displays a mood tracking chart and allows users to log their mood.
*   **`Dashboard/ExerciseList.jsx`**:  Displays a list of recommended exercises.
*   **`Dashboard/HelplineList.jsx`**:  Displays a list of crisis helplines.
*   **`Dashboard/ReferralList.jsx`**:  Displays a list of the user's referrals.
*   **`Dashboard/StrategyList.jsx`**: Displays a list of coping strategies assigned to the user.
*   **`Dashboard/ProgressChart.jsx`**:  Displays a chart of user progress over time (e.g., anxiety levels, mood stability).
*   **`Exercises/ExerciseCard.jsx`**:  Displays a single exercise in a list.
*   **`Exercises/ExerciseDetails.jsx`**:  Displays the details of a specific exercise.
*   **`Mood/MoodEntryForm.jsx`**:  Form for creating a new mood entry.
*   **`Mood/MoodEntryList.jsx`**:  Displays a list of mood entries.
*   **`Mood/MoodEntry.jsx`**:  Displays a single mood entry in a list.
*   **`Referrals/ReferralCard.jsx`**: Displays a single referral card in a list.
*   **`Strategies/StrategyCard.jsx`**: Displays a single strategy card in a list.
*   **`Events/EventList.jsx`**:  Displays a list of upcoming events (reminders).
*   **`Events/EventItem.jsx`**:  Displays a single event in the list.
*   **`Layout/Navbar.jsx`**:  Navigation bar component.  Handles navigation and user authentication status.
*   **`Layout/Footer.jsx`**:  Footer component.
*   **`Common/Button.jsx`**:  Reusable button component.
*   **`Common/Input.jsx`**:  Reusable input component.
*   **`Common/LoadingSpinner.jsx`**:  Loading spinner component.
*   **`Common/ErrorMessage.jsx`**:  Displays error messages.
*   **`pages/Home.jsx`**:  Home page.
*   **`pages/About.jsx`**:  About page.
*   **`pages/Contact.jsx`**: Contact page.
*   **`services/authService.js`**:  Handles authentication API calls (login, register, logout). Uses `api.js`.
*   **`services/moodService.js`**: Handles mood related API calls (create, get, delete). Uses `api.js`.
*   **`services/exerciseService.js`**: Handles exercise-related API calls. Uses `api.js`.
*   **`services/helplineService.js`**:  Handles helpline API calls. Uses `api.js`.
*   **`services/referralService.js`**:  Handles referral API calls. Uses `api.js`.
*   **`services/strategyService.js`**:  Handles strategy API calls. Uses `api.js`.
*   **`services/eventService.js`**: Handles event API calls. Uses `api.js`.
*   **`services/chatbotService.js`**: Handles chatbot API calls. Uses `api.js`.
*   **`context/AuthContext.jsx`**: Provides authentication context to the application (user authentication status, JWT).
*   **`utils/api.js`**:  Handles API requests with JWT authentication.  Sets the `Authorization` header for all requests.  Handles token refresh.
*   **`utils/dateUtils.js`**:  Provides utility functions for working with dates (formatting, parsing).

## 6. Feature Implementation Roadmap

This roadmap outlines the planned implementation phases for MindLeap AI.

**Phase 1: Core Functionality (MVP - Minimum Viable Product) - Target Completion: 4 weeks**

*   **Authentication:**
    *   Implement user registration and login functionality with JWT authentication. (Backend & Frontend)
    *   Store user credentials securely using password hashing (bcrypt). (Backend)
    *   Implement logout functionality. (Backend & Frontend)
*   **Mood Tracking:**
    *   Implement mood entry creation (mood level, description, trigger, notes). (Backend & Frontend)
    *   Implement mood entry retrieval (list and single entry). (Backend & Frontend)
    *   Implement mood entry deletion. (Backend & Frontend)
    *   Basic Mood visualization (Frontend - Line chart of mood levels over time)
*   **AI Chatbot:**
    *   Implement basic AI chatbot functionality