# React Basic Learnings

```jsx
import { useState, useEffect } from "react";

export default function App() {
  const [count, setCount] = useState(0); // State to track count

  useEffect(() => {
    console.log("Count updated: ", count);
  }, [count]); // Runs whenever count changes

  return (
    <div className="p-4 text-center">
      <h1 className="text-2xl">Counter: {count}</h1>
      <button
        onClick={() => setCount(count + 1)}
        className="px-4 py-2 mt-4 bg-blue-500 text-white rounded"
      >
        Increment
      </button>
    </div>
  );
}

```

**Walkthrough:**

1.	**useState for State Management**

•	const [count, setCount] = useState(0); initializes a state variable count with an initial value of 0

•	setCount is the function to update count.

2.	**useEffect for Side Effects**

•	useEffect(() => { console.log("Count updated: ", count); }, [count]);

•	This effect runs every time count changes and logs the new value.

3.	**Rendering the UI**

•	Displays the count inside an <h1>.

•	A button increases count when clicked, triggering a re-render and re-running the effect.
