<!-- Your HTML goes here -->
<template>
  <div>
    <p style="font-size: 15px"> For the First Project in my ECS163 Class, I analyzed the "Fatal Police Shootings in the US" dataset. <br/> </p>
    <div class="chart" id="scatterPlot"></div>
    <p style="font-size: 15px"> I have used a <u>Scatterplot</u> to represent the magnitude of killed officers by year which servers as the overview of the dataset. <br/> 
    <br/> 
    In the scatterplot you can see the general downtrend in police officer deaths, posssibly indicating an increase in peaceful arrests, compliance, and a decrease in violent behavior. This could be because of the legislation that had been passed at the time like certain laws limiting an officer's ability to shoot their gun, or decreased spending in a police department leading to a lack of training/guidance. 
    The downtrend in 2017 is much shorter than the two preceeding years because there wasn't any data for that time period.</p>
    <div class="chart" id="barChart"></div>
    <div class="chart" id="lineChart"></div>
    <p style="font-size: 15px"> For my detailed views, I have implemented a <u>Bar Chart</u> representing the race breakdown of each officer killed, and a <u>Line Chart</u> showing the monthly trend in officer deaths. <br/>
    <br/> 
    Each year has its own respective bar chart representing the deaths in proportion to race, and each year you will see that White Police Officers are killed in a significantly higher amount realtive to Blacks, Hispanics, and Asians. 
    This number is high because of the abudance of White Officers in the police force around the US. Also, the amount of Blacks and Hispanics killed are unproportionately high relative to their population in the department, indicating that they are often more engaged in Shootouts than officers of other race. <br/>
    <br/>
    The Line Chart for each of the three years illustrates the monthly trend in Officer deaths. If you look at each year you will notice that during the first few monthts of the year there is a decline in deaths.
    That number picks back up about halfway through the year, and then declines again heading towards the end of the year. This could mean that the summer time is often a dangerous time in terms of the chances of shootouts ocurring due to things like suitable weather, more outside activities, and an increase in idle behavior. <br/>
   </p>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "View3", // Feel free to rename this and the file
  props: {
    // Feel free to add props to communicate between components
  },
  async mounted() {
    // load data
    let mainFile = await d3.csv("/PoliceKillingsUS.csv");
    await this.createScatterPlot(mainFile);
    await this.createBarChart(mainFile, "16");
    await this.createLineChart(mainFile, "16");
  },
  methods: {
    createScatterPlot(mainFile) {
      let margin = { top: 20, right: 20, bottom: 30, left: 40 },
        width = 400 - margin.left - margin.right,
        height = 300 - margin.top - margin.bottom;

      // setup x
      let xValue = (d) => d.year;
      let xScale = d3.scaleLinear().range([0, width]);
      let xMap = (d) => xScale(xValue(d));
      let xAxis = d3
        .axisBottom()
        .scale(xScale)
        .tickFormat(d3.format(".0f"))
        .ticks(3);

      // setup y
      let yValue = (d) => d.dead;
      let yScale = d3.scaleLinear().range([height, 0]);
      let yMap = (d) => yScale(yValue(d)); // data -> display
      let yAxis = d3.axisLeft().scale(yScale).ticks(12);

      // setup fill color
      let cValue = (d) => d.year;
      let color = d3.scaleOrdinal(d3.schemeCategory10);

      // add the graph canvas to the body of the webpage
      var svg = d3
        .select("#scatterPlot")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom + 50)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      // aggregate the data to create a data set of year by
      // number of cops killed.
      let copsByDate = {};
      for (let i = 0; i < mainFile.length; i++) {
        let yearStr = mainFile[i]["date"].slice(-2);
        if (copsByDate[yearStr]) {
          copsByDate[yearStr] = copsByDate[yearStr] + 1;
        } else {
          copsByDate[yearStr] = 1;
        }
      }
      let copsKilledData = [];
      for (let yearAsStr in copsByDate) {
        copsKilledData.push({
          year: 2000 + parseInt(yearAsStr),
          dead: copsByDate[yearAsStr],
        });
      }

      // don't want dots overlapping axis, so add in buffer to data domain
      xScale.domain([
        d3.min(copsKilledData, xValue) - 1,
        d3.max(copsKilledData, xValue) + 1,
      ]);
      yScale.domain([0, d3.max(copsKilledData, yValue) + 150]);

      // x-axis
      svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + height + ")")
        .call(xAxis);

      // X=axis label
      svg
        .append("text")
        .attr("class", "label")
        .attr(
          "transform",
          "translate(" + width / 2 + " ," + (height + margin.top + 20) + ")"
        )
        .text("Year");

      svg
        .append("text")
        .attr("x", width / 2)
        .attr("y", 8)
        .attr("class", "scattertext")
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("text-decoration", "underline")
        .text("Police Fatalities By Year");

      // y-axis
      svg.append("g").attr("class", "y axis").call(yAxis);

      let currentThis = this;

      // draw dots
      svg
        .selectAll(".dot")
        .data(copsKilledData)
        .enter()
        .append("circle")
        .attr("class", "dot")
        .attr("r", 3.5)
        .attr("cx", xMap)
        .attr("cy", yMap)
        .on("click", function (e, d) {
          e;

          let newYear = d.year.toString().slice(-2);
          currentThis.updateOtherCharts(mainFile, newYear);
        })
        .style("fill", (d) => color(cValue(d)));
    },
    createBarChart(mainFile, selectedYear) {
      let margin = { top: 20, right: 20, bottom: 30, left: 40 },
        width = 400 - margin.left - margin.right,
        height = 300 - margin.top - margin.bottom;

      // set the ranges
      let x = d3.scaleBand().range([0, width]).padding(0.1);
      let y = d3.scaleLinear().range([height, 0]);

      // setup fill color
      let cValue = (d) => d.race;
      let color = d3.scaleOrdinal(d3.schemeCategory10);

      // add the graph canvas to the body of the webpage
      var svg = d3
        .select("#barChart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom + 50)
        .append("g")
        .attr("class", "big")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      svg
        .append("text")
        .attr("x", width / 2)
        .attr("y", 8)
        .attr("class", "bartext")
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("text-decoration", "underline")
        .text("Police Fatalities By Race in Year 20" + selectedYear);

      let data = this.createRaceData(mainFile, selectedYear);

      let currentThis = this;
      // Scale the range of the data in the domains
      x.domain(data.map((d) => currentThis.updateRace(d.race)));
      y.domain([0, d3.max(data, (d) => d.dead) + 60]);

      // append the rectangles for the bar chart
      svg
        .selectAll(".bar")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "bar")
        .attr("x", (d) => x(currentThis.updateRace(d.race)))
        .attr("width", x.bandwidth())
        .attr("y", (d) => y(d.dead))
        .attr("height", function (d) {
          return height - y(d.dead);
        })
        .style("fill", (d) => color(cValue(d)));

      // add the x Axis
      svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x));

      // add the y Axis
      svg.append("g").attr("class", "y axis").call(d3.axisLeft(y));
    },
    createLineChart(mainFile, selectedYear) {
      let margin = { top: 20, right: 20, bottom: 30, left: 40 },
        width = 400 - margin.left - margin.right,
        height = 300 - margin.top - margin.bottom;

      // add the graph canvas to the body of the webpage
      var svg = d3
        .select("#lineChart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom + 50)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      svg
        .append("text")
        .attr("x", width / 2)
        .attr("y", 8)
        .attr("class", "linetext")
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("text-decoration", "underline")
        .text("Police Fatalities By Month in Year 20" + selectedYear);

      let data = this.createMonthData(mainFile, selectedYear);

      let currentThis = this;

      // set the ranges
      var x = d3
        .scaleLinear()
        .domain(d3.extent(data, (d) => d.month))
        .range([0, width]);
      svg
        .append("g")
        .attr("transform", "translate(0," + height + ")")
        .attr("class", "lineX")
        .call(
          d3.axisBottom(x).tickFormat((d) => {
            return currentThis.updateMonth(d);
          })
        );

      // Add Y axis
      var y = d3
        .scaleLinear()
        .domain([0, d3.max(data, (d) => d.dead) + 100])
        .range([height, 0]);
      svg.append("g").call(d3.axisLeft(y)).attr("class", "lineY");

      let lineFunc = d3
        .line()
        .x((d) => x(d.month))
        .y((d) => y(d.dead));
      // Add the line
      svg
        .append("path")
        .data([data])
        .attr("class", "bigLine")
        .attr("fill", "none")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 1.5)
        .attr("d", lineFunc(data));
    },
    createRaceData(mainFile, selectedYear) {
      // aggregate the data to create a data set of year by
      // number of cops killed.
      let copsByRace = {};
      for (let i = 0; i < mainFile.length; i++) {
        let yearStr = mainFile[i].date.slice(-2);
        if (yearStr == selectedYear) {
          let race = mainFile[i].race;
          if (copsByRace[race]) copsByRace[race] = copsByRace[race] + 1;
          else copsByRace[race] = 1;
        }
      }
      let data = [];
      for (let race in copsByRace) {
        data.push({
          race: race,
          dead: copsByRace[race],
        });
      }
      data.sort((a, b) => {
        if (a.race > b.race) {
          return 1;
        }
        if (a.race < b.race) {
          return -1;
        }
        return 0;
      });
      return data;
    },
    updateOtherCharts(mainFile, newYear) {
      this.updateBarChart(mainFile, newYear);
      this.updateLineChart(mainFile, newYear);
    },
    updateLineChart(mainFile, newYear) {
      let margin = { top: 20, right: 20, bottom: 30, left: 40 },
        width = 400 - margin.left - margin.right,
        height = 300 - margin.top - margin.bottom;

      let currentThis = this;
      let data2 = currentThis.createMonthData(mainFile, newYear);
      // set the ranges
      var x2 = d3
        .scaleLinear()
        .domain(d3.extent(data2, (d) => d.month))
        .range([0, width]);
      var y2 = d3
        .scaleLinear()
        .domain([0, d3.max(data2, (d) => d.dead) + 100])
        .range([height, 0]);

      var svg2 = d3.select("#lineChart").transition();

      let lineFunc = d3
        .line()
        .x((d) => x2(d.month))
        .y((d) => y2(d.dead));

      svg2.select(".bigLine").duration(750).attr("d", lineFunc(data2));

      svg2
        .select(".linetext")
        .text("Police Fatalities By Month in Year 20" + newYear);

      svg2
        .select(".lineX")
        .duration(750)
        .call(
          d3.axisBottom(x2).tickFormat((d) => {
            return currentThis.updateMonth(d);
          }).ticks(data2.length)
        );
      svg2.select(".lineY").duration(750).call(d3.axisLeft(y2));
    },
    updateBarChart(mainFile, newYear) {
      let data = this.createRaceData(mainFile, newYear);

      let margin = { top: 20, right: 20, bottom: 30, left: 40 },
        width = 400 - margin.left - margin.right,
        height = 300 - margin.top - margin.bottom;

      let maxVal = d3.max(data, (d) => d.dead) + 60;

      let currentThis = this;

      // set the ranges
      let x4 = d3
        .scaleBand()
        .range([0, width])
        .padding(0.1)
        .domain(data.map((d) => currentThis.updateRace(d.race)));

      let y4 = d3.scaleLinear().range([height, 0]).domain([0, maxVal]);

      // setup fill color
      let cValue = (d) => d.race;
      let color = d3.scaleOrdinal(d3.schemeCategory10);


      // Select the section we want to apply our changes to
      var svg = d3.select("#barChart");
      svg.selectAll(".bar").remove();
      
      // append the rectangles for the bar chart
      let bars = svg
        .selectAll(".big")
        .selectAll(".bar");

      bars
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "bar")
        .attr("x", (d) => x4(currentThis.updateRace(d.race)))
        .attr("width", x4.bandwidth())
        .attr("y", (d) => y4(d.dead))
        .attr("height", function (d) {
          return height - y4(d.dead);
        })
        .style("fill", (d) => color(cValue(d)));

      

      svg = d3.select("#barChart").transition();
      svg.select(".x.axis")
        .duration(750).call(d3.axisBottom(x4));
      svg.select(".y.axis")
        .duration(750).call(d3.axisLeft(y4));
      svg
        .select(".bartext")
        .text("Police Fatalities By Race in Year 20" + newYear);
    },
    updateRace(race) {
      switch (race) {
        case "B":
          return "Black";
        case "W":
          return "White";
        case "H":
          return "Hispanic";
        case "A":
          return "Asian";
        case "O":
          return "Other";
        case "N":
          return "N";
      }
      return "N/A";
    },
    createMonthData(mainFile, selectedYear) {
      let copsByDate = {};
      for (let i = 0; i < mainFile.length; i++) {
        let yearStr = mainFile[i]["date"].slice(-2);
        if (yearStr == selectedYear) {
          let monthStr = mainFile[i]["date"].substring(3, 5);
          if (copsByDate[monthStr]) {
            copsByDate[monthStr] = copsByDate[monthStr] + 1;
          } else {
            copsByDate[monthStr] = 1;
          }
        }
      }
      let data = [];
      let months = {};
      for (let monthStr in copsByDate) {
        let ele = {
          month: parseInt(monthStr),
          dead: copsByDate[monthStr],
        };
        months[monthStr] = ele;
        data.push(ele);
      }

      let finalResults = [
        months["01"] || { month: 1, dead: 0 },
        months["02"] || { month: 2, dead: 0 },
        months["03"] || { month: 3, dead: 0 },
        months["04"] || { month: 4, dead: 0 },
        months["05"] || { month: 5, dead: 0 },
        months["06"] || { month: 6, dead: 0 },
        months["07"] || { month: 7, dead: 0 },
        months["08"] || { month: 8, dead: 0 },
        months["09"] || { month: 9, dead: 0 },
        months["10"] || { month: 10, dead: 0 },
        months["11"] || { month: 11, dead: 0 },
        months["12"] || { month: 12, dead: 0 },
      ];

      while(finalResults[finalResults.length - 1].dead == 0) {
        finalResults.pop();
      }

      return finalResults;
    },
    updateMonth(month) {
      switch (month) {
        case 1:
          return "Jan";
        case 2:
          return "Feb";
        case 3:
          return "Mar";
        case 4:
          return "Apr";
        case 5:
          return "May";
        case 6:
          return "Jun";
        case 7:
          return "Jul";
        case 8:
          return "Aug";
        case 9:
          return "Sep";
        case 10:
          return "Oct";
        case 11:
          return "Nov";
        case 12:
          return "Dec";
      }
      return "";
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.chart {
  display: inline-block;
  margin-right: 20px;
}
</style>