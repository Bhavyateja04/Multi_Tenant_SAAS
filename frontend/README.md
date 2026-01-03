# Frontend – React Application

This frontend application is part of the **Multi-Tenant SaaS Project Management System**.  
It is built using **React** and was initially bootstrapped with **Create React App (CRA)**.

The frontend communicates with the backend REST APIs to handle authentication, tenant-scoped data access, and project/task management.

---

## Tech Stack

- **Framework:** React.js
- **Build Tool:** Create React App
- **Routing:** React Router DOM
- **State Management:** React Context API
- **HTTP Client:** Axios
- **Styling:** CSS3 (Flexbox & Grid)

---

## Available Scripts

In the `frontend` directory, you can run the following commands.

---

### `npm start`

Runs the application in **development mode**.

- Opens automatically at:  
  👉 http://localhost:3000
- Hot reloads on code changes
- Displays linting errors in the console

```bash
npm start
npm test
Runs the test runner in interactive watch mode.

bash
Copy code
npm test
Use this during development to validate UI logic and components.

npm run build
Builds the application for production.

Outputs optimized files to the build/ directory

Minifies JavaScript and CSS

Generates hashed filenames for cache busting

bash
Copy code
npm run build
The generated build can be served using:

Nginx

Docker

Any static hosting service

npm run eject
⚠️ Warning: This is a one-way operation

bash
Copy code
npm run eject
Exposes Webpack, Babel, and ESLint configurations

Gives full control over build setup

Cannot be undone

Note:
This project does not require ejecting.
The default CRA configuration is sufficient for production use.

Environment Configuration
The frontend relies on environment variables for backend communication.

Create a .env file in the frontend/ directory if required:

env
Copy code
REACT_APP_API_BASE_URL=http://localhost:5000/api
Application Structure
text
Copy code
src/
├── context/        # Global state (AuthContext)
├── pages/          # Page components (Login, Dashboard, Projects)
├── components/     # Reusable UI components
├── App.js          # Root component & routing
└── index.js        # Application entry point
Development Notes
Authentication state is managed using JWT stored in memory or localStorage

Tenant context is resolved during login

All API requests are scoped to the authenticated tenant

Unauthorized access is handled gracefully via route guards

Learn More
React Documentation: https://reactjs.org/

Create React App Docs: https://create-react-app.dev/

React Router: https://reactrouter.com/

Deployment
This frontend is typically deployed using Docker and served alongside the backend API.
```
