<script>
import {
    onMount
} from "svelte";
//import IndexText from './IndexText.svelte'

import {tsvParse, csvParse, tsvFormat,csvFormat} from 'd3-dsv';
import IndexText from './IndexText.svelte'

	let data = [];
	let columns=[];
	let tidy=[];
	let name = 'column';
	let tidycolumns=[];
	let href ='';
	

	function readthedata(csv){
 		data=csvParse(csv)
		columns = data.columns
    tidydata(data)
	}
	
	function tidydata(data){
	 	var checked=[];	
		document.querySelectorAll('input[name=checkboxes]:checked').forEach(function(d){
			checked.push(d.value)
		});
		
		var notChecked=[];
		document.querySelectorAll('input[name=checkboxes]:not(:checked)').forEach(function(d){
			notChecked.push(d.value)
		});
		
		var tidied=[]
		data.forEach(function(e){
			checked.forEach(function(d){
				var obj = {}
				obj[name]=d
				notChecked.forEach(function(d){
					obj[d]=e[d]
				})
				obj["value"]=e[d]
				tidied.push(obj)
			})	
		})
		href=encodeURI('data:text/csv;charset=utf-8,'+csvFormat(tidied))
		tidy = tsvParse(tsvFormat(tidied))
		tidycolumns = tidy.columns
	}

const root = "https://raw.githubusercontent.com/ONSvisual/census-charts/main/"; //where this app is getting stuff from on GitHub
//charts is simply a list of names of current templates available on the ONSvisual/census-charts GitHub repo. It can be added to. Bad charts could be commented out before they are fixed.

const charts = [ 
    'bar-chart',
    'comet-plot',
    'dot-plot',
    'grouped-bar-chart',
    'heatmap-per-column',
    'heatmap',
    'population-pyramid-with-dropdown-and-interactive-comparison',
    'population-pyramid-with-dropdown',
    'population-pyramid-with-interactive-comparison',
    'range-plot',
    'split-bar-chart',
    'stacked-horizontal-bar-chart',
    'static-population-pyramid-with-comparison',
    'static-population-pyramid'
]

let chart = 'bar-chart' //index in charts == 'bar-chart'

let config = '', //these are the files we'll be fetching from the GitHub Repo as text strings
    js = '',
    csv = '',
    css = '',
    chartType = ''

let inputs = {
    "chartType": "",
    "js": "",
    "css": "",
    "config": "",
    "csv": ""
} //inputs bundles together fetched files from a GitHub Repo as strings in a JSON object. They will also be saved to LocalStorage and downloadable as a .ons file

//SIMPLE FUNCTIONS
const comments = new RegExp('//.*', 'mg') //this regex can strip out commented out lines from .js files
//END OF - SIMPLE FUNCTIONS

let getTemplate = (string) =>
    //localStorage[string] ? inputs = JSON.parse(localStorage[string]) :
    fetch( //getTemlate follows this order of getting: Config_then_JS_then_CSV_then_CSS. It cleans them up as it progresses
        `${root}${string}/config.js`
    )
    .then((res) => res.text())
    .then((config) =>      
        config
        .replace(/([^{]+)/, '') //all this cleaning is because config is not a clean JSON file
        .replace(comments, '\n')
        .replace(/;/g, '')
        .replace(
            'data.csv',
            `${root}${string}/data.csv`
        )
    )
    .then((cleaned) => {
        inputs["config"] = JSON.parse(cleaned)// DO make it into a JSON object!
        fetch(
                `${root}${string}/script.js` //Now we fetch the .js script from the chart template - maybe this should be manipulable by pro users?
            )
            .then((res) => res.text())
            .then((txt) => {
                js = txt.replace(
                    "('#graphic');",
                    "('#graphic');let config=" + JSON.stringify(inputs["config"]) //And next we add the config into the js script
                );
                fetch(
                        `${root}${string}/data.csv` //Now we get the csv string
                    )
                    .then((res) => res.text())
                    .then((txt) => {
                      console.log("GOTTO_1")
                        csv = txt;//And save it to the csv variable
                        fetch(
                                `${root}${string}/chart.css` //finally we get any custom CSS 
                            )
                            .then((res) => res.text())
                            .then((txt) => {
                                css = txt; //and save the custom CSS to the css variable, though it might need to be injected somewhere?
                                inputs.chartType = string;
                                inputs.css = css;
                                inputs.csv = csv;
                                inputs.js = js;
                                localStorage[string] = JSON.stringify(inputs)
                                gotJavaScript(inputs)
                            })
                    })
            })
    })

//getTemplate(chart)

//gotJavaScript checks if the script.js has been found and loaded from the GitHub repo, then hydrates it into the DOM ***IT NEEDS TO UPDATE ALL 4 FILES***

let gotJavaScript = (inputs) => {
  console.log("INPUTS",inputs)
  readthedata(inputs.csv);  
    if (document.getElementById("legend")) document.getElementById("legend").innerHTML = "";
    if (document.getElementById("graphic")) document.getElementById("graphic").innerHTML = "";
    window.scrollTo(0, 0);
    let func = () => {};
    func = Function(inputs["js"]);
    func();
}

//listening for changes in any of these sources
//End of gotJavaScript

onMount(() => getTemplate(chart))

</script>

{#if inputs.js}

<div id="outputHTML">
    <h5 id="accessibleSummary" class="visuallyhidden"> </h5>
    <div aria-hidden="true" id="legend" />
    <div id="graphic" aria-hidden="true" />
    <h5 id="source"> </h5>
   <!--<code><IndexText {inputs}/></code> -->
<IndexText {inputs}/> 
</div>


<div class="half">

<select bind:value={chart} on:change={() => getTemplate(chart)}>{chart}>
  {#each charts as chart}
    <option value={chart}>
      {chart}
    </option>
  {/each}
</select>

{#if columns}
<div class="tablewrapper">
	<table>
		<thead>
			<tr>
			{#each columns as d}
			<th>{d}</th>
			{/each}
			</tr>
		</thead>
		<tbody>
			{#each data as d}
			<tr>
				{#each Object.keys(d) as e}
				<td>{d[e]}</td>
				{/each}
			</tr>
			{/each}
		</tbody>
	</table>
</div>
{/if}

{#each Object.keys(inputs.config) as main, i}
<h1 style:color="red">{main}</h1>
{#each Object.keys(inputs.config[main]) as sub, ii}
{#if sub=="graphic_data_url"}<a href={inputs.config[main][sub]} target="_blank" rel="noreferrer" download="data.csv">
<h3 style:color="darkorange">{sub}</h3>
</a>
{:else}
<h3 style:color="darkorange">{sub}</h3>
{/if}
<textarea
    class="full"
    on:keyup={(e)=> {
    inputs.config[main][sub] = e.target.value
    .replace(/"/g, '')
    .replace(/\n/g, '');
    localStorage[chart] = JSON.stringify(inputs);
    getTemplate(chart)
    }}
    value={JSON.stringify(inputs.config[main][sub]).trim().replace(/"/g,'')}
    />

    {/each}
    {/each}
  </div>
    {/if}

<style>
.full {
    width: 100%;
    padding: 5px;
}
.half, #outputHTML {
  width:50%;
  float:left;
  height:100vh;
  overflow: scroll;
}
:global(body){
  max-width: 100vw;
}

	.tablewrapper{
		height:150px;
		overflow:scroll
	}
	
	thead tr {
    background-color: red;
    color: #ffffff;
    text-align: left;
}
	th, td {
    padding: 5px;
	}
	
	tbody tr {
			border-bottom: 1px solid #dddddd;
	}

	tbody tr:nth-of-type(even) {
			background-color: #f3f3f3;
	}

	tbody tr:last-of-type {
			border-bottom: 2px solid #206095;
	}

  select{
    font-family: inherit;
    font-size: inherit;
    -webkit-padding: 0.4em 0;
    padding: 0.4em;
    margin: 0 0 0.5em 0;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 2px;
}
</style>
