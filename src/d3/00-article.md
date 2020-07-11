---
css: '/blog/assets/github-extensions.css'
eleventyNavigation:
  order: 0
  parent: D3
  title: D3 SETT Article
layout: topic-layout.njk
---

## Introduction

D3 (<https:/>/d3js.org>) stands for Data-Driven Documents.
It renders, and re-renders, HTML elements (including `svg`) in a web browser
based on data and changes to the data.
D3 is primarily used for data visualizations such as
bar charts, pie charts, line charts, scatter plots, geographic maps, and more.
But as we will see, it has quite a bit of overlap with jQuery
and can be used for many kinds of DOM manipulations
that are not related to data visualization.

So why should I write about D3 now.
After all, it has been around for since February 2011.
Many tutorial articles already exist.
But many of those describe older versions of D3
and each major version included breaking changes.
Version 5 of D3 was released in January, 2018.
So this is my chance to write an up-to-date D3 tutorial
and try to make it very easily to understand.
Of course you will be the judge of whether I am successful.

JavaScript has been my primary programming language
for the entirety of the life of D3.
I have know of its existence for quite a while,
but have ignored it because I was more focused on other JavaScript-based
topics including jQuery, Angular, React, Vue, Svelte, and TypeScript.
It was time for me to stop ignoring D3 and dig in.

D3 was created by Mike Bostock, along with other contributors.
It is open source and uses a BSD license.

D3 uses HTML, CSS, JavaScript, the DOM, and SVG,
so knowledge of those topics is helpful.
To see examples of using D3, click the "Examples" link
near the top of <https://d3.js.org>.

There are 3 levels of D3 usage, using higher-level libraries,
copying and modifying examples, and coding from scratch.

An example of a library that uses D3 is C3 (<https://c3js.org/>).
It uses D3 under the hood to render many kinds of charts.
These charts are customizable,
but not to the extent that coding from scratch provides.
But charts can be implemented quickly this way
and the requirement to have deep knowledge of D3 is removed.
They respond well to changes in data,
including transitions from old to new values.

There are many sites that provided examples of charts implemented with D3.
Perhaps you can find an example that almost exactly what you want.
Copying and customizing the code will often require far less time
than implementing a chart from scratch.

My hope is that by the time you finish reading this article,
you will know enough about D3 to enable a final option
which is implementing an entire chart from scratch.
This provides the most flexibility, but requires the most effort.

The D3 JavaScript API uses CSS selectors
to find elements in the DOM that it modifies.
It also provides methods to append elements and text nodes,
set attributes, and modify text content.
In this respect it has overlap with jQuery functionality.

The functionality of D3 is divided into modules.
A default build includes a collection of commonly used modules.
Custom builds can omit unneeded modules and add ones not in the default build
in order to minimize the amount of code that must be downloaded to browsers.

These modules include:

- Arrays
- Axes
- Brushes
- Chords
- Collections (Objects, Maps, Sets, Nests)
- Colors
- Color Schemes
- Contours
- Dispatches
- Dragging
- Delimiter-Separated Values
- Easings
- Fetches
- Forces
- Number Formats
- Geographies
- Hierarchies
- Interpolators
- Paths
- Polygons
- Quadtrees
- Random Numbers
- Scales
- Selections
- Shapes
- Time Formats
- Time Intervals
- Timers
- Transitions
- Voronoi Diagrams
- Zooming

Let's learn D3 by example.
Many D3 tutorials start with a bar chart and so will we.
But ours will include more features than typically shown in tutorials
in order to introduce more D3 concepts.

You are encouraged to copy the code in the article and try it in a web browser!

### SVG Basics

SVG stands for Scalable Vector Graphics.
It is an XML-based syntax for specifying vector graphics
and is supported by all modern web browsers.

SVG elements include `svg`, `line`, `polygon`, `rect`, `circle`, `path`,
`text`, `image`, `g` (for group), and many more.
For our bar chart will will only need `svg`, `g`, `rect`, and `text`.

Let's get started by drawing a rectangle and putting some text on it.

![D3 SVG demo](/blog/assets/d3-svg-demo.png)

### `svg-demo.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>D3 SVG Demo</title>
    <meta charset="utf-8" />
    <meta name="description" content="D3 SVG Demo" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <link rel="stylesheet" href="./svg-demo.css" />

    <script src="https://d3js.org/d3.v5.min.js"></script>
    <script src="./svg-demo.js" defer></script>
  </head>
  <body>
    <svg id="chart"></svg>
  </body>
</html>
```

### `svg-demo.css`

```css
body {
  background-color: linen;
  font-family: sans-serif;
}

.bar > rect {
  stroke: black;
  stroke-width: 1;
}

.bar > text {
  fill: white;
  text-anchor: middle; /* horizontally centers */
}

#chart {
  background-color: white;
}
```

### `svg-demo.js`

```js
const HEIGHT = 300;
const WIDTH = 400;
const score = 7; // out of 10
const barHeight = HEIGHT * (score / 10);
const barWidth = 50;

const svg = d3.select('#chart').attr('width', WIDTH).attr('height', HEIGHT);

const group = svg.append('g').attr('class', 'bar');

group
  .append('rect')
  .attr('height', barHeight)
  .attr('width', barWidth)
  .attr('x', 0)
  .attr('y', HEIGHT - barHeight)
  .attr('fill', 'cornflowerblue');

group
  .append('text')
  .text(score)
  .attr('x', barWidth / 2) // center horizontally in bar
  .attr('y', HEIGHT - barHeight + 20); // just below top
```

### D3 Selection Objects

A D3 selection object encapsulates a set of DOM elements,
similar to a jQuery object.
To create one, use the d3 methods `select` and `selectAll`.
For example, `d3.select('.bar')` returns a selection object that
encapsulates the first element in the DOM with a CSS class of "bar".
`d3.selectAll('.bar')` is similar, but the returned selection object
encapsulates all the matching DOM elements.

The `select` and `selectAll` methods can also be called on a selection
to find elements that are descendants of the elements in the selection
within the DOM tree.

Here is an example that demonstrates using the `select` and `selectAll` methods.

```html
<html>
  <head>
    <title>D3 Selections</title>
    <script src="https://d3js.org/d3.v5.min.js"></script>
  </head>
  <body>
    <div class="chart" id="chart1">
      <div class="bar">Bar #1</div>
      <div class="bar">Bar #2</div>
      <div class="bar">Bar #3</div>
    </div>

    <div class="chart" id="chart2">
      <div class="bar">Bar #4</div>
      <div class="bar">Bar #5</div>
    </div>

    <script>
      const bars1 = d3.selectAll('.bar');
      console.log('bars1 =', bars1);

      const chart1 = d3.select('#chart1');
      const bars2 = chart1.selectAll('.bar');
      console.log('bars2 =', bars2);

      const charts = d3.selectAll('.chart');
      const bars3 = charts.selectAll('.bar');
      console.log('bars3 =', bars3);
    </script>
  </body>
</html>
```

Here is the output from the `console.log` calls found in the DevTools console.

![selection objects in console](/blog/assets/d3-selections-in-console.png)

Each selection object has the following properties:

- `_parents`
  This is an array of the DOM elements that were searched
  When the `select` and `selectAll` methods are called on the `d3` object,
  this array will contain only the `html` element of the current document.
  When these methods are called on an existing selection,
  this array will contain all the DOM elements in that selection.
- `_groups`
  This is an array of DOM NodeList objects, one per parent element.
  These NodeList objects contain the matching DOM elements.

In the `bars1` selection, `_parents` contains only the `html` element,
indicating that the entire document was searched.
`_groups` holds a single `NodeList` that
contains every DOM element with a CSS class of "bar".

In the `bars2` selection, `_parents`
contains only the element with an id of "chart1".
This happens because the search was performed on
a selection that only contains that element.
`_groups` holds a single `NodeList` that
only contains the DOM elements with a CSS class of "bar"
that are descendants of the element with an id of "chart1".

In the `bars3` selection, `_parents`
contains the two elements with a CSS class of "chart"
because the search was performed on a selection
that only contains those elements.
`_groups` holds two `NodeList` objects, one for each parent element.
Each of these contains the DOM elements with a CSS class of "bar"
that are descendants of their respective parent element.

### The `data` method

Selection objects have a `data` method that associates data in an array
with the DOM elements in the selection.
It does this by adding a `__data__` property to them.

Here is an example that demonstrates calling the `data` method on a selection.

```html
<html>
  <head>
    <title>D3 data method</title>
    <script src="https://d3js.org/d3.v5.min.js"></script>
  </head>
  <body>
    <script>
      const values = [7, 13, 2];
      const bars = d3.selectAll('.bar').data(values);
      console.log('bars =', bars);
    </script>
  </body>
</html>
```

Here is the output from the `console.log` call found in the DevTools console.

![selection from data method in console](/blog/assets/d3-selection-from-data.png)

If the values in the array passed to the `data` method are objects,
a second argument which is a function can be passed.
This is responsible for extracting or computing a value
from each object in the array.
For example, suppose we have an array in a variable names `paintings`
that contains objects that describe paintings and
they have properties like `paintingName`, `width`, and `height`.
We can use the area of each painting as the values as follows:

```js
someSelection.data(paintings, p => p.width * p.height);
```

The `data` method returns a new selection that contains three sub-selections
referred to as "update", "enter", and "exit".

The update sub-selection is held in the `_groups` property
and contains all the DOM elements in the selection
that can be _updated_ with a data value.

The enter sub-selection is held in the `_enter` property
and contains a placeholder for each DOM element
that must be created in order to associated a data value.
These elements will _enter_ the DOM.
Note the `__data__` property values in each of these placeholder objects.

The "exit" sub-selection is held in the `_exit` property
and contains all the DOM elements in the selection
for which no data will be assigned.
Typically these elements _exit_ the DOM by being removed.

There are three scenarios to consider.

1. There are no DOM elements in the selection.
1. There are DOM elements in the selection,
   but fewer than the number of data values.
1. There are DOM elements in the selection,
   but more than the number of data values.

We have already seen the first scenario in previous code example.

To see the second scenario, add the following to the HTML.

```html
<div class="bar"></div>
<div class="bar"></div>
```

Reloading the page in the browser, we see that
the update sub-selection holds these two elements with the values 7 and 13.
The enter sub-selection now contains a single placeholder with the value 2.
The exit sub-selection is empty.

To see the third scenario, add three more `div` elements
with a class of "bar" so there are a total of five.
Reloading the page in the browser, we see that
the update sub-selection holds three elements with the values 7, 13, and 2.
The enter sub-selection is empty.
The exit sub-selection holds two elements that can be removed.

In order to support changing data, both values and the number of values,
we need to handle all three sub-selections.
This means we need a way to iterate over the elements in each.

To iterate over all the elements in the update sub-selection
call methods such as `text` directly on the selection object.
To get the enter sub-selection, call the `enter` method.
To get the exit sub-selection, call the `exit` method.

The following example demonstrates each of these.
It generates an array containing a random number of random integers.
These are used to display values in `div` elements
that have a CSS class of "bar".
Their text indicate whether the element was updated or just entered the DOM.
Each time the "Update" button is pressed, the process repeats
and previously created `div` elements are reused.

Selection objects support many methods,
some which act on all the DOM elements they encapsulate.
An example used in the code below is the `text` method.
These methods take a function that is invoked
once for each encapsulated DOM element.
The function is passed the value of the DOM element **data** property
and the index of the DOM element within the selection.
Just like in jQuery, it is not an error to call such a method on a
selection that is empty, meaning it doesn't encapsulate any DOM elements.

```html
<html>
  <head>
    <title>D3 data method</title>
    <script src="https://d3js.org/d3.v5.min.js"></script>
  </head>
  <body>
    <button onclick="update()">Update</button>

    <div id="chart"></div>

    <script>
      // Gets a random integer between 1 and max inclusive.
      const randomInt = max => Math.floor(Math.random() * max) + 1;

      // Generates an array containing a random number of random integers.
      function getValues() {
        const count = randomInt(7);
        const values = [];
        for (let i = 0; i < count; i++) {
          values.push(randomInt(10));
        }
        return values;
      }

      function update() {
        const values = getValues();
        console.log('values =', values);

        // Create a D3 selection representing the bars
        // which are div elements with a CSS class of "bar"
        // and associate data values with them.
        const bars = d3.select('#chart').selectAll('.bar').data(values);

        // Update the text of .bar elements that are already in the DOM.
        bars.text(d => 'update ' + d);

        // Create new .bar elements that need to enter the DOM
        // and set their text.
        bars
          .enter()
          .append('div')
          .attr('class', 'bar')
          .text(d => 'enter ' + d);

        // Remove .bar elements that need to exit the DOM.
        bars.exit().remove();
      }

      update(); // initial call
    </script>
  </body>
</html>
```

This is a common D3 pattern referred to as the "general update pattern".
It can be simplified in a couple of ways.

The first simplification is the use the `merge` method.
This is called after appending elements to the enter sub-selection.
It creates a new selection by combining elements in the
selection on which it is called (the enter sub-selection in this case)
with the selection passed to it (the update sub-selection in this case).
The call that follow this operates on this new selection.
This approach allows us to replace the
"Update" and "Create" sections above with the following:

```js
bars
  .enter() // returns the enter sub-selection
  .append('div') // returns the enter sub-selection
  .attr('class', 'bar') // returns the enter sub-selection
  .merge(bars) // returns a new selection
  .text(d => 'value ' + d);
```

We have lost the ability to specify different text for
updated and entered elements, but that is not typically needed.

The second simplification is to use the `join` method.
This takes three functions, one to handle each sub-selection.
They must be in the order enter, update, and exit.
Using this approach, the code above, including the code for
handling the exit sub-selection, can be replaced by the following.

```js
bars.join(
  enter =>
    enter
      .append('div')
      .attr('class', 'bar')
      .text(d => 'enter ' + d),
  update => update.text(d => 'update ' + d),
  // This is the default behavior for the exit sub-selection
  // and can be omitted.
  exit => exit.remove()
);
```

This regains the ability to have different text
for updated and entered elements.
But if that is not needed, we can shorten the code even more like this:

```js
bars.join(enter =>
  enter
    .append('div')
    .attr('class', 'bar')
    .text(d => 'value ' + d)
);
```

We can even take this one step farther and pass the name of the element to be created for each element in the enter sub-selection like this:

```js
bars
  .join('div')
  .attr('class', 'bar')
  .text(d => 'value ' + d);
```

Selection objects are immutable, meaning
the set of DOM elements they encapsulate cannot be changed.
However, there are many methods on selection objects that return
a new selection, including `map` and `filter`.

That's a lot to take in!
But if you're feeling somewhat comfortable with the notion of selection objects,
their three sub-selections, and the general update pattern,
you are ready to put that knowledge to use for creating your first chart.

### Drawing Bars

Earlier we learned how to draw a single bar.
Let's combine that with what we have learned about selections
to draw one bar for each piece of data in an array.
We will create a bar chart like the following:

![D3 Bar Chart](/blog/assets/bar-chart.png)

To see this in action, browse
{% aTargetBlank 'https://bar-chart.vercel.app/', 'Bar Chart on Vercel' %}.

The "Update" button generates new, random data
and updates the bars.
This is implemented by the files
`index.html`, `bar-chart.css`, and `bar-chart.js`
which are shown below.

This code gives us a good base for adding features.
There are many D3 concepts to cover here.
See comments in the code for details.

### `index.html`

```html
<html>
  <head>
    <title>D3 Bar Chart</title>
    <meta charset="utf-8" />
    <meta name="description" content="D3 Bar Chart" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <link rel="stylesheet" href="./bar-chart.css" />

    <script src="https://d3js.org/d3.v5.min.js"></script>
    <script src="./bar-chart.js" defer></script>
  </head>
  <body>
    <!-- The bars will be added in this svg element. -->
    <svg id="chart"></svg>

    <div>
      <!-- Render a new version of the chart when this is pressed. -->
      <button onclick="updateData()">Update</button>
    </div>
  </body>
</html>
```

### `bar-chart.css`

```css
body {
  background-color: linen;
  font-family: sans-serif;
}

.bar > rect {
  stroke: black;
  stroke-width: 1;
}

button {
  margin-top: 1rem;
}

#chart {
  background-color: white;
}
```

### `bar-chart.js`

```js
// Later these will be adjusted to make room
// for a vertical and horizontal axis.
const BOTTOM_PADDING = 10;
const LEFT_PADDING = 10;
const RIGHT_PADDING = 10;
const TOP_PADDING = 10;

// Full size of the svg element.
const HEIGHT = 300;
const WIDTH = 400;

// Size that can be used for the bars.
const usableHeight = HEIGHT - TOP_PADDING - BOTTOM_PADDING;
const usableWidth = WIDTH - LEFT_PADDING - RIGHT_PADDING;

// Random data will be selected from this array.
const allData = [
  {name: 'apple', colorIndex: 1},
  {name: 'banana', colorIndex: 2},
  {name: 'cherry', colorIndex: 3},
  {name: 'date', colorIndex: 4},
  {name: 'grape', colorIndex: 5},
  {name: 'mango', colorIndex: 6},
  {name: 'peach', colorIndex: 7},
  {name: 'raspberry', colorIndex: 8},
  {name: 'strawberry', colorIndex: 9},
  {name: 'tangerine', colorIndex: 10},
  {name: 'watermelon', colorIndex: 11}
];

let barPadding, barWidth, xScale, yScale;

// This is used to select bar colors based on their score.
const colorScale = d3.scaleOrdinal(d3.schemePaired); // 12 colors

// This returns a random integer from 1 to max inclusive.
const random = max => Math.floor(Math.random() * max + 1);

// This returns an array of objects taken from allData.
// A "score" property with a random value from 1 to 10
// is added to each object.
function getRandomData() {
  const count = random(allData.length);
  const shuffled = allData.sort(() => 0.5 - Math.random());
  const data = shuffled.slice(0, count);
  data.sort((f1, f2) => f1.name.localeCompare(f2.name));
  for (const item of data) {
    item.score = random(10);
  }
  return data;
}

// This updates the attributes of an SVG rect element
// that represents a bar.
function updateRect(rect) {
  rect
    // Each fruit will keep the same color as its score changes.
    .attr('fill', d => colorScale(d.colorIndex))
    .attr('width', barWidth - barPadding * 2)
    .attr('height', d => usableHeight - yScale(d.score))
    .attr('x', barPadding)
    .attr('y', d => TOP_PADDING + yScale(d.score));
}

// This updates the bar chart with random data.
function updateData() {
  const data = getRandomData();

  // Calculate padding on sides of bars based on # of bars.
  barPadding = Math.ceil(30 / data.length);

  // Calculate the width of each bar based on # of bars.
  barWidth = usableWidth / data.length;

  // Create a scale to map data index values to x coordinates.
  // This is a function that takes a value in the "domain"
  // and returns a value in the "range".
  xScale = d3
    .scaleLinear()
    .domain([0, data.length])
    .range([LEFT_PADDING, LEFT_PADDING + usableWidth]);

  // Create a scale to map data score values to y coordinates.
  // The range is flipped to account for
  // the SVG origin being in the upper left corner.
  // Like xScale, this is a function that takes a value in the "domain"
  // and returns a value in the "range".
  // The d3.max function computes the largest data value in a given array
  // where values are computed by the 2nd argument function.
  const max = d3.max(data, d => d.score);
  yScale = d3.scaleLinear().domain([0, max]).range([usableHeight, 0]);

  // Create a D3 selection object that represents the svg element
  // and set the size of the svg element.
  const svg = d3.select('#chart').attr('width', WIDTH).attr('height', HEIGHT);

  // This is the most critical part to understand!
  // You learned about about selections and the general update pattern
  // in the previous section.
  const groups = svg
    .selectAll('.bar')
    .data(data, d => d.name)
    .join(enter => {
      // Create a new SVG group element for each placeholder
      // to represent a new bar.
      // For now the only thing in each group will be a rect element,
      // but later we will add a text element to display the value.
      const groups = enter.append('g').attr('class', 'bar');

      // Create a new SVG rect element for each group.
      groups
        .append('rect')
        .attr('height', 0)
        .attr('y', TOP_PADDING + usableHeight);

      return groups;
    });

  // The join method call above returns a selection that combines
  // the update and enter sub-selections into its update selection.
  // This allows operations needed on elements in both
  // to be performed on the new selection.

  // Translate the groups for each bar to their
  // appropriate x coordinate based on its index.
  groups.attr('transform', (_, i) => `translate(${xScale(i)}, 0)`);

  // Update all the rect elements using their newly associated data.
  groups.select('rect').call(updateRect);
}

// Render the first version of the chart.
updateData();
```

## Adding Y Axis

Now let's add a y axis to the chart the indicates the bar values.
We want this to be dynamic so that the highest value
matches the highest value of any of the bars that are present.

Here are the steps:

1. Increase the value of `LEFT_PADDING` from 10 to 25
   to leave room for the y axis.

1. Add the variable `yAxisGroup` to the `let` statement near the top
   as follows:

   ```js
   let barPadding, barWidth, xScale, yAxisGroup, yScale;
   ```

1. Add the function `updateYAxis` shown below:

   ```js
   function updateYAxis(svg, data, max) {
     if (!yAxisGroup) {
       // Create an SVG group that will hold the y axis and
       // translate the group to the appropriate position in the SVG.
       yAxisGroup = svg
         .append('g')
         .attr('class', 'y-axis')
         .attr('transform', `translate(${LEFT_PADDING}, ${TOP_PADDING})`);
     }

     // Create an array with values from zero to max
     // that will be used as the tick values on the y axis.
     const tickValues = Array.from(Array(max + 1).keys());

     // Create an axis generator function that renders the yAxis.
     const yAxis = d3
       .axisLeft(yScale)
       .tickValues(tickValues)
       .tickFormat(n => n.toFixed(0));

     // Pass the selection for the group to the
     // axis generator function to render it.
     yAxis(yAxisGroup);
     // An equivalent way to do this is yAxisGroup.call(yAxis);
   }
   ```

1. Add the following at the bottom of the `updateData` function:

   ```js
   updateYAxis(svg, data, max);
   ```

The result looks like this:

![D3 bar chart with y axis](/blog/assets/d3-bar-chart-with-y-axis.png)

## Adding X Axis

Now let's add an x axis to the chart whose values
are labels associated with each bar.
In our example the labels are names of fruits
and the values represent scores in a poll
where respondents name their favorite fruit.
Of course all our data is randomly generated.

Here are the steps:

1. Add the following to `bar-chart.css` to position and rotate
   the x-axis labels:

```css
.x-axis > .tick > text {
  /* Rotate labels so they fit below bars better. */
  transform: translate(-8px, 15px) rotate(-45deg);
}
```

1. Increase the value of `BOTTOM` from 10 to 50
   to leave room for the x axis.

1. Add the variable `yAxisGroup` to the `let` statement near the top
   as follows:

   ```js
   let barPadding, barWidth, xAxisGroup, xScale, yAxisGroup, yScale;
   ```

1. Add the function `updateXAxis` shown below:

   ```js
   function updateXAxis(svg, data) {
     if (!xAxisGroup) {
       // Create the x-axis that displays fruit names.
       xAxisGroup = svg
         .append('g')
         .attr('class', 'x-axis')
         .attr('transform', `translate(0, ${TOP_PADDING + usableHeight})`);
     }

     const xAxisScale = d3
       .scaleBand()
       .domain(data.map(item => item.name)) // fruit names
       .range([LEFT_PADDING, LEFT_PADDING + usableWidth]);
     const xAxis = d3.axisBottom(xAxisScale).ticks(data.length);
     xAxisGroup.call(xAxis);
   }
   ```

1. Add the following at the bottom of the `updateData` function:

   ```js
   updateXAxis(svg, data);
   ```

The result looks like this:

![D3 bar chart with x axis](/blog/assets/d3-bar-chart-with-x-axis.png)

## Adding Text on Bars

This includes selecting a text color based on the bar color
using relative luminance.
See the `getTextColor` function.
This includes horizontal centering with the
CSS `text-anchor` property set to `middle`.

## Adding Animation

This includes using `transition`, `duration`, and `easing` methods.
It also explains use of the `myTransition` function.