<script>
    export let headContent
</script>

<svelte:head>
{#if headContent}
{@html '<st'+'yle>' +  headContent +  '</st'+'yle>'}
{/if}
</svelte:head>