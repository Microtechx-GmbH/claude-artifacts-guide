# The Ultimate Guide to Building Claude Artifacts
### From Zero to Production-Grade Interactive Apps — Inside a Chat Window

> **Author**: Hira Jabeen
> **Last Updated**: March 2026
> **Audience**: Developers, Educators, Content Creators, Product Builders

---

## Table of Contents

1. [What Are Claude Artifacts?](#1-what-are-claude-artifacts)
2. [The Six Renderable Artifact Types](#2-the-six-renderable-artifact-types)
3. [When to Use (and NOT Use) Artifacts](#3-when-to-use-and-not-use-artifacts)
4. [HTML Artifacts — The Swiss Army Knife](#4-html-artifacts--the-swiss-army-knife)
5. [React (JSX) Artifacts — Stateful Power](#5-react-jsx-artifacts--stateful-power)
6. [SVG Artifacts — Precision Graphics](#6-svg-artifacts--precision-graphics)
7. [Mermaid Artifacts — Instant Diagrams](#7-mermaid-artifacts--instant-diagrams)
8. [Markdown Artifacts — Structured Documents](#8-markdown-artifacts--structured-documents)
9. [PDF Artifacts — Professional Documents](#9-pdf-artifacts--professional-documents)
10. [Available Libraries & Imports](#10-available-libraries--imports)
11. [Persistent Storage API](#11-persistent-storage-api)
12. [The Anthropic API Inside Artifacts (Claude-in-Claude)](#12-the-anthropic-api-inside-artifacts-claude-in-claude)
13. [Design Principles That Separate Good from Great](#13-design-principles-that-separate-good-from-great)
14. [Common Patterns & Recipes](#14-common-patterns--recipes)
15. [Mistakes Everyone Makes (and How to Avoid Them)](#15-mistakes-everyone-makes-and-how-to-avoid-them)
16. [Prompt Engineering for Artifacts](#16-prompt-engineering-for-artifacts)
17. [Real-World Use Cases](#17-real-world-use-cases)
18. [Quick Reference Cheat Sheet](#18-quick-reference-cheat-sheet)

---

## 1. What Are Claude Artifacts?

Claude Artifacts are **live, interactive files** that Claude creates and renders directly inside the chat interface. Unlike a typical code block in a chat response, artifacts are functional — they run, they render, they respond to user input.

Think of them as **mini-applications** built on demand, embedded right in your conversation.

**What makes them special:**

- They render in a dedicated panel alongside the chat
- They support interactivity (clicks, inputs, animations, state)
- They can persist data across sessions using a storage API
- They can even call the Anthropic API internally ("Claude-in-Claude")
- They are single-file by design — everything lives in one file

**The mental model:** An artifact is a self-contained, single-file web app that runs in a sandboxed iframe inside Claude's interface.

---

## 2. The Six Renderable Artifact Types

Claude can produce any file type, but **six specific types get special rendering** in the UI — meaning they show up as live, interactive content, not just downloadable files.

| Type | Extension | Best For |
|------|-----------|----------|
| **HTML** | `.html` | Full pages, games, dashboards, tools |
| **React (JSX)** | `.jsx` | Stateful components, complex UIs, data apps |
| **SVG** | `.svg` | Icons, illustrations, diagrams, logos |
| **Mermaid** | `.mermaid` | Flowcharts, sequence diagrams, Gantt charts |
| **Markdown** | `.md` | Reports, guides, documentation |
| **PDF** | `.pdf` | Formal documents, certificates |

**Key insight:** HTML and React are the workhorses. If you are building anything interactive, you will use one of these two 90% of the time.

---

## 3. When to Use (and NOT Use) Artifacts

### ✅ USE Artifacts For

- Custom applications and tools (calculators, converters, simulators)
- Data visualizations and interactive charts
- Games and interactive experiences
- Long-form creative writing (stories, essays, scripts)
- Structured reference content (meal plans, study guides, workout routines)
- Code longer than 20 lines
- Content meant to be edited, expanded, or reused
- Standalone documents longer than 20 lines or 1500 characters

### ❌ DO NOT Use Artifacts For

- Short code snippets (under 20 lines)
- Quick answers, definitions, or explanations
- Short poems, haikus, or limericks
- Simple lists, tables, or bullet points
- Brief emails or one-paragraph responses
- Content where the user explicitly asked for something short

**The rule of thumb:** If someone would want to download, share, keep, or interact with it — make it an artifact. If it is a quick answer they will read and move on — keep it inline.

---

## 4. HTML Artifacts — The Swiss Army Knife

HTML artifacts are the most flexible type. Everything — HTML, CSS, and JavaScript — goes into a single `.html` file.

### Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Artifact</title>
  <style>
    /* All CSS goes here */
  </style>
</head>
<body>
  <!-- All HTML goes here -->
  <script>
    // All JavaScript goes here
  </script>
</body>
</html>
```

### Key Rules

1. **Single file only** — no separate CSS or JS files
2. **External scripts** can be imported from `https://cdnjs.cloudflare.com`
3. **No localStorage or sessionStorage** — these APIs are blocked in the sandbox
4. **Use in-memory variables** for all state management
5. **Animations** — prefer CSS-only solutions for performance

### Example: Interactive Color Palette Generator

```html
<style>
  :root {
    --bg: #0a0a0a;
    --text: #f5f5f5;
  }
  body {
    font-family: 'Georgia', serif;
    background: var(--bg);
    color: var(--text);
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
    margin: 0;
    padding: 2rem;
  }
  .palette {
    display: flex;
    gap: 0.5rem;
    margin-top: 2rem;
  }
  .swatch {
    width: 80px;
    height: 120px;
    border-radius: 12px;
    cursor: pointer;
    transition: transform 0.2s;
  }
  .swatch:hover { transform: scale(1.1); }
</style>

<h1>Palette Generator</h1>
<button onclick="generate()">Generate New Palette</button>
<div class="palette" id="palette"></div>

<script>
  function randomHSL() {
    return `hsl(${Math.random()*360}, ${50+Math.random()*40}%, ${40+Math.random()*30}%)`;
  }
  function generate() {
    const el = document.getElementById('palette');
    el.innerHTML = '';
    for (let i = 0; i < 5; i++) {
      const div = document.createElement('div');
      div.className = 'swatch';
      div.style.background = randomHSL();
      div.onclick = () => navigator.clipboard?.writeText(div.style.background);
      el.appendChild(div);
    }
  }
  generate();
</script>
```

### When to Choose HTML over React

- Simple interactivity (clicks, toggles, basic animations)
- No complex state management needed
- You want maximum control over the DOM
- Performance-critical animations
- Quick prototypes and one-off tools

---

## 5. React (JSX) Artifacts — Stateful Power

React artifacts use functional components with hooks. They are ideal for complex, stateful UIs.

### Structure

```jsx
import { useState, useEffect, useRef } from "react";

// Optional library imports
import { LineChart, XAxis, YAxis, Line, Tooltip } from "recharts";
import { Search, Plus, Trash2 } from "lucide-react";

export default function MyApp() {
  const [state, setState] = useState(initialValue);

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      {/* Your UI here */}
    </div>
  );
}
```

### Key Rules

1. **Default export required** — use `export default function`
2. **No required props** — component must work with zero props (or provide defaults)
3. **Tailwind CSS only** — use core utility classes, no custom Tailwind config
4. **No localStorage** — use `useState` and `useReducer` for all state
5. **No `<form>` tags** — use `onClick` and `onChange` handlers directly
6. **Hooks must be imported** — `import { useState } from "react"`

### Available Libraries (Pre-installed)

```jsx
// Icons
import { Camera, Heart, Star } from "lucide-react";

// Charts
import { LineChart, BarChart, PieChart, XAxis, YAxis, Tooltip } from "recharts";

// Math
import * as math from "mathjs";

// Utilities
import _ from "lodash";

// Data Visualization
import * as d3 from "d3";
import * as Plotly from "plotly";

// 3D Graphics
import * as THREE from "three";

// UI Components
import { Alert, AlertDescription } from "@/components/ui/alert";

// Audio
import * as Tone from "tone";

// Data Processing
import * as Papa from "papaparse";   // CSV
import * as XLSX from "sheetjs";      // Excel

// Machine Learning
import * as tf from "tensorflow";
```

### Example: Task Manager with Filtering

```jsx
import { useState } from "react";
import { Plus, Trash2, Check } from "lucide-react";

export default function TaskManager() {
  const [tasks, setTasks] = useState([]);
  const [input, setInput] = useState("");
  const [filter, setFilter] = useState("all");

  const addTask = () => {
    if (!input.trim()) return;
    setTasks(prev => [...prev, {
      id: Date.now(),
      text: input.trim(),
      done: false
    }]);
    setInput("");
  };

  const toggle = (id) => {
    setTasks(prev => prev.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    ));
  };

  const remove = (id) => {
    setTasks(prev => prev.filter(t => t.id !== id));
  };

  const filtered = tasks.filter(t =>
    filter === "all" ? true :
    filter === "done" ? t.done :
    !t.done
  );

  return (
    <div className="max-w-md mx-auto p-6 min-h-screen bg-stone-50">
      <h1 className="text-2xl font-bold mb-4 text-stone-800">Tasks</h1>

      <div className="flex gap-2 mb-4">
        <input
          className="flex-1 px-3 py-2 border rounded-lg"
          value={input}
          onChange={e => setInput(e.target.value)}
          onKeyDown={e => e.key === "Enter" && addTask()}
          placeholder="Add a task..."
        />
        <button
          onClick={addTask}
          className="p-2 bg-stone-800 text-white rounded-lg"
        >
          <Plus size={20} />
        </button>
      </div>

      <div className="flex gap-2 mb-4">
        {["all", "active", "done"].map(f => (
          <button
            key={f}
            onClick={() => setFilter(f)}
            className={`px-3 py-1 rounded-full text-sm capitalize ${
              filter === f
                ? "bg-stone-800 text-white"
                : "bg-stone-200 text-stone-600"
            }`}
          >
            {f}
          </button>
        ))}
      </div>

      {filtered.map(task => (
        <div key={task.id}
          className="flex items-center gap-3 p-3 mb-2 bg-white rounded-lg shadow-sm"
        >
          <button onClick={() => toggle(task.id)}>
            <Check size={18} className={task.done ? "text-green-500" : "text-stone-300"} />
          </button>
          <span className={`flex-1 ${task.done ? "line-through text-stone-400" : ""}`}>
            {task.text}
          </span>
          <button onClick={() => remove(task.id)}>
            <Trash2 size={16} className="text-red-400" />
          </button>
        </div>
      ))}
    </div>
  );
}
```

### When to Choose React over HTML

- Complex state with multiple interdependent values
- List rendering with dynamic updates
- Component composition (reusable UI pieces)
- You need Recharts, Lucide icons, or shadcn/ui
- Forms with multiple inputs and validation

---

## 6. SVG Artifacts — Precision Graphics

SVG artifacts create scalable vector graphics. Perfect for diagrams, icons, illustrations, and data visualizations.

### Structure

```svg
<svg viewBox="0 0 800 600" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- Gradients, patterns, filters -->
  </defs>

  <!-- Shapes, text, paths -->
  <rect x="10" y="10" width="200" height="100" rx="8" fill="#3b82f6" />
  <text x="110" y="65" text-anchor="middle" fill="white">Hello</text>
</svg>
```

### Key Elements

| Element | Purpose | Example |
|---------|---------|---------|
| `<rect>` | Rectangles, cards | `<rect x="0" y="0" width="100" height="50" rx="8"/>` |
| `<circle>` | Circles, dots | `<circle cx="50" cy="50" r="30"/>` |
| `<line>` | Connectors | `<line x1="0" y1="0" x2="100" y2="100"/>` |
| `<path>` | Complex shapes | `<path d="M10 10 L90 10 L50 90 Z"/>` |
| `<text>` | Labels | `<text x="50" y="50" text-anchor="middle">Hi</text>` |
| `<g>` | Groups | `<g transform="translate(100,50)">...</g>` |

### Tips

- Use `viewBox` for responsive scaling
- Group related elements with `<g>` and `transform`
- Define reusable elements in `<defs>` and reference with `<use>`
- Add animations with `<animate>` or CSS within `<style>` tags

---

## 7. Mermaid Artifacts — Instant Diagrams

Mermaid lets you create diagrams from text descriptions. Fast, readable, and great for documentation.

### Supported Diagram Types

```mermaid
%% Flowchart
graph TD
    A[Start] --> B{Decision}
    B -- Yes --> C[Action 1]
    B -- No --> D[Action 2]
    C --> E[End]
    D --> E

%% Sequence Diagram
sequenceDiagram
    Client->>Server: POST /api/login
    Server->>Database: Validate credentials
    Database-->>Server: User found
    Server-->>Client: 200 OK + JWT

%% Class Diagram
classDiagram
    Animal <|-- Dog
    Animal <|-- Cat
    Animal : +String name
    Animal : +makeSound()

%% Gantt Chart
gantt
    title Project Timeline
    dateFormat YYYY-MM-DD
    section Phase 1
    Research      :a1, 2026-01-01, 30d
    Design        :a2, after a1, 20d
    section Phase 2
    Development   :b1, after a2, 60d
```

### When to Use Mermaid

- Quick flowcharts and process diagrams
- API sequence diagrams
- Project timelines (Gantt)
- Class/entity relationship diagrams
- Git branching visualizations
- Documentation diagrams

---

## 8. Markdown Artifacts — Structured Documents

For standalone written content, reports, guides, and creative writing.

### Best Practices

- Use for content the user will reference, save, or share
- Appropriate for reports, guides, tutorials, and long-form writing
- Not for conversational responses or quick answers
- Supports standard Markdown: headers, bold, italic, links, code blocks, tables, blockquotes

---

## 9. PDF Artifacts — Professional Documents

PDF artifacts render in the UI and are downloadable. Use for formal documents, certificates, and print-ready materials.

---

## 10. Available Libraries & Imports

### For React Artifacts

| Library | Import | Use Case |
|---------|--------|----------|
| **Lucide React** (v0.383.0) | `import { Icon } from "lucide-react"` | 1000+ icons |
| **Recharts** | `import { LineChart, ... } from "recharts"` | Charts & graphs |
| **MathJS** | `import * as math from "mathjs"` | Math operations |
| **Lodash** | `import _ from "lodash"` | Utility functions |
| **D3** | `import * as d3 from "d3"` | Advanced data viz |
| **Plotly** | `import * as Plotly from "plotly"` | Scientific plots |
| **Three.js** (r128) | `import * as THREE from "three"` | 3D graphics |
| **Chart.js** | `import * as Chart from "chart.js"` | Canvas charts |
| **Tone.js** | `import * as Tone from "tone"` | Audio synthesis |
| **PapaParse** | `import * as Papa from "papaparse"` | CSV parsing |
| **SheetJS** | `import * as XLSX from "sheetjs"` | Excel files |
| **Mammoth** | `import * as mammoth from "mammoth"` | Word docs |
| **TensorFlow.js** | `import * as tf from "tensorflow"` | Machine learning |
| **shadcn/ui** | `import { Alert } from "@/components/ui/alert"` | UI components |

### For HTML Artifacts

You can import any script from the Cloudflare CDN:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js"></script>
```

---

## 11. Persistent Storage API

Artifacts can **save and load data across sessions** using a key-value storage API. This enables journals, trackers, leaderboards, and any app that needs memory.

### API Methods

```javascript
// Store data (personal, only you can see it)
await window.storage.set("notes:123", JSON.stringify(data));

// Store shared data (visible to all users of the artifact)
await window.storage.set("leaderboard:alice", JSON.stringify(score), true);

// Retrieve data
const result = await window.storage.get("notes:123");
const parsed = result ? JSON.parse(result.value) : null;

// Delete data
await window.storage.delete("notes:123");

// List keys by prefix
const keys = await window.storage.list("notes:");
```

### Key Rules

- **Keys**: Under 200 characters, no whitespace, slashes, or quotes
- **Values**: Under 5MB per key, text/JSON only
- **Scope**: `shared: false` (default, personal) or `shared: true` (visible to all)
- **Concurrency**: Last-write-wins
- **Rate limiting**: Batch related data into single keys to minimize calls

### Design Pattern: Hierarchical Keys

Use `table:record_id` format:

```
todos:todo_1
todos:todo_2
users:user_abc
settings:theme
```

### Critical: Batch Your Data

```javascript
// ❌ BAD — multiple sequential calls
await window.storage.set("cards", cardsData);
await window.storage.set("benefits", benefitsData);
await window.storage.set("completion", completionData);

// ✅ GOOD — single call with combined data
await window.storage.set("app-state", JSON.stringify({
  cards: cardsData,
  benefits: benefitsData,
  completion: completionData
}));
```

### Error Handling

```javascript
try {
  const result = await window.storage.get("my-key");
  // Use result.value
} catch (error) {
  // Key doesn't exist or storage unavailable
  console.log("Not found:", error);
}
```

---

## 12. The Anthropic API Inside Artifacts (Claude-in-Claude)

This is the most powerful feature: artifacts can **call the Anthropic API directly**, enabling AI-powered applications built entirely inside a chat message.

### Basic API Call

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [
      { role: "user", content: "Explain quantum computing in one paragraph." }
    ],
  })
});

const data = await response.json();
const text = data.content
  .filter(item => item.type === "text")
  .map(item => item.text)
  .join("\n");
```

### With Web Search

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{ role: "user", content: "Latest AI news this week" }],
    tools: [{ type: "web_search_20250305", name: "web_search" }]
  })
});
```

### With MCP Servers (External Services)

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{ role: "user", content: "Create a task in Asana" }],
    mcp_servers: [
      { type: "url", url: "https://mcp.asana.com/sse", name: "asana-mcp" }
    ]
  })
});
```

### Processing MCP Responses

```javascript
// Find tool results by type, not position
const toolResults = data.content
  .filter(item => item.type === "mcp_tool_result")
  .map(item => item.content?.[0]?.text || "");

const textResponses = data.content
  .filter(item => item.type === "text")
  .map(item => item.text);
```

### With File Inputs (PDF/Images)

```javascript
// Convert file to base64
const base64Data = await new Promise((res, rej) => {
  const r = new FileReader();
  r.onload = () => res(r.result.split(",")[1]);
  r.onerror = () => rej(new Error("Read failed"));
  r.readAsDataURL(file);
});

// Send with message
messages: [{
  role: "user",
  content: [
    {
      type: "document",
      source: { type: "base64", media_type: "application/pdf", data: base64Data }
    },
    { type: "text", text: "Summarize this document." }
  ]
}]
```

### Key Constraints

- Always use `claude-sonnet-4-20250514` as the model
- Always set `max_tokens: 1000`
- No API key needed — it is handled automatically
- Include full conversation history for multi-turn interactions
- Never use `<form>` tags in React — use `onClick` handlers

---

## 13. Design Principles That Separate Good from Great

### Typography

- Avoid generic fonts (Arial, Inter, Roboto, system fonts)
- Pair a distinctive display font with a refined body font
- Import from Google Fonts via CDN link in HTML artifacts

### Color

- Commit to a cohesive palette — use CSS variables
- One dominant color + sharp accents outperforms evenly-distributed colors
- Avoid the cliché "purple gradient on white" look

### Layout

- Use asymmetry, overlap, diagonal flow for visual interest
- Generous negative space OR controlled density — choose one
- Grid-breaking elements create memorability

### Animation

- Focus on high-impact moments: staggered page-load reveals
- Scroll-triggered and hover states that surprise
- CSS-only for HTML artifacts, Motion library for React

### Backgrounds & Texture

- Gradient meshes, noise textures, geometric patterns
- Layered transparencies, dramatic shadows
- Never default to plain solid white or grey backgrounds

---

## 14. Common Patterns & Recipes

### Pattern 1: Tab Navigation

```jsx
const [activeTab, setActiveTab] = useState("overview");

const tabs = ["overview", "details", "settings"];

<div className="flex gap-1 bg-gray-100 p-1 rounded-lg">
  {tabs.map(tab => (
    <button
      key={tab}
      onClick={() => setActiveTab(tab)}
      className={`px-4 py-2 rounded-md capitalize ${
        activeTab === tab ? "bg-white shadow-sm font-medium" : "text-gray-500"
      }`}
    >
      {tab}
    </button>
  ))}
</div>
```

### Pattern 2: Search & Filter

```jsx
const [query, setQuery] = useState("");
const filtered = items.filter(item =>
  item.name.toLowerCase().includes(query.toLowerCase())
);
```

### Pattern 3: Modal / Dialog

```jsx
const [showModal, setShowModal] = useState(false);

{showModal && (
  <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
    <div className="bg-white rounded-xl p-6 max-w-md w-full mx-4">
      <h2 className="text-lg font-bold mb-4">Title</h2>
      <p>Content here</p>
      <button onClick={() => setShowModal(false)}
        className="mt-4 px-4 py-2 bg-gray-800 text-white rounded-lg"
      >
        Close
      </button>
    </div>
  </div>
)}
```

### Pattern 4: Data Table with Sort

```jsx
const [sortKey, setSortKey] = useState("name");
const [sortDir, setSortDir] = useState(1);

const sorted = [...data].sort((a, b) =>
  a[sortKey] > b[sortKey] ? sortDir : -sortDir
);

const toggleSort = (key) => {
  if (sortKey === key) setSortDir(d => -d);
  else { setSortKey(key); setSortDir(1); }
};
```

### Pattern 5: Responsive Grid

```jsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => (
    <div key={item.id} className="bg-white rounded-xl p-4 shadow-sm">
      {item.name}
    </div>
  ))}
</div>
```

### Pattern 6: Loading States

```jsx
const [loading, setLoading] = useState(false);

{loading ? (
  <div className="flex items-center justify-center p-8">
    <div className="w-6 h-6 border-2 border-gray-300 border-t-gray-800 rounded-full animate-spin" />
  </div>
) : (
  <div>{/* Content */}</div>
)}
```

---

## 15. Mistakes Everyone Makes (and How to Avoid Them)

### ❌ Mistake 1: Using localStorage

```javascript
// WILL FAIL — localStorage is blocked
localStorage.setItem("key", "value");
```

**Fix:** Use `useState` in React, regular variables in HTML, or the persistent storage API (`window.storage`).

### ❌ Mistake 2: Using `<form>` Tags in React

```jsx
// WILL BREAK
<form onSubmit={handleSubmit}>
  <input />
  <button type="submit">Go</button>
</form>
```

**Fix:** Use `onClick` and `onChange` directly:

```jsx
<input onChange={e => setValue(e.target.value)} />
<button onClick={handleSubmit}>Go</button>
```

### ❌ Mistake 3: Separate CSS/JS Files

Claude creates single-file artifacts. You cannot reference external files you create.

**Fix:** Inline everything into one file.

### ❌ Mistake 4: Using THREE.CapsuleGeometry

`CapsuleGeometry` was introduced in Three.js r142, but artifacts use r128.

**Fix:** Use `CylinderGeometry`, `SphereGeometry`, or custom geometries.

### ❌ Mistake 5: Importing OrbitControls from Three.js

```javascript
// WILL FAIL — not on CDN
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
```

**Fix:** Implement basic camera controls manually or use the available r128 features.

### ❌ Mistake 6: Not Handling Storage Errors

```javascript
// ❌ Will crash if key doesn't exist
const result = await window.storage.get("nonexistent");

// ✅ Always wrap in try-catch
try {
  const result = await window.storage.get("nonexistent");
} catch (e) {
  // Handle missing key
}
```

---

## 16. Prompt Engineering for Artifacts

### Be Specific About What You Want

```
❌ "Make me a dashboard"
✅ "Create a React dashboard with 4 metric cards (revenue, users,
   orders, conversion rate), a line chart showing monthly revenue
   for the past 12 months, and a table of recent orders with
   sortable columns. Use a dark theme with blue accents."
```

### Specify the Technology

```
"Build this as an HTML artifact" (vs React)
"Use Recharts for the chart"
"Use Tailwind classes for styling"
```

### Request Iteration

```
"Add a dark mode toggle to the existing artifact"
"Replace the bar chart with a stacked area chart"
"Add persistent storage so the data saves between sessions"
```

### Ask for the Anthropic API

```
"Build a React artifact that includes a text input where users type
a topic, and it calls the Anthropic API to generate a summary,
then displays it below the input."
```

---

## 17. Real-World Use Cases

| Use Case | Best Type | Key Features |
|----------|-----------|--------------|
| **Portfolio website** | HTML | Animations, responsive layout, sections |
| **Expense tracker** | React + Storage | CRUD operations, charts, persistence |
| **Flashcard app** | React + Storage | Spaced repetition, progress tracking |
| **API documentation** | Markdown | Headers, code blocks, tables |
| **System architecture** | Mermaid | Flowcharts, sequence diagrams |
| **AI writing assistant** | React + API | Claude-in-Claude, text generation |
| **Quiz generator** | React + API | AI generates questions, scores answers |
| **Project timeline** | Mermaid | Gantt charts with dependencies |
| **Color palette tool** | HTML | HSL manipulation, clipboard copy |
| **Data visualizer** | React + D3 | CSV upload, dynamic charts |
| **Music synth** | React + Tone.js | Keyboard input, oscillators |
| **3D scene viewer** | HTML + Three.js | Camera controls, lighting |
| **Meeting summarizer** | React + API + MCP | Record → transcribe → summarize |
| **Habit tracker** | React + Storage | Daily logging, streak visualization |

---

## 18. Quick Reference Cheat Sheet

### File Extensions That Render

```
.html  → Full HTML page
.jsx   → React component
.svg   → SVG graphic
.mermaid → Mermaid diagram
.md    → Markdown document
.pdf   → PDF document
```

### Critical Constraints

```
✗ No localStorage / sessionStorage
✗ No <form> tags in React
✗ No separate CSS/JS files
✗ No THREE.CapsuleGeometry (use r128 features)
✗ No OrbitControls import
✗ No required props in React components
✗ Storage keys: no whitespace, slashes, or quotes
```

### Storage Quick Reference

```javascript
await window.storage.set(key, value, shared?)   // Save
await window.storage.get(key, shared?)           // Load
await window.storage.delete(key, shared?)        // Delete
await window.storage.list(prefix?, shared?)      // List keys
```

### API Quick Reference

```javascript
fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{ role: "user", content: "..." }],
    // Optional:
    tools: [{ type: "web_search_20250305", name: "web_search" }],
    mcp_servers: [{ type: "url", url: "...", name: "..." }]
  })
})
```

### React Import Cheat Sheet

```jsx
import { useState, useEffect, useRef, useMemo, useCallback } from "react";
import { LineChart, BarChart, PieChart, XAxis, YAxis, Tooltip, ResponsiveContainer, Line, Bar, Pie, Cell } from "recharts";
import { Search, Plus, Trash2, Edit, Check, X, ChevronDown, Menu, Settings, User, Home } from "lucide-react";
import _ from "lodash";
import * as d3 from "d3";
```

---

## About This Guide

This guide was built from hands-on experience creating hundreds of artifacts across every type and complexity level. It is designed to be a living reference — bookmark it, share it, and come back to it whenever you are building something new.

**Connect with me:**
- LinkedIn: [Hira Jabeen](https://www.linkedin.com/in/hira-jabeen-cloud-engineer/)
- GitHub: [HiraJabeen01](https://github.com/HiraJabeen01)

---

*Built with Claude by Anthropic. Last updated March 2026.*
