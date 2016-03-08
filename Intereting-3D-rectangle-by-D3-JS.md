title: About
date: 2016-03-8 14:55:55
tags:
- JS
- 有趣的玩意
categories: 技术相关

---

一个有趣的JS 3D特效

<!-- more -->

# 下面是个有趣的D3特效
> 第一次遇到还有点吓人的感觉，看久了觉得真__鬼畜__=。=
> 出自[这里](http://bl.ocks.org/mbostock/1123639), 依靠[D3](https://github.com/mbostock/d3)制作而成

<div id="body1">
如果觉得不错可以去学习一下<a href="http://d3js.org/" target="_blank" rel="external">D3</a>
</div>
<script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>
<script>
var mouse = [480, 250],
    count = 0;

var svg = d3.select("#body1").append("svg")
    .attr("width", 960)
    .attr("height", 500);

var g = svg.selectAll("g")
    .data(d3.range(25))
  .enter().append("g")
    .attr("transform", "translate(" + mouse + ")");

g.append("rect")
    .attr("rx", 6)
    .attr("ry", 6)
    .attr("x", -12.5)
    .attr("y", -12.5)
    .attr("width", 25)
    .attr("height", 25)
    .attr("transform", function(d, i) { return "scale(" + (1 - d / 25) * 20 + ")"; })
    .style("fill", d3.scale.category20c());

g.datum(function(d) {
  return {center: [0, 0], angle: 0};
});

svg.on("mousemove", function() {
  mouse = d3.mouse(this);
});

d3.timer(function() {
  count++;
  g.attr("transform", function(d, i) {
    d.center[0] += (mouse[0] - d.center[0]) / (i + 5);
    d.center[1] += (mouse[1] - d.center[1]) / (i + 5);
    d.angle += Math.sin((count + i) / 10) * 7;
    return "translate(" + d.center + ")rotate(" + d.angle + ")";
  });
});

</script>

