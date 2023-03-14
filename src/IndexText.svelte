<script>
export let chart
//let inputs = JSON.parse(sessionStorage[chart])

let output;
import html2canvas from 'html2canvas'
let global
fetch("/global.css").then(css=>css.text()).then(text=>global=text)
let graph = document.getElementById("graph")

function getPNG(){

let graph = document.getElementById("graph")

function download(canvas) {
  const data = canvas.toDataURL("image/png;base64");
  const donwloadLink = document.querySelector("#download");
  donwloadLink.download = "image.png";
  donwloadLink.href = data;
  donwloadLink.click()
}

html2canvas(document.querySelector("#graph")).then((canvas) => {
  // document.body.appendChild(canvas);
  download(canvas);
});
}

let codeBase = (inputs) =>
'<!DOCTYPE html>'
+'<html lang="en">'
+'<!-- TEMPLATE https://github.com/ONSvisual/census-charts/tree/main/'+inputs.chartType+' -->'
+'<head>'
+'  <link href="https://fonts.googleapis.com/css?family=Open+Sans:400,600,700" rel="stylesheet" type="text/css">'
+'  <title>'+inputs.title+'</title>'
+'  <meta name="template" content="bar-chart">'
+'  <meta name="viewport" content="width=device-width, initial-scale=1" />'
+'  <meta charset="utf-8" />'
+'  <meta name="robots" content="noindex" />'
+'  <meta name="googlebot" content="indexifembedded" />'
+'  <link rel="stylesheet" href="https://www.ons.gov.uk/visualisations/dvc2233/lib/globalStyle.css" />' //NOTE: There is unecessary dublication of this file
+'	<link href="https://fonts.googleapis.com/css?family=Open+Sans:400,600,700" rel="stylesheet" type="text/css">'
+'  <scr'+'ipt src="https://cdn.ons.gov.uk/vendor/d3/6.3.0/d3.min.js" type="text/javascript"></scr'+'ipt>'
+'  <scri'+'pt src="https://cdn.ons.gov.uk/vendor/pym/1.3.2/pym.js" type="text/javascript"></scr'+'ipt>'
+'  <scr'+'ipt src="https://cdnjs.cloudflare.com/ajax/libs/simple-statistics/7.8.3/simple-statistics.min.js" type="text/javascript"></scr'+'ipt>'
+'  <scr'+'ipt src="https://cdnjs.cloudflare.com/ajax/libs/chroma-js/2.1.0/chroma.min.js" type="text/javascript"></scr'+'ipt>'
+'  <scr'+'ipt src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.3/jquery.min.js"></scr'+'ipt>'
+'	<scr'+'ipt src="https://cdnjs.cloudflare.com/ajax/libs/chosen/1.8.7/chosen.jquery.min.js"></scr'+'ipt>'
+'  <style>'+inputs.css + global + '</st'+'yle>'
+'</he'+'ad>'
+'<body>'
+'  <h5 id="accessibleSummary" class="visuallyhidden"></h5>'
+'  <div id="select"></div>'
+'  <div id="nav"></div>'
+'  <div aria-hidden="true" id="titles"></div>'
+'  <div aria-hidden="true" id="legend"></div>'
+'  <div id="graphic" aria-hidden="true">'
+'  <img src="fallback.png" alt='+inputs.config.essential.accessibleSummary+'/>'
+'  </div>'
+'  <h5 id="source">'
+'  </h5>'
+'  <scr'+'ipt>'+inputs.combined+'</scr'+'ipt>'
+'  </bo'+'dy>'
+'  </ht'+'ml>'

function downloadHTML(filename) {

  let text = codeBase(JSON.parse(sessionStorage[chart]));
  console.log("TEXT",text)
  var element = document.createElement('a');
  element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
  element.setAttribute('download', filename);
  element.style.display = 'none';
  document.body.appendChild(element);
  element.click();
  document.body.removeChild(element);
}
function downloadCSV(filename) {

let text = JSON.parse(sessionStorage[chart])["csv"];
console.log("TEXT",text)
var element = document.createElement('a');
element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
element.setAttribute('download', filename);
element.style.display = 'none';
document.body.appendChild(element);
element.click();
document.body.removeChild(element);
}

</script>

<br><br><br>
<button class="btn" on:click={()=>downloadHTML("index.html")}>download HTML</button>
<button class="btn" on:click={()=>downloadCSV("data.csv")}>download CSV</button>
<button class="btn" on:click={()=>getPNG()}>download PNG</button>

<a id="download" style="color:white" >image</a>

<style>
.btn {
    font-family: open sans,Helvetica,Arial,sans-serif;
    font-weight: 400;
    font-size: 14px;
    display: inline-block;
    width: auto;
    cursor: pointer;
    padding: 6px 16px 10px;
    border: 0;
    text-align: center;
    transition: background-color .25s ease-out;
    text-decoration: none;
    line-height: 24px;
    background-color: #0f8243;
    color: #fff;
    margin:10px;
}
</style>