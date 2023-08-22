<script>
import Chart from "./Chart.svelte";
import Table from "./Table.svelte";

    const title = "Stockrunner";

    let display = { switchOn: false };

    function toggle() {
		display.switchOn = !display.switchOn;
	}

    let buttonMsg = "Turn display on";
    // Use $ label to make this a reactive declaration bound to display.switchOn
    $: if (!display.switchOn) {
        buttonMsg = "Get started";
    } else {
        buttonMsg = "Hide table";
    }

    // Make an async GET request to NYC Open Data
    async function getLobbyistData() {
        console.log("Fetching data...");
        const res = await fetch('https://data.cityofnewyork.us/resource/fmf3-knd8.json');
        const data = await res.json();

        if (res.ok) {
            console.log(data[0])
			return data;
		} else {
			throw new Error(data);
		}
	}

    // First we initiate the variable for scope, but don't call the getLobbyist function
	let promise;

	function fetchData() {
		promise = getLobbyistData();
	}

</script>

<h1>{ title }</h1>

<div>
    <!-- TODO: Conditional logic to make each button do the right thing at right time -->
    <button on:click="{ toggle }">{ buttonMsg }</button>
    <button on:click="{ fetchData }">Fetch data</button>
</div>

<Chart />

<hr>

{#if display.switchOn}
    <div>
        DISPLAY <strong>ON</strong>
        <!-- Load the table only after response from .fetch() request -->
        {#await promise}
            <p>Waiting for response...</p>
        {:then tableData}
            <Table {tableData} />
        {:catch error}
            <p style="color: red">{error.message}</p>
        {/await}
    </div>
{:else} 
    <div>
        DISPLAY <strong>OFF</strong>
    </div>
{/if}



<!-- STYLING -->

<style>
    h1 {
        color: rgb(22, 22, 175);
    }
</style>