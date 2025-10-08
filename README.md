# React Node.js Full-Stack Application

A full-stack web application built with React frontend and Node.js backend, featuring user management functionality with a complete workflow from frontend to backend.

## 🏗️ Architecture Overview

This application follows a modern full-stack architecture with:
- **Frontend**: React.js with Bootstrap UI components
- **Backend**: Node.js with Express.js server
- **Containerization**: Docker for deployment
- **Build Process**: Multi-stage Docker build for optimization

## 📁 Project Structure

```
react-nodejs-example-master/
├── my-app/                    # React Frontend
│   ├── src/
│   │   ├── components/        # React Components
│   │   │   ├── CreateUser.js
│   │   │   ├── Users.js
│   │   │   ├── DisplayBoard.js
│   │   │   └── Header.js
│   │   ├── services/          # API Services
│   │   │   └── UserService.js
│   │   └── App.js            # Main App Component
│   ├── public/               # Static Assets
│   └── build/                # Production Build
├── api/                      # Node.js Backend
│   ├── server.js            # Express Server
│   ├── package.json         # Backend Dependencies
│   └── server.bundle.js     # Bundled Server
├── Dockerfile               # Docker Configuration
└── key-pair/                # AWS Key Pairs
```

## 🔄 Frontend to Backend Workflow

### 1. **User Interface Layer (React Components)**

#### CreateUser Component
- **Location**: `my-app/src/components/CreateUser.js`
- **Purpose**: Form for creating new users
- **Features**:
  - Form inputs for firstName, lastName, email
  - Real-time form state management
  - Submit button triggers user creation

#### Users Component
- **Location**: `my-app/src/components/Users.js`
- **Purpose**: Display list of all users
- **Features**:
  - Table format display
  - Dynamic user count
  - Responsive design

### 2. **Service Layer (API Communication)**

#### UserService.js
- **Location**: `my-app/src/services/UserService.js`
- **Purpose**: Handles all API communication
- **Functions**:
  ```javascript
  // Fetch all users from backend
  getAllUsers() → GET /api/users
  
  // Create new user
  createUser(data) → POST /api/user
  ```

### 3. **State Management (React Hooks)**

#### App.js State Flow
```javascript
const [user, setUser] = useState({})           // Current user form data
const [users, setUsers] = useState([])         // All users list
const [numberOfUsers, setNumberOfUsers] = useState(0)  // User count
```

#### Data Flow Process:
1. **Form Input** → `onChangeForm()` → Updates `user` state
2. **Form Submit** → `userCreate()` → Calls `createUser()` service
3. **Service Call** → API request to backend
4. **State Update** → Updates `users` array and `numberOfUsers` count
5. **UI Re-render** → Components reflect new data

### 4. **Backend API Layer (Node.js/Express)**

#### Server Configuration
- **Port**: 3080
- **Static Files**: Serves React build from `/my-app/build`
- **API Endpoints**: `/api/*` routes

#### API Endpoints

##### GET /api/users
```javascript
// Returns all users
app.get("/api/users", (req, res) => {
  console.log("api/users called!");
  res.json(users);
});
```

##### POST /api/user
```javascript
// Creates new user
app.post("/api/user", (req, res) => {
  const user = req.body.user;
  console.log("Adding user:::::", user);
  users.push(user);
  res.json("user addedd");
});
```

##### GET /
```javascript
// Serves React application
app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "../my-app/build/index.html"));
});
```

## 🔄 Complete Data Flow

### User Creation Workflow:

1. **Frontend Form** (`CreateUser.js`)
   - User fills form fields (firstName, lastName, email)
   - `onChangeForm()` updates component state

2. **Form Submission** (`App.js`)
   - User clicks "Create" button
   - `userCreate()` function triggered
   - Calls `createUser(user)` service

3. **API Service** (`UserService.js`)
   - `createUser()` makes POST request to `/api/user`
   - Sends user data as JSON payload
   - Returns response to component

4. **Backend Processing** (`server.js`)
   - Express receives POST request at `/api/user`
   - Extracts user data from request body
   - Adds user to in-memory array
   - Returns success response

5. **State Update** (`App.js`)
   - Updates `numberOfUsers` count
   - Triggers re-render of components

6. **UI Refresh** (`Users.js`)
   - Displays updated user list
   - Shows new user count

### User Retrieval Workflow:

1. **Component Mount** (`App.js`)
   - `useEffect()` hook triggers on component load
   - Calls `fetchAllUsers()` function

2. **API Service** (`UserService.js`)
   - `getAllUsers()` makes GET request to `/api/users`
   - Returns user array from backend

3. **Backend Response** (`server.js`)
   - Express serves user data from in-memory array
   - Returns JSON response

4. **State Update** (`App.js`)
   - Updates `users` state with fetched data
   - Updates `numberOfUsers` count

5. **UI Display** (`Users.js`)
   - Renders user table with fetched data

## 🚀 Setup and Installation

### Prerequisites
- Node.js (v18+)
- npm
- Docker (optional)

### Development Setup

#### Frontend Development
```bash
cd my-app
npm install
npm start
```
- Runs on: http://localhost:3001
- Proxy configured to backend on port 80

#### Backend Development
```bash
cd api
npm install
npm run dev
```
- Runs on: http://localhost:3080
- Hot reload with nodemon

### Production Deployment

#### Docker Build
```bash
docker build -t react-nodejs-app .
docker run -p 3080:3080 react-nodejs-app
```

#### Multi-stage Build Process:
1. **UI Build Stage**: Builds React application
2. **Server Build Stage**: Sets up Node.js server
3. **Production Stage**: Combines frontend build with backend

## 🛠️ Technologies Used

### Frontend
- **React 16.13.1**: UI framework
- **Bootstrap 4.5.0**: CSS framework
- **React Bootstrap**: Component library
- **Fetch API**: HTTP requests

### Backend
- **Node.js 18**: Runtime environment
- **Express.js 4.17.1**: Web framework
- **Body Parser**: Request parsing
- **HTTP Proxy Middleware**: API proxying

### DevOps
- **Docker**: Containerization
- **Multi-stage Build**: Optimized production builds
- **Webpack**: Module bundling
- **Gulp**: Build automation

## 📊 API Documentation

### Endpoints

| Method | Endpoint | Description | Request Body | Response |
|--------|----------|-------------|--------------|----------|
| GET | `/api/users` | Get all users | - | Array of user objects |
| POST | `/api/user` | Create new user | `{user: {firstName, lastName, email}}` | Success message |
| GET | `/` | Serve React app | - | HTML page |

### User Object Structure
```javascript
{
  firstName: "string",
  lastName: "string", 
  email: "string"
}
```

## 🔧 Configuration

### Frontend Proxy
```json
// my-app/package.json
{
  "proxy": "http://localhost:80"
}
```

### Docker Configuration
```dockerfile
# Multi-stage build
FROM node:18 AS ui-build    # Frontend build
FROM node:18 AS server-build # Backend setup
EXPOSE 3080                 # Port configuration
```

## 🚀 Deployment

### AWS Deployment
- Key pairs included in `key-pair/` directory
- Docker container ready for EC2 deployment
- Static file serving configured

### Local Development
1. Start backend: `cd api && npm run dev`
2. Start frontend: `cd my-app && npm start`
3. Access application at http://localhost:3001

## 📝 Features

- ✅ User creation with form validation
- ✅ Real-time user list display
- ✅ User count tracking
- ✅ Responsive Bootstrap UI
- ✅ RESTful API endpoints
- ✅ Docker containerization
- ✅ Production-ready build process

## 🤝 Contributing

1. Fork the repository
2. Create feature branch
3. Commit changes
4. Push to branch
5. Create Pull Request

## 📄 License

This project is licensed under the ISC License.

---

**Repository**: [https://github.com/Nitinkumargits/ReactNode_website.git](https://github.com/Nitinkumargits/ReactNode_website.git)
