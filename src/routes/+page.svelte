<script lang="ts">
    import { FFmpeg } from "@ffmpeg/ffmpeg";
    import { confetti } from "@neoconfetti/svelte";
    import { onMount } from "svelte";
    import { tweened } from "svelte/motion";
    import { fade } from "svelte/transition";

    type State = "loading" | "loaded" | "conv.start" | "conv.err" | "conv.done";

    let state: State = "loading";
    let err = "";
    let ffmpeg: FFmpeg;
    let progress = tweened(0);

    async function onDrop(e: DragEvent) {
        if (!e.dataTransfer) return;
        if (e.dataTransfer.files.length > 1) err = "Upload 1 file";

        if (e.dataTransfer.files[0].type == "video/webm") {
            err = "";
            const [file] = e.dataTransfer.files;
            const webmVideo = await toWebM(file);
            setTimeout(() => downloadVideo(webmVideo), 1000);
        } else {
            err = "Only webm is supported";
        }
    }

    async function readFile(file: File): Promise<Uint8Array> {
        return new Promise((resolve) => {
            const fileReader = new FileReader();

            fileReader.onload = () => {
                const { result } = fileReader;
                if (result instanceof ArrayBuffer) {
                    resolve(new Uint8Array(result));
                }
            };

            fileReader.onerror = () => {
                err = "could not read file";
            };

            fileReader.readAsArrayBuffer(file);
        });
    }

    async function toWebM(file: File) {
        state = "conv.start";

        const videoAsUintArr = await readFile(file);
        await ffmpeg.writeFile("input.webm", videoAsUintArr);
        await ffmpeg.exec(["-i", "input.webm", "output.mp4"]);
        const data = await ffmpeg.readFile("output.mp4");

        state = "conv.done";
        return data as Uint8Array;
    }

    async function loadFFmpeg() {
        const baseURL =
            "https://cdn.jsdelivr.net/npm/@ffmpeg/core@0.12.10/dist/esm";

        ffmpeg = new FFmpeg();

        ffmpeg.on("progress", (e) => {
            $progress = e.progress * 100;
        });

        await ffmpeg.load({
            coreURL: `${baseURL}/ffmpeg-core.js`,
            wasmURL: `${baseURL}/ffmpeg-core.wasm`,
        });

        state = "loaded";
    }

    onMount(() => loadFFmpeg());

    $: console.log(state);

    function downloadVideo(webmVideo: Uint8Array) {
        const a = document.createElement("a");
        a.href = URL.createObjectURL(
            new Blob([webmVideo.buffer], { type: "video/mp4" }),
        );
        a.download = "video.mp4";
        a.click();
    }
</script>

<h1 class="title">WebM To Mp4 Converter</h1>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
    on:drop|preventDefault={onDrop}
    on:dragover|preventDefault={() => {}}
    data-state={state}
    class="drop"
>
    {#if state == "loading"}
        <p in:fade>Loading FFmpeg</p>
    {:else if state == "loaded"}
        <p in:fade>Drag video here</p>
    {:else if state == "conv.start"}
        <p in:fade>Converting video</p>

        <div class="progress-bar">
            <div class="progress" style:--progress="{$progress}%">
                {$progress.toFixed(0)}%
            </div>
        </div>
    {:else if state == "conv.done"}
        <div use:confetti></div>
        <p in:fade>Done! 🎉</p>
    {/if}

    {#if err}
        <p in:fade class="err">{err}</p>
    {/if}
</div>

<style>
    .title {
        text-align: center;
    }

    .drop {
        width: 600px;
        height: 400px;
        display: grid;
        place-content: center;
        margin-block-start: 2rem;
        border: 10px dashed hsl(220, 10%, 20%);
        border-radius: 0.5rem;

        & p {
            font-size: 2rem;
            text-align: center;

            &.err {
                color: hsl(9, 100%, 64%);
            }
        }
    }

    .progress-bar {
        --bar-clr: hsl(180, 100%, 50%);
        --txt-clr: hsl(0, 0%, 0%);

        width: 300px;
        height: 60px;
        position: relative;

        font-weight: 700;

        background-color: hsl(200, 10%, 14%);

        border-radius: 0.5rem;

        &.progress {
            width: var(--progress);
            height: 100%;

            position: absolute;
            left: 0;

            display: grid;
            place-content: center;

            background: var(--bar-clr);
            color: var(--txt-clr);

            border-radius: 0.5rem;
        }
    }
</style>
