---
layout: laborfolio
title: Ondoj kaj frekvencoj
js:
  - folio-0c
---

<!--
https://ib-lenhardt.com/kb/glossary/signal-modulation
https://www.gaussianwaves.com/2015/11/interpreting-fft-results-obtaining-magnitude-and-phase-information/
-->

<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/14.6.0/math.js" type="text/javascript"></script>
<script src="https://cdn.plot.ly/plotly-1.35.2.min.js"></script>

<div id="func"></div>
<div id="plot1"></div>
<div id="plot2"></div>
<div id="plot3"></div>

<script type="text/javascript">
  // test
  console.log(math.sqrt(-4).toString()) // 2i

  N = 256;
  frekv1 = 10; // 10 Hz
  fazo1 = 0;
  frekv2 = 5; // 5Hz
  fazo2 = Math.PI/7;
  daŭro = 2; // 2s
  R = N/daŭro; // testfrekvenco 128 Hz
  dt = 1/R; // tempintervalo de testoj
  
  sojlo = 1e-12; // ne kalkulu la fazon por magnitudoj proks. 0

  // desegnu sinusfunkcion
  //const signalo = "sin(2*Math.PI*frekv1*t+fazo1) + cos(2*Math.PI*frekv2*t+fazo2)"; 

  let xValues, yValues;
  let magn = [], phi = [];

  try {
        // compile the expression once
        //const expression = document.getElementById('eq').value
        //"f(x) = sin(x) + cos(x/2) - 0.5";        
        //const expr = math.compile(signalo);

        // evaluate the expression repeatedly for different values of x
        xValues = math.range(0, daŭro, dt).toArray()
        yValues = xValues.map(function (t) {
            // expr.evaluate({t: t, frekv1: ...,fazo2: ...})
            //return Math.sin(2*Math.PI*frekv1*t+fazo1) + Math.cos(2*Math.PI*frekv2*t+fazo2);
            return Math.cos(2*Math.PI*frekv1*t+fazo1) + Math.cos(2*Math.PI*frekv2*t+fazo2);
        });

    } catch (err) {
        console.error(err)
        alert(err)
    }

  function draw() {

      // render the plot using plotly
      const trace1 = {
        x: xValues,
        y: yValues,
        type: 'scatter'
      }
      const data = [trace1]
      Plotly.newPlot('plot1', data)
  }


  function draw2() {

    // frekvenc-spektro 0..R, inkremento R/N
    const xValues = math.range(0,R,R/N).toArray();
    // render the plot using plotly
    const trace1 = {
      x: xValues,
      y: magn,
      //y: phi,
      type: 'scatter'
    }
    const data = [trace1]
    Plotly.newPlot('plot2', data)
  }


  function draw3() {

    // frekvenc-spektro 0..R, inkremento R/N
  const xValues = math.range(0,R,R/N).toArray();
    // render the plot using plotly
    const trace1 = {
      x: xValues,
      y: phi,
      //y: phi,
      type: 'scatter'
    }
    const data = [trace1]
    Plotly.newPlot('plot3', data)
  }  

  function fft() {
    const ft = math.fft(yValues);
    //console.log(ft);
    for (const n of ft) {
      const mg = math.sqrt(math.square(n.re) + math.square(n.im));
        magn.push(mg);
        phi.push(mg>sojlo?math.atan2(n.im,n.re):0);
    }
  }
/*
  document.getElementById('func').onsubmit = function (event) {
    event.preventDefault()
    draw()
  }*/

    draw();
    fft();
    draw2();
    draw3();
</script>
