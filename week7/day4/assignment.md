# Week 7 - Day 4 Assignment

## Title
Rendering Lists, Conditional UI, and the Task Manager Pre-Weekend

## Overview
Day 4 is pre-weekend polish day. Today you combine everything this week has taught: components, props, state, lists, conditional rendering -- all in service of a Task Manager app. The Task Manager is THE canonical first-React project for a reason: it forces you to manage a list of items in state, add to it, remove from it, and render it dynamically.

## Learning Objectives Assessed
- Manage an array in state correctly (immutably)
- Render a list with unique keys
- Conditionally render based on state
- Use `&&` and ternary for inline conditionals
- Begin the weekend project

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 60/40. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Extending the Task Manager with AI after your manual version has add/complete/delete working.
- **NOT ALLOWED FOR:** Generating the first version of the Task Manager.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: TaskManager component, manual version

**What to do:**
Create `src/components/TaskManager.jsx`. Requirements:

1. State: an array of task objects `[{ id, text, completed }]` and a string for the current input.
2. An input field and an Add button. On click (or Enter), adds a new task to the list.
3. A list of tasks. Clicking a task toggles `completed`.
4. Each task has a delete button next to it.
5. The list uses `key={task.id}`.

You may use `Date.now()` or `crypto.randomUUID()` for IDs.

```jsx
import { useState } from "react";

function TaskManager() {
  const [tasks, setTasks] = useState([]);
  const [input, setInput] = useState("");

  function addTask() {
    if (!input.trim()) return;
    setTasks([...tasks, { id: Date.now(), text: input, completed: false }]);
    setInput("");
  }

  function toggleTask(id) {
    setTasks(tasks.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t)));
  }

  function deleteTask(id) {
    setTasks(tasks.filter((t) => t.id !== id));
  }

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={(e) => e.key === "Enter" && addTask()}
        placeholder="New task..."
      />
      <button onClick={addTask}>Add</button>
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>
            <span
              onClick={() => toggleTask(task.id)}
              style={{ textDecoration: task.completed ? "line-through" : "none", cursor: "pointer" }}
            >
              {task.text}
            </span>
            <button onClick={() => deleteTask(task.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default TaskManager;
```

Type it yourself. Use it in `App.jsx`.

**Expected output:**
Working Task Manager. Add, complete, delete all work.

### Task 2: Conditional rendering -- empty state

**What to do:**
When the list is empty, show a message "No tasks yet -- add your first one above" instead of an empty `<ul>`:

```jsx
{tasks.length === 0 ? (
  <p>No tasks yet -- add your first one above</p>
) : (
  <ul>
    {/* ... */}
  </ul>
)}
```

**Expected output:**
Empty state visible on page load.

### Task 3: Count of remaining tasks

**What to do:**
Below the list, show "N tasks remaining" where N is the count of incomplete tasks:

```jsx
<p>{tasks.filter((t) => !t.completed).length} tasks remaining</p>
```

Verify by adding tasks and marking some complete.

**Expected output:**
Remaining count updates correctly.

### Task 4: Immutability check

**What to do:**
In `day4-notes.md`, write 3-5 sentences explaining:
- Why do we use `[...tasks, newTask]` instead of `tasks.push(newTask)`?
- What happens if you mutate state directly?
- Why does React need a new array reference to re-render?

Your own words. Read the React docs if needed.

**Expected output:**
Notes committed.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 7 Day 4 Pre-Weekend Checklist

- [ ] React project runs with `npm run dev`
- [ ] First 5+ components were hand-typed, no AI
- [ ] Counter and Toggle components working
- [ ] TaskManager: add, toggle, delete all working
- [ ] Empty state shows when no tasks
- [ ] "N remaining" count updates correctly
- [ ] No console warnings about keys
- [ ] AI_AUDIT.md has manual-first log and build-first entries
- [ ] Repo pushed and clean
```

Tick every box honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Add a filter: "All", "Active", "Completed" -- three buttons that change what gets rendered.
- Persist tasks in localStorage (preview of `useEffect` next week).
- Add an edit button that lets you change a task's text inline.

## Submission Requirements

- **What to submit:** Repo with `TaskManager.jsx`, `day4-notes.md`, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| TaskManager core (add, toggle, delete) | 30 | All three operations work. Keys correct. No warnings. |
| Empty state | 10 | "No tasks yet" visible when list is empty. |
| Remaining count | 10 | Count updates correctly. |
| Immutability notes | 15 | Three questions answered in student's own words. |
| Pre-weekend checklist | 10 | All boxes honest. |
| AI Audit | 15 | Manual-first log plus one build-first entry. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Mutating the tasks array directly.** `tasks.push(newTask)` does not trigger a re-render. Always create a new array: `setTasks([...tasks, newTask])`.
- **Using array index as key.** Breaks on reordering. Use `task.id` from your data.
- **Forgetting to clear the input after adding.** After `setTasks`, also `setInput("")` so the field is ready for the next task.

## Resources

- Day 4 reading: [Rendering Lists and Managing UI Logic.md](./Rendering%20Lists%20and%20Managing%20UI%20Logic.md)
- Week 7 AI boundaries: [../ai.md](../ai.md)
- react.dev: Updating Arrays in State: https://react.dev/learn/updating-arrays-in-state
