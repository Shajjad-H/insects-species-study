<script>
  /*
  let testMulti=10;
  function getWidth1(d) {
    if(isNaN(d.value)) {
      console.log("getWidth1_", d);
    }
    return (testMulti*(d.value)-testMulti*(0)-1);
  }
  function getX1(d) {
    if(false || isNaN(d.value)) {
      console.log("getX1", d.value);
    }
    return testMulti*(d.value)-8;
    //return (10*(d.value)-10*(0)-1);
  }
  function getX2(d) {
    if(false || isNaN(d.value)) {
      console.log("getX2", d.value);
    }
    return testMulti*(d.value)-5;
  }*/


  // Feel free to change or delete any of the code you see in this editor!
  console.log("before kcreate svg")
  /*
  var svg = d3.select("body").append("svg")
    .attr("width", 960)
    .attr("height", 600);
  */
  var  svg = d3.select("#race_chart").append("svg")
    .attr("width", 960)
    .attr("height", 600);
  
  var tickDuration = 2000;
  
  var top_n = 10;
  var height = 600;
  var width = 960;
  
  const margin = {
    top: 80,
    right: 0,
    bottom: 5,
    left: 2
  };

  let barPadding = (height-(margin.bottom+margin.top))/(top_n*5);

  let pause = false;

  function getContentYear_Country(bombusData, years, countries) {
    
    var content = {};
    for (var idx1 = 0; idx1 < years.length ; idx1++) {
        var year = years[idx1];
        //totalByYear[year] = 0;
        content[year] = {};
        for (var idx2 = 0; idx2 < countries.length; idx2++ ) {
            var country = countries[idx2];
            content[year][country] = 0;
        }
    }
    //console.log("content1", content);
    for (var idx = 0; idx < bombusData.length; idx++) {
        var row = bombusData[idx];
        var year =  row.Year
        var country = row.Country;
        if( countries.indexOf(country)>=0) {
            //console.log();
            content[year][country] = content[year][country] + 1*row.Frequency;
        }
    }
    return content;
}

/**
* Delay for a number of milliseconds
*/
function sleep(delay) {
    console.log("sleep : begin");
    var start = new Date().getTime();
    while (new Date().getTime() < start + delay) {
        // TO nothing
        if(!pause) {
          return;
        }
    }
    console.log("sleep : end");
    return;
}


let year1 = 1997;

d3.csv('bombus_terrestris_freqs.csv').then(function(data) {
    //if (error) throw error;
    
    console.log(data);
    var  svg = d3.select("#race_chart").append("svg")
      .attr("width", 960)
      .attr("height", 600);

    let title = svg.append('text')
                  .attr("x", (width / 2))
                  .attr("y", 35)
                  .attr("text-anchor", "middle")
                  .style("fill", "#5a5a5a")
                  .style("font-family", "Raleway")
                  .style("font-weight", "300")
                  .style("font-size", "24px")
    .html("Evolution of bees over time by country");



    let subTitle = svg.append("text")
    .attr("class", "subTitle")
    .attr("y", 55)
    .html("");

    let caption = svg.append('text')
    .attr('class', 'caption')
    .attr('x', width)
    .attr('y', height-5)
    .style('text-anchor', 'end')
    .html('Source: https://observablehq.com');

    var years = data
          .map((d) => parseInt(d.Year))
          .filter((value, index, self) => self.indexOf(value) === index)
          .sort();

    var countries = data
    .map((d) => d.Country)
    .filter((value, index, self) => self.indexOf(value) === index)
    .sort();

    var data0 = getContentYear_Country(data, years, countries);
    console.log(data0);
    
    var data1 = [];
    var minYear = 1890
    for (var idx = 0; idx < years.length ; idx++) {
          var year = years[idx];
          var values = data0[year]
          // Create items array
          var items = Object.keys(values).map(function(key) {
            return [key, values[key]];
          });
          // Sort the array based on the second element
          items.sort(function(first, second) {
            return second[1] - first[1];
          });
          //console.log(items);
          if(year>=minYear) {
              for (var idx2 = 0; idx2 < countries.length; idx2++ ) {
                  var country = countries[idx2];
                  var item = {}
                  item["name"] = country
                  item["value"] = data0[year][country];
                  item["year"] = year;

                  if (data0.hasOwnProperty(year-1) ) {
                    item["lastValue"] =  data0[year-1][country];
                  }
                  else item["lastValue"] = 0;
                  for(var i = 0 ; i < items.length; i++){
                    if(items[i][0] == country){
                      item["rank"] = i;
                    }
                  }
                  data1.push(item);
              }
              
          }
      }
    
    
    console.log(data1);

    data1.forEach(d => {
    d.value = +d.value,
    d.lastValue = +d.lastValue,
    d.value = isNaN(d.value) ? 0 : d.value,
    d.year = +d.year,
    d.colour = d3.hsl(Math.random()*360,0.75,0.75)


    d3.select("#bt_pause") .on("click", function(){
      var bt = document.getElementById("bt_pause");
      pause = !pause;

      if(pause){
        bt.style.backgroundImage = "url(play.png)";
      }
      else{
        bt.style.backgroundImage = "url(pause2.png)";
      }
      bt.style.backgroundRepeat = "no-repeat";
      bt.style.width="40px";//({ width: 200, height: 200 })
      bt.style.height="40px";
      bt.style.backgroundSize = "40px";
    });
    //d3.select("#bt_resume") .on("click", function(){ 
    //    pause = false;
    //    console.log("pause", pause);
    //});
  });
  console.log(data1);

   let yearSlice = data1.filter(d => d.year == year1 && !isNaN(d.value))
    .sort((a,b) => b.value - a.value)
    .slice(0, top_n);

    yearSlice.forEach((d,i) => d.rank = i);
  
   console.log('yearSlice: ', yearSlice)

   let x = d3.scaleLinear()
      .domain([0, d3.max(yearSlice, d => d.value)])
      .range([margin.left, width-margin.right-65]);

      

   let y = d3.scaleLinear()
      .domain([top_n, 0])
      .range([height-margin.bottom, margin.top]);

   let xAxis = d3.axisTop()
      .scale(x)
      .ticks(width > 500 ? 5:2)
      .tickSize(-(height-margin.top-margin.bottom))
      .tickFormat(d => d3.format(',')(d));

   svg.append('g')
     .attr('class', 'axis xAxis')
     .attr('transform', `translate(0, ${margin.top})`)
     .call(xAxis)
     .selectAll('.tick line')
     .classed('origin', d => d == 0);

   svg.selectAll('rect.bar')
      .data(yearSlice, d => d.name)
      .enter()
      .append('rect')
      .attr('class', 'bar')
      .attr('x', x(0)+1)
      .attr('width', d => x(d.value)-x(0)-1)
    //  .attr('width', d => getWidth1(d))  
      .attr('y', d => y(d.rank)+5)
      .attr('height', y(1)-y(0)-barPadding)
      .style('fill', d => d.colour);
    
   svg.selectAll('text.label')
      .data(yearSlice, d => d.name)
      .enter()
      .append('text')
      .attr('class', 'label')
      .attr('x', d => x(d.value)-8)
      //.attr('x', d => getX1(d))
      .attr('y', d => y(d.rank)+5+((y(1)-y(0))/2)+1)
      .style('text-anchor', 'end')
      .html(d => d.name);
    
  svg.selectAll('text.valueLabel')
    .data(yearSlice, d => d.name)
    .enter()
    .append('text')
    .attr('class', 'valueLabel')
    .attr('x', d => x(d.value)+5)
    //.attr('x', d => getX2(d))
    .attr('y', d => y(d.rank)+5+((y(1)-y(0))/2)+1)
    .text(d => d3.format(',.0f')(d.lastValue));

  let yearText = svg.append('text')
    .attr('class', 'yearText')
    .attr('x', width-margin.right)
    .attr('y', height-25)
    .style('text-anchor', 'end')
    .style('font-size', '4em')
    .html(~~year1)
    .call(halo, 10);
   
 let ticker = d3.interval(e => {
    console.log("ticker CALL");
    if(pause) {
      console.log("This is pause!");
      return;
    }
    yearSlice = data1.filter(d => d.year == year1 && !isNaN(d.value))
      .sort((a,b) => b.value - a.value)
      .slice(0,top_n);

    yearSlice.forEach((d,i) => d.rank = i);
    console.log('yearSlice: ', year1, yearSlice);

    x.domain([0, d3.max(yearSlice, d => d.value)]); 

   
    svg.select('.xAxis')
      .transition()
      .duration(tickDuration)
      .ease(d3.easeLinear)
      .call(xAxis);
  
     let bars = svg.selectAll('.bar').data(yearSlice, d => d.name);
  
     bars
      .enter()
      .append('rect')
      .attr('class', d => `bar ${d.name.replace(/\s/g,'_')}`)
      .attr('x', x(0)+1)
      .attr( 'width', d => x(d.value)-x(0)-1)
      //.attr( 'width', d => getWidth1(d))
      .attr('y', d => y(top_n+1)+5)
      .attr('height', y(1)-y(0)-barPadding)
      .style('fill', d => d.colour)
      .transition()
        .duration(tickDuration)
        .ease(d3.easeLinear)
        .attr('y', d => y(d.rank)+5);
        
     bars
      .transition()
        .duration(tickDuration)
        .ease(d3.easeLinear)
        .attr('width', d => x(d.value)-x(0)-1)
        //.attr( 'width', d => getWidth1(d))
        .attr('y', d => y(d.rank)+5);
          
     bars
      .exit()
      .transition()
        .duration(tickDuration)
        .ease(d3.easeLinear)
        .attr('width', d => x(d.value)-x(0)-1)
        //.attr( 'width', d => getWidth1(d))
        .attr('y', d => y(top_n+1)+5)
        .remove();

     let labels = svg.selectAll('.label')
        .data(yearSlice, d => d.name);
   
     labels
      .enter()
      .append('text')
      .attr('class', 'label')
      .attr('x', d => x(d.value)-8)
     // .attr('x', d => getX1(d))
      .attr('y', d => y(top_n+1)+5+((y(1)-y(0))/2))
      .style('text-anchor', 'end')
      .html(d => d.name)    
      .transition()
        .duration(tickDuration)
        .ease(d3.easeLinear)
        .attr('y', d => y(d.rank)+5+((y(1)-y(0))/2)+1);
           
  
      labels
        .transition()
        .duration(tickDuration)
          .ease(d3.easeLinear)
          .attr('x', d => x(d.value)-8)
          //.attr('x', d => getX1(d))
          .attr('y', d => y(d.rank)+5+((y(1)-y(0))/2)+1);
   
     labels
        .exit()
        .transition()
          .duration(tickDuration)
          .ease(d3.easeLinear)
          .attr('x', d => x(d.value)-8)
          //.attr('x', d => getX1(d))
          .attr('y', d => y(top_n+1)+5)
          .remove();
       

   
     let valueLabels = svg.selectAll('.valueLabel').data(yearSlice, d => d.name);
  
     valueLabels
        .enter()
        .append('text')
        .attr('class', 'valueLabel')
        .attr('x', d => x(d.value)+5)
        //.attr('x', d => getX2(d))
        .attr('y', d => y(top_n+1)+5)
        .text(d => d3.format(',.0f')(d.lastValue))
        .transition()
          .duration(tickDuration)
          .ease(d3.easeLinear)
          .attr('y', d => y(d.rank)+5+((y(1)-y(0))/2)+1);
          
     valueLabels
        .transition()
          .duration(tickDuration)
          .ease(d3.easeLinear)
          .attr('x', d => x(d.value)+5)
          //.attr('x', d => getX2(d))
          .attr('y', d => y(d.rank)+5+((y(1)-y(0))/2)+1)
          .tween("text", function(d) {
             let i = d3.interpolateRound(d.lastValue, d.value);
             return function(t) {
               this.textContent = d3.format(',')(i(t));
            };
          });
    
   
    valueLabels
      .exit()
      .transition()
        .duration(tickDuration)
        .ease(d3.easeLinear)
        .attr('x', d => x(d.value)+5)
        //.attr('x', d => getX2(d))
        .attr('y', d => y(top_n+1)+5)
        .remove();
  
    yearText.html(~~year1);
   
   if(year1 == 2010) ticker.stop();
   year1 =+year1 + 1;
 },tickDuration);

 var bt = document.getElementById("bt_pause");

  bt.style.backgroundImage = "url(pause2.png)";
  bt.style.backgroundRepeat = "no-repeat";
  bt.style.width="40px";//({ width: 200, height: 200 })
  bt.style.height="40px";
  bt.style.backgroundSize = "40px";

});
  
const halo = function(text, strokeWidth) {
text.select(function() { return this.parentNode.insertBefore(this.cloneNode(true), this); })
  .style('fill', '#ffffff')
   .style( 'stroke','#ffffff')
   .style('stroke-width', strokeWidth)
   .style('stroke-linejoin', 'round')
   .style('opacity', 1);
 
}   



</script>

<div>
<div>
  <table>
      <tr>
          <td></td>
          <td class='explain_text'><div class='explain_text'>
              &nbsp; This view represents the classification and evolution by year of bee populations in Europe.
              <br>&nbsp; <u>Please note</u> : As the amount of data is less for years prior to 1997, this graph may not show realistic end-to-end change over a range beyond 50 years.
              <br>&nbsp; It is however interesting to note the countries that appear are those with the 10 having the highest population of bees.
              <p/>
              </div>
          </td>
          <td></td>
      </tr>
  </table>
  </div>
  <div id="race_chart"></div>
</div>
<input id='bt_pause' class="bt_selector"   type="button">

<style>
  text{
    font-size: 16px;
    font-family: Open Sans, sans-serif;
  }
  text.title{
    font-size: 24px;
    font-weight: 500;
  }
    text.subTitle{
      font-weight: 500;
      fill: #777777;
    }
    text.caption{
      font-weight: 400;
      font-size: 14px;
      fill: #777777;
    }
    text.label{
      font-weight: 600;
    }
  
    text.valueLabel{
     font-weight: 300;
    }
  
    text.yearText{
      font-weight: 700;
      opacity: 0.25;
    }
    .tick text {
      fill: #777777;
    }
    .xAxis .tick:nth-child(2) text {
      text-anchor: start;
    }
    .tick line {
      shape-rendering: CrispEdges;
      stroke: #dddddd;
    }
    .tick line.origin{
      stroke: #aaaaaa;
    }
    path.domain{
      display: none;
    }

    .bt_selector{
      background-image:url(play.png) no-repeat left top;
    }
</style>

