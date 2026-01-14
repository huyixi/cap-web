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
		const safeBottom = getSafeArea('--safe-bottom');

		const chromeHeight = paddingTop + paddingBottom + rowGap;
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

	onMount(async () => {
		await tick();
		recalculateTextareaHeight();

		const viewport = window.visualViewport;
		window.addEventListener('resize', recalculateTextareaHeight);
		viewport?.addEventListener('resize', recalculateTextareaHeight);
		viewport?.addEventListener('scroll', recalculateTextareaHeight);

		return () => {
			window.removeEventListener('resize', recalculateTextareaHeight);
			viewport?.removeEventListener('resize', recalculateTextareaHeight);
			viewport?.removeEventListener('scroll', recalculateTextareaHeight);
		};
	});
</script>

<svelte:head>
	<title>Cap</title>
</svelte:head>

<main class="min-h-svh bg-white text-[#171717]">
	<div
		class="mx-auto flex min-h-svh w-full max-w-225 flex-col px-4 pt-[calc(var(--safe-top))] pb-[calc(20px+var(--safe-bottom))] sm:px-6 lg:px-8"
	>
		<div class="flex flex-1 flex-col overflow-hidden border border-[rgba(23,23,23,0.08)] bg-white">
			<section
				class="sticky top-0 z-10 grid gap-3 border-b border-[rgba(23,23,23,0.12)] bg-[rgba(23,23,23,0.02)] px-4 pt-[calc(18px+var(--safe-top))] pb-3.5 sm:px-6"
				bind:this={inputShell}
			>
				<div class="flex flex-col gap-3 sm:flex-row sm:items-end sm:justify-between"></div>

				<div class="relative">
					<textarea
						bind:this={textareaEl}
						bind:value={draft}
						class="min-h-30 w-full resize-none appearance-none border border-[rgba(23,23,23,0.12)] bg-[rgba(23,23,23,0.03)] p-4 pr-14 pb-12 text-base leading-[1.6] text-[#171717] shadow-none transition-colors duration-200 outline-none focus:border-[rgba(23,23,23,0.3)] focus:bg-[rgba(23,23,23,0.05)] focus:outline-none"
						rows={3}
						on:input={recalculateTextareaHeight}
						on:keydown={handleKeydown}
					></textarea>

					<button
						type="button"
						class="absolute right-3 bottom-3 inline-flex h-9 w-9 items-center justify-center rounded-[6px] border border-[rgba(23,23,23,0.15)] bg-[#171717] text-white transition-colors duration-200 hover:bg-black disabled:cursor-not-allowed disabled:border-[rgba(23,23,23,0.08)] disabled:bg-[rgba(23,23,23,0.2)]"
						disabled={!draft.trim()}
						aria-label="Send memo"
						on:click={submitMemo}
					>
						<svg
							aria-hidden="true"
							viewBox="0 0 24 24"
							class="h-4 w-4"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<path d="M22 2L11 13" />
							<path d="M22 2l-7 20-4-9-9-4 20-7z" />
						</svg>
					</button>
				</div>
			</section>

			<section
				class="flex flex-1 flex-col gap-4 overflow-auto px-4 pt-4 pb-[calc(24px+var(--safe-bottom))] sm:px-6"
				aria-live="polite"
			>
				<div class="flex flex-col divide-y divide-[rgba(23,23,23,0.12)]">
					{#each memos as memo (memo.id)}
						<MemoItem {memo} />
					{/each}
				</div>
			</section>
		</div>
	</div>
</main>
