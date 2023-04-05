<script>
	import Power from './Power.svelte';	
</script>

<svelte:head>
	<title>Test app</title>
	<meta name="description" content="test app" />
</svelte:head>

<section>
	<Power />
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}
</style>
