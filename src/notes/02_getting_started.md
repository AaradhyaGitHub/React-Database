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

We can improve on that. The then blocks chain method is good but the async await
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

### **Enhancing UX for Slow Network Conditions**

When fetching data over a **slow 3G connection**, users experience a delay before content appears. Instead of showing **"No places available"** immediately, we provide:

✅ **A loading state** → Displays `"Fetching Places..."` while waiting for data.  
✅ **Better UX** → Prevents confusion when data is still loading.

---

### **1️⃣ Updating `AvailablePlaces.jsx` to Handle Loading State**

We add a **state variable** `isFetching` to track whether data is still being fetched.

```jsx
import { useState, useEffect } from "react";
import Places from "./Places.jsx";

export default function AvailablePlaces({ onSelectPlace }) {
  const [isFetching, setIsFetching] = useState(false);
  const [availablePlaces, setAvailablePlaces] = useState([]);

  useEffect(() => {
    setIsFetching(true); // Start loading
    async function fetchPlaces() {
      try {
        const response = await fetch("http://localhost:3000/places");
        const resData = await response.json();
        setAvailablePlaces(resData.places);
      } catch (error) {
        console.error("Error fetching places:", error);
      } finally {
        setIsFetching(false); // Stop loading when done
      }
    }
    fetchPlaces();
  }, []);

  return (
    <Places
      title="Available Places"
      places={availablePlaces}
      isLoading={isFetching}
      loadingText="Fetching Places..."
      fallbackText="No places available."
      onSelectPlace={onSelectPlace}
    />
  );
}
```

---

### **2️⃣ Enhancing `Places.jsx` to Display a Loading State**

We modify `Places.jsx` to **conditionally** show a **loading message** while data is being fetched.

```jsx
export default function Places({
  title,
  places,
  fallbackText,
  onSelectPlace,
  isLoading,
  loadingText
}) {
  return (
    <section className="places-category">
      <h2>{title}</h2>

      {/* Show loading text if data is still being fetched */}
      {isLoading && <p className="fallback-text">{loadingText}</p>}

      {/* Show fallback text if no places are available AFTER loading */}
      {!isLoading && places.length === 0 && (
        <p className="fallback-text">{fallbackText}</p>
      )}

      {/* Show places list when data is available */}
      {!isLoading && places.length > 0 && (
        <ul className="places">
          {places.map((place) => (
            <li key={place.id} className="place-item">
              <button onClick={() => onSelectPlace(place)}>
                <img
                  src={`http://localhost:3000/${place.image.src}`}
                  alt={place.image.alt}
                />
                <h3>{place.title}</h3>
              </button>
            </li>
          ))}
        </ul>
      )}
    </section>
  );
}
```

---

### **3️⃣ What’s Happening Here?**

1️⃣ **`isFetching` State Added**

- Set to `true` before fetching starts.
- Set to `false` after fetching completes (whether successful or failed).

2️⃣ **`useEffect` Fetching Logic**

- **Try-Catch Block** handles errors.
- **Finally Block** ensures `setIsFetching(false)` runs **regardless of success or failure**.

3️⃣ **`Places.jsx` Adjustments**

- Shows `"Fetching Places..."` while data loads.
- Shows `"No places available."` only **after** loading is done and no data exists.

---

### **4️⃣ Benefits of This Approach**

✅ **Better UX:** Users see a clear loading message instead of an empty screen.  
✅ **Handles Slow Connections:** Reactively updates the UI when data arrives.  
✅ **Prevents Misleading Messages:** `"No places available."` only appears **after** the request completes.
