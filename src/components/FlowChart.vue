<template>
  <div class="app-container">
    <div class="controls-container">
      <textarea
        v-model="inputText"
        class="input-textarea"
        placeholder="Enter your flowchart steps (e.g., Login → Verify SMS → Home)"
      />
      <div class="button-container">
        <button class="btn" @click="generateFlowchart">Generate Flowchart</button>
        <button class="btn" @click="exportAsImage">Export as PNG</button>
      </div>
    </div>
    <div class="graph-container" ref="graphContainer"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { Graph } from '@antv/x6';
import html2canvas from 'html2canvas';

const inputText = ref('');
const graph = ref(null);
const graphContainer = ref(null);

onMounted(() => {
  graph.value = new Graph({
    container: graphContainer.value,
    grid: true,
    panning: true,
    connecting: {
      connector: 'smooth',
      allowBlank: false,
      snap: true,
    },
    width: graphContainer.value.offsetWidth, // Auto adjust width
    height: graphContainer.value.offsetHeight, // Auto adjust height
  });

  graph.value.on('node:dblclick', ({ node }) => {
    const currentLabel = node.getLabel();
    const newLabel = prompt('Edit node label:', currentLabel);
    if (newLabel !== null) {
      node.setLabel(newLabel);
    }
  });
});

const generateFlowchart = () => {
  if (!graph.value) return;
  graph.value.resetCells([]);

  const steps = inputText.value.split('→').map((step) => step.trim());
  const nodes = [];
  const edges = [];

  steps.forEach((step, index) => {
    const node = graph.value.addNode({
      id: `node-${index}`,
      x: index * 200, // Increase horizontal spacing for clarity
      y: 100,
      width: 120,
      height: 40,
      label: step,
    });
    nodes.push(node);

    if (index > 0) {
      edges.push(
        graph.value.addEdge({
          source: `node-${index - 1}`,
          target: `node-${index}`,
          attrs: {
            line: {
              stroke: '#5F95FF',
              strokeWidth: 2,
              targetMarker: {
                name: 'classic',
                size: 8,
              },
            },
          },
        })
      );
    }
  });
};

const exportAsImage = () => {
  if (!graphContainer.value) return;
  html2canvas(graphContainer.value).then((canvas) => {
    const link = document.createElement('a');
    link.download = 'flowchart.png';
    link.href = canvas.toDataURL();
    link.click();
  });
};
</script>

<style>
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.controls-container {
  padding: 16px;
  background-color: #f3f4f6;
}

.input-textarea {
  width: 100%;
  height: 80px;
  padding: 8px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
}

.button-container {
  margin-top: 8px;
  display: flex;
  gap: 8px;
}

.btn {
  padding: 8px 16px;
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn:hover {
  background-color: #4338ca;
}

.graph-container {
  flex: 1;
  background-color: white;
  position: relative;
  width: 100%;
  height: 100%; /* Adjust to take the full container height */
  overflow: hidden; /* Ensure content is contained within the container */
}
</style>
