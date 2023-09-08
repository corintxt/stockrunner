<script>
import key from "$lib/local.js"
import Chart from "./Chart.svelte";

    const title = "Stockrunner";

    const tickers = ["AAPL","AMZN","GOOGL","META","TSLA"];

    let display = { switchOn: false };

    function toggle() {
		display.switchOn = !display.switchOn;
	}

    let buttonMsg = "Turn display on";
    // Use $ label to make this a reactive declaration bound to display.switchOn
    $: if (!display.switchOn) {
        buttonMsg = "Start game";
    } else {
        buttonMsg = "Hide game";
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
    // Handle key press
    function onKeyDown(e) {
		 switch(e.keyCode) {
			 case 38:
                direction = 'up'
				 break;
			 case 40:
                direction = 'down'
				 break;
			 case 37:
				//  left -= 50;
				 break;
			 case 39:
				//  left += 50;
				 break;
		 }
	}

</script>

<div class="container mt-5">

    <h1>{ title }</h1>

    <p>Pick a stock:</p>

    {#each tickers as ticker}
        <button type="button" class="btn btn-secondary" on:click="{ () => fetchData(ticker) }">{ticker}</button>
    {/each}

    <hr>

    <div>
        <!-- TODO: Conditional logic to make each button do the right thing at right time -->
        <button class="btn btn-primary" type="button" on:click="{ toggle }">{ buttonMsg }</button>

    </div>

    <hr>

    {#if display.switchOn}
        <div>
            GAME DISPLAY <strong>ON</strong>
            <!-- Load the table only after response from .fetch() request -->
            {#await promise}
                <p>Waiting for response...</p>
            {:then stockData}
                <Chart stockData={stockData} direction={direction} gameState={true} />
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
    h1 {
        color: rgb(22, 22, 175);
    }
</style>

<!-- // Prevent default behavior of arrow keys -->
<svelte:window on:keydown|preventDefault={onKeyDown} />