Frontend Event Service

This is the Vue frontend for the Event Service in the Event Ticketing and Venue Management System.

The frontend communicates with the Event Service backend using:

GET http://localhost:8083/events

The backend returns real PostgreSQL data in JSON format.

Example response:

[
  {
    "eventId": 1,
    "title": "Concert",
    "description": "Music event",
    "category": "Music",
    "status": "DRAFT"
  }
]
Frontend Components

Main frontend files:

src/
│
├── components/
│   └── EventList.vue
│
├── App.vue
└── main.js
EventList Component

The EventList.vue component:

sends API requests to the backend
receives JSON responses
stores data using Vue refs
dynamically renders event cards
Run Locally

From the frontend folder:

npm install
npm run dev

Open:

http://localhost:5173
Run Backend

The Spring Boot backend must also be running.

Backend endpoint:

http://localhost:8083/events

Start backend:

mvn spring-boot:run
Docker Support

The backend service supports Docker containerization.

Build Docker image:

docker build -t event-service .

Run Docker container:
docker run -p 8083:8083 event-service
Technologies Used
Vue 3
Vite
JavaScript
Spring Boot
PostgreSQL
Docker

Real backend data displayed dynamically.
Build
npm run build
