# Vue 3 + TypeScript Hierarchy Tree

An interactive tree visualization built with **Vue 3**, **TypeScript**, and **D3.js**.  
Displays hierarchical data, supports node selection, and shows node details in a sidebar.

---

## Features
- Hierarchical visualization using D3 tree layout  
- Interactive node selection and deselection  
- Sidebar with node details
- Type‑safe architecture (Vue 3 + TypeScript + Vite)

---

## Installation

npm init vue@latest

1. Clone the repository
git clone https://github.com/<your-username>/hierarchy-tree.git
cd hierarchy-tree

2. Install dependencies
npm install
npm install d3

3. Start development server
npm run dev

## Build

Output files will be created in the `/dist` directory.

---

## Implementation Summary

1. Data is transformed from flat to nested via `buildHierarchy()`.
2. D3 `tree()` layout calculates node positions (`x`, `y`).
3. Nodes and links are drawn in an `<svg>` container.
4. Clicking a node updates `selectedNode`, opening the sidebar.
5. Clicking the close button clears selection.

---
