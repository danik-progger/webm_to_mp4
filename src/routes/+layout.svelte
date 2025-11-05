<script lang="ts">
    import favicon from "$lib/assets/favicon.svg";
    import { onMount } from "svelte";
    import "../app.css";

    async function detectSWUpdate() {
        const reg = await navigator.serviceWorker.ready;

        reg.addEventListener("updatefound", () => {
            const newSW = reg.installing;
            newSW?.addEventListener("statechange", () => {
                if (newSW.state === "installed") {
                    if (confirm("New update avaliable! Reload to update?")) {
                        newSW.postMessage({ type: "SKIP_WAITING" });
                        window.location.reload();
                    }
                }
            });
        });
    }

    onMount(() => {
        detectSWUpdate();
    });

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
