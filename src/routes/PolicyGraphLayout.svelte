<script>
    import * as d3 from 'd3';
    import {onMount} from "svelte";
    import ScatterPlot from './ScatterPlot.svelte';
    import PolicyGraph from './PolicyGraph.svelte';

    let data = [];
    let selectedHouses =[];
    let property_taxAmount = 0;
    let transfer_taxAmount = 0;

    function handleBrushingChanged(event) {
        selectedHouses = event.detail.detail;
    }

    $:{
        property_taxAmount = d3.sum(selectedHouses, d => d.property_tax)
        transfer_taxAmount = d3.sum(selectedHouses, d => d.transfer_tax)
    }

    $: housingpotential = Math.ceil((property_taxAmount+transfer_taxAmount)/500000)

    onMount(async() => {

        data = await d3.csv("https://raw.githubusercontent.com/usonia09/zipcode-affordability/main/data/filtered_policies_boston_residential_sales.csv", house => ({
            ...house, 

            year: Number(house.year),
            price: Number(house.price),
            property_tax: Number(house.property_tax),
            transfer_tax: Number(house.transfer_tax),
            yearbuilt: Number(house.yearbuilt)

        }))

    })
    
    $: data = data.filter((data) => data.price <= 30000000)

</script>
<div class="container">
    <div class="scatterplot" style="margin: 10px">
        <div class="x-label">
            <div class="y-label">
                <text class="axis-label">Cost of House</text>
                <ScatterPlot on:brushChanged={handleBrushingChanged} data={data}/>
            </div>
            <text> <center>Year Sold</center></text>
        </div>

    </div>
    <div class="policy">
        <h3>Projected Revenue Per Policy Implementation</h3>
        <!-- <PolicyGraph property_taxAmount={property_taxAmount} transfer_taxAmount={transfer_taxAmount}/> -->
            <h4 style="color: #610C63">From Property Tax: $ {property_taxAmount}</h4>
            <h4 style="color: #610C63">From Transfer Fees: $ {transfer_taxAmount}</h4>

        <h3>Number of Homes That Could Be Built </h3>
                <span> {housingpotential}</span>  
        
    </div>
    
</div>


<style>
@import url("$lib/global.css");

.y-label {
    display: flex;
    align-items: center;
}

.axis-label {
    writing-mode: vertical-lr;
    transform: rotate(180deg);
}


.container {
    /* border: 1px solid black; */
    width: 100%;
    display: grid;
    grid-template-columns: 2fr 1fr;
    column-gap: 50px;
    align-items: center;
}

.policy {
    display: flex;
    flex-direction: column;
}


h3 {
    text-align: center;
}
span {
    font-size: 40px;
    font-weight: bold;
    align-self: center;
}

</style>