<script lang="ts">
    import type { AvailableLanguageTag } from "$lib/paraglide/runtime";
    import { i18n } from "$lib/i18n";
    import { page } from "$app/state";
    import { goto, preloadCode } from "$app/navigation";
    import * as m from "$lib/paraglide/messages.js";
    import { Patterns } from "$lib/patterns";
    import { onMount } from "svelte";
    import FontList, { fonts } from "../components/ui/FontList.svelte";

    function switchToLanguage(newLanguage: AvailableLanguageTag) {
        const canonicalPath = i18n.route(page.url.pathname);
        const localisedPath = i18n.resolveRoute(canonicalPath, newLanguage);
        goto(localisedPath);
    }

    let backgroundImageSrc = "";
    let goodScreenshotImageSrc = "";
    let badScreenshotImageSrc = "";
    let avatorImageSrc = "";

    let titleText = "";
    const titleFontSizePt = 60;
    let selectedTitleFontIndex = 0;
    let selectedTitleWeightIndex = 7;

    let goodText = "";
    const goodFontSizePt = 36;
    let selectedGoodFontIndex = 1;
    let selectedGoodWeightIndex = 5;

    let badText = "";
    const badFontSizePt = 36;
    let selectedBadFontIndex = 1;
    let selectedBadWeightIndex = 5;

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

    function handleGoodScreenshotFileChange(event) {
        const file = event.target.files[0];

        if (!file) {
            return;
        }

        const render = new FileReader();
        render.onload = (e) => {
            goodScreenshotImageSrc = e.target?.result;
            draw();
        };

        render.readAsDataURL(file);
    }

    function handleBadScreenshotFileChange(event) {
        const file = event.target.files[0];

        if (!file) {
            return;
        }

        const render = new FileReader();
        render.onload = (e) => {
            badScreenshotImageSrc = e.target?.result;
            draw();
        };

        render.readAsDataURL(file);
    }

    function handleAvatorFileChange(event) {
        const file = event.target.files[0];

        if (!file) {
            return;
        }

        const render = new FileReader();
        render.onload = (e) => {
            avatorImageSrc = e.target?.result;
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

    function handlePatternGoodAndBadChange(event) {
        pattern = Patterns.GoodAndBad;

        draw();
    }

    function handlePAtternGoodOnlyChange(event) {
        pattern = Patterns.GoodOnly;

        draw();
    }

    function handleSetAvatorChange(event) {
        enable_avator = event.target.checked;

        draw();
    }

    function drawBoxes(ctx, callback) {
        const totalCount = pattern === Patterns.GoodAndBad ? 2 : 1;
        const GoodX = pattern === Patterns.GoodAndBad ? 32 : 320;
        const BadX = 672;
        const Y = 311;

        let loaded = 0;

        const onLoad = () => {
            loaded++;

            if (loaded === totalCount && callback) {
                ctx.drawImage(goodBoxImage, GoodX, Y);
                if (pattern === Patterns.GoodAndBad) {
                    ctx.drawImage(badBoxImage, BadX, Y);
                }

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

    function drawGoodScreenShot(ctx) {
        const img = new Image();
        img.onload = () => {
            const x = pattern === Patterns.GoodAndBad ? 32 : 320;
            const y = 396;
            const w = 576;
            const h = 356;
            ctx.drawImage(img, x, y, w, h);
        };
        img.src = goodScreenshotImageSrc;
    }

    function drawBadScreenShot(ctx) {
        const img = new Image();
        img.onload = () => {
            const x = 672;
            const y = 396;
            const w = 576;
            const h = 356;
            ctx.drawImage(img, x, y, w, h);
        };
        img.src = badScreenshotImageSrc;
    }

    function drawAvator(ctx) {
        const img = new Image();
        img.onload = () => {
            const x = 16;
            const y = 30;
            const w = 253;
            const h = 253;
            ctx.drawImage(img, x, y, w, h);
        };
        img.src = avatorImageSrc;
    }

    function drawText(
        ctx,
        text: string,
        startX: number,
        startY: number,
        fontSizePt: number,
        fontName: string,
        fontWeight: number,
    ) {
        const letterSpacing = -0.0;
        const lineHeight = fontSizePt * 1.5;
        const _startX = startX;
        const _startY = startY + fontSizePt;

        ctx.fontKerning = "normal";
        ctx.font = `${fontWeight} ${fontSizePt}pt "${fontName}", sans-serif`;
        ctx.fillStyle = "#363636";

        console.info(ctx.font);

        text.split("\n").forEach((line, index) => {
            let x = _startX;
            for (let i = 0; i < line.length; i++) {
                const c = line[i];
                ctx.fillText(c, x, _startY + index * lineHeight);
                const width = ctx.measureText(c).width;
                x += width + width * letterSpacing;
            }
        });
    }

    function drawTitle(ctx, text: string) {
        const font = fonts[selectedTitleFontIndex];
        const fontName = font.name;
        const fontWeight = font.weight[selectedTitleWeightIndex];

        const X = enable_avator ? 304 : 31;
        const Y = 69;

        drawText(ctx, text, X, Y, titleFontSizePt, fontName, fontWeight);
    }

    function drawGoodText(ctx, text: string) {
        const font = fonts[selectedGoodFontIndex];
        const fontName = font.name;
        const fontWeight = font.weight[selectedGoodWeightIndex];

        const X = pattern === Patterns.GoodAndBad ? 118 : 438;
        const Y = 336;

        drawText(ctx, text, X, Y, goodFontSizePt, fontName, fontWeight);
    }

    function drawBadText(ctx, text: string) {
        const font = fonts[selectedBadFontIndex];
        const fontName = font.name;
        const fontWeight = font.weight[selectedBadWeightIndex];

        drawText(ctx, text, 758, 336, badFontSizePt, fontName, fontWeight);
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
            drawGoodScreenShot(ctx);
            if (pattern === Patterns.GoodAndBad) {
                drawBadScreenShot(ctx);
            }
            drawAvator(ctx);

            return;
        }

        const img = new Image();
        img.onload = () => {
            canvas.width = 1280;
            canvas.height = 791;

            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

            drawBoxes(ctx, drawOverlays);
            drawGoodScreenShot(ctx);
            if (pattern === Patterns.GoodAndBad) {
                drawBadScreenShot(ctx);
            }
            drawAvator(ctx);
        };

        img.src = backgroundImageSrc;
    }

    async function preLoadFonts() {
        const loadPromises = [60, 36].flatMap((s) =>
            fonts.flatMap((f) =>
                f.weight.map((w) => {
                    const font = `${w} ${Math.ceil(s * 1.333)}px "${f.name}"`;
                    console.log(`loading: ${font}`);
                    return document.fonts.load(font);
                }),
            ),
        );

        await Promise.all(loadPromises);
        console.log("font loaded");
    }

    async function init() {
        await preLoadFonts();
        draw();
    }

    onMount(() => {
        document.fonts.ready.then(() => {
            init();
        });
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

<fieldset class="border p-4 rounded">
    <legend class="">{m.pattern_legend()}</legend>

    <div>
        <div>
            <input
                type="radio"
                id="good_and_bad"
                class="mr-2"
                value={Patterns.GoodAndBad}
                on:change={handlePatternGoodAndBadChange}
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
                on:change={handlePAtternGoodOnlyChange}
                bind:group={pattern}
            />
            <label for="good_only">{m.good_only_exmaple()}</label>
        </div>

        <div>
            <input
                type="checkbox"
                id="set_avator"
                class="mr-2"
                on:change={handleSetAvatorChange}
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
        on:change={handleGoodScreenshotFileChange}
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
            on:change={handleBadScreenshotFileChange}
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
            on:change={handleAvatorFileChange}
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

    <FontList
        defaultFontIndex={selectedTitleFontIndex}
        defaultWeightIndex={selectedTitleWeightIndex}
        onFontListChangeCallback={(index) => {
            selectedTitleFontIndex = index;
            draw();
        }}
        onWeightListChangeCallback={(index) => {
            selectedTitleWeightIndex = index;
            draw();
        }}
    />
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

<FontList
    defaultFontIndex={selectedGoodFontIndex}
    defaultWeightIndex={selectedGoodWeightIndex}
    onFontListChangeCallback={(index) => {
        selectedGoodFontIndex = index;
        selectedBadFontIndex = index;
        draw();
    }}
    onWeightListChangeCallback={(index) => {
        selectedGoodWeightIndex = index;
        selectedBadWeightIndex = index;
        draw();
    }}
/>

<canvas
    class="border border-darkgray bg-[#dbc9ad] rounded w-[1280px] h-[791px]"
    bind:this={canvas}
></canvas>

<style>
    @import url("https://fonts.googleapis.com/css2?family=Kosugi&family=Kosugi+Maru&family=Noto+Sans+JP:wght@100..900&family=Noto+Serif+JP:wght@200..900&family=Shippori+Mincho+B1:wght@400;500;600;700;800&family=Zen+Kaku+Gothic+New:wght@300;400;500;700;900&family=Zen+Maru+Gothic:wght@300;400;500;700;900&family=Zen+Old+Mincho:wght@400;500;600;700;900&display=swap");
</style>
