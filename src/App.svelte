<script>
	import { onMount } from 'svelte';
	import { App } from 'octokit';
	import { HSplitPane } from 'svelte-split-pane';
	import { toCSS, toJSON } from './cssjson/cssjson';

	//const root = 'https://raw.githubusercontent.com/ONSvisual/census-charts/main/' //where this app is getting stuff from on GitHub
	//charts is simply a list of names of current templates available on the ONSvisual/census-charts GitHub repo. It can be added to. Bad charts could be commented out before they are fixed.
	const root = 'https://raw.githubusercontent.com/ONSvisual/Charts/main/';
	let menuItems;
	const app = new App({
		appId: 337269,
		privateKey: 'Iv1.15a31c21d3cd0dc5'
	});

	console.log(app);
	//import IndexText from './IndexText.svelte'
	import Head from './Head.svelte';
	import { tsvParse, csvParse, tsvFormat, csvFormat } from 'd3-dsv';
	import * as d3 from 'd3';
	let tree,
		myTree,
		libTree,
		libFiles = {};

	async function getGit() {
		const octokit = await app.getInstallationOctokit(8758367);
		if (octokit) {
			console.log('ockto', octokit);
			let result;
			await fetch(
				'https://api.github.com/repos/ONSvisual/Charts/git/trees/main?recursive=1'
			)
				.then((res) => (result = res.json()))
				.then((result) => {
					console.log('result', result);
					tree = result.tree;
					libTree = tree
						.filter((e) => e.path.startsWith('lib/'))
						.map((e) => e.path);
					localStorage.gitTree = JSON.stringify(tree);
					localStorage.libTree = libTree;
					//  if (!localStorage.libFiles)
					libTree.forEach((path) =>
						fetch(root + path)
							.then((res) => res.text())
							.then((text) => (libFiles[path] = text))
							.then((e) => (localStorage.libFiles = JSON.stringify(libFiles)))
					);
					console.log('Tree', tree);
					menuItems = [
						...new Set(
							tree
								.map((e) => e.path.split('/')[0])
								.filter(
									(e) =>
										(e[0] != '.') & (e != 'vendor') & !e.split('').includes('.')
								)
								.sort()
						)
					];
				});
		}
	}
	getGit(); //get the REPL directory listing as JSON

	function filterTreeAndFetchFiles() {
		let filteredTree = tree.filter((e) => e.path.startsWith(chart + '/'));
		let filesToFetch = filteredTree.map((el) => el.path);
		console.log('filteredTree', filteredTree);
		console.log('filesToFetch', filesToFetch);
		filesToFetch.forEach((e) => {
			fetch(
				//getTemlate follows this order of getting: Config_then_JS_then_CSV_then_CSS. It cleans them up as it progresses
				`https://raw.githubusercontent.com/ONSvisual/all-charts-before/main/${e}`
				//https://raw.githubusercontent.com/ONSvisual/all-charts-before/main/slope/config.json
			)
				.then((res) => res.text())
				.then((fileContent) => console.log(e, fileContent));
		});
	}
	//$: console.log(libTree, libFiles)

	let d3Format = [
		['35.792 => 36', '.0f'],
		['35.792 => 35.8', '.1f'],
		['35.792 => 35.79', '.2f'],
		['0.357 => 36%', '.0%'],
		['0.357=> 35.7%', '.1%'],
		['-0.357 => (35.7%)', '(.1%'],
		['0.357 => +35.7%', '+.1%'],
		['0.357 => +0.4', '+.1f']
	];

	import IndexText from './IndexText.svelte';
	import { each } from 'svelte/internal';

	let data = [];
	let columns = [];
	let tidy = [];
	let name = 'column';
	let tidycolumns = [];
	let href = '';

	function tidyList(theString) {
		var newString = theString
			.toLowerCase()
			.replace(/(^\s*\w|[\.\!\?]\s*\w)/g, function (c) {
				return c.toUpperCase();
			});
		newString = newString.replace(/-/g, ' ');
		return newString;
	}

	function readthedata(csv) {
		data = csvParse(csv);
		columns = data.columns;
		tidydata(data);
	}

	function tidydata(data) {
		var checked = [];
		document
			.querySelectorAll('input[name=checkboxes]:checked')
			.forEach(function (d) {
				checked.push(d.value);
			});

		var notChecked = [];
		document
			.querySelectorAll('input[name=checkboxes]:not(:checked)')
			.forEach(function (d) {
				notChecked.push(d.value);
			});

		var tidied = [];
		data.forEach(function (e) {
			checked.forEach(function (d) {
				var obj = {};
				obj[name] = d;
				notChecked.forEach(function (d) {
					obj[d] = e[d];
				});
				obj['value'] = e[d];
				tidied.push(obj);
			});
		});
		href = encodeURI('data:text/csv;charset=utf-8,' + csvFormat(tidied));
		tidy = tsvParse(tsvFormat(tidied));
		tidycolumns = tidy.columns;
	}

	const charts = [
		'bar-chart-horizontal-sm',
		'bar-chart-horizontal-split',
		'bar-chart-horizontal-stacked',
		'bar-chart-horizontal-stacked-sm',
		'bar-chart-horizontal-stacked-clustered-grouped',
		'bar-chart-horizontal-stacked-clustered',
		'bar-chart-horizontal-stacked-grouped',
		'bar-chart-horizontal',
		'line-chart',
		'comet-plot',
		'dot-plot',
		'grouped-bar-chart',
		'heatmap-per-column',
		'heatmap',
		'range-plot',
		'split-bar-chart',
		'stacked-horizontal-bar-chart',
		'static-population-pyramid-with-comparison',
		'static-population-pyramid',
		'population-pyramid-with-dropdown-and-interactive-comparison',
		'population-pyramid-with-dropdown',
		'population-pyramid-with-interactive-comparison',
		'bar-chart-horizontal-grouped-clustered'
	];

	let chart = 'split-bar-chart'; //index in charts == 'bar-chart'

	let config = '', //these are the files we'll be fetching from the GitHub Repo as text strings
		js = '',
		csv = '',
		css = '',
		chartType = '',
		combined = '';

	let inputs = {
		chartType: '',
		js: '',
		css: '',
		config: '',
		csv: '',
		combined: ''
	}; //inputs bundles together fetched files from a GitHub Repo as strings in a JSON object. They will also be saved to sessionStorage and downloadable as a .ons file

	//this var will be used to store the data from the csv upload box
	let inputData = '';

	let useInputData = false;

  //Tis function will then load the new data

function loadNewData() {
  // Store user data in sessionStorage
  sessionStorage.setItem('userChartData', inputData);
  
  // Create a Blob from the user data
  let blob = new Blob([inputData], { type: 'text/csv' });
  
  // Create a URL from the Blob
  let url = URL.createObjectURL(blob);

  // Update the graphic_data_url to point to the new data
 inputs.config.essential.graphic_data_url = url;
 
 // test code.s
 inputs.csv = inputData;
  
  console.log("New data loaded, useInputData is now:", useInputData);
  console.log("Data stored in session storage:", sessionStorage.getItem('userChartData'));
  console.log("New data URL:", inputs.config.essential.graphic_data_url);

updateChart(inputData)
 

 //potential solution updateChart(inputs);

  //console.log(url)
}



//intercept this request and return the user's data instead 



//This function will update the chart
function updateChart(csvData) {
    // Update the inputs with the new data

    console.log("Updating chart with data", csvData);

    inputs.csv = csvData;

    // Update the combined script
    inputs.combined = inputs.js.replace(
        "('#graphic');",
        "('#graphic');let config=" + JSON.stringify(inputs['config'])
    );

    // Update sessionStorage
    sessionStorage[chart] = JSON.stringify(inputs);

    // Re-render the chart
    gotJavaScript(inputs);
}


	//SIMPLE FUNCTIONS
	const comments = new RegExp('//.*', 'mg'); //this regex can strip out commented out lines from .js files
	//END OF - SIMPLE FUNCTIONS
	let suffix = '?token=GHSAT0AAAAAACCIIGLXPO5PWY6A6YRVGZBWZC4UEHQ';

	// reworked this 
	let getTemplate = (string) => {
  return fetch(`${root}${string}/config.js${suffix}`)
    .then((res) => res.text())
    .then((config) => {
      let cleaned = config
        .replace(/([^{]+)/, '')
        .replace(comments, '\n')
        .replace(/;/g, '')
        .replace('data.csv', `${root}${string}/data.csv`)
        .replace(/d3.*\n/, /"d3.*"\n/)
        .replace('comparison.csv', `${root}${string}/comparison.csv`)
        .replace('comparison-time.csv', `${root}${string}/comparison-time.csv`);

      try {
        inputs['config'] = JSON.parse(cleaned);
      } catch (e) {
        console.log(e, cleaned);
      }

      return fetch(`${root}${string}/script.js`)
        .then((res) => res.text())
        .then((txt) => {
          let js = txt;

          return fetch(`${root}${string}/data.csv${suffix}`)
            .then((res) => res.text())
            .then((txt) => {
              let csv = txt;

              return fetch(`${root}${string}/chart.css${suffix}`)
                .then((res) => res.text())
                .then((txt) => {
                  let css = txt;

                  return fetch(`${root}${string}/index.html${suffix}`)
                    .then((res) => res.text())
                    .then((text) => {
                      inputs['markup'] = text;

                      let cssJSON = toJSON(css);
                      let json = {};
                      Object.keys(cssJSON.children).forEach(
                        (e) => (json[e] = cssJSON.children[e].attributes)
                      );

                      inputs.chartType = string;
                      inputs.css = css;
                      inputs.csv = csv;
                      inputs.js = js;
                      inputs.config.css = json;

                      let combined = js.replace(
                        "('#graphic');",
                        "('#graphic');let config=" + JSON.stringify(inputs['config'])
                      );

                      inputs.combined = combined;
                      sessionStorage[string] = JSON.stringify(inputs);
                      gotJavaScript(inputs);
                    });
                });
            });
        });
    });
};


	//gotJavaScript checks if the script.js has been found and loaded from the GitHub repo, then hydrates it into the DOM ***IT NEEDS TO UPDATE ALL 4 FILES***
	let updateCode = (inputs) => {
  if (document.getElementById('legend'))
    document.getElementById('legend').innerHTML = '';
  if (document.getElementById('graphic'))
    document.getElementById('graphic').innerHTML = '';

  // Check if user data is stored in sessionStorage
  if (sessionStorage.getItem('userChartData')) {
    // Create a Blob from the user data
    let blob = new Blob([sessionStorage.getItem('userChartData')], { type: 'text/csv' });

    // Create a URL from the Blob
    let url = URL.createObjectURL(blob);

    // Update the graphic_data_url in the config to point to the new data
    inputs.config.essential.graphic_data_url = url;
  }

  inputs.combined = '';
  inputs.combined = inputs.js.replace(
    "('graphic');",
    "('graphic');let config=" + JSON.stringify(inputs['config']) // And next we add the config into the js script
  );

  sessionStorage[chart] = JSON.stringify(inputs);
  gotJavaScript(inputs);
};


let gotJavaScript = (inputs) => {
  //console.log("INPUTS",inputs)
  readthedata(inputs.csv);
  if (document.getElementById('select'))
    document.getElementById('select').innerHTML = '';
  if (document.getElementById('nav'))
    document.getElementById('nav').innerHTML = '';
  if (document.getElementById('titles'))
    document.getElementById('titles').innerHTML = '';
  if (document.getElementById('legend'))
    document.getElementById('legend').innerHTML = '';
  let func = () => {};

  if (inputs['combined'].length) func = Function(inputs['combined']);

  func();
};

	//listening for changes in any of these sources
	//End of gotJavaScript

	// this will update the chart data
	// function updateChart(newData) {
	// 	// update data
	// 	chart.data(newData);

	// 	// redraw chart
	// 	chart.draw();
	// }

	let renderCode = () => {
		if (sessionStorage[chart]) {
			inputs = JSON.parse(sessionStorage[chart]);
			gotJavaScript(inputs);
      
		} else {
			getTemplate(chart);
		}
	};
	renderCode();
	onMount(setTimeout(() => renderCode(), 1000));



</script>

{#if inputs.combined}
	<Head headContent={inputs.css} />

	<HSplitPane>
		<left slot="left" class="splitScreen">
			<span id="accessibleSummary" class="visuallyhidden" />
			{#if inputs.config.elements.select}
				<div id="select" />
			{/if}
			{#if inputs.config.elements.nav}
				<div id="nav" />
			{/if}
			{#if inputs.config.elements.titles}
				<div aria-hidden="true" id="titles" />
			{/if}
			{#if inputs.config.elements.legend}
				<div aria-hidden="true" id="legend" />
			{/if}
			<div id="graphic" aria-hidden="true" />
			<span id="source" />
			<IndexText {chart} />
			<div id="graph" />
		</left>

		<right slot="right" class="splitScreen">
			<b color="#0f8243">
				<a
					href="https://docs.google.com/spreadsheets/d/1qlDgJIJCdumMRwLmI1_KF4yAKwtDoHmToYKEwZMq_zU/edit?usp=sharing"
					rel="noreferrer"
					target="_blank"
				>
					Please use this link to share feedback (opens in a new tab)
				</a>
			</b>
			<h3 style:color="slategrey">chart selection</h3>
			<select class="full" bind:value={chart} on:change={renderCode}>
				{#each charts as chart}
					<option value={chart}>{tidyList(chart)}</option>
				{/each}
			</select>

      
			<!-- Adding the csv upload attribute. -->

			<h3>View your own data in this chart</h3>
			<textarea
      bind:value={inputData} rows=10 columns=10 id="csvdata"
			/>
      <button on:click={loadNewData}>Load New Data</button>

			<!--  <h3 style:color="slategrey">full menu</h3>
      {#if menuItems}
      <select class="full" bind:value={chart} on:change={filterTreeAndFetchFiles}>
        {#each menuItems as chart}
          <option value={chart}>{tidyList(chart)}</option>
        {/each}
      </select>
      {/if}-->
			{#if columns && inputs.config.essential}
				<a
					href={inputs.config.essential.graphic_data_url}
					target="_blank"
					rel="noreferrer"
					download="data.csv"
				>
					<h3 style:color="slategrey">data source</h3>
				</a>
				<input
					type="url"
					class="full"
					on:change={(e) => {
						inputs.config.essential.graphic_data_url = e.target.value;
						sessionStorage[chart] = JSON.stringify(inputs);
						updateCode(inputs);
					}}
					value={inputs.config.essential.graphic_data_url}
				/>
				<h3 style:color="slategrey">example data</h3>

				<table>
					<thead>
						<tr>
							{#each columns as d}
								<th>{d}</th>
							{/each}
						</tr>
					</thead>
					<tbody>
						{#each data as d, i}
							{#if i < 1}
								<tr>
									{#each columns as e}
										<td>{d[e]}</td>
									{/each}
								</tr>
							{/if}
						{/each}
					</tbody>
				</table>
			{/if}

			{#each Object.keys(inputs.config) as main, i}
				{#if main != 'elements' && main !== 'chart_build'}
					<h1 style:color="#666">config: {main}</h1>
					{#each Object.keys(inputs.config[main]) as sub, ii}
						{#if sub !== 'graphic_data_url'}
							{#if main == 'css'}
								<h3
									style:color="slategrey"
									on:mouseover={document
										.querySelectorAll(sub)
										.forEach((element) => {
											element.classList.add('highlighted');
										})}
									on:mouseout={document
										.querySelectorAll(sub)
										.forEach((element) => {
											element.classList.remove('highlighted');
										})}
								>
									{sub}
								</h3>
							{:else}
								<h3 style:color="slategrey">{sub}</h3>
							{/if}

							{#if inputs.config.chart_build}
								{#if inputs.config.chart_build[sub] == 'number'}
									<input
										type="number"
										bind:value={inputs.config[main][sub]}
										on:change={(e) => {
											inputs.config[main][sub] = e.target.value;
											sessionStorage[chart] = JSON.stringify(inputs);
											updateCode(inputs);
										}}
									/>
								{:else if inputs.config.chart_build[sub] == 'radio'}
									<div>
										{#each inputs.config.chart_build[sub + '_options'] as option}
											<label for={option}>{option}</label>
											<input
												type="radio"
												name={inputs.config.chart_build[sub]}
												id={option}
												bind:value={option}
												checked={option == inputs.config[main][sub]}
												on:change={(e) => {
													inputs.config[main][sub] = e.target.value;
													sessionStorage[chart] = JSON.stringify(inputs);
													updateCode(inputs);
												}}
											/>
										{/each}
									</div>
								{:else if inputs.config.chart_build[sub] == 'colour'}
									<div>
										{#each inputs.config.chart_build[sub + '_options'] as option, i}
											<label for={option}>{option}:</label>
											<input
												type="color"
												name={inputs.config.chart_build[sub][i]}
												id={option}
												bind:value={option}
												on:change={(e) => {
													inputs.config[main][sub][i] = e.target.value;
													sessionStorage[chart] = JSON.stringify(inputs);
													updateCode(inputs);
												}}
											/>
										{/each}
									</div>
								{:else if inputs.config.chart_build[sub] == 'textarea'}
									<textarea
										class="full"
										type="text"
										name={inputs.config.chart_build[sub]}
										id={inputs.config.chart_build[sub]}
										bind:value={inputs.config[main][sub]}
										on:change={(e) => {
											inputs.config[main][sub] = e.target.value;
											sessionStorage[chart] = JSON.stringify(inputs);
											updateCode(inputs);
										}}
									/>
								{:else if inputs.config.chart_build[sub] == 'dThreeFormat'}
									<select
										class="full"
										bind:value={inputs.config[main][sub]}
										on:change={(e) => {
											inputs.config[main][sub] = e.target.value;
											sessionStorage[chart] = JSON.stringify(inputs);
											updateCode(inputs);
										}}
									>
										{#each d3Format as option, i}
											<option value={option[1]}>
												{option[0]} ({option[1]})
											</option>
										{/each}
									</select>
								{:else if typeof inputs.config[main][sub] == 'object'}
									{#each Object.keys(inputs.config[main][sub]) as subsub}
										{#if typeof inputs.config[main][sub][subsub] == 'object'}
											<h4>Object {subsub}</h4>
											<div>
												{#each Object.keys(inputs.config[main][sub][subsub]) as subsubsub}
													<label for={'field_' + subsubsub} class="label">
														{subsubsub}:
													</label>
													<input
														type="number"
														class="full"
														id="field_{subsubsub}"
														on:change={(e) => {
															inputs.config[main][sub][subsub][subsubsub] =
																e.target.value;
															sessionStorage[chart] = JSON.stringify(inputs);
															updateCode(inputs);
														}}
														value={inputs.config[main][sub][subsub][subsubsub]}
													/>
												{/each}
											</div>
										{:else if main == 'css'}
											<label for={'field_' + subsub} class="label">
												{subsub}:
											</label>
											<input
												type="text"
												class="full"
												id="field_css_{sub}_{subsub}"
												on:change={(e) => {
													inputs.config[main][sub][subsub] = e.target.value;
													sessionStorage[chart] = JSON.stringify(inputs);
													inputs.css = toCSS(inputs.config.css);
													updateCode(inputs); //NEED TO ADD NEW CSS * ** *** ****
												}}
												value={inputs.config[main][sub][subsub]}
											/>
										{/if}
									{/each}
								{:else}
									<textarea
										class="full"
										on:change={(e) => {
											inputs.config[main][sub] = e.target.value
												.replace(/"/g, '')
												.replace(/\n/g, '');
											sessionStorage[chart] = JSON.stringify(inputs);
											updateCode(inputs);
										}}
										value={inputs.config[main][sub]}
									/>
								{/if}
							{:else if typeof inputs.config[main][sub] == 'object'}
								{#each Object.keys(inputs.config[main][sub]) as subsub}
									{#if typeof inputs.config[main][sub][subsub] == 'object'}
										<h4>{subsub}</h4>
										<div>
											{#each Object.keys(inputs.config[main][sub][subsub]) as subsubsub}
												{#if typeof inputs.config[main][sub][subsub][subsubsub] == 'string' && inputs.config[main][sub][subsub][subsubsub][0] == '#'}
													<label for="colour_{subsubsub}">{subsubsub}:</label>

													<input
														type="color"
														id="colour_{subsubsub}"
														on:change={(e) => {
															inputs.config[main][sub][subsub][subsubsub] =
																e.target.value
																	.replace(/"/g, '')
																	.replace(/\n/g, '');
															sessionStorage[chart] = JSON.stringify(inputs);
															updateCode(inputs);
														}}
														value={inputs.config[main][sub][subsub][subsubsub]}
													/>
												{:else if typeof inputs.config[main][sub][subsub][subsubsub] == 'string'}
													<label for="text_{subsubsub}">{subsubsub}:</label>
													<input
														type="number"
														id="text_{subsubsub}"
														on:change={(e) => {
															inputs.config[main][sub][subsub][subsubsub] =
																e.target.value
																	.replace(/"/g, '')
																	.replace(/\n/g, '');
															sessionStorage[chart] = JSON.stringify(inputs);
															updateCode(inputs);
														}}
														value={inputs.config[main][sub][subsub][subsubsub]}
													/>
												{:else if !isNaN(inputs.config[main][sub][subsub][subsubsub])}
													<label for="number_{subsubsub}">{subsubsub}:</label>
													<input
														type="number"
														id="number_{subsubsub}"
														on:change={(e) => {
															inputs.config[main][sub][subsub][subsubsub] =
																e.target.value;
															sessionStorage[chart] = JSON.stringify(inputs);
															updateCode(inputs);
														}}
														value={inputs.config[main][sub][subsub][subsubsub]}
													/>
												{:else}
													<label for={'field_' + subsubsub} class="label">
														{subsub}:
													</label>
													<input
														type="number"
														id="field_{subsubsub}"
														on:change={(e) => {
															inputs.config[main][sub][subsub][subsubsub] =
																e.target.value;
															sessionStorage[chart] = JSON.stringify(inputs);
															updateCode(inputs);
														}}
														value={inputs.config[main][sub][subsub][subsubsub]}
													/>
												{/if}
											{/each}
										</div>
									{:else}
										<label for={'input_' + subsub} class="label">
											{subsub}:
										</label>
										{#if typeof inputs.config[main][sub][subsub] == 'string' && inputs.config[main][sub][subsub][0] == '#'}
											<input
												type="color"
												id="input_{subsub}"
												on:change={(e) => {
													inputs.config[main][sub][subsub] = e.target.value
														.replace(/"/g, '')
														.replace(/\n/g, '');
													sessionStorage[chart] = JSON.stringify(inputs);
													updateCode(inputs);
												}}
												value={inputs.config[main][sub][subsub]}
											/>
										{:else if typeof inputs.config[main][sub][subsub] == 'string'}
											<input
												type="text"
												id="input_{subsub}"
												class="full"
												on:change={(e) => {
													inputs.config[main][sub][subsub] = e.target.value
														.replace(/"/g, '')
														.replace(/\n/g, '');
													sessionStorage[chart] = JSON.stringify(inputs);
													updateCode(inputs);
												}}
												value={inputs.config[main][sub][subsub]}
											/>
										{:else if !isNaN(inputs.config[main][sub][subsub])}
											<input
												type="number"
												on:change={(e) => {
													inputs.config[main][sub][subsub] = e.target.value;
													sessionStorage[chart] = JSON.stringify(inputs);
													updateCode(inputs);
												}}
												value={inputs.config[main][sub][subsub]}
											/>
										{:else}
											<textarea
												class="full"
												on:change={(e) => {
													inputs.config[main][sub][subsub] = e.target.value;
													sessionStorage[chart] = JSON.stringify(inputs);
													updateCode(inputs);
												}}
												value={inputs.config[main][sub][subsub]}
											/>
										{/if}
									{/if}
								{/each}
							{:else}
								<textarea
									class="full"
									on:change={(e) => {
										inputs.config[main][sub] = e.target.value
											.replace(/"/g, '')
											.replace(/\n/g, '');
										sessionStorage[chart] = JSON.stringify(inputs);
										updateCode(inputs);
									}}
									value={inputs.config[main][sub]}
								/>
							{/if}
						{/if}
						<hr />
					{/each}
				{/if}
			{/each}
			{#if inputs.css}
				<textarea
					class="css"
					bind:value={inputs.css}
					on:keyup={(e) => {
						console.log('CSS', cssToJS(inputs.css));
						sessionStorage[chart] = JSON.stringify(inputs);
						updateCode(inputs);
					}}
				/>
			{/if}
		</right>
	</HSplitPane>
{/if}
<div class="highlighted" style="width:0; height:0;" />

<style>
	input[type='number'] {
		width: 50px;
		margin-right: 30px;
	}
	.full {
		width: 95%;
		padding: 5px;
	}

	:global(body) {
		max-width: 100vw;
	}

	.tablewrapper {
		height: 150px;
		overflow: scroll;
		border: 1px solid black;
	}

	thead tr {
		background-color: #206095;
		color: #ffffff;
		text-align: middle;
	}
	th,
	td {
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

	select {
		-webkit-appearance: none;
		-moz-appearance: none;
		appearance: none;
		font-family: inherit;
		font-size: inherit;
		-webkit-padding: 0.4em 0;
		padding: 0.4em;
		margin: 0 0 0.5em 0;
		box-sizing: border-box;
		border: 2px solid #5a5a5a;
		border-radius: 2px;
		font-weight: bolder;
		max-width: 99%;
		background: transparent;
		background-image: url("data:image/svg+xml;utf8,<svg fill='black' height='24' viewBox='8 4 24 24' width='24' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l10 10 10-10z'/><path d='M0 0h24v24H0z' fill='none'/></svg>");
		background-repeat: no-repeat;
		background-position-x: 100%;
		background-position-y: 5px;
	}
	h3 {
		margin-bottom: 5px;
		margin-top: 10px;
	}
	h4 {
		margin-bottom: 5px;
		margin-top: 10px;
		color: #999797;
	}
	hr {
		width: 90%;
		text-align: left;
		margin-left: 5%;
		height: 5px;
		background: rgb(252, 237, 249);
		border: none;
	}
	label {
		margin-left: 15px;
	}
	.css {
		width: 100%;
		height: 300px;
	}
	:global(.left, .right) {
		padding: 20px;
	}
	:global(.highlighted) {
		outline: solid;
	}
</style>
