<script>
    import P5 from 'p5-svelte';

    // Export this variable so that it can be a prop that gets passed in on 
    // component creation
    export let stockData;
    export let direction;
    export let gameState;

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

    // Define variables for game objects
    let collision = false;
    let speed = 1;

    let stockParticles = [];
    let player;

    /// P5 SKETCH STARTS HERE.
    const sketch = (p5) => {
      let height = 300;
      let width = 500;

      p5.setup = () => {
        p5.createCanvas(width, height);
        p5.background(220);

        // Create a player
        player = new Player(width/2, height/2, 'blue');

        // Create stock particles
        let xPos = 0; 
        for (let [date, prices] of stockArray){
            let yPos = prices['2. high'];
            const s = new Particle(xPos,yPos, speed);
            stockParticles.push(s);
            xPos += 5; //currently there are 100 particles & 500px canvas
        }
      };

      // GAME DRAW LOOP
      p5.draw = () => {
        p5.background(220);

        updatePlayer();
        updateParticles();

        if (collision) {
            p5.background('#EC4646');
            setTimeout(collisionReset, 5);
            // gameState = false;
            // setCollision(true);
        }

      };    

      // PARTICLE AND PLAYER CLASSES
      class Particle {
        constructor(x, y, z) {
            this.pos = p5.createVector(x, y);
            this.vec = z;
            this.displace = 0;
            this.speed = 1;
        }
        render() {
            p5.stroke('purple');
            p5.strokeWeight(3);      
            p5.point(this.pos.x, this.pos.y);
      
        }
        bounce(range) {            
            this.pos.y -= this.vec;
            this.pos.x -= this.speed;

            this.displace += this.vec;

            // Switch displacement vector at limit of range of motion
            // (multiplying by -1 switches positive/negative)
            if (this.displace > range) {
                this.vec *= -1;
            } else if (this.displace < -range) {
                this.vec *= -1;
            };

            // Reset at the edges of screen
            if (this.pos.y < 0) {
                this.pos.y = height;
            }
            if (this.pos.x < 0) {
                this.pos.x = width;
            }
        }
        collisionCheck() {
            if (p5.dist(this.pos.x, this.pos.y, player.pos.x, player.pos.y) < 10) {
                // gameState = false;
                collision = true;
            };
        }
      };

      class Player {
        constructor(x, y, color='white') {
            this.pos = p5.createVector(x, y);
            this.speed = 1;
            this.color=color;
            // this.vel = p5.createVector(0, 0);
            // this.acc = p5.createVector(0, 0);

        }
        render() {
            p5.stroke(this.color);
            p5.strokeWeight(10);  
            p5.point(this.pos.x, this.pos.y);
        }
        move() {
            this.pos.x += this.speed;
            // Reset on edges
            if (this.pos.x > width) {
                this.pos.x = 0;
            }
            // Can't pass through floor/ceiling
            if (this.pos.y < 0) {
                this.pos.y = 3;
            } else if (this.pos.y > height) {
                this.pos.y = height - 3;
            }
            // Direction controls
            if (direction == 'up'){
                this.pos.y -= this.speed;
            } else if (direction == 'down'){
                this.pos.y += this.speed;
            }
        // }
        // accelerate() {
        //     if (direction == 'up'){
        //         this.vel.y -= this.speed;
           
        //     }
        }
        adjustSpeed(s) {
            this.speed = s;
        }
        
      }

      // FUNCTIONS FOR DRAW LOOP
      function updateParticles() {
        for (const p of stockParticles) {
            p.render();
            p.collisionCheck();
            p.bounce(50);
        }
      };

      function updatePlayer() {
        player.render();
        player.adjustSpeed(speed);
        player.move();
      };

      // GAME STATE AND CONTROL FUNCTIONS
      function playGame() {
        // console.log("Game is running");
        p5.loop();
      }

      function pauseGame() {
        // console.log("Game is paused");
        p5.noLoop();
      }

      function checkGameState() {
        if (gameState) {
            playGame();
        } else {
            pauseGame();
        }
    }

    function collisionReset(){
        console.log("Collision reset");
        collision = false;
        p5.background(220);
    }

    // Is there a better way to do this with reactivity?
    setInterval(checkGameState, 500);

}; //END P5 SKETCH

    // Game on/off control – could become part of prop passed to sketch component.
    function toggleGameState() {
        gameState = !gameState; 
    }
</script>

<style>
    canvas {
        border: 1px solid black;
    }
</style>

<div>
    <h2>Stock data:</h2>
    <p>You chose <b>{tickerName}</b></p>

    <P5 {sketch} />
    
    {#if gameState}
    <button type="button" class="btn btn-warning" on:click="{ toggleGameState }">Pause</button>
    {:else}
    <button type="button" class="btn btn-success" on:click="{ toggleGameState }">Play</button>
    {/if}

    <label>
        Speed
        <input type="range" bind:value={speed} min="0.5" max="2.5" step="0.5" />
        {speed*2}
    </label>

    <h4>Direction: {direction}</h4>

</div>

<!-- <svelte:window on:keydown|preventDefault={onKeyDown} /> -->