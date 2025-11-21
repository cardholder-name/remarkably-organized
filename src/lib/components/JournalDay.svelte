<script lang="ts">
	// Journal Day component with 5 evenly-spaced prompts and lined sections
	const prompts = [
		'Act: Last night after work I...',
		'Act: Today I want to accomplish...',
		'Reflect: What cool things did I do today?',
		'Reflect: The things I struggled with today were...',
		'Next: One thing I\'m excited about right now is...',
	];
	
	// Each section gets approximately 1/5 of the page height
	// Account for prompt header space
	const linesPerSection = 6;
</script>

<div class="journal-day">
	{#each prompts as prompt, index (index)}
		<div class="journal-section">
			<div class="prompt-header">{prompt}</div>
			<div class="lined-section">
				{#each new Array(linesPerSection) as _, i (i)}
					<div class="line"></div>
				{/each}
			</div>
		</div>
	{/each}
</div>

<style lang="scss">
	.journal-day {
		display: flex;
		flex-direction: column;
		height: 100%;
		width: 100%;
		padding: 0 2rem 1rem;
		gap: 0.5rem;
	}
	
	.journal-section {
		flex: 1;
		display: flex;
		flex-direction: column;
		min-height: 0;
	}
	
	.prompt-header {
		font-family: var(--font);
		font-size: 0.95em;
		font-weight: var(--font-weight-6);
		color: var(--text);
		margin-bottom: 0.25rem;
		padding-top: 0.25rem;
	}
	
	.lined-section {
		display: flex;
		flex-direction: column;
		flex: 1;
		justify-content: space-evenly;
	}
	
	.line {
		color: var(--text);
		border-bottom: solid 1px var(--outline);
		width: 100%;
		min-height: 1.5rem;
	}
</style>
