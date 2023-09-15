<script>
    import P5 from 'p5-svelte';
    import Scoreboard from "./Scoreboard.svelte";

    // Export this variable so that it can be a prop that gets passed in on 
    // component creation
    export let stockData;
    export let direction;
    export let gameState;
    export let speed;

    // The stockData object is a prop passed in to <Jump>
    let data = stockData;

    console.log(data["Meta Data"])
    const tickerName = data["Meta Data"]["2. Symbol"];
    const stockPriceSeries = data["Time Series (Daily)"];
    const firstDate = Object.keys(stockPriceSeries)[99];
    const lastDate = Object.keys(stockPriceSeries)[0];

    // Convet obj to array for easy iteration in draw function
    const stockArray = Object.entries(stockPriceSeries);
    const prices = stockArray.map(a => parseFloat(a[1]['2. high']));
    const priceMin = Math.min(...prices);
    const priceMax = Math.max(...prices);
    const firstPoint = prices[0]
    console.log("First day high:" + " $" + firstPoint)

    // Define outside of p5 scoped variables for game objects
    let time = 60;
    let lives = 3;
    let score = 0;
    let collision = false;
    let kill = false;
    let stockParticles = [];
    let stockSpeed = 1;
    const bounceRange = 30;
    let player;
    let killer;
    let finishState = 'play';
    let killGraphic = false;

    /// P5 SKETCH STARTS HERE.
    const sketch = (p5) => {
      const height = 400;
      const width = 1000;
      const ground = bounceRange; // Defines how far from bottom ground is
      const size = 15; // Size of player
      const g = 1; // Gravity

      // Calculate price range + translate to a height
      const priceRange = priceMax - priceMin;
      const scalar = .9;
      const rangeFix = (height*scalar)/priceRange;

      function calcHeight(price){
        return (((priceMax - price) * rangeFix) + bounceRange);
      }       

      p5.setup = () => {
        p5.createCanvas(width, height);
        p5.background(220);

        // Create a player
        player = new Player(10, height/2, '#0394fc', size);

        // Create stock particles
        let xPos = 0; 
        for (let [date, prices] of stockArray){
            let yPos = calcHeight(prices['2. high']);
            // let yPos = height - (((priceMax - prices['2. high']) * rangeFix)+ bounceRange);
            const s = new Particle(xPos,yPos, stockSpeed);
            stockParticles.push(s);
            xPos += width/100; //currently there are 100 particles, as 100 stock details
        }

        // Create a killer (eventually could make multiple)
        killer = new Killer(width, height/2, 'red', 15, 2);

      };

      // GAME DRAW LOOP
      p5.draw = () => {
        p5.background(220);

        // // Make price marks
        p5.strokeWeight(0);  
        p5.textSize(15);
        p5.text(`High: $${priceMax}`, 5, calcHeight(priceMax));
        p5.text(`Low: $${priceMin}`, 5, calcHeight(priceMin));

        // Draw ground line
        p5.strokeWeight(5);  
        p5.stroke('grey');
        let gy = height - ground;
        p5.line(0, gy, width, gy);

        // Update all game sprites
        updatePlayer();
        updateParticles();
        updateKiller();
        checkKillScreen(killGraphic);

        // Check for collisions, kills, lives
        if (collision) {
            p5.background('#cff08d');
            setTimeout(collisionReset, 5);
            score += 1;
            // setCollision(true);
        }
        if (kill) {
            kill = false;
            lives--;
            killGraphic = true;
            setTimeout(()=>{
                killGraphic = false;
            },300)
        }
        if (lives==0) {
            setFinishState('gameover')
        };
        if (score==99){
            setFinishState('win')
        }
      }; // End draw loop

      // PARTICLE CLASS
      class Particle {
        constructor(x, y, z) {
            this.pos = p5.createVector(x, y);
            this.vec = z;
            this.displace = 0;
            this.speed = 1;
        }
        render() {
            p5.stroke('#ad3af0');
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
                collision = true;
                const hit = stockParticles.indexOf(this);
                stockParticles.splice(hit, 1);
            };
        }
      };

      // PLAYER CLASS
      class Player {
        constructor(x, y, color='white', size) {
            this.pos = p5.createVector(x, y);
            this.speed = 1;
            this.color=color;
            this.size=size;
            this.vy = 0;
            this.jump = 10;
        }
        render() {
            p5.stroke(this.color);
            p5.strokeWeight(10);
            p5.fill(this.color);
            p5.ellipse(this.pos.x, this.pos.y, this.size, this.size);
        }               
        adjustSpeed(s) {
            this.speed = s;
        }
        move() {
            this.pos.x += this.speed;
            // Reset on edges of X axis
            if (this.pos.x > width) {
                this.pos.x = 1;
            } else if (this.pos.x < 0) {
                this.pos.x = width-1;
            }
            // Can't pass through floor/ceiling on Y axis
            if (this.pos.y < 0) {
                this.pos.y = 3;
            } else if (this.pos.y > height) {
                this.pos.y = height - 3;
            }
            // // Direction controls
            // if (direction == 'left'){
            //     this.pos.y -= this.speed;
            // } else if (direction == 'right'){
            //     this.pos.y += this.speed;
            // }
        }
        gravity() {
            // Adapted from: https://editor.p5js.org/tnishida/sketches/Wv_-BBBaA
            this.pos.y += this.vy;
            if(this.pos.y < height - ground - size / 2){ // in the air
                this.vy += g;
            }
            else{
                this.vy = 0;
                this.pos.y = height - ground - size / 2;
            }
        }
        hop() {
            this.vy = -this.jump;
        }
        
      };

      // KILLER CLASS
      class Killer {
        constructor(x, y, color='red', size, speed) {
            this.pos = p5.createVector(x, y);
            this.speed = speed;
            this.color=color;
            this.size=size;
        }
        render() {
            p5.stroke(this.color);
            p5.strokeWeight(5);
            p5.fill('black');
            p5.triangle(this.pos.x, this.pos.y, this.pos.x + this.size, this.pos.y + this.size, this.pos.x + this.size, this.pos.y - this.size);
        }
        sweep() {
            this.pos.x -= this.speed;
            // Reset on edges of X axis
            if (this.pos.x < 0) {
                this.pos.x = width;
                // Come in at random height and faster
                this.pos.y = getRandomHeight(10, height-10)
                if (this.speed < 8){
                    this.speed += .5;
                }
            }
        }
        killCheck() {
            if (p5.dist(this.pos.x, this.pos.y, player.pos.x, player.pos.y) < 25) {
                kill = true;
                console.log("Kill: true")
                this.pos.x = width;
                this.pos.y = getRandomHeight(10, height-10);
            };
        }
      }               

      // FUNCTIONS FOR DRAW LOOP
      function updateParticles() {
        for (const p of stockParticles) {
            p.render();
            p.collisionCheck();
            p.bounce(bounceRange);
        }
      };

      function updatePlayer() {
        player.render();
        player.adjustSpeed(speed);
        player.move();
        player.gravity();
        if (direction == 'jump'){
            player.hop();   
        }
      };

      function updateKiller() {
        killer.render();
        killer.killCheck();
        killer.sweep();
      };
      function checkKillScreen(killSwitch){
        if (killSwitch){
            p5.background('red');
            p5.strokeWeight(0);  
            p5.textSize(100);
            p5.text("KILL", width/2-50, height/2); 
        };
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
        if (finishState!='play'){
             gameState = false;
        }
    }

    function collisionReset(){
        collision = false;
        p5.background(220);
    }

    // Function to make random height (for killer)
    function getRandomHeight(min, max){
        return Math.random() * (max - min) + min;
    };

    // Is there a better way to do this with reactivity?
    setInterval(checkGameState, 500);

}; //END P5 SKETCH

    // Game on/off control – could become part of prop passed to sketch component.
    function toggleGameState() {
        gameState = !gameState; 
    }

    function setFinishState(state){
        finishState = state;
    }

    function countTime() {
        if (time > 0 && finishState=='play'){ 
            time-=1;
        } else if (time == 0){
            // console.log("Time's up!")
            setFinishState("timeup")
        }
    };
    // Ideally we can stop this when game ends!
    setInterval(countTime, 1000);

</script>

<style>
    #score {
        color: #25ffa3;
    }
    #time {
        color: #25afff;
    }
    #lives {
        color: #ff79be;
    }
</style>

<div>
    <h2>Level data</h2>
    <div class="row">
        <div class="col"><b>{tickerName} stocks:</b> {firstDate} — {lastDate}</div>
        <div class="col">
            <div class="col" id="score">Score: {score}</div>
            <div class="col" id="lives">Lives: {lives}</div>
            <div class="col" id="time">Time: {time}</div>
        </div>
    </div>
    
    {#if finishState=='play'}
        <P5 {sketch} />
    {:else}
        <Scoreboard {finishState} {score}/>
    {/if}
    <div>
        {#if gameState}
        <button type="button" class="btn btn-warning" on:click="{ toggleGameState }">Pause</button>
        {:else}
        <button type="button" class="btn btn-success" on:click="{ toggleGameState }">Play</button>
        {/if}

        <label>
            Speed
            <input type="range" bind:value={speed} min="-3" max="3" step="0.5" />
            {speed}
        </label>
        <span class="text-right">
            |  <em>Left/right arrows adjust speed, spacebar to boost</em>
        </span>
    </div>
</div>