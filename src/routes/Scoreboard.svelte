<script>
    export let finishState;
    export let score;

    let message;

    if (finishState=='timeup'){
        message = "Time's up!"
    } else if (finishState=='gameover') {
        message = "Game over!"
    } else if (finishState=='win'){
        message = "You win!"
    }

</script>
<style>
    .scoreblock{
        color: rgb(175, 10, 10);
        text-align: center;
        display:block;
        width:100%;
        height:400px;
        background-color:rgb(0, 0, 0);
        margin-top: 10px;
        margin-bottom: 20px;
        padding: 150px;
    }

</style>

<div class="scoreblock">
    <h1>{message}</h1>
    <h2>Score: {score}</h2>
    <p>press Reset to play again</p>
</div>