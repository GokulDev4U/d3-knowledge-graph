<template>
  <div ref="svgContainer" style="width: 100%; height: 600px;"></div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "D3Graph",
  mounted() {
    const nodes = [
      { id: 1, label: "Node 1" },
      { id: 2, label: "Node 2" },
      { id: 3, label: "Node 3" },
      { id: 4, label: "Node 4" },
    ];

    const links = [
      { source: 1, target: 2 },
      { source: 1, target: 3 },
      { source: 3, target: 4 },
    ];

    this.createGraph(nodes, links);
  },
  methods: {
    createGraph(nodes, links) {
      const width = 800;
      const height = 600;

      // Create SVG container
      const svg = d3
        .select(this.$refs.svgContainer)
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      // Simulation setup
      const simulation = d3
        .forceSimulation(nodes)
        .force(
          "link",
          d3.forceLink(links).id((d) => d.id).distance(100)
        )
        .force("charge", d3.forceManyBody().strength(-200))
        .force("center", d3.forceCenter(width / 2, height / 2));

      // Draw links (edges)
      const link = svg
        .selectAll(".link")
        .data(links)
        .enter()
        .append("line")
        .attr("class", "link")
        .attr("stroke", "lightgray")
        .attr("stroke-width", 2);

      // Draw nodes
      const node = svg
        .selectAll(".node")
        .data(nodes)
        .enter()
        .append("circle")
        .attr("class", "node")
        .attr("r", 10)
        .attr("fill", "darkgray")
        .call(
          d3
            .drag()
            .on("start", dragStarted)
            .on("drag", dragged)
            .on("end", dragEnded)
        );

      // Add labels
      svg
        .selectAll(".label")
        .data(nodes)
        .enter()
        .append("text")
        .attr("class", "label")
        .attr("text-anchor", "middle")
        .attr("dy", -15)
        .text((d) => d.label);

      // Update simulation on tick
      simulation.on("tick", () => {
        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);

        node.attr("cx", (d) => d.x).attr("cy", (d) => d.y);

        svg
          .selectAll(".label")
          .attr("x", (d) => d.x)
          .attr("y", (d) => d.y);
      });

      // Dragging behavior
      function dragStarted(event, d) {
        if (!event.active) simulation.alphaTarget(0.3).restart();

        d.fx = d.x;
        d.fy = d.y;

        // Highlight connected nodes and edges
        highlightConnected(d.id);
      }

      function dragged(event, d) {
        d.fx = event.x;
        d.fy = event.y;
      }

      function dragEnded(event, d) {
        if (!event.active) simulation.alphaTarget(0);

        d.fx = null;
        d.fy = null;

        // Reset colors
        resetColors();
      }

      // Highlight connected nodes and edges
      function highlightConnected(nodeId) {
        // Find connected links
        link.attr("stroke", (d) =>
          d.source.id === nodeId || d.target.id === nodeId ? "yellow" : "lightgray"
        );

        // Highlight connected nodes
        node.attr("fill", (d) =>
          d.id === nodeId ||
          links.some(
            (link) =>
              (link.source.id === nodeId && link.target.id === d.id) ||
              (link.target.id === nodeId && link.source.id === d.id)
          )
            ? "yellow"
            : "lightgray"
        );
      }

      // Reset colors
      function resetColors() {
        link.attr("stroke", "lightgray");
        node.attr("fill", "darkgray");
      }
    },
  },
};
</script>

<style>
/* Optional: Add styles for better visuals */
.link {
  stroke-opacity: 0.6;
}

.node {
  cursor: pointer;
}
</style>
