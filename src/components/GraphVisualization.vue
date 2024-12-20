<template>
  <div ref="svgContainer" style="width: 100%; height: 600px;"></div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "D3Graph",
  mounted() {
    // const nodes = [
    //   { id: 1, label: "Node 1" },
    //   { id: 2, label: "Node 2" },
    //   { id: 3, label: "Node 3" },
    //   { id: 4, label: "Node 4" },
    // ];

    // const links = [
    //   { source: 1, target: 2 },
    //   { source: 1, target: 3 },
    //   { source: 3, target: 4 },
    // ];

    // this.createGraph(nodes, links);

     // Convert nodes object to array
     const nodes = Object.entries({
      0: { name: "0. Foundations" },
      1: { name: "1. Basic ROS2" },
      2: { name: "2. ROS Basics" },
      3: { name: "3. Intermediate ROS2" },
      4: { name: "4. Robotics Theory" },
      5: { name: "5. Navigation ROS2" },
      6: { name: "C++ Basics for robotics" },
      7: { name: "7. Manipulation" },
      8: { name: "8. Robot Creation" },
      9: { name: "9. Artificial Intelligence" },
      10: { name: "10. ROS Debugging" },
      11: { name: "11. Course of product" },
      12: { name: "12. Web Devel for Robots" },
      13: { name: "13. Simulation" },
      14: { name: "14. Enterprise" },
      15: { name: "Advanced Modern C++ for robotics" },
      16: { name: "Linux for robotics" },
      17: { name: "Behavior Trees for ROS2" },
      18: { name: "ROS2 Control" },
      19: { name: "ROS2 Basics in 5 days (C++)" },
      20: { name: "ROS2 Basics in 5 days (Python)" },
      21: { name: "ROS2 Security" },
      22: { name: "ROS2 Basics in 3 days (Rust)" },
      23: { name: "RTAB" },
      24: { name: "Nav2 Basics" },
      25: { name: "Nav Advanced" },
      26: { name: "TF in ROS2" },
      27: { name: "URDF robot modelling ROS2" },
      28: { name: "ROS1-ROS2 Migration" },
      29: { name: "Python 3 for robotics" },
      30: { name: "Gazebo Sim with ROS2" },
      31: { name: "create a link" },
      32: { name: "Intermediate ROS2 (Python)" },
      33: { name: "Intermediate ROS2 (C++)" },
    }).map(([id, data]) => ({ id: +id, label: data.name }));

    const edges = [
      { source: 0, target: 1 },
      { source: 1, target: 3 },
      { source: 1, target: 5 },
      { source: 1, target: 27 },
      { source: 1, target: 28 },
      { source: 3, target: 19 },
      { source: 3, target: 20 },
      { source: 3, target: 32 },
      { source: 3, target: 33 },
      { source: 3, target: 17 },
      { source: 3, target: 18 },
      { source: 5, target: 22 },
      { source: 5, target: 23 },
      { source: 5, target: 24 },
      { source: 5, target: 25 },
      { source: 2, target: 6 },
      { source: 2, target: 13 },
      { source: 0, target: 16 },
      { source: 1, target: 26 },
      { source: 1, target: 29 },
      { source: 2, target: 30 },
      { source: 0, target: 31 },
      { source: 4, target: 3 },
      { source: 8, target: 30 },
      { source: 12, target: 10 },
    ];

    this.createGraph(nodes, edges);
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
