<template>
  <v-container>
    <div id="tree"/>
  </v-container>
</template>

<script>
/* eslint-disable */
export default {
  name: 'sdD-sdf',
  data: () => ({
    nodeId: null
  }),
  mounted() {
    const treeData = {
      name: "Osteoporosis",
      children: [
        {
          name: "Exercise",
          description: 'description'
        },
        {
          name: "Protein Intake",
        },
        {
          name: "Age",
        },
        {
          name: "Sex",
        },
        {
          name: "Some text",
        },
        {
          name: "Bone fractures",
        },
        {
          name: "height loss",
        },
        {
          name: "posture change",
        },
        {
          name: "DEXA scan result",
          description: 'description'
        },
      ],
    };

    // Set the dimensions and margins of the diagram
    const margin = {
          top: 50,
          right: 0,
          bottom: 30,
          left: 90,
        },
        width = 1260 - margin.left - margin.right,
        height = 700 - margin.top - margin.bottom;

    const svg = d3
        .select("#tree")
        .append("svg")
        .attr("width", width + margin.right + margin.left)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    let i = 0,
        duration = 750,
        root;

    const treemap = d3.tree().size([width, height]);

    root = d3.hierarchy(treeData, function (d) {
      return d.children;
    });
    root.x0 = height / 2;
    root.y0 = 0;

    update(root);

    function collapse(d) {
      if (d.children) {
        d._children = d.children;
        d._children.forEach(collapse);
        d.children = null;
      }
    }

    function update(source) {
      const treeData = treemap(root);
      const nodes = treeData.descendants(),
          links = treeData.descendants().slice(1);

      nodes.forEach(function (d) {
        d.y = d.depth * 180;
      });

      // ****************** Nodes section ***************************

      // Update the nodes...
      const node = svg.selectAll("g.node").data(nodes, (d) => {
        return d.id || (d.id = ++i);
      })

      // Enter any new modes at the parent's previous position.
      const nodeEnter = node
          .enter()
          .append("g")
          .attr("class", "node")
          .attr("transform", function (e, d) {
            return "translate(" + source.x0 + "," + source.y0 + ")";
          })
          .on("click", click.bind(this));

      // Add Circle for the nodes
      nodeEnter
          .append("circle")
          .attr("class", "node")
          .attr("r", 1e-6)
          .style("fill", function (d) {
            return d._children ? "lightsteelblue" : "#fff";
          });

      // Add labels for the nodes
      nodeEnter
          .append("text")
          .attr("dy", ".35em")
          .attr("x", function (d) {
            return d.children || d._children ? -13 : 13;
          })
          .attr("text-anchor", function (d) {
            return d.children || d._children ? "end" : "start";
          })
          .text(function (d) {
            return d.data.name;
          });

      // UPDATE
      const nodeUpdate = nodeEnter.merge(node);

      // Transition to the proper position for the node
      nodeUpdate
          .transition()
          .duration(duration)
          .attr("transform", function (d) {
            return "translate(" + d.x + "," + d.y + ")";
          });

      // Update the node attributes and style
      nodeUpdate
          .select("circle.node")
          .attr("r", 10)
          .style("fill", (d) => {
            return d._children ? "lightsteelblue" : "#fff";
          })
          .attr("cursor", "pointer");

      // Remove any exiting nodes
      const nodeExit = node
          .exit()
          .transition()
          .duration(duration)
          .attr("transform", (d) => {
            return "translate(" + source.x + "," + source.y + ")";
          })
          .remove();

      // On exit reduce the node circles size to 0
      nodeExit.select("circle").attr("r", 1e-6);

      // On exit reduce the opacity of text labels
      nodeExit.select("text").style("fill-opacity", 1e-6);

      // ****************** links section ***************************

      // Update the links...
      const link = svg.selectAll("path.link").data(links, (d) => {
        return d.id;
      });

      // Enter any new links at the parent's previous position.
      const linkEnter = link
          .enter()
          .insert("path", "g")
          .attr("class", "link")
          .attr("d", function (d) {
            var o = {
              x: source.x0,
              y: source.y0,
            };
            return diagonal(o, o);
          });

      link
          .enter()
          .insert("text", "g")
          .attr("font-family", "Arial, Helvetica, sans-serif")
          .attr("fill", "Orange")
          .style("font", "normal 12px Arial")
          .attr("transform", function (d) {
            return (
                "translate(" +
                (d.parent.x + d.x) / 2 +
                "," +
                (d.parent.y + d.y) / 2 +
                ")"
            );
          })
          .attr("dy", ".35em")
          .attr("text-anchor", "middle")
          .text(function (d) {
            return d.data.description;
          });

      // UPDATE
      const linkUpdate = linkEnter.merge(link);

      // Transition back to the parent element position
      linkUpdate
          .transition()
          .duration(duration)
          .attr("d", function (d) {
            return diagonal(d, d.parent);
          });

      // Remove any exiting links
      const linkExit = link
          .exit()
          .transition()
          .duration(duration)
          .attr("d", function (d) {
            var o = {
              x: source.x,
              y: source.y,
            };
            return diagonal(o, o);
          })
          .remove();

      // Store the old positions for transition.
      nodes.forEach(function (d) {
        d.x0 = d.x;
        d.y0 = d.y;
      });

      function diagonal(s, d) {
        let path = `M ${s.x} ${s.y}C ${(s.x + d.x) / 2} ${s.y}, ${(s.x + d.x) / 2} ${d.y},${d.x} ${d.y}`;
        return path;
      }

      function click(e, d) {
        update(d);
      }
    }
  }
}
</script>
<style>
#tree {
  margin-top: 50px;
}

.node circle {
  fill: #fff;
  stroke: steelblue;
  stroke-width: 3px;
}

.node text {
  font: 12px sans-serif;
}

.link {
  fill: none;
  stroke: #ccc;
  stroke-width: 2px;
}
</style>
