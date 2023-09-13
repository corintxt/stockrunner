<script>
import key from "$lib/local.js"
import Jump from "./Jump.svelte";

    const title = "Stockrunner";

    const tickers = ["AAPL","AMZN","GOOGL","META","TSLA"];

    let display = { switchOn: false };

    function toggle() {
		display.switchOn = !display.switchOn;
	}

    let buttonMsg = "";
    // Use $ label to make this a reactive declaration bound to display.switchOn
    $: if (!display.switchOn) {
        buttonMsg = "Start game";
    } else {
        buttonMsg = "Reset game";
    }

    // Make an async GET request to load stocks
    async function getStockData(symbol) {
        console.log("Fetching data...");
        const res = await fetch(`https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=${symbol}&apikey=${key}`);
        const data = await res.json();

        if (res.ok) {
            console.log("Stock data received.")
			return data;
		} else {
			throw new Error(data);
		}
	}

    // First we initiate the variable for scope, but don't call the getStockData function immediately
	let promise;

	function fetchData(symbol) {
		promise = getStockData(symbol);
	}

    let direction = 'none';
    let gameSpeed = 1;
    let maxSpeed = 3;

    // Handle key press
    function onKeyDown(e) {
		 switch(e.keyCode) {
			 case 38:
                // direction = 'up'
				 break;
			 case 40:
                // direction = 'down'
				 break;
             case 32:
                direction = 'jump'
				 break;
			 case 37:
                direction = 'left'
                if (gameSpeed > -3){
                    gameSpeed -= 1;
                }
                break;
			 case 39:
				direction = 'right'
                if (gameSpeed < maxSpeed){
                    gameSpeed += 1;
                }
				 break;
		 }
         
	}
    function keyClear() {
        direction = 'none';
    }

</script>

<div class="container mt-5">

    <h1>{ title }</h1>

    <p>Pick a stock to start:</p>

    {#each tickers as ticker}
        <button type="button" class="btn btn-secondary btn-space" on:click="{ () => fetchData(ticker) }">{ticker}</button>
    {/each}

    <hr>

    <div>
        <!-- TODO: Conditional logic to make each button do the right thing at right time -->
        <button class="btn btn-dark" type="button" on:click="{ toggle }">{ buttonMsg }</button>
    </div>

    <hr>

    {#if display.switchOn}
        <div>
            <!-- Load the table only after response from .fetch() request -->
            {#await promise}
                <p>Waiting for response...</p>
            {:then stockData}
                <Jump stockData={stockData} direction={direction} speed={gameSpeed} gameState={true} />
                <!-- This will become a Chart with chartData -->
            {:catch error}
                <p style="color: red">{error.message}</p>
            {/await}
        </div>
    {:else} 
        <div>
            GAME DISPLAY <strong>OFF</strong>
        </div>
    {/if}   
</div>


<!-- STYLING -->

<style>
    :global(body) {
        color: #fff779;
        background-color: #313131;
    }

    h1 {
        font-size: 4.5rem;
    }

    * {
        font-family: M PLUS Code Latin;
    }

    .btn-space {
    margin-right: 5px;
}
</style>

<!-- // Prevent default behavior of arrow keys -->
<svelte:window on:keydown|preventDefault={onKeyDown} on:keyup|preventDefault={keyClear} />

<svelte:head>
  <link href="https://fonts.googleapis.com/css?family=M+PLUS+Code+Latin" rel="stylesheet">
</svelte:head>
