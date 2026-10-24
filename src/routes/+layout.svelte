<script lang="ts">
	let { children } = $props();
</script>

<div class="shell">
	<main>
		{@render children()}
	</main>

	<footer>SvelteKit + Vercel / SSR</footer>
</div>

<style>
	.shell {
		max-width: 42rem;
		margin: 0 auto;
		padding: 1.5rem 1rem 3rem;
		min-height: 100dvh;
		display: flex;
		flex-direction: column;
	}

	main {
		flex: 1;
	}

	footer {
		padding-top: 2rem;
		font-size: 0.75rem;
		color: var(--muted);
		text-align: center;
	}
</style>
