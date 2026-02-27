Question 1: Render Phase vs Commit Phase

Answer:

🔹 Render Phase
1. React calculates what changed
2. Runs component functions
3. Creates and compares Virtual DOM
4. No DOM updates
5. Can be paused/interrupted (React 18)

🔹 Commit Phase
1. React updates the real DOM
2. Runs useEffect and refs
3. Cannot be interrupted
4. Side effects happen here

🎯 One-Line Interview Answer:
- Render phase computes changes, commit phase applies them to the DOM and runs side effects.
- Render is about computation, commit is about mutation.
  
Question 2: Virtual DOM vs. Real DOM (The correct mental model)

Answer:

The Virtual DOM is a lightweight JavaScript representation of the UI. 
When state changes, React creates a new Virtual DOM tree, compares it with the previous one, and calculates the minimal changes required. 
Those changes are then applied to the Real DOM during the commit phase. This makes updates efficient and predictable.

✅ Correct Mental Model

🔹 Real DOM
1. The actual browser DOM
2. Tree of real nodes (div, p, button)
3. Updating it is expensive
4. Causes layout + repaint

🔹 Virtual DOM
The Virtual DOM is a lightweight JavaScript representation of what the UI should look like.

It is:
1. Not the browser DOM
2. Just plain JS objects
3. Used for comparison (diffing)

Question 3: Fiber Reconciliation: Step-by-step flow

Answer:

1. When state changes, React schedules the update and starts the render phase.
2. In this phase, it runs the components again and creates a new version of the UI (Work-In-Progress tree).
3. It then compares this new tree with the current tree to see what changed (reconciliation)
4. During this comparison, React marks what needs to be added, updated, or removed. No real DOM changes happen here — it’s only calculation.
5. After that, React moves to the commit phase. Here it applies the necessary changes to the real DOM, runs useLayoutEffect, and then runs useEffect.
6. Once this is done, the new tree becomes the current tree, and the old tree is no longer used and gets garbage collected.

- Because Fiber breaks rendering into small tasks, the render phase can pause, resume, or prioritize updates in React 18, but the commit phase always runs fully without interruption.
- In short, steps of Fiber reconciliation — scheduling the update, building the work-in-progress tree during the render phase,
  comparing it with the current tree, marking effects, committing DOM changes, and finally swapping the trees.

Bonus Points
🎯 What Makes It Specifically “Fiber”?

Two key things:

🔹 Work-In-Progress Tree

React keeps two trees:

- Current tree (what’s on screen)
- WIP tree (what’s being prepared)

This double-buffering is a Fiber concept.

🔹 Interruptible Render Phase

Fiber allows:

- Pausing
- Resuming
- Prioritizing updates

Old React (Stack Reconciler) could not do this.

Question 4: How React schedules priority updates (Lanes)?

Answer:
In React 18, lanes are a priority-based scheduling system. When a state update occurs, React assigns it a lane that represents its priority. Internally, React uses bitmasks to represent these lanes, which allows multiple priorities to be tracked efficiently at the same time. React processes higher-priority lanes first and can pause lower-priority rendering if a more urgent update arrives. This enables concurrent rendering and improves UI responsiveness.

Bonus:
1. A bitmask is just a number written in binary (0s and 1s).
2. Each bit represents one priority lane.

Question 5:  • Rules of Hooks and how they are mapped internally?

Answer:
1) Only call Hooks at the top level
   
❌ Don’t call inside:
- if conditions
- loops
- nested functions
- try/catch

Because React tracks Hooks by their call order.
If a Hook runs sometimes and skips other times, the order changes between renders, 
so React assigns the wrong state to the wrong Hook → bugs/crashes.

React relies on the order of Hook calls to associate each Hook with its state. 
React doesn't know which Hook is which by name because it tracks them by call order.

✅ Always call hooks at the top of your component / custom hook.

2) Only call Hooks from React functions

✅ Allowed:

- React function components
- custom hooks (functions starting with use)

❌ Not allowed:

- normal JS functions
- class components

Question 5:  What happens if you call a setState inside useEffect with no dependency array?

Answer: If we call setState inside a useEffect without a dependency array, the effect runs after every render. Since setState triggers a re-render, it causes an infinite render loop, leading to a "Maximum update depth exceeded" error. To prevent this, we either provide a dependency array or conditionally update state.

❌ Bad Example:
function App() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    setCount(count + 1);
  });

  return <h1>{count}</h1>;
}

Here,
useEffect runs after every render
setState causes a new render
That new render runs the effect again, so creates a loop
Render → Effect → setState → Render → Effect → setState → ...

✅ Correct Fix
useEffect(() => {
  setCount(1);
}, []);

Question 6:  what is tearing in React? and how does concurrent mode solve it?
Tearing happens when different components read different versions of the same state during rendering so the UI becomes inconsistent.

🧠 Simple Example
Imagine you have a global store:

const store = {
  value: 0
};

Two components read from this store:
function ComponentA() {
  return <h1>{store.value}</h1>;
}

function ComponentB() {
  return <h2>{store.value}</h2>;
}

Now imagine:
1. React starts rendering
2. ComponentA reads value = 0
3. Before ComponentB renders, store updates to 1
4. ComponentB reads value = 1

Now UI shows:
ComponentA → 0
ComponentB → 1

❌ Same state source
❌ Same render cycle
❌ Different values

That inconsistency is called tearing.

🔹 Why Does Tearing Happen?

Tearing mainly happens in Concurrent Rendering when:

1. React can pause rendering
2. External store updates in between
3. Components read different snapshots
4. It usually occurs when using:
External stores (Redux, Zustand, custom store) Without proper subscription handling
