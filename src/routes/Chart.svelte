<script>
    import P5 from 'p5-svelte';

    // Export this variable so that it can be a prop that gets passed in on 
    // component creation
    export let stockData;

    // Data processing functions go here
    let data = stockData;
    console.log(data["Meta Data"])
    // console.log(Object.keys(data["Time Series (Daily)"][0]))

    const sketch = (p5) => {
      p5.setup = () => {
        p5.createCanvas(500, 300);
      };
  
      p5.draw = () => {
        p5.background(220);
        p5.line(0, 0, 150, 200);
        };
    };

</script>

<div>
    <h2>Stock data here:</h2>

    <P5 {sketch} />

</div>