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
