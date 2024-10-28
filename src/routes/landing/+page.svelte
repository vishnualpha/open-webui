<script lang="ts">
	import { WEBUI_BASE_URL  } from '$lib/constants';
	import { WEBUI_NAME, LANDING_MESSAGE, LANDING_SUBMESSAGE  } from '$lib/stores';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	let mounted = false;
	let roles = ['best Software Professional', 'best Developer', 'best Quality Engineer', 'best DevOps Engineer', 'Alpha']; // Array of roles
	let currentRoleIndex = 0; // Index to track the current role
	let roleText = ''; // Variable to display the role text with animation
	let showCursor = true; // Toggle to show/hide the blinking cursor
	const staticText = 'Be the'; // Constant text
	let prompt = ''; // Chat input binding
	let chatTextAreaElement; // To bind the text area element
	let suggestions = ['How do I improve my skills?', 'Can you guide me in DevOps?', 'Help me write clean code']; // Example prompts

	// Typing effect for role
	const typeRole = async (role) => {
		showCursor = true;
		for (let i = 0; i < role.length; i++) {
			roleText += role[i];
			await new Promise((r) => setTimeout(r, 100)); // Adjust typing speed
		}
		//showCursor = false;
		await new Promise((r) => setTimeout(r, 1000)); // Small pause before switching role
	};

	// Function to cycle through roles and animate them
	const cycleRoles = async () => {
		while (true) {
			roleText = ''; // Clear the text before typing a new role
			await typeRole(roles[currentRoleIndex]);
			currentRoleIndex = (currentRoleIndex + 1) % roles.length; // Move to the next role in a loop
		}
	};

	// Handle prompt submission
	const handlePromptSubmission = () => {
		if (prompt.trim()) {
			localStorage.setItem('selectedPrompt',prompt);
			goto('/auth');
		}
	};

	// Adjust textarea height dynamically
	const adjustTextareaHeight = (event) => {
		const element = event.target;
		element.style.height = '48px';
		element.style.height = `${element.scrollHeight}px`;
	};

	// Function to set prompt from suggestions
	const setPromptFromSuggestion = (suggestedPrompt) => {
		prompt = suggestedPrompt;
		chatTextAreaElement?.focus();
	};

	onMount(() => {
		mounted = true;
		chatTextAreaElement?.focus(); // Focus on text area element
		cycleRoles(); // Start cycling through roles with animation
	});
</script>

<svelte:head>
	<title>{$WEBUI_NAME}</title>
</svelte:head>

{#key mounted}
	<!-- Main container for the layout -->
	<div class="relative h-screen bg-white dark:bg-gray-950 font-primary">

		<!-- Logo in the top-left corner -->
		<div class="fixed top-10 left-10">
			<a href="/">
			<img
				crossorigin="anonymous"
				src="{WEBUI_BASE_URL}/static/logo.png"
				class="w-36 rounded-full"
				alt="logo"
			/>
			</a>
		</div>

		<!-- Centered chat section -->
		<div class="flex justify-center items-center h-full">
			<div class="max-w-4xl w-full px-8 lg:px-20 text-center">
				<!-- Greeting with animated role text -->
				<div class="text-7xl text-gray-800 dark:text-gray-100 font-large mb-8">
					<!--{staticText}
					<span class="gradient-text">{roleText}</span>
					{#if showCursor}
						<span class="blinking-cursor"></span>
					{/if}-->
					<span class="gradient-text">{$LANDING_MESSAGE}</span>
				</div>
				<!-- Explanation Below Greeting Text -->
				<div class="text-md text-gray-600 dark:text-gray-400 mb-8">
					{$LANDING_SUBMESSAGE}
				</div>

				<!-- Chat Input Form -->
				<div class="bg-white dark:bg-gray-900 p-6 rounded-3xl">
					<form class="w-full flex gap-1.5" on:submit|preventDefault={handlePromptSubmission}>
						<!-- Chat Input Field -->
						<div class="flex-1 flex flex-col relative w-full rounded-3xl px-1.5 bg-gray-50 dark:bg-gray-850 dark:text-gray-100">
							<div class="flex">
								<div class="ml-0.5 self-end mb-1.5 flex space-x-1">
									<!-- Add Prompt Button -->
									<button type="button" class="bg-gray-50 hover:bg-gray-100 text-gray-800 dark:bg-gray-850 dark:text-white dark:hover:bg-gray-800 transition rounded-full p-2 outline-none focus:outline-none" aria-label="More">
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="size-5">
											<path d="M8.75 3.75a.75.75 0 0 0-1.5 0v3.5h-3.5a.75.75 0 0 0 0 1.5h3.5v3.5a.75.75 0 0 0 1.5 0v-3.5h3.5a.75.75 0 0 0 0-1.5h-3.5v-3.5Z"></path>
										</svg>
									</button>
								</div>

								<!-- Textarea for Chat Input -->
								<textarea
									id="chat-textarea"
									bind:this={chatTextAreaElement}
									class="scrollbar-hidden bg-gray-50 dark:bg-gray-850 dark:text-gray-100 outline-none w-full py-3 px-1 rounded-xl resize-none h-[48px]"
									placeholder="Type something or Select a prompt from below to get started..."
									rows="1"
									bind:value={prompt}
									on:input={adjustTextareaHeight}
									on:keydown={(e) => {
										if (e.key === 'Enter' && !e.shiftKey) {
											e.preventDefault(); 
											handlePromptSubmission(); 
										}
									}}
								></textarea>

								<div class="self-end mb-2 flex space-x-1 mr-1">
									<!-- Voice Input Button -->
									<button id="voice-input-button" class="text-gray-600 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-850 transition rounded-full p-1.5 mr-0.5 self-center" type="button" aria-label="Voice Input">
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
											<path d="M7 4a3 3 0 016 0v6a3 3 0 11-6 0V4z"></path>
											<path d="M5.5 9.643a.75.75 0 00-1.5 0V10c0 3.06 2.29 5.585 5.25 5.954V17.5h-1.5a.75.75 0 000 1.5h4.5a.75.75 0 000-1.5h-1.5v-1.546A6.001 6.001 0 0016 10v-.357a.75.75 0 00-1.5 0V10a4.5 4.5 0 01-9 0v-.357z"></path>
										</svg>
									</button>
								</div>
							</div>
						</div>

						<!-- Send Message Button -->
						<div class="flex items-end w-10">
							<button
								class="{prompt.trim() !== '' ? 'bg-black text-white hover:bg-gray-900' : 'bg-gray-200 text-gray-600'} transition rounded-full p-1.5 m-0.5 self-center"
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
					</form>

					<!-- Prompt Suggestions -->
					<div class="mt-4">
						{#each suggestions as suggestion,i}
							<button
								class="text-gray-800 dark:text-gray-200 underline hover:text-yellow-600"
								on:click={() => setPromptFromSuggestion(suggestion)}
							>
								{suggestion}
							</button>
							{#if i !== suggestions.length - 1} <!-- Check if it's not the last item -->
								<span> | </span>
							{/if}
						{/each}
					</div>
				</div>
			</div>
		</div>

		<!-- Footer Links -->
		<footer class="absolute bottom-0 left-0 right-0 p-4 text-center text-sm text-gray-600 dark:text-gray-400">
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

	.h-screen {
		height: 100vh;
	}

	.flex-col {
		display: flex;
		flex-direction: column;
	}

	.justify-center {
		justify-content: center;
	}

	.items-center {
		align-items: center;
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

	.mt-4 {
		margin-left: -3rem;
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
