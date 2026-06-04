# Realtime Spotify Application ✨

This repository contains a full-stack, production-ready **Realtime Spotify Clone** designed as a final year college project. It features music streaming, an admin panel for content management, user-to-user chatting, real-time presence indicators, and advanced data aggregation analytics.

Below you will find the complete system design resources, including the **Entity-Relationship Diagram (ERD)** and the **Data Flow Diagrams (DFD)** (Context Level 0 and Process Level 1) to assist in your project documentation, reports, and diagrams.

---

## 📸 Demo Preview

![Demo App](/frontend/public/screenshot-for-readme.png)

---

## 🛠️ Tech Stack & System Architecture

The application is built using a modern **MERN** architecture coupled with real-time WebSockets:

*   **Frontend**: React (Vite), TailwindCSS, Shadcn UI, Zustand (State Management), Lucide Icons, React Router DOM, Socket.io-client.
*   **Backend**: Node.js, Express.js, REST API, Socket.io (WebSockets for real-time status and chat), Node-Cron (temporary file clean-up).
*   **Database**: MongoDB (NoSQL) with Mongoose Object Modeling.
*   **Authentication & Session Management**: Clerk (OAuth 2.0, Secure Login, Session Sync).
*   **Cloud Media Hosting**: Cloudinary (for storing and streaming audio files and images).

---

## 📊 Entity-Relationship Diagram (ERD)

The database is built on MongoDB using Mongoose. The diagram below illustrates the collections, fields, data types, and relationships (using Mermaid syntax).

```mermaid
erDiagram
    USER {
        string clerkId PK "Unique Clerk User ID (Unique Index)"
        string fullName "User's Full Name"
        string imageUrl "User Avatar URL"
        date createdAt "Timestamp of creation"
        date updatedAt "Timestamp of last update"
    }

    ALBUM {
        ObjectId id PK "Unique MongoDB Object ID"
        string title "Album Name"
        string artist "Album Artist Name"
        string imageUrl "Cloudinary Cover Image URL"
        number releaseYear "Release Year"
        ObjectIdArray songs "Array of Song IDs (References Song)"
        date createdAt "Timestamp of creation"
        date updatedAt "Timestamp of last update"
    }

    SONG {
        ObjectId id PK "Unique MongoDB Object ID"
        string title "Song Name"
        string artist "Song Artist Name"
        string imageUrl "Cloudinary Album Art URL"
        string audioUrl "Cloudinary Audio file stream URL"
        number duration "Duration of track in seconds"
        ObjectId albumId FK "Refers to Album ID (Optional)"
        date createdAt "Timestamp of creation"
        date updatedAt "Timestamp of last update"
    }

    MESSAGE {
        ObjectId id PK "Unique MongoDB Object ID"
        string senderId FK "Clerk User ID of sender"
        string receiverId FK "Clerk User ID of receiver"
        string content "Text Content of the message"
        date createdAt "Timestamp of creation"
        date updatedAt "Timestamp of last update"
    }

    USER ||--o{ MESSAGE : "sends/receives"
    ALBUM ||--o{ SONG : "contains"
```

### Schema & Entity Details

1.  **User Entity (`User`)**:
    *   `clerkId`: Primary Identifier synced from Clerk Authentication.
    *   `fullName`: Full name of the user.
    *   `imageUrl`: Link to the user profile avatar.
2.  **Song Entity (`Song`)**:
    *   `title` & `artist`: Song details.
    *   `imageUrl` & `audioUrl`: Media stream URLs hosted securely on Cloudinary.
    *   `duration`: Duration of the song in seconds.
    *   `albumId`: Optional Foreign Key pointing to the `Album` entity.
3.  **Album Entity (`Album`)**:
    *   `title` & `artist`: Album details.
    *   `imageUrl`: Album art hosted on Cloudinary.
    *   `releaseYear`: Release year of the album.
    *   `songs`: Array of ObjectIds representing the songs assigned to this album.
4.  **Message Entity (`Message`)**:
    *   `senderId` & `receiverId`: References to the Clerk ID of the sender and receiver.
    *   `content`: The text content of the message.

---

## 🔄 Data Flow Diagrams (DFD)

The following diagrams show how data propagates through the system, from external entities (User, Admin, Clerk, Cloudinary) to the internal database and real-time processes.

### DFD Level 0: Context Diagram
This diagram represents the boundary of the system, showing the inputs and outputs between the System and the External Entities.

```mermaid
graph TD
    User([User / Listener])
    Admin([Admin User])
    Clerk([Clerk Authentication])
    Cloudinary([Cloudinary Media Storage])
    
    System[[Realtime Spotify App System]]

    %% User Interactions
    User -->|Credentials / Actions| System
    System -->|Real-time Activity / Audio stream / Chat stream| User

    %% Admin Interactions
    Admin -->|Manage Songs & Albums / Files| System
    System -->|Analytics & Statistics| Admin

    %% External Systems
    System -->|Verify Token & Sync Profile| Clerk
    Clerk -->|Secure Token & Profile Data| System
    System -->|Media Upload / Audio & Images| Cloudinary
    Cloudinary -->|Secure Asset CDN URLs| System
```

---

### DFD Level 1: Detailed Process Flow Diagram
This diagram decomposes the system into major functional components, showcasing the specific data stores (MongoDB collections) and interactions.

```mermaid
graph TD
    subgraph External Entities
        User([User / Listener])
        Admin([Admin User])
        Clerk([Clerk Auth Service])
        Cloudinary([Cloudinary CDN])
    end

    subgraph Core Processes
        P1(1.0 Authentication & User Sync)
        P2(2.0 Music Streaming & Browsing)
        P3(3.0 Media Catalog Management)
        P4(4.0 Real-time Messaging & Status)
        P5(5.0 Statistics & Aggregations)
    end

    subgraph Data Stores
        DS1[(MongoDB: Users Collection)]
        DS2[(MongoDB: Songs Collection)]
        DS3[(MongoDB: Albums Collection)]
        DS4[(MongoDB: Messages Collection)]
    end

    %% 1.0 Auth Sync
    User -->|Sign-up / Sign-in Request| P1
    P1 -->|Verify Clerk JWT & Profile| Clerk
    Clerk -->|Validated Profile Metadata| P1
    P1 -->|Create/Sync User Record| DS1
    DS1 -->|User Data| P1
    P1 -->|Session Response| User

    %% 2.0 Music Stream
    User -->|Browse Songs / Play Album| P2
    P2 -->|Query Tracks & Albums| DS2
    P2 -->|Query Album Details| DS3
    DS2 -->|Tracks Metadata & Cloudinary URLs| P2
    DS3 -->|Album Metadata & Track references| P2
    P2 -->|Audio streams & Catalog| User

    %% 3.0 Media Mgmt (Admin)
    Admin -->|Add Song/Album & files| P3
    P3 -->|Raw Audio & Image files| Cloudinary
    Cloudinary -->|CDN URLs| P3
    P3 -->|Save Song with CDN URLs| DS2
    P3 -->|Save Album details & references| DS3
    P3 -->|Operation Confirmation| Admin
    Admin -->|Delete Song/Album request| P3
    P3 -->|Remove records| DS2
    P3 -->|Remove records| DS3

    %% 4.0 Chat & Status
    User -->|Socket Conn / User Online| P4
    P4 -->|Broadcast Listener Activities| User
    User -->|Send Chat Message| P4
    P4 -->|Write Chat Log| DS4
    DS4 -->|Chat History Log| P4
    P4 -->|Dispatch Realtime Messages| User

    %% 5.0 Admin Stats
    Admin -->|View Dashboard Statistics| P5
    P5 -->|Query Document Counts & Unique Artists| DS1
    P5 -->|Query Document Counts & Unique Artists| DS2
    P5 -->|Query Document Counts & Unique Artists| DS3
    DS1 -->|Stats Counts| P5
    DS2 -->|Stats Counts| P5
    DS3 -->|Stats Counts| P5
    P5 -->|Aggregated Analytics Report| Admin
```

---

## 🚀 Installation & Setup

Follow these steps to set up and run the project locally.

### Prerequisites
*   Node.js (v18+)
*   MongoDB Atlas Account
*   Clerk Account (for authentication)
*   Cloudinary Account (for media uploads)

### 1. Setup Environment Variables

Create a `.env` file in the **`backend`** folder:
```env
PORT=5000
MONGODB_URI=your_mongodb_connection_string
ADMIN_EMAIL=your_admin_email_registered_on_clerk
NODE_ENV=development

CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name

CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key
```

Create a `.env` file in the **`frontend`** folder:
```env
VITE_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
```

### 2. Run the Application

#### Automated Scripts
You can install and build both client and server from the root directory:
```bash
# Install all dependencies and build frontend
npm run build

# Start the backend server
npm start
```

#### Running in Development Mode
To run client and server independently:

**Start Backend server:**
```bash
cd backend
npm install
npm run dev
```

**Start Frontend client:**
```bash
cd frontend
npm install
npm run dev
```
The frontend will run at `http://localhost:3000` (or `http://localhost:5173` depending on ports configured).

---

## 📈 Key API Endpoints Reference

*   **Auth Routes** (`/api/auth`)
    *   `POST /callback`: Handles new user signup/login syncing with clerk details.
*   **User Routes** (`/api/users`)
    *   `GET /`: Fetches all registered users (excluding current active user).
    *   `GET /messages/:userId`: Fetches chat logs between current user and requested user.
*   **Admin Routes** (`/api/admin`) *(Requires admin email authorization)*
    *   `GET /check`: Verifies admin status.
    *   `POST /songs`: Uploads and publishes a song to Cloudinary and MongoDB.
    *   `DELETE /songs/:id`: Deletes song record and database relationship.
    *   `POST /albums`: Creates a new music album.
    *   `DELETE /albums/:id`: Deletes album and dependencies.
*   **Song Routes** (`/api/songs`)
    *   `GET /featured`: Gets curated tracks list.
    *   `GET /made-for-you`: Gets customized recommendations.
    *   `GET /trending`: Gets trending tracks list.
*   **Album Routes** (`/api/albums`)
    *   `GET /`: Gets all albums.
    *   `GET /:albumId`: Gets specific album details and associated song items.
*   **Stat Routes** (`/api/stats`) *(Admin only)*
    *   `GET /`: Returns total count of songs, albums, users, and unique artists.
