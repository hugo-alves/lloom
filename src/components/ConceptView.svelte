<script>
    // Component to render table in notebook
    // Imports
    import SvelteTable from "svelte-table";
    import ConceptSummaryChart from "./ConceptSummaryChart.svelte";
    import { onMount } from "svelte";

    // Properties
    export let data;
    export let el;
    export let filterItems;
    export let selectedTitle;
    export let selectedMetadata;
    export let sliceCol;

    let rows = [];
    let columns = [];
    let rowsMatchAll = [];
    let rowsNonMatchAll = [];
    let rowsMatch = [];
    let remount = true;
    let ignore_cols = ["concept_score_orig"];
    let sortBy = "concept score";
    let sortOrder = -1;
    let domain = [];

    onMount(() => {
        renderTable(data);
    });

    function renderTable(data_in) {
        if (data_in != undefined) {
            let div = document.createElement("div");
            let data_json = JSON.parse(data_in);
            rows = data_json;

            for (var i = 0; i < Object.keys(rows[0]).length; i++) {
                const key = Object.keys(rows[0])[i];
                if (!ignore_cols.includes(key)) {
                    columns.push({
                        key,
                        title: key,
                        value: (v) => v[key],
                        sortable: true,
                        parseHTML: true,
                    });
                }
            }
            remount = false;
            setTimeout(() => (remount = true), 0);
            el.appendChild(div);

            // Adjust filters for highlights
            rowsMatchAll = rows.filter(row => (row["concept_score_orig"] == 1.0 && row["id"] == "All"));
            rowsNonMatchAll = rows.filter(row => (row["concept_score_orig"] < 1.0 && row["id"] == "All"));
            rowsMatch = rows.filter(row => (row["concept_score_orig"] == 1.0));

            // rowsMatch = rows.filter(row => (row["concept_score_orig"] == 1.0));
            // rowsNonMatch = rows.filter(row => (row["concept_score_orig"] == 0.0));

            let fullDomain = rows.map(d => d["id"]);
            domain = [...new Set(fullDomain)];
        }
    }
</script>

<div>
    {#if selectedTitle}
        <h2>{selectedTitle}</h2>
    {/if}
    
    <div class="row">
        {#if selectedMetadata}
            <h3 class="card-title">CONCEPT DETAILS</h3>
            <div class="overview-card">  
                <div class="overview-card-left">
                    {#each Object.entries(selectedMetadata) as [key, value]}
                        <p><b>{key}</b>: {@html value}</p>
                    {/each}
                </div>
                <div class="overview-card-right">
                    <ConceptSummaryChart rows={rowsMatch} {filterItems} {sliceCol} {domain} />
                </div>
            </div>
        {/if}

        <h3 class="card-title">POTENTIAL CONCEPT MATCHES</h3>
        <div class="highlight-card">  
            {#if remount}
                <SvelteTable
                    {columns}
                    rows={rowsMatchAll}
                    filterSelections={filterItems}
                >
                </SvelteTable>
            {/if}
        </div>

        <h3 class="card-title" style="margin-top: 20px">CONCEPT NON-MATCHES</h3>
        <div class="highlight-card">
            {#if remount}
                <SvelteTable
                    {columns}
                    rows={rowsNonMatchAll}
                    filterSelections={filterItems}
                    sortBy={sortBy}
                    sortOrder={sortOrder}
                >
                </SvelteTable>
            {/if}
        </div>
    </div>
</div>

<style>
    :global(td) {
        height: 1px; /* ignored, allows setting height of div within */
    }

    :global(table, th, td) {
        border-bottom: 1px solid #e6e6e6;
        border-collapse: collapse;
    }
    
    :global(thead th) {
        position: sticky;
        top: 0;
        z-index: 1;
        background: #fff;
        padding-block: 10px;
        padding-inline: 10px;
    }

    :global(.score-col) {
        font-size: 14px; 
        text-align: center;
        height: 100%;
    }

    :global(.overview-card) {
        padding-block: 10px;
        padding-inline: 20px;
        border-radius: 10px;
        border: 1px solid #e6e6e6;
        font-size: 16px;
        overflow-y: scroll;
    }

    :global(.overview-card-left) {
        float: left;
        width: 60%; 
        height: 100%; 
    }

    :global(.overview-card-right) {
        float: right;
        width: 40%; 
        height: 100%;
    }

    :global(.highlight-card) {
        padding-inline: 20px;
        border-radius: 10px; 
        border: 1px solid #e6e6e6;
        overflow-y: scroll;
        max-height: 600px; 
    }

    :global(.row) {
        clear: both;
        margin-bottom: 21px;
    }
</style>
