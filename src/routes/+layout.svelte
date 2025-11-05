<script lang="ts">
    import favicon from "$lib/assets/favicon.svg";
    import "../app.css";

    let { children } = $props();
</script>

<svelte:head>
    <link rel="icon" href={favicon} />
    <title>Video Converter</title>
    <meta
        name="description"
        content="A simple and fast online tool to convert WebM videos to MP4 format."
    />
</svelte:head>

{@render children()}
