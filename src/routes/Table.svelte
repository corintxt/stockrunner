<script>
    // Export this variable so that it can be a prop that gets passed in on 
    // component creation
    export let tableData;

    const rowNames = [
        "Lobbyist Name",
        "Client Name",
        "Client Industry"
    ];

</script>

<table class="table">
    <thead>
        <tr>
          {#each rowNames as name}
          <th>{name}</th>
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each tableData as row, i}
        <tr>
            <td>{row.lobbyist_name}</td>
            <td>{row.client_name}</td>
            <td>{row.client_industry}</td>
        </tr>
        {/each}
      </tbody>
</table>