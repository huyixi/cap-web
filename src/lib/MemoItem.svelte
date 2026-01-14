<script lang="ts">
	export let memo: { text: string; createdAt: number };

	const formatter = new Intl.DateTimeFormat(undefined, {
		month: 'short',
		day: '2-digit',
		hour: '2-digit',
		minute: '2-digit'
	});

	$: timeLabel = formatter.format(new Date(memo.createdAt));
</script>

<article class="py-4">
	<div
		class="mb-2 flex justify-between font-['JetBrains_Mono','IBM_Plex_Mono',monospace] text-[12px] uppercase tracking-[0.12em] text-[rgba(23,23,23,0.6)]"
	>
		<span>{timeLabel}</span>
	</div>
	<p class="m-0 whitespace-pre-wrap text-[15px] leading-[1.6] text-[#171717]">{memo.text}</p>
</article>
