<script lang="ts">
	import { onMount } from 'svelte';

	interface SponsorAd {
		id: number;
		name: string;
		tier: string;
		logo: string;
		message: string;
		url: string;
		pdf?: string;
	}

	interface Props {
		ads: SponsorAd[];
		duration?: number;
		delayBetween?: number;
	}

	let { ads, duration = 5000, delayBetween = 10000 }: Props = $props();

	let currentAdIndex = $state(0);
	let visible = $state(false);
	let expanded = $state(false);
	let progress = $state(0);
	let dismissed = $state(false);

	let progressInterval: ReturnType<typeof setInterval>;
	let hideTimeout: ReturnType<typeof setTimeout>;
	let nextAdTimeout: ReturnType<typeof setTimeout>;

	function showAd() {
		if (dismissed || ads.length === 0) return;

		visible = true;
		progress = 0;

		// Animate progress bar
		const startTime = Date.now();
		progressInterval = setInterval(() => {
			const elapsed = Date.now() - startTime;
			progress = Math.min((elapsed / duration) * 100, 100);
		}, 50);

		// Hide after duration
		hideTimeout = setTimeout(() => {
			hideAd();
		}, duration);
	}

	function hideAd() {
		clearInterval(progressInterval);
		clearTimeout(hideTimeout);
		visible = false;
		expanded = false;
		progress = 0;

		// Schedule next ad
		if (!dismissed && ads.length > 1) {
			nextAdTimeout = setTimeout(() => {
				currentAdIndex = (currentAdIndex + 1) % ads.length;
				showAd();
			}, delayBetween);
		}
	}

	function dismissAll() {
		dismissed = true;
		clearInterval(progressInterval);
		clearTimeout(hideTimeout);
		clearTimeout(nextAdTimeout);
		visible = false;
		expanded = false;
	}

	function expandAd() {
		// Pause the auto-hide timer when expanded
		clearInterval(progressInterval);
		clearTimeout(hideTimeout);
		expanded = true;
	}

	function collapseAd() {
		expanded = false;
		// Resume auto-hide after collapse
		hideAd();
	}

	function handleBackdropClick(e: MouseEvent) {
		if (e.target === e.currentTarget) {
			collapseAd();
		}
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Escape' && expanded) {
			collapseAd();
		}
	}

	onMount(() => {
		// Show first ad after a short delay
		const initialDelay = setTimeout(() => {
			showAd();
		}, 2000);

		return () => {
			clearTimeout(initialDelay);
			clearInterval(progressInterval);
			clearTimeout(hideTimeout);
			clearTimeout(nextAdTimeout);
		};
	});

	let currentAd = $derived(ads[currentAdIndex]);
</script>

<svelte:window onkeydown={handleKeydown} />

{#if visible && currentAd}
	<!-- Toast (collapsed state) -->
	{#if !expanded}
		<div class="toast-container" role="region" aria-label="Sponsor message">
			<div class="sr-only" aria-live="polite" aria-atomic="true">
				{currentAd.message}. {currentAd.name}, {currentAd.tier} sponsor.
			</div>
			<div
				class="toast"
				onclick={expandAd}
				onkeydown={(e) => e.key === 'Enter' && expandAd()}
				role="button"
				tabindex="0"
				aria-expanded="false"
			>
				<div class="progress-bar" style="width: {progress}%"></div>

				<img src={currentAd.logo} alt="{currentAd.name} logo" class="sponsor-logo" />

				<div class="toast-content">
					<span class="sponsor-message">{currentAd.message}</span>
					<span class="sponsor-name">{currentAd.name}</span>
					<span class="sponsor-tier">{currentAd.tier} sponsor</span>
				</div>

				<button
					class="close-btn"
					onclick={(e) => { e.stopPropagation(); hideAd(); }}
					aria-label="Close"
				>
					<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<path d="M18 6L6 18M6 6l12 12"/>
					</svg>
				</button>
			</div>
		</div>
	{/if}

	<!-- Modal (expanded state) -->
	{#if expanded}
		<div
			class="modal-backdrop"
			onclick={handleBackdropClick}
			onkeydown={(e) => e.key === 'Escape' && collapseAd()}
			role="dialog"
			aria-modal="true"
			aria-labelledby="sponsor-modal-title"
			tabindex="-1"
		>
			<div class="modal">
				<button
					class="modal-close"
					onclick={collapseAd}
					aria-label="Close sponsor details"
				>
					<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<path d="M18 6L6 18M6 6l12 12"/>
					</svg>
				</button>

				<div class="modal-content" class:has-pdf={currentAd.pdf}>
				{#if currentAd.pdf}
					<div class="modal-pdf-header">
						<img src={currentAd.logo} alt="{currentAd.name} logo" class="modal-logo-small" />
						<div class="modal-pdf-info">
							<span class="sponsor-tier">{currentAd.tier}</span>
							<h2 id="sponsor-modal-title" class="modal-title-small">{currentAd.name}</h2>
						</div>
					</div>
					<div class="pdf-container">
						<iframe src={currentAd.pdf} title="{currentAd.name} information" class="pdf-viewer"></iframe>
					</div>
					<a
						href={currentAd.url}
						target="_blank"
						rel="noopener noreferrer"
						class="modal-cta"
					>
						Visit Website
						<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
							<path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
							<polyline points="15 3 21 3 21 9"/>
							<line x1="10" y1="14" x2="21" y2="3"/>
						</svg>
					</a>
				{:else}
					<img src={currentAd.logo} alt="{currentAd.name} logo" class="modal-logo" />

					<div class="modal-header">
						<span class="sponsor-tier">{currentAd.tier} sponsor</span>
					</div>

					<h2 id="sponsor-modal-title" class="modal-title">{currentAd.name}</h2>

					<p class="modal-message">{currentAd.message}</p>

					<a
						href={currentAd.url}
						target="_blank"
						rel="noopener noreferrer"
						class="modal-cta"
					>
						Visit Website
						<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
							<path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
							<polyline points="15 3 21 3 21 9"/>
							<line x1="10" y1="14" x2="21" y2="3"/>
						</svg>
					</a>
				{/if}
				</div>
			</div>
		</div>
	{/if}
{/if}

<style>
	.toast-container {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		padding: var(--space-md);
		z-index: 1000;
		animation: slide-up 0.3s ease-out;
	}

	@keyframes slide-up {
		from {
			transform: translateY(100%);
			opacity: 0;
		}
		to {
			transform: translateY(0);
			opacity: 1;
		}
	}

	.toast {
		display: flex;
		align-items: center;
		gap: var(--space-md);
		padding: var(--space-md);
		background: var(--color-surface);
		border-radius: var(--radius-lg);
		box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.15);
		position: relative;
		overflow: hidden;
		max-width: 500px;
		margin: 0 auto;
		border: 1px solid var(--color-primary);
		width: 100%;
		text-align: left;
		cursor: pointer;
	}

	@media (prefers-color-scheme: dark) {
		.toast {
			border-color: var(--color-yellow);
		}
	}

	.progress-bar {
		position: absolute;
		top: 0;
		left: 0;
		height: 3px;
		background: var(--color-primary);
		transition: width 0.05s linear;
	}

	.sponsor-logo {
		width: 48px;
		height: 48px;
		object-fit: contain;
		border-radius: var(--radius-sm);
		flex-shrink: 0;
	}

	.toast-content {
		flex: 1;
		min-width: 0;
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	.sponsor-header {
		display: flex;
		align-items: center;
		gap: var(--space-sm);
	}

	.sponsor-label {
		font-size: var(--text-xs);
		color: var(--color-text-muted);
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.sponsor-tier {
		font-size: var(--text-xs);
		text-transform: uppercase;
		letter-spacing: 0.05em;
		color: var(--color-text-muted);
	}

	.sponsor-name {
		font-size: var(--text-sm);
		font-weight: 700;
		color: var(--color-text);
	}

	.sponsor-message {
		font-size: var(--text-sm);
		color: var(--color-gray-600);
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.close-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 32px;
		height: 32px;
		border-radius: var(--radius-full);
		color: var(--color-gray-600);
		flex-shrink: 0;
		transition: all var(--transition-fast);
	}

	.close-btn:hover,
	.close-btn:focus-visible {
		background: var(--color-gray-100);
		color: var(--color-text);
	}

	/* Modal Styles */
	.modal-backdrop {
		position: fixed;
		inset: 0;
		background: rgba(0, 0, 0, 0.7);
		z-index: 1001;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: var(--space-md);
		animation: fade-in 0.2s ease-out;
	}

	@keyframes fade-in {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	.modal {
		background: var(--color-surface);
		border-radius: var(--radius-xl);
		max-width: 400px;
		width: 100%;
		position: relative;
		animation: scale-in 0.2s ease-out;
		overflow: hidden;
	}

	.modal:has(.has-pdf) {
		max-width: 600px;
		max-height: 90vh;
		overflow-y: auto;
	}

	@keyframes scale-in {
		from {
			transform: scale(0.95);
			opacity: 0;
		}
		to {
			transform: scale(1);
			opacity: 1;
		}
	}

	.modal-close {
		position: absolute;
		top: var(--space-sm);
		right: var(--space-sm);
		width: 40px;
		height: 40px;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: var(--radius-full);
		color: var(--color-gray-600);
		transition: all var(--transition-fast);
		z-index: 1;
	}

	.modal-close:hover,
	.modal-close:focus-visible {
		background: var(--color-gray-100);
		color: var(--color-text);
	}

	.modal-content {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: var(--space-2xl) var(--space-xl);
		text-align: center;
	}

	.modal-logo {
		width: 100px;
		height: 100px;
		object-fit: contain;
		border-radius: var(--radius-md);
		margin-bottom: var(--space-lg);
	}

	.modal-header {
		margin-bottom: var(--space-sm);
	}

	.modal-title {
		font-size: var(--text-2xl);
		font-weight: 700;
		color: var(--color-text);
		margin-bottom: var(--space-md);
	}

	.modal-message {
		font-size: var(--text-base);
		line-height: 1.6;
		color: var(--color-text-muted);
		margin-bottom: var(--space-xl);
	}

	.modal-cta {
		display: inline-flex;
		align-items: center;
		gap: var(--space-sm);
		padding: var(--space-md) var(--space-xl);
		background: var(--color-primary);
		color: var(--color-text-light);
		border-radius: var(--radius-md);
		font-weight: 600;
		font-size: var(--text-base);
		transition: background var(--transition-fast);
	}

	.modal-cta:hover,
	.modal-cta:focus-visible {
		background: var(--color-primary-dark);
	}

	/* PDF Modal Styles */
	.modal-content.has-pdf {
		padding: var(--space-lg);
	}

	.modal-pdf-header {
		display: flex;
		align-items: center;
		gap: var(--space-md);
		width: 100%;
		margin-bottom: var(--space-md);
	}

	.modal-logo-small {
		width: 50px;
		height: 50px;
		object-fit: contain;
		border-radius: var(--radius-sm);
	}

	.modal-pdf-info {
		text-align: left;
	}

	.modal-title-small {
		font-size: var(--text-lg);
		font-weight: 700;
		color: var(--color-text);
		margin: 0;
	}

	.pdf-container {
		width: 100%;
		margin-bottom: var(--space-md);
		border-radius: var(--radius-md);
		overflow: hidden;
		background: var(--color-gray-100);
	}

	.pdf-viewer {
		width: 100%;
		height: 400px;
		border: none;
	}

	.has-pdf .modal-cta {
		width: 100%;
		justify-content: center;
	}

	@media (min-width: 768px) {
		.toast-container {
			padding: var(--space-lg);
		}

		.sponsor-logo {
			width: 56px;
			height: 56px;
		}

		.modal-logo {
			width: 120px;
			height: 120px;
		}

		.pdf-viewer {
			height: 500px;
		}

		.modal:has(.has-pdf) {
			max-width: 700px;
		}
	}

	.sr-only {
		position: absolute;
		width: 1px;
		height: 1px;
		padding: 0;
		margin: -1px;
		overflow: hidden;
		clip: rect(0, 0, 0, 0);
		white-space: nowrap;
		border: 0;
	}
</style>
