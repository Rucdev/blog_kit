<script lang="ts">
	interface SearchPost {
		slug: string;
		title: string;
		excerpt: string;
		tags: string[];
		dateDisplay: string;
		readingTime: number;
	}

	let { posts }: { posts: SearchPost[] } = $props();

	let query = $state('');
	let activeTag = $state('all');

	const allTags = $derived.by(() => {
		const counts = new Map<string, number>();
		for (const post of posts) {
			for (const tag of post.tags) {
				counts.set(tag, (counts.get(tag) ?? 0) + 1);
			}
		}
		return [...counts.entries()].sort((a, b) => b[1] - a[1]);
	});

	const results = $derived.by(() => {
		const q = query.trim().toLowerCase();
		return posts.filter((post) => {
			if (activeTag !== 'all' && !post.tags.includes(activeTag)) return false;
			if (!q) return true;
			return (
				post.title.toLowerCase().includes(q) ||
				post.excerpt.toLowerCase().includes(q) ||
				post.tags.some((tag) => tag.toLowerCase().includes(q))
			);
		});
	});

	function highlight(text: string, q: string) {
		const trimmed = q.trim();
		if (!trimmed) return [{ text, mark: false }];
		const lower = text.toLowerCase();
		const idx = lower.indexOf(trimmed.toLowerCase());
		if (idx === -1) return [{ text, mark: false }];
		return [
			{ text: text.slice(0, idx), mark: false },
			{ text: text.slice(idx, idx + trimmed.length), mark: true },
			{ text: text.slice(idx + trimmed.length), mark: false },
		];
	}
</script>

<div class="search">
	<div class="input-row">
		<span class="prompt">/</span>
		<input
			type="text"
			bind:value={query}
			placeholder="タイトル・タグ・本文で検索"
			aria-label="記事を検索"
			autocomplete="off"
		/>
	</div>

	<div class="chips">
		<button type="button" class:active={activeTag === 'all'} onclick={() => (activeTag = 'all')}>
			すべて {posts.length}
		</button>
		{#each allTags as [tag, count] (tag)}
			<button type="button" class:active={activeTag === tag} onclick={() => (activeTag = tag)}>
				#{tag} {count}
			</button>
		{/each}
	</div>

	<div class="results">
		{#each results as post (post.slug)}
			<a class="result" class:selected={false} href={`/blog/${post.slug}/`}>
				<div class="result-meta">
					<span>{post.dateDisplay}</span>
					{#if post.tags[0]}<span>#{post.tags[0]}</span>{/if}
					<span>{post.readingTime} min</span>
				</div>
				<p class="result-title">
					{#each highlight(post.title, query) as chunk}
						{#if chunk.mark}<mark>{chunk.text}</mark>{:else}{chunk.text}{/if}
					{/each}
				</p>
				<p class="result-excerpt">
					{#each highlight(post.excerpt, query) as chunk}
						{#if chunk.mark}<mark>{chunk.text}</mark>{:else}{chunk.text}{/if}
					{/each}
				</p>
			</a>
		{:else}
			<p class="no-results">一致する記事が見つかりませんでした。</p>
		{/each}
	</div>

	<p class="hint">{results.length} 件 · タグで絞り込み</p>
</div>

<style>
	.search {
		max-width: 720px;
		margin: 0 auto;
		padding: 26px 26px 30px;
	}
	.input-row {
		display: flex;
		align-items: center;
		gap: 10px;
		padding: 13px 16px;
		background: var(--surface);
		border: 1px solid var(--border-strong);
		border-radius: 5px;
	}
	.prompt {
		font: 500 13px var(--font-mono);
		color: var(--accent);
	}
	input {
		flex: 1;
		border: none;
		background: none;
		font: 400 14px var(--font-mono);
		color: var(--ink);
		outline: none;
	}
	.chips {
		display: flex;
		flex-wrap: wrap;
		gap: 6px;
		margin-top: 12px;
	}
	.chips button {
		padding: 4px 9px;
		background: var(--surface);
		border: 1px solid var(--border-strong);
		border-radius: 3px;
		font: 400 11px var(--font-mono);
		color: var(--muted);
		cursor: pointer;
	}
	.chips button.active {
		background: var(--header-bg);
		border-color: var(--header-bg);
		color: var(--header-fg);
	}
	.results {
		margin-top: 20px;
		display: flex;
		flex-direction: column;
		gap: 2px;
	}
	.result {
		display: block;
		padding: 14px 16px;
		border-bottom: 1px solid var(--border-soft);
		text-decoration: none;
	}
	.result:hover {
		background: var(--surface);
		border-radius: 5px;
	}
	.result-meta {
		display: flex;
		justify-content: space-between;
		gap: 12px;
		margin-bottom: 6px;
		font: 400 11px var(--font-mono);
		color: var(--faint);
	}
	.result-title {
		margin: 0 0 6px;
		font: 500 14.5px/1.6 var(--font-sans);
		color: var(--ink);
	}
	.result-excerpt {
		margin: 0;
		font: 400 12px/1.8 var(--font-sans);
		color: var(--muted);
	}
	.result mark {
		background: var(--mark-bg);
		color: var(--ink);
		padding: 0 2px;
		border-radius: 2px;
	}
	.no-results {
		padding: 20px 4px;
		font: 400 13px var(--font-sans);
		color: var(--muted);
	}
	.hint {
		margin: 16px 2px 0;
		font: 400 10.5px var(--font-mono);
		color: var(--faint);
	}
</style>
