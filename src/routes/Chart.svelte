<script>
    import P5 from 'p5-svelte';

    // Export this variable so that it can be a prop that gets passed in on 
    // component creation
    export let stockData;

    // Data processing functions go here
    let data = stockData;

    console.log(data["Meta Data"])

    // console.log(Object.keys(data["Time Series (Daily)"][0]))

    let tickerName = data["Meta Data"]["2. Symbol"];
    let stockPriceSeries = data["Time Series (Daily)"];

    // Access values from stock
    console.log(Object.keys(stockPriceSeries)[0])
    console.log(stockPriceSeries[Object.keys(stockPriceSeries)[0]])

    // Get the most recent daily high value of the stock
    const firstPoint = stockPriceSeries[Object.keys(stockPriceSeries)[0]]['2. high']

    console.log(firstPoint)

    const sketch = (p5) => {
      p5.setup = () => {
        p5.createCanvas(500, 300);
      };
  
      p5.draw = () => {
        p5.background(220);
        // p5.line(0, 0, 150, 200);

        p5.stroke('purple');
        p5.strokeWeight(10);
        p5.point(100, firstPoint);
        };
        
    
    };

</script>

<div>
    <h2>Stock data here:</h2>
    <p>You chose: {tickerName}</p>

    <P5 {sketch} />

</div>