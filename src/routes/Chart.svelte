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

    // Get the most recent daily high value of the stock
    const firstPoint = stockPriceSeries[Object.keys(stockPriceSeries)[0]]['2. high']
    console.log("First day high:" + " $" + firstPoint)

    // Convet obj to array for easy iteration in draw function
    let stockArray = Object.entries(stockPriceSeries);

    let stockParticles = [];

    const sketch = (p5) => {
      let height = 300;
      let width = 500;

      p5.setup = () => {
        p5.createCanvas(width, height);
        p5.background(220);

        let xPos = 0;
        // maybe a switch could control this later:
        let speed = 1
 
        for (let [date, prices] of stockArray){
            let yPos = prices['2. high'];
            const s = new Particle(xPos,yPos, speed);
            stockParticles.push(s);
            xPos += 5;
        } 

      };  

      p5.draw = () => {
        p5.background(220);

        for (const p of stockParticles) {
            p.render();
        }
        for (const p of stockParticles) {
            p.bounce(50);
        }
      };    

      class Particle {
        constructor(x, y, z) {
            this.pos = p5.createVector(x, y);
            this.displace = 0;
            this.vec = z;
        }
        render() {
            p5.point(this.pos.x, this.pos.y);
            p5.stroke('purple');
            p5.strokeWeight(3);            
        }
        bounce(range) {
            // let vec = p5.createVector(-1, -1)
            // this.pos = this.pos.add(vec);
            
            this.pos.y -= this.vec;
            this.pos.x -= 1;

            this.displace += this.vec;

            // Switch displace vector
            if (this.displace > range) {
                this.vec *= -1;
            } else if (this.displace < -range) {
                this.vec *= -1;
            };

            // Reset at the bottom
            if (this.pos.y < 0) {
                this.pos.y = height;
            }
            if (this.pos.x < 0) {
                this.pos.x = width;
            }
        }
      };
    };

    

</script>

<div>
    <h2>Stock data here:</h2>
    <p>You chose: {tickerName}</p>

    <P5 {sketch} />

</div>