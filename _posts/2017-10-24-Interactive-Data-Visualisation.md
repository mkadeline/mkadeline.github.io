---
layout: post
title: "Data Visualisation and Angular"
categories: web
excerpt_separator: <!--more-->
imageid: datapi.jpg
---

D3.js is by far my favourite charting library for web development and wherever possible, I'll opt to send data to the front end and visualise it through D3 rather <!--more--> than render graphs in the back end. This can howsever add some layers of complexity. While the API is fairly simple, it is a powerful library and requires some getting used to and the methods we choose to pass the data can also add complexity.

In this post, I explored the use of Angular and D3.js to create some interactive data visualisations. 

Firstly, in setting up, I found the easiest method for importing the required D3 library in Angular was to create a bridge in the form of an export file, called d3.ts.

```typescript
export * from 'd3-array';
export * from 'd3-axis';
export * from 'd3-brush';
export * from 'd3-chord';
export * from 'd3-collection';
export * from 'd3-color';
export * from 'd3-dispatch';
export * from 'd3-drag';
export * from 'd3-dsv';
export * from 'd3-ease';
export * from 'd3-force';
export * from 'd3-format';
export * from 'd3-geo';
export * from 'd3-hierarchy';
export * from 'd3-interpolate';
export * from 'd3-path';
export * from 'd3-polygon';
export * from 'd3-quadtree';
export * from 'd3-queue';
export * from 'd3-random';
export * from 'd3-request';
export * from 'd3-scale';
export * from 'd3-selection';
export * from 'd3-shape';
export * from 'd3-time';
export * from 'd3-time-format';
export * from 'd3-timer';
export * from 'd3-transition';
export * from 'd3-voronoi';
export * from 'd3-zoom';
```

Then, referencing this file allowed me to import all d3-modules at once. This approach can be a little heavy handed, but we can control what D3.js libraries we need through this d3.ts and it also allows us to change the export statements in one place only if module names change, as they might during the migration to v4.

This file is then imported as:
```typescript
import * as d3 from /path/to/d3.ts
```

In this example, I used Json data in .ts export files, however we could use other formats after the correct parsing.
The json data used:
- Financial Data
- Temperature Data
- Housing Data

D3 is a remarkably flexible library, particularly when dealing with data and conversion or parsing it into the correct form. This flexibility was however limited by Typescript, with the benefit of that being its typesafe nature.

There were some slight alterations to usual D3 code.

For example, defining the data and domains for some Data sources required the creation of a date object and correct type declarations when mapping data to variables. Different data forms may not need as much consideration but it's something to keep in mind.
```Typescript
this.data = allInvestments.map((v) => v.values.map((v) => new Date(v.date)  ))[0];
...
this.line = d3.line()
                    .curve(d3.curveLinear)
                    .x( (d: any) => this.x(new Date(d.date)) )
                    .y( (d: any) => this.y(d.value) );
...
this.x.domain(d3.extent(this.data, (d: any) => d ));
```

The final step was providing some level of interaction. Allowing clients to control the data they see and change inputs to change the visualised data before their eyes can be very powerful. 
The simplest method for the reshuffled data was to call a new function on a button press, one that would reset and replace the existing data with the chosen data.

In my component I'd have a setUpGraph() function to initialise the axes and space, followed by the function to present the original graph. Finally, linked a button with a function and passing some information through allowed me to call updateLines( chosenDataSource ) and interactively change the data.

For example, for the Financial Data Component:
```Typescript
import { Component, OnInit } from '@angular/core';
import { FinGraphService } from './../services/fin-graph-service/fin-graph.service';


@Component({
  selector: 'app-fin-data',
  templateUrl: './fin-data.component.html',
  styleUrls: ['./fin-data.component.css']
})
export class FinDataComponent implements OnInit {

  investment: string = 'savings'; // The first graph to produce

  setUpGraph(): void { this.finGraphService.setUpLine(); }; // called inside produceBlueGraph
  updateLines():void { this.finGraphService.updateLine(this.investment); }
  produceBlueGraph(): void { this.finGraphService.produceBlueGraph(this.investment); }


  constructor(private finGraphService: FinGraphService) { }

  ngOnInit() {
    this.produceBlueGraph();
  }

}
```
And the functions are called through buttons, with the relevant parameter passed by updating a variable in the component:
```html
<div class="panel-body">
    <div class="col-sm-3 text-center">
      <button
        class="btn btn-primary"
        (click)=" investment = 'savings'; updateLines()">
        Savings
      </button>
    </div>
    <div class="col-sm-3 text-center">
      <button
        class="btn btn-primary"
        (click)=" investment = 'bonds'; updateLines()">
        Bonds
      </button>
    </div>
    <div class="col-sm-3 text-center">
      <button
        class="btn btn-primary"
        (click)=" investment = 'property'; updateLines()">
        Property
      </button>
    </div>
    <div class="col-sm-3 text-center">
      <button
        class="btn btn-primary"
        (click)=" investment = 'stocks'; updateLines()">
        Stocks
      </button>
    </div>
</div>
```
The actual change of data and updating of the line is accomplished in several lines in updateLines():
```Typescript
updateLine(loc:string): void {

    //...
    // Code to reset the axes
    // ...

    // Use the passed parameter to change the data
    if (loc == "savings") {
    var svg = d3.select(".orangeContainer")
                .data(savings)
                .transition();
    } else if (loc == "bonds") {
    var svg = d3.select(".orangeContainer")
          .data(bonds)
          .transition();
    } else if (loc == "property") {
    var svg = d3.select(".orangeContainer")
          .data(property)
          .transition();
    } else if (loc == "stocks") {
    var svg = d3.select(".orangeContainer")
          .data(stocks)
          .transition();
    }

    // Allow a smooth transition
    svg.select(".line")
       .duration(750)
       .attr("d", (d) => this.line(d.values) )
       .style("stroke", (d) => this.z(d.id) )
```
As the data was all of the same size, there was no need for enter() or exit(). However if needed they can be added to the if block.

The entire process is only a slight extension on the original data visualisation and provides and excellent way for users to compare data across inputs, in this case across investments or cities, and bring the data to life.

A demonstration is available at our [Angular demo site](angular.adlyss.com)