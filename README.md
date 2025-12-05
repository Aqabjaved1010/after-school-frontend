After School Classes – Full Stack App
This project is my CST3144 Coursework 1: a full stack web application for buying after‑school classes and activities. It uses a Vue.js front‑end and a Node.js / Express back‑end with MongoDB Atlas for data storage.​

Vue.js Front-End
GitHub repository (Vue app):
https://github.com/Aqabjaved1010/after-school-frontend

Live app on GitHub Pages:
https://aqabjaved1010.github.io/after-school-frontend/

The front-end:

Displays a list of lessons (subject, location, price, spaces, icon) using v-for.

Allows sorting by subject, location, price, and spaces in ascending or descending order.

Supports search (including “search as you type”) via a REST API to the back-end.

Implements add to cart, cart view, and checkout with JavaScript validation:

Name: letters only.

Phone: numbers only.​

Express.js Back-End
GitHub repository (Express app):
https://github.com/Aqabjaved1010/after-school-backend

Deployed API on Render (base URL):
https://after-school-backend-r4mc.onrender.com

Main routes:

GET /lessons – returns all lessons as JSON from MongoDB Atlas.
https://after-school-backend-r4mc.onrender.com/lessons

GET /search?q=term – searches lessons in MongoDB and returns matching results.
https://after-school-backend-r4mc.onrender.com/search?q=english

GET /orders – returns existing orders (used for inspection/demo).
https://after-school-backend-r4mc.onrender.com/orders

POST /orders – creates a new order document with customerName, customerPhone and items.
https://after-school-backend-r4mc.onrender.com/orders
Sample Data; {
  "customerName": "Testing User",
  "customerPhone": "07123756481",
  "items": [
    {
      "lessonId": "652b90cc0def08c132fd2700",
      "spaces": 1
    }
  ]
}


PUT /lessons/:id/spaces – updates the available spaces for a lesson.
(https://after-school-backend-r4mc.onrender.com/lessons/692ae6b6d65aa005c29d1cc4/spaces)
Sample Data; {
  "change": -1
}

Middleware:

Logger middleware that logs timestamp, HTTP method and URL for every request.
https://dashboard.render.com/web/srv-d4lfce0dl3ps7385hgn0/logs?r=1h

Static middleware:

app.use('/images', express.static('public/images'))

This serves lesson images from backend/public/images and returns a 404 error when a file does not exist.​
serves lesson image (https://after-school-backend-r4mc.onrender.com/images/maths.jpg)
returns 404 error (https://after-school-backend-r4mc.onrender.com/images/english.jpg)

Database (MongoDB Atlas)
Database name: after_school

Collections:

lessons

Fields include: topic, subject, location, price, spaces, icon.

orders

Fields include: customerName, customerPhone, items array (each item contains lesson and spaces information).​

The exported lessons and orders collections (JSON files) are included in the coursework ZIP as required.​

Postman Collection
The ZIP also contains an exported Postman collection with prepared requests for:

GET /lessons

GET /search

GET /orders

POST /orders

PUT /lessons/:id/spaces

These are used to test and demonstrate the REST API routes.​

How to Run Locally
Clone the repositories

Front-end:
git clone https://github.com/Aqabjaved1010/after-school-frontend.git

Back-end:
git clone https://github.com/Aqabjaved1010/after-school-backend.git​​

Back-end setup

Go into the backend folder:

cd after-school-backend

Install dependencies:

npm install

Create a .env file with:

MONGO_URL=<mongodb+srv://aqabjaved01:<db_password>@fullstack-cluster.exfltov.mongodb.net/?appName=fullstack-cluster>
PLEASE CONTACT ME FOR THE REAL PASSWORD IF NEEDED.

DB_NAME=after_school

Seed the database:

node seed.js

Start the server:

node server.js

The server will run on the port defined in .env.​

Front-end setup (development)

Go into the Vue project folder:

cd after-school-frontend/frontend/vue-project 

Install dependencies:

npm install

Make sure the API base URL in the front-end matches the back-end (for example https://after-school-backend-r4mc.onrender.com).

Run the dev server:

npm run dev

Open the local URL shown in the terminal in the browser.​

Build for GitHub Pages
To build and update the GitHub Pages deployment:

In the Vue project folder run:

npm run build

Copy the contents of dist into the docs folder of the front-end repo.

Commit and push:

git add .

git commit -m "Update GitHub Pages build"

git push

GitHub Pages is configured to serve from the main branch and /docs folder.​