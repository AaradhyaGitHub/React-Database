## **1. Understanding the Three States**
These three states help manage the request lifecycle and UI updates:

1. **`isFetching` (Boolean)**
   - Purpose: Tracks whether data is currently being fetched.
   - Usage: It starts as `false`, is set to `true` before fetching, and then back to `false` after the request is complete.

2. **`availablePlaces` (Array)**
   - Purpose: Stores the fetched list of places from the backend.
   - Usage: Initially an empty array (`[]`), it updates when the API responds with data.

3. **`error` (Object or `undefined`)**
   - Purpose: Stores error details if the request fails.
   - Usage: If an error occurs, it holds an object with an error message. Otherwise, it remains `undefined`.

---

## **2. `useEffect` Hook**
```javascript
useEffect(() => {
  setIsFetching(true);
  async function fetchPlaces() {
    try {
      const response = await fetch("http://localhost:3000/places");
      const resData = await response.json();

      if (!response.ok) {
        throw new Error("Failed to fetch places");
      }
      setAvailablePlaces(resData.places);
    } catch (error) {
      setError({
        message: error.message || "Could not fetch places. Please try again later"
      });
    }
    setIsFetching(false);
  }
  fetchPlaces();
}, []);
```
### **How `useEffect` Works**
- `useEffect(() => { ... }, []);` runs the provided function **once** when the component mounts (due to the empty `[]` dependency array).
- This is where side effects (like fetching data from an API) are handled.

### **Why We Use It Here**
- It ensures that the API request runs once when the component loads.
- Prevents unnecessary re-renders.

---

## **3. `async/await` for Fetching Data**
```javascript
async function fetchPlaces() {
  try {
    const response = await fetch("http://localhost:3000/places");
    const resData = await response.json();
    
    if (!response.ok) {
      throw new Error("Failed to fetch places");
    }

    setAvailablePlaces(resData.places);
  } catch (error) {
    setError({ message: error.message || "Could not fetch places. Please try again later" });
  }

  setIsFetching(false);
}
```
### **Breakdown**
- **`async` function** allows the use of `await` inside it.
- **`await fetch("http://localhost:3000/places")`**:
  - Sends a GET request to the backend.
  - Waits for the response before moving to the next line.
- **`await response.json()`**:
  - Converts the response from JSON format into a JavaScript object.
- **`if (!response.ok) throw new Error("Failed to fetch places")`**:
  - If the request fails (e.g., `404 Not Found` or `500 Server Error`), we manually throw an error.

---

## **4. Error Handling with `try/catch`**
```javascript
try {
  const response = await fetch("http://localhost:3000/places");
  const resData = await response.json();

  if (!response.ok) {
    throw new Error("Failed to fetch places");
  }
  setAvailablePlaces(resData.places);
} catch (error) {
  setError({
    message: error.message || "Could not fetch places. Please try again later"
  });
}
```
### **How `try/catch` Works**
- **`try { ... }` block**:
  - Runs the API request.
  - If everything works, it updates `availablePlaces`.
- **`catch (error) { ... }` block**:
  - Catches any error from `fetch` or JSON parsing.
  - Updates `error` state to display an error message.

---

## **5. Conditional Rendering Based on `error`**
```javascript
if (error) {
  return <Error title="An Error occurred!" message={error.message} />;
}
```
- If `error` is set, the `Error` component is displayed.
- This prevents rendering an empty or broken UI.

---

## **6. Rendering the Data**
```javascript
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
```
- The `Places` component:
  - Displays the list of places (`availablePlaces`).
  - Shows a loading text if `isFetching` is `true`.
  - Displays `fallbackText` if no places are available.

---

## **7. Why These Three States Work Together**
1. **`isFetching`**
   - Tells the UI when to show a loading indicator.
   - Helps prevent user actions before data is ready.
2. **`availablePlaces`**
   - Stores the fetched data so it can be displayed in the UI.
   - Prevents unnecessary re-fetching.
3. **`error`**
   - Allows the UI to display an error message instead of failing silently.
   - Provides feedback when something goes wrong.

Together, these states create a **smooth and user-friendly experience** while handling data fetching.

---

### **Final Thoughts**
- `useEffect` ensures the fetch request happens when the component loads.
- `async/await` makes handling API requests cleaner.
- `try/catch` prevents crashes by handling errors gracefully.
- `useState` keeps track of loading status, data, and errors.
