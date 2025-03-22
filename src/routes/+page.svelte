<script lang="ts">
    import type { AvailableLanguageTag } from "$lib/paraglide/runtime";
    import { i18n } from "$lib/i18n";
    import { page } from "$app/state";
    import { goto } from "$app/navigation";
    import * as m from "$lib/paraglide/messages.js";
    import { Patterns } from "$lib/patterns";
    import { onMount } from "svelte";

    function switchToLanguage(newLanguage: AvailableLanguageTag) {
        const canonicalPath = i18n.route(page.url.pathname);
        const localisedPath = i18n.resolveRoute(canonicalPath, newLanguage);
        goto(localisedPath);
    }

    let backgroundImageSrc = "";

    let goodScreenshotImageSrc = "";
    let badScreenshotImageSrc = "";

    let titleText = "";
    let goodText = "";
    let badText = "";

    let canvas: HTMLCanvasElement;

    let enable_avator = false;
    let pattern = Patterns.GoodAndBad;

    function handleBackgroundFileChange(event) {
        const file = event.target.files[0];

        if (!file) {
            return;
        }

        const render = new FileReader();
        render.onload = (e) => {
            backgroundImageSrc = e.target?.result;
            draw();
        };

        render.readAsDataURL(file);
    }

    function handleTitleTextInput(event) {
        titleText = event.target.value;

        draw();
    }

    function handleGoodTextInput(event) {
        goodText = event.target.value;

        draw();
    }

    function handleBadTextInput(event) {
        badText = event.target.value;

        draw();
    }

    function drawBoxes(ctx, callback) {
        let loaded = 0;

        const onLoad = () => {
            loaded++;

            if (loaded === 2 && callback) {
                ctx.drawImage(goodBoxImage, 32, 311);
                ctx.drawImage(badBoxImage, 672, 311);

                callback(ctx);
            }
        };

        let goodBoxImage = new Image();
        goodBoxImage.onload = onLoad;
        goodBoxImage.src = "/good.png";

        let badBoxImage = new Image();
        badBoxImage.onload = onLoad;
        badBoxImage.src = "/bad.png";
    }

    function drawText(
        ctx,
        text: string,
        startX: number,
        startY: number,
        fontSizePt: number,
    ) {
        const lineHeight = fontSizePt * 1.5;
        const _startX = startX;
        const _startY = startY + fontSizePt;

        ctx.font = `${fontSizePt}pt Yu Gothic,sans-serif`;
        ctx.fillStyle = "#363636";

        text.split("\n").forEach((line, index) =>
            ctx.fillText(line, _startX, _startY + index * lineHeight),
        );
    }

    function drawTitle(ctx, text: string) {
        drawText(ctx, text, 31, 69, 60);
    }

    function drawGoodText(ctx, text: string) {
        drawText(ctx, text, 118, 336, 36);
    }

    function drawBadText(ctx, text: string) {
        drawText(ctx, text, 758, 336, 36);
    }

    function drawOverlays(ctx) {
        drawTitle(ctx, titleText);
        drawGoodText(ctx, goodText);
        if (pattern === Patterns.GoodAndBad && badText) {
            drawBadText(ctx, badText);
        }
    }

    function draw() {
        if (!canvas) {
            console.warn("invalid canvas");
            return;
        }

        const ctx = canvas.getContext("2d");
        if (!ctx) {
            console.warn("invalid context");
            return;
        }

        if (!backgroundImageSrc) {
            console.info("background image not loaded");

            canvas.width = 1280;
            canvas.height = 791;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            drawBoxes(ctx, drawOverlays);

            return;
        }

        const img = new Image();
        img.onload = () => {
            canvas.width = 1280;
            canvas.height = 791;

            ctx.drawImage(img, 0, 0);

            drawBoxes(ctx, drawOverlays);
        };

        img.src = backgroundImageSrc;
    }

    onMount(() => {
        draw();
    });
</script>

<button
    class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
    on:click={() => switchToLanguage("en")}>English</button
>
<button
    class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
    on:click={() => switchToLanguage("ja")}>日本語</button
>

<h1>Welcome to SvelteKit</h1>
<p>
    Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the
    documentation
</p>

<fieldset class="border p-4 rounded">
    <legend class="">{m.pattern_legend()}</legend>

    <div>
        <div>
            <input
                type="radio"
                id="good_and_bad"
                class="mr-2"
                value={Patterns.GoodAndBad}
                bind:group={pattern}
            />
            <label for="good_and_bad">{m.good_and_bad_example()}</label>
        </div>

        <div>
            <input
                type="radio"
                id="good_only"
                class="mr-2"
                value={Patterns.GoodOnly}
                bind:group={pattern}
            />
            <label for="good_only">{m.good_only_exmaple()}</label>
        </div>

        <div>
            <input
                type="checkbox"
                id="set_avator"
                class="mr-2"
                bind:checked={enable_avator}
            />
            <label for="set_avator">{m.set_avator_label()}</label>
        </div>
    </div>
</fieldset>

<div>
    <label for="choose_background">{m.choose_background_image()}</label>
    <input
        type="file"
        accept="image/*"
        on:change={handleBackgroundFileChange}
        class="mb-4 p-2 border rounded"
        id="choose_background"
    />
</div>

<div>
    <label for="choose_good_screenshot"
        >{m.choose_good_screenshot_image()}</label
    >
    <input
        type="file"
        accept="image/*"
        on:change={handleBackgroundFileChange}
        class="mb-4 p-2 border rounded"
        id="choose_good_screenshot"
    />
</div>

{#if pattern === Patterns.GoodAndBad}
    <div>
        <label for="choose_bad_screenshot"
            >{m.choose_bad_screenshot_image()}</label
        >
        <input
            type="file"
            accept="image/*"
            on:change={handleGoodScreenshotFileChange}
            class="mb-4 p-2 border rounded"
            id="choose_bad_screenshot"
        />
    </div>
{/if}

{#if enable_avator}
    <div>
        <label for="choose_avator">{m.choose_avator_image()}</label>
        <input
            type="file"
            accept="image/*"
            on:change={handleBadScreenshotFileChange}
            class="mb-4 p-2 border rounded"
            id="choose_avator"
        />
    </div>
{/if}

<div>
    <label for="title_text" class="">{m.title_text_label()}</label>
    <textarea
        class="w-full p-2 border rounded"
        placeholder={m.textarea_placeholder({})}
        id="title_text"
        on:input={handleTitleTextInput}
        bind:value={titleText}
    ></textarea>
</div>

<div class="my-4">
    <label for="good_text" class="block font-medium mb-1"
        >{m.good_text_label()}</label
    >
    <input
        type="text"
        placeholder={m.good_text_placeholder()}
        id="good_text"
        on:input={handleGoodTextInput}
        bind:value={goodText}
    />
</div>

{#if pattern === Patterns.GoodAndBad}
    <div class="my-4">
        <label for="bad_text" class="block font-medium mb-1"
            >{m.bad_text_label()}</label
        >
        <input
            type="text"
            placeholder={m.bad_text_placeholder()}
            id="bad_text"
            on:input={handleBadTextInput}
            bind:value={badText}
        />
    </div>
{/if}

<canvas
    class="border border-darkgray bg-[#dbc9ad] rounded w-[1280px] h-[791px]"
    bind:this={canvas}
></canvas>
