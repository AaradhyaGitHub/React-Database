### **Establishing a Connection Between React and a Dummy Backend**

Now that our **backend is up and running**, let’s break down how to properly fetch data from it in our React app.

---

### **1️⃣ Running the Backend & Frontend Servers**

Before fetching data, ensure both servers are running:

- **Backend:**
  ```sh
  cd backend
  npm install  # Install dependencies
  node app.js  # Start the backend server
  ```
- **Frontend:**
  ```sh
  npm run dev  # Start Vite server
  ```

At this point, we have:  
✅ **Backend** handling API requests  
✅ **Frontend** (React) ready to fetch data

---

### **2️⃣ Why We Need to Send an HTTP Request**

Before, we stored places locally:

```jsx
const places = localStorage.getItem("places");
const [availablePlaces, setAvailablePlaces] = useState([]);
```

🚨 **This won’t work anymore!** 🚨  
Now, we are fetching data from a backend, which means:

- The frontend **cannot** directly access the data.
- We need to send an **HTTP request** to get the data.
- The request takes **time**, so the component **renders first**, then updates once the data arrives.

---

### **3️⃣ Fetching Data from the Backend**

Here’s how we modify `AvailablePlaces.jsx` to fetch data:

```jsx
import { useState, useEffect } from "react";
import Places from "./Places.jsx";

export default function AvailablePlaces({ onSelectPlace }) {
  const [availablePlaces, setAvailablePlaces] = useState([]);

  useEffect(() => {
    fetch("http://localhost:3000/places")
      .then((response) => {
        return response.json();
      })
      .then((resData) => {
        setAvailablePlaces(resData.places);
      });
  }, []);

  return (
    <Places
      title="Available Places"
      places={availablePlaces}
      fallbackText="No places available."
      onSelectPlace={onSelectPlace}
    />
  );
}
```

We can improve on that. The then blocks chain is good but the async await
method is prefered and just better

```jsx
seEffect(() => {
  async function fetchPlaces() {
    const response = await fetch("http://localhost:3000/places");
    const resData = await response.json();
    setAvailablePlaces(resData.places);
  }
  fetchPlaces();
}, []);
```

---
