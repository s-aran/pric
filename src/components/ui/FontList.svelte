<script context="module" lang="ts">
  type Font = {
    name: string;
    weight: number[];
  };

  export const fonts: Font[] = [
    {
      name: "Noto Serif JP",
      weight: [200, 300, 400, 500, 600, 700, 800, 900],
    },
    {
      name: "Noto Sans JP",
      weight: [200, 300, 400, 500, 600, 700, 800, 900],
    },
    // {
    //   name: "Kosugi",
    //   weight: [400],
    // },
    // {
    //   name: "Kosugi Maru",
    //   weight: [400],
    // },
    {
      name: "Zen Maru Gothic",
      weight: [300, 400, 500, 700, 900],
    },
    {
      name: "Zen Kaku Gothic New",
      weight: [300, 400, 500, 700, 900],
    },
    {
      name: "Zen Old Mincho",
      weight: [400, 500, 600, 700, 900],
    },
    {
      name: "Shippori Mincho B1",
      weight: [400, 500, 600, 700, 800],
    },
  ];
</script>

<script lang="ts">
  export let defaultFontIndex = 0;
  export let defaultWeightIndex = 0;

  let selectedFontIndex = defaultFontIndex;
  let selectedWeightIndex = defaultWeightIndex;

  export let onFontListChangeCallback = () => {};
  export let onWeightListChangeCallback = () => {};

  $: availableWeights = fonts[selectedFontIndex].weight;

  function onFontListChange(event: Event) {
    onFontListChangeCallback(event.target.value as number);
    selectedWeightIndex = 0;
    console.info(event.target);
  }

  function onWeightListChange(event: Event) {
    onWeightListChangeCallback(event.target.value as number);
    console.info(event.target);
  }
</script>

<div>
  <select
    bind:value={selectedFontIndex}
    on:change={onFontListChange}
    class="border rounded"
  >
    {#each fonts as font, index}
      <option value={index}>{font.name}</option>
    {/each}
  </select>

  <select
    bind:value={selectedWeightIndex}
    on:change={onWeightListChange}
    class="border rounded"
  >
    {#each availableWeights as weight, index}
      <option value={index}>{weight}</option>
    {/each}
  </select>
</div>
