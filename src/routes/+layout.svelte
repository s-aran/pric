<script lang="ts">
	import { i18n } from "$lib/i18n";
	import { ParaglideJS } from "@inlang/paraglide-sveltekit";
	import "../app.css";
	let { children } = $props();
</script>

<div class="bg-gray-100 dark:bg-gray-800 text-gray-900 dark:text-gray-50">
	<ParaglideJS {i18n}>
		{@render children()}
	</ParaglideJS>
</div>
