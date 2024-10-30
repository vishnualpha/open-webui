<script lang="ts">
	import { WEBUI_BASE_URL } from '$lib/constants';
	import { WEBUI_NAME, LANDING_MESSAGE, LANDING_SUBMESSAGE } from '$lib/stores';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { track } from '@amplitude/analytics-browser';

	// Track page visit
	onMount(() => {
		track('Page Visited', {
			page: window.location.pathname,
			referrer: document.referrer,
			timestamp: new Date().toISOString(),
		});
	});

	const trackLinkClick = (linkName) => {
		track('Link Clicked', {
			linkName,
			timestamp: new Date().toISOString(),
		});
	};

	const trackChatAreaFocus = () => {
		track('Chat Area Focused', {
			timestamp: new Date().toISOString(),
		});
	};

	const trackChatAreaSubmission = () => {
		track('Chat Area Submission', {
			messageLength: prompt.trim().length,
			timestamp: new Date().toISOString(),
		});
	};
	let mounted = false;
	let roles = ['best Software Professional', 'best Developer', 'best Quality Engineer', 'best DevOps Engineer', 'Alpha'];
	let currentRoleIndex = 0;
	let roleText = '';
	let showCursor = true;
	const staticText = 'Be the';
	let prompt = '';
	let chatTextAreaElement;
	let suggestions = ['How do I improve my skills?', 'Can you guide me in DevOps?', 'Help me write clean code'];

	const typeRole = async (role) => {
		showCursor = true;
		for (let i = 0; i < role.length; i++) {
			roleText += role[i];
			await new Promise((r) => setTimeout(r, 100));
		}
		await new Promise((r) => setTimeout(r, 1000));
	};

	const cycleRoles = async () => {
		while (true) {
			roleText = '';
			await typeRole(roles[currentRoleIndex]);
			currentRoleIndex = (currentRoleIndex + 1) % roles.length;
		}
	};

	const handlePromptSubmission = () => {
		if (prompt.trim()) {
			localStorage.setItem('selectedPrompt', prompt);
			goto('/auth');
		}
	};

	const adjustTextareaHeight = (event) => {
		const element = event.target;
		element.style.height = '48px';
		element.style.height = `${element.scrollHeight}px`;
	};

	const setPromptFromSuggestion = (suggestedPrompt) => {
		prompt = suggestedPrompt;
		chatTextAreaElement?.focus();
	};

	onMount(() => {
		mounted = true;
		chatTextAreaElement?.focus();
		cycleRoles();
	});
</script>

<svelte:head>
	<title>{$WEBUI_NAME}</title>
</svelte:head>

{#key mounted}
	<div class="relative min-h-screen bg-white dark:bg-gray-950 font-primary">

		<div class="fixed top-10 left-4 md:left-10">
			<a href="/">
				<img
					crossorigin="anonymous"
					src="{WEBUI_BASE_URL}/static/logo.png"
					class="w-20 md:w-36 rounded-full"
					alt="logo"
				/>
			</a>
		</div>

		<div class="fixed top-10 right-4 md:right-10">
			<button
				class="bg-gray-900 hover:bg-gray-800 rounded-2xl text-white font-medium text-sm py-2 px-4 transition"
				on:click={() => {
					trackLinkClick('auth');
					goto('/auth?mode=signin')
				}}
			>
				Sign In
			</button>
		</div>

		<div class="flex justify-center items-center min-h-screen px-4 md:px-0">
			<div class="max-w-4xl w-full text-center">
				<div class="text-3xl md:text-6xl text-gray-800 dark:text-gray-100 font-large mb-8">
					<span class="gradient-text">{$LANDING_MESSAGE}</span>
				</div>

				<div class="text-sm md:text-md text-gray-600 dark:text-gray-400 mb-8">
					{$LANDING_SUBMESSAGE}
				</div>

				<div class="bg-white dark:bg-gray-900 p-4 md:p-6 rounded-3xl">
					<form class="flex flex-col md:flex-row gap-1.5" on:submit|preventDefault={handlePromptSubmission}>
						<div class="flex-1 flex flex-col w-full rounded-3xl px-1.5 bg-gray-50 dark:bg-gray-850 dark:text-gray-100">
							<div class="flex">
								<div class="self-end mb-1.5 flex space-x-1">
									<button type="button" class="bg-gray-50 hover:bg-gray-100 text-gray-800 dark:bg-gray-850 dark:text-white dark:hover:bg-gray-800 transition rounded-full p-2" aria-label="More">
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="size-5">
											<path d="M8.75 3.75a.75.75 0 0 0-1.5 0v3.5h-3.5a.75.75 0 0 0 0 1.5h3.5v3.5a.75.75 0 0 0 1.5 0v-3.5h3.5a.75.75 0 0 0 0-1.5h-3.5v-3.5Z"></path>
										</svg>
									</button>
								</div>

								<textarea
									id="chat-textarea"
									bind:this={chatTextAreaElement}
									class="scrollbar-hidden bg-gray-50 dark:bg-gray-850 dark:text-gray-100 outline-none w-full py-3 px-1 rounded-xl resize-none h-[48px]"
									placeholder="Type something or Select a prompt from below to get started..."
									rows="1"
									bind:value={prompt}
									on:focus={trackChatAreaFocus}
									on:input={adjustTextareaHeight}
									on:keydown={(e) => {
										if (e.key === 'Enter' && !e.shiftKey) {
											e.preventDefault();
											trackChatAreaSubmission();
											handlePromptSubmission();
										}
									}}
								></textarea>

								<div class="self-end mb-2 flex space-x-1 mr-1">
									<button id="voice-input-button" class="text-gray-600 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-850 transition rounded-full p-1.5 self-center" type="button" aria-label="Voice Input">
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
											<path d="M7 4a3 3 0 016 0v6a3 3 0 11-6 0V4z"></path>
											<path d="M5.5 9.643a.75.75 0 00-1.5 0V10c0 3.06 2.29 5.585 5.25 5.954V17.5h-1.5a.75.75 0 000 1.5h4.5a.75.75 0 000-1.5h-1.5v-1.546A6.001 6.001 0 0016 10v-.357a.75.75 0 00-1.5 0V10a4.5 4.5 0 01-9 0v-.357z"></path>
										</svg>
									</button>
								</div>
								<div class="flex items-end w-10">
									<button
										class="{prompt.trim() !== '' ? 'bg-black text-white hover:bg-gray-900' : 'bg-gray-200 text-gray-600'} transition rounded-full p-1.5 self-center"
										type="submit"
										disabled={prompt.trim() === ''}
									>
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="size-5">
											<path
												fill-rule="evenodd"
												d="M8 14a.75.75 0 0 1-.75-.75V4.56L4.03 7.78a.75.75 0 0 1-1.06-1.06l4.5-4.5a.75.75 0 0 1 1.06 0l4.5 4.5a.75.75 0 0 1-1.06 1.06L8.75 4.56v8.69A.75.75 0 0 1 8 14Z"
												clip-rule="evenodd"
											/>
										</svg>
									</button>
								</div>
							</div>
						</div>

						
					</form>

					<div class="mt-4 text-sm md:text-base">
						{#each suggestions as suggestion, i}
							<button
								class="text-gray-800 dark:text-gray-200 underline hover:text-yellow-600"
								on:click={() => setPromptFromSuggestion(suggestion)}							>
								{suggestion}
							</button>
							{#if i !== suggestions.length - 1}
								<span> | </span>
							{/if}
						{/each}
					</div>
				</div>
			</div>
		</div>

		<footer class="absolute bottom-0 left-0 right-0 p-4 text-center text-xs md:text-sm text-gray-600 dark:text-gray-400">
			<a href="/privacy" class="hover:text-yellow-600">Privacy Policy</a> |
			<a href="/terms" class="hover:text-yellow-600">Terms of Service</a> |
			<a href="/contact" class="hover:text-yellow-600">Contact Us</a>
		</footer>
	</div>
{/key}

<style>
	.scrollbar-hidden::-webkit-scrollbar {
		display: none;
	}

	.min-h-screen {
		min-height: 100vh;
	}

	.gradient-text {
		background: linear-gradient(0deg, #FFB94F, #dfad1593);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.blinking-cursor {
		border-right: 2px solid #FFB94F;
		animation: blink 1s step-end infinite;
	}

	@keyframes blink {
		50% {
			border-color: transparent;
		}
	}

	@media (min-width: 768px) {
		.mt-4 {
			margin-left: 0;
		}
	}

	footer a {
		margin: 0 8px;
		text-decoration: none;
		color: inherit;
	}

	footer a:hover {
		color: #FFB94F;
	}
</style>


