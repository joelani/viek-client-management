# VIEK Client Management — Debugging Assessment

## Overview

Debugged a React/Vite client management application and Node.js/Express REST API. Fixed 11 issues involving rendering, API contracts, state management, request handling, ID generation, filtering, and credential exposure.

## Bugs Identified and Fixed

| #   | Issue                                  | Root Cause                                           | Fix                                                       |
| --- | -------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------- |
| 1   | Blank page                             | Missing React import                                 | Added the React import in `App.jsx`.                      |
| 2   | Clients list not rendering             | API returned `data` instead of `clients`             | Standardized the response property to `clients`.          |
| 3   | Projects render crash                  | Project state initialized as `undefined`             | Initialized project state with `useState([])`.            |
| 4   | Client creation returned 400           | Missing JSON content-type header                     | Added `Content-Type: application/json`.                   |
| 5   | Password exposed in login response     | Handler returned the complete user object            | Returned only the token, ID, name, and email.             |
| 6   | Client deletion failed                 | Compared numeric IDs with a string route parameter   | Converted the route ID to a number.                       |
| 7   | Project filtering returned no results  | Compared numeric IDs with a string query parameter   | Converted the query parameter to a number.                |
| 8   | Projects displayed raw client IDs      | Rendered the ID instead of resolving the client      | Displayed the associated client name with an ID fallback. |
| 9   | Duplicate client IDs after deletion    | Generated IDs using `clients.length + 1`             | Generated the next ID from the highest existing ID.       |
| 10  | Failed requests could corrupt UI state | Loaders updated state without checking `response.ok` | Added response guards before updating state.              |
| 11  | Inconsistent client creation response  | Create endpoint returned `data`                      | Standardized the response to `{ message, client }`.       |

## Setup

Requirements: Node.js and npm.

Start the backend:

```bash
cd server
npm install
npm start
```

In a separate terminal, start the frontend:

```bash
cd client
npm install
npm run dev
```

Default development URLs:

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:4000`

## Testing

All 11 regression tests passed.

- Valid login and password-free login response.
- Invalid login rejection.
- Client list retrieval and rendering.
- Valid client creation.
- Invalid client input rejection.
- Client deletion without affecting other clients.
- Unique ID generation after deletion and creation.
- Correct rendering of all seeded projects and client names.
- Correct project filtering by client.
- Rejection of unauthenticated requests on protected endpoints.
- `404 Not Found` when deleting a nonexistent client.

The backend and frontend also started successfully, and the login screen rendered after the React import fix.

## Security and Limitations

The login response no longer exposes the user's password. This fix does not establish production-grade authentication. Password hashing, robust token generation and verification, and appropriate CORS restrictions require further review before production use.

Client data and generated IDs are held in application memory. This implementation does not provide persistent storage or database-level uniqueness guarantees.

## Reflection

I traced errors across frontend components and backend endpoints, checked API response contracts and data types, and applied targeted fixes. I then tested the affected user flows, including authentication, client management, project filtering, and error handling.
