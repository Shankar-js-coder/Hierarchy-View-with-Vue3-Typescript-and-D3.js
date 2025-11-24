<template>
  <div class="tree-container">
    <svg ref="svgRef" class="tree-svg"></svg>

    <!-- When a node is selected (by clicking it), we show the sidebar -->
    <div v-if="selectedNode" class="sidebar">
      <button class="close-btn" @click="deselectNode">×</button>
      <h3>{{ selectedNode.name }}</h3>
      <p>{{ selectedNode.description }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import * as d3 from 'd3';
import { ref, onMounted, watch } from 'vue';
import { sampleData, NodeData } from '../data/sampleData';

const svgRef = ref<SVGSVGElement | null>(null); // Gives us access to the <svg> tag in the DOM.
const selectedNode = ref<NodeData | null>(null); // Keeps track of which node is currently clicked

const width = 800;
const height = 400;

const drawTree = () => {
  if (!svgRef.value) return;

  const svg = d3.select(svgRef.value);
  svg.selectAll('*').remove();

  //We first find the root node (the one with no parent)
  const rootData = sampleData.find((d) => d.parent === '');
  if (!rootData) return;

  const buildHierarchy = (node: NodeData): any => {
    // Create a shallow copy of the node
    const treeNode: any = { ...node, children: [] };
    // Find all child nodes of the current node
    const children = sampleData.filter((d) => d.parent === node.name);
    // Recursively build each child subtree
    for (const child of children) {
      const childHierarchy = buildHierarchy(child);
      treeNode.children.push(childHierarchy);
    }
    // Return the node with its nested children
    return treeNode;
  };

  // Create D3 Tree Layout
  const root = d3.hierarchy(buildHierarchy(rootData)); //d3.hierarchy() converts our data into a tree-like object.

  const treeLayout = d3.tree<NodeData>().size([height - 30, width - 160]); //.tree() arranges it into (x, y) positions, so we can draw lines and circles.
  treeLayout(root);

  // Draw links
  // d3.linkHorizontal() - This adds curved lines between nodes using D3’s built-in linkHorizontal() function.
  svg
    .append('g')
    .selectAll('path')
    .data(root.links())
    .join('path')
    .attr('fill', 'none')
    .attr('stroke', '#999') //lines color code
    .attr('stroke-width', 2) //lines width
    .attr(
      'd',
      d3
        .linkHorizontal()
        .x((d: any) => d.y)
        .y((d: any) => d.x)
    );

  // Draw nodes
  // Each node is a circle. When clicked, it updates selectedNode. The fill color changes if the node is selected
  const nodes = svg
    .append('g')
    .selectAll('circle')
    .data(root.descendants())
    .join('circle')
    .attr('cx', (d) => d.y+20)
    .attr('cy', (d) => d.x)
    .attr('r', 20)
    .attr('fill', (d) =>
      selectedNode.value && selectedNode.value.name === d.data.name
        ? '#00897b'
        : '#7f8c8d'
    )
    .attr('cursor', 'pointer')
    .on('click', (_, d) => {
      selectedNode.value = d.data;
    });

  // Add labels
  //This prints node names beside each circle.
  svg
    .append('g')
    .selectAll('text')
    .data(root.descendants())
    .join('text')
    .attr('x', (d) => d.y + 50)
    .attr('y', (d) => d.x + 5)
    .text((d) => d.data.name)
    .attr('font-size', 18)
    .attr('fill', '#333');
};

// If the user clicks the 'X' button, we clear the selection. After that, v-if="selectedNode" hides the sidebar again.
const deselectNode = () => {
  selectedNode.value = null;
};

onMounted(drawTree);

//Watching for Node Changes
watch(selectedNode, () => drawTree()); //We use Vue’s watch() to redraw the tree whenever a node is selected or deselected — so the selected node color updates.
</script>

<style scoped>
.tree-container {
  display: flex;
  position: relative;
}
.tree-svg {
  width: 800px;
  height: 400px;
  background-color: #f8f8f8;
  border: 1px solid #ddd;
  padding: 50px;
}
.sidebar {
  position: absolute;
  right: 0;
  top: 0;
  width: 250px;
  background-color: #fff;
  border-left: 1px solid #ddd;
  padding: 16px;
  box-shadow: -2px 0 6px rgba(0, 0, 0, 0.1);
}
.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  float: right;
  cursor: pointer;
}
</style>
