<script>
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    async function changeHandler(event) {
        const file = event.target.files[0];
        if (!file) {
            return;
        }
        const reader = new FileReader();
        reader.onload = (event) => {
            const userCsvDataUrl = event.target.result;
            dispatch('fileuploaded', userCsvDataUrl);
        };
        reader.readAsDataURL(file);
    }
</script>

<input type="file" accept=".csv" on:change="{changeHandler}" />