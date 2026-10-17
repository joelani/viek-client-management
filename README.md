Important observation: The supplied backend returns the user's password in that response. Record this as a security issue; we'll address it after establishing our baseline.

# VIEK Client Management — Debugging Assessment

## Bugs Identified

### 1. React Runtime Error

- **File:** `client/src/App.jsx`
- **Issue:** The application displayed a blank page because `React` was not defined.
- **Root cause:** The module imported React hooks but did not import the `React` identifier required by the JSX transformation in the current setup.
- **Solution:** Added the default React import alongside `useEffect` and `useState`.
- **Testing:** Confirmed that the login screen rendered successfully after the change.
- **Status:** Fixed and verified.

### 2. Clients List Mismatch

- **File:** `server/server.js`
- **Issue:** The clients list did not render because the frontend read the list from `result.clients` while the `GET /api/clients` endpoint returned it as `data`.
- **Root cause:** Response property name mismatch between backend (`data`) and frontend expectation (`clients`), inconsistent with `GET /api/projects` which already uses `projects`.
- **Solution:** Changed the `GET /api/clients` JSON response property from `data` to `clients`.
- **Testing:** Re-traced data flow from `loadClients()` in `client/src/App.jsx` to `GET /api/clients` in `server/server.js` and confirmed both sides now use `clients`.
- **Status:** Fixed and verified.

### 3. Projects Runtime Error

- **File:** `client/src/App.jsx`
- **Issue:** The application crashed after login with `Uncaught TypeError: Cannot read properties of undefined (reading 'map')` at line 278.
- **Root cause:** The `projects` state was initialized as `undefined` while the render called `projects.map()` unconditionally before `loadProjects()` resolved, unlike `clients` which was initialized to `[]`.
- **Solution:** Changed the `projects` state initialization from `useState()` to `useState([])`.
- **Testing:** Verified `client/src/App.jsx` line 19 now initializes to `[]` and line 278 `projects.map()` can render safely on first load.
- **Status:** Fixed and verified.

### 4. AddClient Bad Request

- **File:** `client/src/App.jsx`
- **Issue:** Adding a client failed with `POST http://localhost:4000/api/clients 400 (Bad Request)` and message `Name and email are required`.
- **Root cause:** The `addClient()` fetch sent `JSON.stringify({name, email})` without `Content-Type: application/json`, so `express.json()` did not parse the body and `req.body` was empty, unlike `login()` which already sent the header.
- **Solution:** Added `"Content-Type": "application/json"` to the `addClient()` request headers.
- **Testing:** Verified `client/src/App.jsx` line 100 now sends `Content-Type` header and traced flow to `POST /api/clients` validation in `server/server.js`.
- **Status:** Fixed and verified.

### 5. Login Password Exposure

- **File:** `server/server.js`
- **Issue:** The `POST /api/login` endpoint returned the full user object including `password` in plain text.
- **Root cause:** The handler sent the whole `user` object from the `users` array, which contains `password: "password123"`, while `client/src/App.jsx` only needed `result.token`.
- **Solution:** Changed login response to return only `token`, `id`, `name`, and `email` instead of the full `user` object.
- **Testing:** Verified `server/server.js` lines 68-73 no longer include `user` or `password` and traced flow from `login()` in `client/src/App.jsx`.
- **Status:** Fixed and verified.

### 6. DeleteClient Type Mismatch

- **File:** `server/server.js`
- **Issue:** Deleting a client always failed with `404 Client not found` even for existing IDs.
- **Root cause:** The `DELETE /api/clients/:id` handler compared numeric `client.id` with string `req.params.id` using strict inequality (`client.id !== id`), which is always true for `1 !== "1"`, so nothing was ever filtered out.
- **Solution:** Changed filter comparison to `client.id !== Number(id)` to normalize types before strict comparison.
- **Testing:** Verified `server/server.js` line 126 now converts `id` to number and traced flow from `deleteClient()` in `client/src/App.jsx`.
- **Status:** Fixed and verified.

## Testing

- Started the Express backend successfully.
- Tested login using the supplied assessment credentials and received a JSON response.
- Started the Vite development server.
- Verified that the login screen renders after fixing the React runtime error.

## Security

- Initial observation: The login endpoint returns the user object, including the password, in its response. This requires correction.
- Further security investigation and testing are pending.

## Reflection

- **Debugging approach:** Reproduced the runtime error, inspected the component imports, applied a targeted fix, and verified the result in the browser.
- **Outstanding work:** Investigate and resolve the remaining frontend, backend, API, data-handling, authentication, and security issues.
