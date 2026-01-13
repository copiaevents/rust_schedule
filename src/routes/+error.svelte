<script lang="ts">
	import { page } from '$app/stores';
	import { base } from '$app/paths';
</script>

<svelte:head>
	<title>Error - Rust Nation UK</title>
</svelte:head>

<div class="error-page">
	<div class="error-content">
		<div class="error-icon">
			<svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
				<circle cx="12" cy="12" r="10"/>
				<path d="M12 8v4M12 16h.01"/>
			</svg>
		</div>

		<h1 class="error-title">
			{#if $page.status === 404}
				Page not found
			{:else}
				Something went wrong
			{/if}
		</h1>

		<p class="error-message">
			{#if $page.status === 404}
				The page you're looking for doesn't exist or has been moved.
			{:else if $page.error?.message}
				{$page.error.message}
			{:else}
				We encountered an unexpected error. Please try again.
			{/if}
		</p>

		<div class="error-code">
			Error {$page.status}
		</div>

		<div class="error-actions">
			<button class="btn-retry" onclick={() => window.location.reload()}>
				<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
					<path d="M1 4v6h6M23 20v-6h-6"/>
					<path d="M20.49 9A9 9 0 0 0 5.64 5.64L1 10m22 4l-4.64 4.36A9 9 0 0 1 3.51 15"/>
				</svg>
				Try again
			</button>
			<a href="{base}/" class="btn-home">
				<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
					<path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/>
					<polyline points="9 22 9 12 15 12 15 22"/>
				</svg>
				Go home
			</a>
		</div>
	</div>
</div>

<style>
	.error-page {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: var(--space-xl);
		background: var(--color-gray-100);
		min-height: 60vh;
	}

	.error-content {
		text-align: center;
		max-width: 400px;
	}

	.error-icon {
		display: flex;
		justify-content: center;
		margin-bottom: var(--space-lg);
		color: var(--color-primary);
	}

	.error-title {
		font-size: var(--text-2xl);
		font-weight: 700;
		color: var(--color-text);
		margin-bottom: var(--space-sm);
	}

	.error-message {
		font-size: var(--text-base);
		color: var(--color-text-muted);
		margin-bottom: var(--space-md);
		line-height: 1.5;
	}

	.error-code {
		display: inline-block;
		padding: var(--space-xs) var(--space-sm);
		background: var(--color-gray-200);
		border-radius: var(--radius-sm);
		font-size: var(--text-xs);
		font-weight: 600;
		color: var(--color-gray-600);
		margin-bottom: var(--space-xl);
	}

	.error-actions {
		display: flex;
		gap: var(--space-sm);
		justify-content: center;
		flex-wrap: wrap;
	}

	.btn-retry,
	.btn-home {
		display: flex;
		align-items: center;
		gap: var(--space-sm);
		padding: var(--space-sm) var(--space-lg);
		border-radius: var(--radius-md);
		font-weight: 600;
		font-size: var(--text-sm);
		transition: all var(--transition-fast);
	}

	.btn-retry {
		background: var(--color-primary);
		color: var(--color-text-light);
	}

	.btn-retry:hover {
		background: var(--color-primary-dark);
	}

	.btn-home {
		background: var(--color-white);
		color: var(--color-text);
		border: 1px solid var(--color-gray-300);
	}

	.btn-home:hover {
		background: var(--color-gray-100);
		border-color: var(--color-gray-400);
	}
</style>
