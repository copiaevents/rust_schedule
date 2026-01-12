<script lang="ts">
	interface Break {
		time: string;
		duration: number;
		type: string;
		label: string;
	}

	interface Props {
		breakItem: Break;
	}

	let { breakItem }: Props = $props();

	const icons: Record<string, string> = {
		coffee: '\u2615',
		lunch: '\ud83c\udf7d\ufe0f'
	};
</script>

<div class="break-card">
	<span class="break-icon" aria-hidden="true">{icons[breakItem.type] || '\u23f8'}</span>
	<span class="break-time">{breakItem.time} <span class="timezone">GMT</span></span>
	<span class="break-label">{breakItem.label}</span>
	<span class="break-duration">{breakItem.duration} min</span>
</div>

<style>
	.break-card {
		display: flex;
		align-items: center;
		gap: var(--space-sm);
		padding: var(--space-sm) var(--space-md);
		background: var(--color-green);
		border-radius: var(--radius-md);
		color: var(--color-white);
	}

	.break-icon {
		font-size: var(--text-base);
	}

	.break-time {
		font-weight: 700;
		font-size: var(--text-sm);
		display: flex;
		align-items: baseline;
		gap: var(--space-xs);
	}

	.timezone {
		font-size: 10px;
		font-weight: 500;
		opacity: 0.8;
		text-transform: uppercase;
		letter-spacing: 0.03em;
	}

	.break-label {
		flex: 1;
		font-weight: 500;
		font-size: var(--text-sm);
	}

	.break-duration {
		font-size: var(--text-xs);
		opacity: 1;
	}
</style>
