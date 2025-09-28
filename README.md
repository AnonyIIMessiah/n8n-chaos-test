ShopEasy - A Compact Three-Tier E-Commerce Web Application
Overview
ShopEasy is a streamlined e-commerce web application designed for browsing, purchasing, and managing products online. It follows a three-tier architecture to ensure modularity, scalability, and maintainability. The app supports product listings, a shopping cart, user authentication, and order processing, making it ideal for small-scale online stores or as a foundation for larger platforms.
Key features:

User authentication (signup/login).
Product browsing and search.
Shopping cart management (add/remove items).
Order creation and tracking.
Persistent storage for users, products, and orders.

Architecture
ShopEasy is structured as a three-tier application, separating concerns for better organization and independent scaling:

Presentation Tier (Frontend):

Provides an intuitive user interface for browsing products, managing carts, and placing orders.
Built with React.js for dynamic, component-driven UI.
Communicates with the backend via RESTful APIs (e.g., GET /api/products for product listings, POST /api/cart for cart updates).
Key components: Product catalog, Cart view, Checkout form, User profile.
Served via a web server (e.g., Nginx) or bundled with the backend in development.
Notable features: Responsive design, client-side form validation, real-time cart updates.


Application Tier (Backend):

Handles business logic, API routing, and request processing.
Developed using Node.js with Express.js for efficient API management.
Key endpoints: 
/api/auth/register and /api/auth/login for user authentication.
/api/products for product CRUD operations.
/api/cart for cart management.
/api/orders for order creation and retrieval.


Uses JWT for secure authentication and Mongoose for MongoDB interactions.
Deployed on port 3000 by default; supports asynchronous request handling.
Scalability: Can be containerized or deployed across multiple instances.


Data Tier (Database):

Stores data for users, products, carts, and orders.
Utilizes MongoDB for a flexible, schema-less NoSQL database.
Schemas:
User: username, email, password hash, role (customer/admin).
Product: name, description, price, stock, category.
Cart: userId, items (productId, quantity).
Order: userId, items, total, status, createdAt.


Connected via MongoDB URI (e.g., mongodb://localhost:27017/shopeasy).
Features indexes (e.g., on product category, user email) and transactions for order processing.
Supports data export via mongodump.



Inter-Tier Communication:

Frontend ↔ Backend: HTTP/REST with JSON payloads.
Backend ↔ Database: MongoDB driver queries.This separation enables independent development, testing, and deployment of each tier.

Technologies Used

Frontend: React.js (v18+), Axios for API requests, Tailwind CSS for styling.
Backend: Node.js (v20+), Express.js (v4+), Mongoose (v8+), JWT for authentication.
Database: MongoDB (v7+).
Tools: npm for package management, Docker (optional), Git for version control.

Prerequisites

Node.js (v20 or higher).
MongoDB server (local or cloud, e.g., MongoDB Atlas).
Git for repository cloning.

Installation

Clone the repository:
git clone https://github.com/yourusername/shopeasy.git
cd shopeasy


Install backend dependencies:
cd backend
npm install


Install frontend dependencies:
cd ../frontend
npm install


Configure environment variables:

Create .env in the backend directory:PORT=3000
MONGO_URI=mongodb://localhost:27017/shopeasy
JWT_SECRET=your_jwt_secret_key




Start MongoDB (if local):
mongod



Running the Application

Start the backend:
cd backend
npm start


Accessible at http://localhost:3000.


Start the frontend:
cd frontend
npm start


Accessible at http://localhost:3001 (proxies API calls to backend).


Open http://localhost:3001 in your browser to:

Sign up/login.
Browse products, add to cart, and place orders.



For production, build the frontend (npm run build) and serve via the backend or a static host.
Configuration

Database: Update MONGO_URI in .env for custom MongoDB instances.
API Endpoints:
/api/auth/register: POST for signup.
/api/auth/login: POST for login.
/api/products: GET/POST for product listing/creation (admin only for POST).
/api/cart: GET/POST/PUT for cart operations.
/api/orders: POST/GET for order creation/retrieval.


Security: JWT protects all routes except auth endpoints; admin routes use role-based access.

Testing

Unit Tests: Jest for backend APIs/models (npm test in backend).
Integration Tests: Use Postman or scripts for end-to-end testing.
Frontend Tests: React Testing Library (npm test in frontend).
Verify: API responses, cart calculations, order status updates.

Deployment

Docker: Dockerfile in root for containerization.
Build: docker build -t shopeasy .
Run: docker run -p 3000:3000 -e MONGO_URI=your_mongo_uri shopeasy


Cloud: Deploy on AWS, Heroku, or Vercel (frontend) + Render (backend).
Ensure .env variables are configured in the deployment environment.

Troubleshooting

Database Errors: Check MongoDB URI and server status.
CORS Issues: Backend includes CORS middleware; verify allowed origins.
JWT Issues: Refresh token on login if expired.
Logs: Console-based; use Winston for production logging.

Contributing
Fork the repo, create a feature branch, and submit a pull request. Adhere to ESLint conventions.
License
Licensed under the MIT License. See LICENSE for details.
