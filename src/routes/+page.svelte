<script lang="ts">
	import { onMount, tick } from 'svelte';
	import MemoItem from '$lib/MemoItem.svelte';

	type Memo = {
		id: number;
		text: string;
		createdAt: number;
	};

	let memos: Memo[] = [];
	let draft = '';
	let textareaEl: HTMLTextAreaElement | null = null;
	let inputShell: HTMLDivElement | null = null;
	let inputHeader: HTMLDivElement | null = null;

	const parsePx = (value: string) => Number.parseFloat(value) || 0;

	const getSafeArea = (name: string) => {
		const rootStyles = getComputedStyle(document.documentElement);
		return parsePx(rootStyles.getPropertyValue(name));
	};

	const getViewportHeight = () => window.visualViewport?.height ?? window.innerHeight;

	const getMaxTextareaHeight = () => {
		if (!inputShell) return 0;
		const viewportHeight = getViewportHeight();
		const rect = inputShell.getBoundingClientRect();
		const styles = getComputedStyle(inputShell);
		const paddingTop = parsePx(styles.paddingTop);
		const paddingBottom = parsePx(styles.paddingBottom);
		const rowGap = parsePx(styles.rowGap);
		const headerHeight = inputHeader?.offsetHeight ?? 0;
		const safeBottom = getSafeArea('--safe-bottom');

		const chromeHeight = paddingTop + paddingBottom + headerHeight + rowGap;
		const available = viewportHeight - rect.top - safeBottom - chromeHeight;
		return Math.max(80, Math.floor(available));
	};

	const recalculateTextareaHeight = () => {
		if (!textareaEl) return;
		const maxHeight = getMaxTextareaHeight();
		if (maxHeight <= 0) return;

		const previousScrollTop = textareaEl.scrollTop;
		textareaEl.style.height = 'auto';
		const nextHeight = Math.min(textareaEl.scrollHeight, maxHeight);
		textareaEl.style.height = `${nextHeight}px`;

		const shouldScroll = textareaEl.scrollHeight > maxHeight + 1;
		textareaEl.style.overflowY = shouldScroll ? 'auto' : 'hidden';
		if (shouldScroll) {
			textareaEl.scrollTop = previousScrollTop;
		}
	};

	const submitMemo = async () => {
		const trimmed = draft.trim();
		if (!trimmed) return;

		const timestamp = Date.now();
		memos = [{ id: timestamp, text: trimmed, createdAt: timestamp }, ...memos];
		draft = '';
		await tick();
		recalculateTextareaHeight();
	};

	const handleKeydown = async (event: KeyboardEvent) => {
		if (event.key !== 'Enter' || !event.shiftKey || event.isComposing) return;
		event.preventDefault();
		await submitMemo();
	};

	const handleResize = () => {
		recalculateTextareaHeight();
	};

	const handleInput = () => {
		recalculateTextareaHeight();
	};

	onMount(async () => {
		await tick();
		recalculateTextareaHeight();

		const viewport = window.visualViewport;
		window.addEventListener('resize', handleResize);
		viewport?.addEventListener('resize', handleResize);
		viewport?.addEventListener('scroll', handleResize);

		return () => {
			window.removeEventListener('resize', handleResize);
			viewport?.removeEventListener('resize', handleResize);
			viewport?.removeEventListener('scroll', handleResize);
		};
	});
</script>

<svelte:head>
	<title>Cap</title>
	<link
		rel="stylesheet"
		href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600&family=Space+Grotesk:wght@400;500;600;700&display=swap"
	/>
</svelte:head>

<main class="min-h-svh bg-white font-['Space_Grotesk','IBM_Plex_Sans',sans-serif] text-[#171717]">
	<div
		class="mx-auto flex min-h-svh w-full max-w-225 flex-col px-4 pt-[calc(var(--safe-top))] pb-[calc(20px+var(--safe-bottom))] sm:px-6 lg:px-8"
	>
		<div class="flex flex-1 flex-col overflow-hidden border border-[rgba(23,23,23,0.08)] bg-white">
			<section
				class="sticky top-0 z-10 grid gap-3 border-b border-[rgba(23,23,23,0.12)] bg-[rgba(23,23,23,0.02)] px-4 pt-[calc(18px+var(--safe-top))] pb-3.5 sm:px-6"
				bind:this={inputShell}
			>
				<div
					class="flex flex-col gap-3 sm:flex-row sm:items-end sm:justify-between"
					bind:this={inputHeader}
				>
					<div>
						<h1 class="mt-1.5 text-[clamp(22px,3vw,32px)] tracking-[-0.02em]">Cap</h1>
					</div>
				</div>

				<textarea
					bind:this={textareaEl}
					bind:value={draft}
					class="min-h-30 w-full resize-none appearance-none border border-[rgba(23,23,23,0.12)] bg-[rgba(23,23,23,0.03)] p-4 text-base leading-[1.6] text-[#171717] shadow-none transition-colors duration-200 outline-none focus:border-[rgba(23,23,23,0.3)] focus:bg-[rgba(23,23,23,0.05)] focus:outline-none"
					rows={3}
					on:input={handleInput}
					on:keydown={handleKeydown}
				></textarea>
			</section>

			<section
				class="flex flex-1 flex-col gap-4 overflow-auto px-4 pt-4 pb-[calc(24px+var(--safe-bottom))] sm:px-6"
				aria-live="polite"
			>
				<div
					class="flex items-center justify-between text-[12px] tracking-[0.16em] text-[rgba(23,23,23,0.6)] uppercase"
				>
					<h2 class="m-0 text-[12px]">History</h2>
					<span>{memos.length} memos</span>
				</div>

				<div class="flex flex-col divide-y divide-[rgba(23,23,23,0.12)]">
					{#each memos as memo (memo.id)}
						<MemoItem {memo} />
					{/each}
				</div>
			</section>
		</div>
	</div>
</main>
