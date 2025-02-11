# Vnodes

Vue components to create svg interactive graphs, diagrams or node visual tools.

### Demo

https://tiagolr.github.io/vnodes/

### Install

```bash
npm install vnodes
```

## Vnodes 2.0 is here!

With 2.0 the rendering method was changed, instead of having foreignObjects inside svg, the `screen` now uses different layers for svg and html while sinchronizing the transforms between them. This fixes issues with Safari that were partially patched by @metanas, the layout of nodes becomes simpler as there is no need to account for margins, clipping, lack of support of absolute positioning inside nodes or opacity in browsers based on WebKit.

Markers were also revamped among other changes, see [CHANGELOG.md](./CHANGELOG.md) for more details.

### Get started
```html
<template>
  <screen ref="screen">
    <!-- svg content can be placed in #edges template -->
    <template #edges>
      <edge v-for="edge in graph.edges" :data="edge" :nodes="graph.nodes" :key="edge.id">
      </edge>
    </template>

      <!-- html content can be placed on #nodes template -->
    <template #nodes>
      <!-- nodes can have any html content, defaults to <div>{{node.id}}</div> -->
      <node v-for="node in graph.nodes" :data="node" :key="node.id">
      </node>
    </template>
  </screen>
</template>
```

Previously all svg and html nodes were placed inside screen default slot, in 2.0 that changed and it uses different layers for different types like `#nodes` (html), `#edges` (svg) and `#overlay` (svg).

The rest of the API remains the same but there were a few minor tweaks.

```js
import { Screen, Node, Edge, graph } from 'vnodes'
export default {
  components: {
     Screen,
     Node,
     Edge
  }
  data () {
    return {
      graph: new graph()
    }
  }
  created () {
    this.graph.createNode('a')
    this.graph.createNode('b')
    this.graph.createEdge('a', 'b')
    this.graph.graphNodes()
  }
}
```

## Components

### Screen

Main container of html and svg content, handles zoom panning and applies the same transforms to all its layers.

```html
<screen>
  <template #edges>
    <circle cx="50" cy="50" r="50" fill="red"/>
  </template>
</screen>
```

#### Screen Options
Screen component uses [svg-pan-zoom](https://www.npmjs.com/package/svg-pan-zoom) under the hood
and screen takes options prop like this
```html
<screen :options="options">
  <template #edges>
    <circle cx="50" cy="50" r="50" fill="red"/>
  </template>
</screen>
```
you can refer to available options [here](https://www.npmjs.com/package/svg-pan-zoom#how-to-use)
```javascript
{
  viewportSelector: string,
  panEnabled: boolean,
  controlIconsEnabled: boolean,
  zoomEnabled: boolean,
  dblClickZoomEnabled: boolean,
  mouseWheelZoomEnabled: boolean,
  preventMouseEventsDefault: boolean,
  zoomScaleSensitivity: number,
  minZoom: number,
  maxZoom: number,
  fit: boolean,
  contain: boolean,
  center: boolean,
  refreshRate: 'auto',
  beforeZoom: function(){},
  onZoom: function(){},
  beforePan: function(){},
  onPan: function(){},
  onUpdatedCTM: function(){},
  customEventsHandler: {},
  eventsListenerElement: null
}
```

### Node

Div containers with handlers for data updates based on dimensions and positioning, also provides dragging by default which can be disabled.

```html
<node :data="{
    id: 'test',
    x: 100,
    y: 100,
    width: 250,
    height: 150
  }"
>
  <h1>My First Node!</h1>
</node>
```

### Edge

Connects nodes using svg paths

```html
<edge :data="{
  from: { x: 0, y: 0},
  to: { x: 100, y: 100}}"
></edge>
```

Edges require node references `{ from: id|Object, to: String|Object }`, if nodes are refered by `id(String)` an array `nodes` must be passed:

```html
<edge
  :data="{from: 'A', to: 'B'}"
  :nodes="[{id: 'A' ... ]">
</edge>
```

Edges can take **anchor** information to offset their position relative to a node,

```html
<edge :data="{
  from: nodes[0],
  to: nodes[1],
  fromAnchor: 'center',
  toAnchor: 'top-left',
}">
```
 anchors format can be:

* String `'center', 'left', 'right', 'top', 'top-left', 'top-right', 'bottom', 'bottom-left', 'bottom-right', 'cirlce', 'rect'`
* Object `{ x?:Number|String, y?: Number|String, align?: String, snap?: String }`

Examples of valid anchors:

```js
null
{ x: 0, y: 0}
{ x: 10, y: 10 }
{ x: '50%', '50%' }
{ x: '50%', '50%', snap: 'rect' }
{ align: 'bottom-right' }
'center'
'top-left'
'circle'   // snaps offset to circle with radius node.width/2
'rect'     // snaps offset to node rectangle
```

### Group

Surrounds a group of nodes with a rectangle, allows dragging multiple nodes.

```html
<group :nodes="nodes">
  <h1>Group Label</h1>
</group>
```

### Port

Placed inside a node, automatically offsets edges to a their position inside the nodes.

### Label

Create a label node that is positioned along an edge

```html
<v-label :edge="graph.edges[0]" :perc="50" :offset="{x: 0, y: -50}">
  <h4>Content</h4>
</v-label>
```

### graph.js

Can be used to store edges and nodes.
Contains utility methods to build graphs, layouts, remove and create nodes, edges and so on.

## Styling

The simplest way to style nodes and edges is using CSS

```css
<style>
svg .node .content {
  border-radius: 50%;
  background-color: red;
}

svg .edge {
  stroke-width: 10;
  stroke: blue;
  marker-start: url(#arrow-start);
}
</style>
```

### Markers

There are two ways to create makers for eges, one is using [SVG markers](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/marker), creating definitions  `<defs></defs>` in `#edges` slot and then assign them to edges using CSS.

In 2.0 the old markers helper was removed and a new `Marker.vue` component was added that provides embed svg markers which are more versatile than SVG markers, here is how it can be used:

```html
<screen ref="screen">
  <template #edges>
    <edge v-for="edge in edges" :data="edge" :nodes="graph.nodes" :key="edge.id">
    </edge>
    <v-marker v-for="edge in edges" :edge="edge" :perc="100">
      <rect x="0" y="0" width="10" height="10" :fill="markerColor">
    </v-marker>
  </template>
</screen>
```

The marker can be any svg content, the component handles rotations and translations to the correct place along the edge given a percentage `perc` and the edge to place on.
These markers are more versatile then defining them in svg using `<defs></defs>` although probably more expensive in terms of computation.
The svg content should be centered at the origin for the transforms to work properly, the `offset` property can be used to correct alignments.
