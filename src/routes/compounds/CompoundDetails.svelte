<script lang="ts">
	import type { Compound } from '$lib/types';

	import Details from '$lib/components/Details.svelte';
	import Link from '$lib/components/Link.svelte';
	import X from '$lib/components/X.svelte';

	let possibleCompound: Compound | null;
	export { possibleCompound as compound };
</script>

<Details
	bind:value={possibleCompound}
	key={compound => compound.compound}
	let:value={compound}
>
	<div class="flex">
		<h2 class="text-2xl">{compound.compound}</h2>

		<button
			class="ml-auto p-1 interactable"
			on:click={() => {
				possibleCompound = null;
			}}
		>
			<X />
		</button>
	</div>

	<p class="font-pona text-4xl">
		{compound.glyphs?.join(' ') || compound.compound}
	</p>

	<p>
		see
		{#each compound.compound.split(' ') as word, i (word)}
			{i !== 0 ? ',' : ''}
			<Link href="/{word}">{word}</Link>
		{/each}
	</p>

	<div class="flex flex-col mt-2">
		{#each Object.entries(compound.uses) as [use, count]}
			<p>
				{use}
				<span class="faded">
					&middot; {count}%
				</span>
			</p>
		{/each}
	</div>
</Details>
