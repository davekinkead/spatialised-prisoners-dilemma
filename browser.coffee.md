

# Browser Code

This is the browser file where we pull together different parts of the simulation and wrap it up in the browser.  We'll start out by importing our libraries and defining any global variables we might need.


    d3 = require './assets/d3.min.js'
    simulation = require './simulation.coffee.md'
    running = false
    height = window.innerHeight  - 40 || 600
    width = window.innerWidth - 40 || 600


Next, we'll grab create our svg canvas, apply some event listeners and add it to the DOM.


    canvas = d3.select("#space")
                .append("svg:svg")
                .attr("height", height)
                .attr("width", width)
                .on "click", () ->
                  running = if running is true then false else true
                

Now we need to createsvg circles to represent our agents and bind them to the actual agents from the simulation.  The `d` in the functions here is the D3js accessor to the agent data.

  
    populate = () ->
      canvas.selectAll "circle"
        .data simulation.agents(height, width)
        .enter().append "circle"
        .style "fill", (d) -> d.strategy.color 
        .style "opacity", 0.5
        .attr "title", (d) -> "#{d.strategy.name}"
        .attr "r", 8
        .attr "cx", (d) -> d.x
        .attr "cy", (d) -> d.y

    populate()


We then write a loop where each svg circle triggers the move function for its bound agent.  


    move = () ->
      circles = canvas.selectAll "circle"
      circles.each (d) ->
        d = simulation.contest d
      .each (d) ->
        d = simulation.update d
      .transition()
      .duration 500
      .attr "cx", (d) -> d.x
      .attr "cy", (d) -> d.y
      .attr "r", (d) -> 8
      .attr "title", (d) -> "#{d.strategy.name} - #{d.score}"
      .style "fill", (d) -> d.strategy.color

    run = () ->
      move() unless running is false


Finally, we run the loop continuous with a half second pause.


    setInterval run, 1000

