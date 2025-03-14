<script lang="ts">
	import type { BookName, CoinedEra, UsageCategory, Word } from '$lib/types';

	import { categoryColors } from '$lib/util';
	import type { PageData } from './$types';

	import Details from '$lib/components/Details.svelte';
	import Link from '$lib/components/Link.svelte';
	import X from '$lib/components/X.svelte';

	export let data: PageData;

	$: linku = data.linku;
	$: words = Object.values(linku.data);

	$: keys = words.reduce((acc, word) => {
		Object.keys(word).forEach(key => {
			if (!acc.includes(key as keyof Word)) acc.push(key as keyof Word);
		});
		return acc;
	}, [] as (keyof Word)[]);

	$: frequencies = keys.map(key => {
		const values = words.map(word => word[key]);
		const existing = values.filter(value => value !== undefined);
		const unique = new Set(existing).size;

		return { key, count: existing.length, unique };
	});

	$: nearCompleteKeys = frequencies
		.filter(({ count }) => count > words.length - 20 && count < words.length)
		.map(({ key }) => key);

	$: usageCategoryFrequencies = words.reduce(
		(acc, word) => {
			acc[word.usage_category]++;
			return acc;
		},
		{
			core: 0,
			widespread: 0,
			common: 0,
			uncommon: 0,
			rare: 0,
			obscure: 0,
			marginal: 0
		} satisfies Record<UsageCategory, number>
	);

	function getCategoryColor(category: string) {
		return category in categoryColors
			? categoryColors[category as UsageCategory]
			: '';
	}

	$: booksFrequencies = words.reduce(
		(acc, word) => {
			acc[word.book]++;
			return acc;
		},
		{
			pu: 0,
			'ku suli': 0,
			'ku lili': 0,
			none: 0
		} satisfies Record<BookName, number>
	);

	$: coinedEraFrequencies = words.reduce(
		(acc, word) => {
			acc[word.coined_era ?? 'unknown']++;
			return acc;
		},
		{
			'pre-pu': 0,
			'post-pu': 0,
			'post-ku': 0,
			unknown: 0
		} satisfies Record<CoinedEra | 'unknown', number>
	);

	$: compounds = Object.values(data.compounds);

	$: glyphFrequencies = compounds.reduce((acc, compound) => {
		acc[compound.glyphs?.length ?? 0] ??= 0;
		acc[compound.glyphs?.length ?? 0]++;
		return acc;
	}, [] as Record<number, number>);

	let detailsOpen = false;
</script>

<svelte:head>
	<title>super secret testing page &ndash; nimi.li</title>

	<meta name="author" content="jan Tani" />
	<meta name="description" content="interactive toki pona dictionary" />

	<meta property="og:title" content="super secret testing page – nimi.li" />
	<meta property="og:author" content="jan Tani" />
	<meta property="og:description" content="interactive toki pona dictionary" />
	<meta property="og:url" content="https://nimi.li/test" />
	<meta property="og:site_name" content="nimi.li" />
	<meta property="og:type" content="website" />
</svelte:head>

<h1 class="text-4xl">super secret testing page</h1>

<h2 class="mt-4 text-3xl">styles</h2>

<p class="mt-2">
	<button
		class="px-2 py-1 interactable"
		on:click={() => {
			detailsOpen = !detailsOpen;
		}}
	>
		toggle details
	</button>
</p>

<Details value={detailsOpen} key={() => ''}>
	<div class="flex">
		<h2 class="text-2xl">title</h2>

		<button
			class="ml-auto p-1 interactable"
			on:click={() => {
				detailsOpen = false;
			}}
		>
			<X />
		</button>
	</div>

	<p class="font-pona text-4xl">ni li seme a</p>

	<p class="mt-2">
		mi sona <Link href="/ala">ala</Link>
	</p>
</Details>

<h2 class="mt-4 text-3xl" data-sveltekit-preload-data="off">
	<Link href="/data/linku">linku data</Link>
	<span class="font-text text-sm faded">
		({words.length})
	</span>
</h2>

<p class="mt-2">
	<span class="font-bold">Note:</span> nimi.li modifies and adds a few words, so
	a few things might not match up.
</p>

<h3 class="mt-4 text-2xl">keys</h3>

<table class="mt-2">
	<thead>
		<tr class="text-left">
			<th>key</th>
			<th># of words with value</th>
			<th># of non-empty unique values</th>
		</tr>
	</thead>
	<tbody>
		{#each frequencies as { key, count, unique }}
			<tr>
				<td>
					<span class="flex items-center gap-2">
						<span
							class="w-2 h-2 rounded-full {count === words.length &&
							count === unique
								? 'bg-green-500'
								: count === words.length
								? 'bg-blue-500'
								: count > words.length - 20
								? 'bg-yellow-500'
								: 'bg-red-500'}"
						/>

						{key}

						{#if count !== words.length && count > words.length - 20}
							<span class="faded">
								{words.length - count} missing
							</span>
						{/if}
					</span>
				</td>
				<td>{count}</td>
				<td>{unique}</td>
			</tr>
		{/each}
	</tbody>
</table>

<!-- <p class="mt-2">
	missing source_language:
	{words
		.filter(w => !w.source_language)
		.map(w => w.word)
		.join(', ')}
</p>
<p>
	missing etymology:
	{words
		.filter(w => !w.etymology)
		.map(w => w.word)
		.join(', ')}
</p>
<p>
	missing coined_era:
	{words
		.filter(w => !w.coined_era)
		.map(w => w.word)
		.join(', ')}
</p>
<p>
	missing creator:
	{words
		.filter(w => !w.creator)
		.map(w => w.word)
		.join(', ')}
</p>
<p>
	missing sitelen_pona:
	{words
		.filter(w => !w.sitelen_pona)
		.map(w => w.word)
		.join(', ')}
</p> -->

<h3 class="mt-4 mb-2 text-2xl">near complete keys</h3>

{#each nearCompleteKeys as key}
	<p>
		missing {key}:
		{#each words.filter(w => !w[key]) as word, i}
			{#if i > 0}, {/if}
			<Link href="/{word.id}">
				{word.word}
			</Link>
		{/each}
	</p>
{/each}

<h3 class="mt-4 text-2xl">more stats</h3>

<table class="inline-table mt-2">
	<thead>
		<tr class="text-left">
			<th>usage</th>
			<th>count</th>
		</tr>
	</thead>
	<tbody>
		{#each Object.entries(usageCategoryFrequencies) as [category, count]}
			<tr>
				<td>
					<span class="flex items-center gap-2">
						<span class="w-2 h-2 rounded-full {getCategoryColor(category)}" />
						{category}
					</span>
				</td>
				<td>{count}</td>
			</tr>
		{/each}
	</tbody>
</table>

<table class="inline-table mt-2">
	<thead>
		<tr class="text-left">
			<th>book</th>
			<th>count</th>
		</tr>
	</thead>
	<tbody>
		{#each Object.entries(booksFrequencies) as [book, count]}
			<tr>
				<td>{book}</td>
				<td>{count}</td>
			</tr>
		{/each}
	</tbody>
</table>

<table class="inline-table mt-2">
	<thead>
		<tr class="text-left">
			<th>coined era</th>
			<th>count</th>
		</tr>
	</thead>
	<tbody>
		{#each Object.entries(coinedEraFrequencies) as [era, count]}
			<tr>
				<td>{era}</td>
				<td>{count}</td>
			</tr>
		{/each}
	</tbody>
</table>

<h3 class="mt-4 text-2xl">links</h3>

<p class="mt-2 flex flex-wrap gap-1">
	{#each words as word}
		<Link href="/{word.id}">
			{word.id}
		</Link>
	{/each}
</p>

<h2 class="mt-4 text-3xl" data-sveltekit-preload-data="off">
	<Link href="/data/compounds">compounds</Link>
	<span class="font-text text-sm faded">
		({compounds.length})
	</span>
</h2>

<table class="inline-table mt-2">
	<thead>
		<tr class="text-left">
			<th>unique glyphs</th>
			<th>count</th>
		</tr>
	</thead>
	<tbody>
		{#each Object.entries(glyphFrequencies) as [amount, count]}
			<tr>
				<td>{amount}</td>
				<td>{count}</td>
			</tr>
		{/each}
	</tbody>
</table>

<style lang="postcss">
	th,
	td {
		@apply px-2 py-0.5 border border-gray-200;
	}

	:global(.dark) :is(th, td) {
		@apply border-gray-800;
	}
</style>
