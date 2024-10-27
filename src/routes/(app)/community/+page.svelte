<script>
	import { onMount, onDestroy } from 'svelte';
	import { getAllSharedChats } from '$lib/apis/chats';
	import { getUserById } from '$lib/apis/users';
	import { synthesizeOpenAISpeech } from '$lib/apis/audio';
	let sharedChats = [];
	let page = 0;
	let loading = false;
	const limit = 10;
	let interval;
	let audioStates = {}; // Store audio state for each chat by its share_id

	function createAudioState() {
		return {
			currentAudio: null,
			isPlaying: false,
			audioParts: [],
			currentAudioIdx: 0,
			pausedPosition: 0
		};
	}

	function stopAllAudio() {
	  Object.values(audioStates).forEach((audioState) => {
	    if (audioState.isPlaying && audioState.currentAudio) {
	      audioState.currentAudio.pause();
	      audioState.currentAudio.currentTime = 0;
	      audioState.isPlaying = false;
	      audioState.currentAudio = null;
	    }
	  });
	}

	const loadMoreChats = async () => {
		if (loading) return;
		loading = true;
		try {
			const newChats = await getAllSharedChats(localStorage.token, page, limit);
			await Promise.all(
				newChats.map(async (chat) => {
					try {
						const response = await getUserById(localStorage.token, chat.user_id);
						chat.user = response;
					} catch (error) {
						chat.user = { name: 'Unknown User', profile_image_url: 'default-profile.png' };
					}
					audioStates[chat.share_id] = createAudioState(); // Initialize audio state for each chat
				})
			);
			sharedChats = [...sharedChats, ...newChats];
			page += 1;
		} catch (error) {
			console.error('Failed to load more chats:', error);
		} finally {
			loading = false;
		}
	};

	onMount(() => {
		loadMoreChats();
		window.addEventListener('scroll', onScroll);
	});

	onDestroy(() => {
		clearInterval(interval);
		window.removeEventListener('scroll', onScroll);
		stopAllAudio(); // Stop any playing audio when the component is destroyed
	});

	const onScroll = () => {
		const windowHeight = window.innerHeight || document.documentElement.offsetHeight;
		const docHeight = Math.max(
			document.body.scrollHeight,
			document.body.offsetHeight,
			document.documentElement.clientHeight,
			document.documentElement.scrollHeight,
			document.documentElement.offsetHeight
		);
		const windowBottom = windowHeight + window.pageYOffset;
		if (windowBottom >= docHeight - 100 && !loading) {
			loadMoreChats();
		}
	};

	async function playPauseAudio(chatId) {
	const audioState = audioStates[chatId];

	if (audioState.isPlaying && audioState.currentAudio) {
		audioState.pausedPosition = audioState.currentAudio.currentTime;
		audioState.currentAudio.pause();
		audioState.isPlaying = false;
		audioStates = { ...audioStates };  // Trigger reactive update
		return;
	}

	stopAllAudio(); // Stop all other audio before playing

	audioState.audioParts = []; // Initialize in case of replay
	audioState.currentAudioIdx = 0;
	audioState.pausedPosition = 0;
	audioState.isPlaying = true;
	audioStates = { ...audioStates };  // Trigger reactive update

	const chat = sharedChats.find((chat) => chat.share_id === chatId);

	function chunkText(text, sentencesPerChunk = 3) {
	const sentenceEndRegex = /(?<=[.!?])\s+/g;
	const sentences = text.split(sentenceEndRegex);
	const chunks = [];

	for (let i = 0; i < sentences.length; i += sentencesPerChunk) {
		chunks.push(sentences.slice(i, i + sentencesPerChunk).join(' '));
	}
	
	return chunks;
}

async function synthesizeAndPlay(index) {
    if (index >= chat.chat.messages.length) {
        audioState.isPlaying = false;
        audioStates = { ...audioStates };  // Update state
        return;
    }

    const message = chat.chat.messages[index];
    const voice = index % 2 === 0 ? 'nova' : 'alloy';
    const content = message.content || '';
    if (!content.trim()) {
        synthesizeAndPlay(index + 1);
        return;
    }

    const contentChunks = chunkText(content);

    try {
        for (let chunk of contentChunks) {
            const res = await synthesizeOpenAISpeech(localStorage.token, voice, chunk);
            if (res) {
                const blob = await res.blob();
                const blobUrl = URL.createObjectURL(blob);
                audioState.audioParts.push(blobUrl);

                audioState.currentAudio = new Audio(blobUrl);
                audioState.currentAudio.currentTime = audioState.pausedPosition;
                await new Promise((resolve) => {
                    audioState.currentAudio.onended = resolve;
                    audioState.currentAudio.play();
                });
            }
        }
        audioState.pausedPosition = 0;
        audioState.currentAudioIdx++;
        synthesizeAndPlay(index + 1);
    } catch (error) {
        console.error('Error synthesizing speech:', error);
        synthesizeAndPlay(index + 1); // Continue with the next message
    }
}


	synthesizeAndPlay(audioState.currentAudioIdx);
}

	async function playCurrentAudio(audioState) {
		if (audioState.currentAudioIdx < audioState.audioParts.length) {
			audioState.currentAudio = new Audio(audioState.audioParts[audioState.currentAudioIdx]);
			audioState.currentAudio.currentTime = audioState.pausedPosition;
			audioState.currentAudio.play();
			audioState.isPlaying = true;

			audioState.currentAudio.onended = () => {
				audioState.pausedPosition = 0;
				audioState.currentAudioIdx++;
				if (audioState.currentAudioIdx < audioState.audioParts.length) {
					playCurrentAudio(audioState);
				} else {
					audioState.currentAudioIdx = 0;
					audioState.isPlaying = false;
					audioState.currentAudio = null;
					audioStates = { ...audioStates };  // Ensure state change is visible
				}
			};
		}
	}

	function stopAudio(chatId) {
		const audioState = audioStates[chatId];
		if (audioState.currentAudio) {
			audioState.currentAudio.pause();
			audioState.currentAudio.currentTime = 0;
			audioState.pausedPosition = 0;
			audioState.currentAudioIdx = 0;
			audioState.isPlaying = false;
			audioState.currentAudio = null;
			audioStates = { ...audioStates };  // Ensure state change is visible
		}
	}
</script>


<div class="shared-chats">
	<ul class="tile-container">
		{#each sharedChats as chat (chat.share_id)}
			<li class="tile">
        {#if audioStates[chat.share_id]?.isPlaying}
          <div class="wave-container">
              <div class="wave"></div>
              <div class="wave"></div>
              <div class="wave"></div>
          </div>
        {/if}
				<div class="user-info">
					<img
						class="profile-image"
						src={chat.user?.profile_image_url}
						alt="{chat.user?.name}'s profile image"
					/>
					<span class="user-name">{chat.user?.name}</span>
				</div>
				<a href={'/s/' + chat.share_id} target="_blank" class="chat-title">{chat.title}</a>
				<div class="tile-footer">
					Model: {chat.chat.models}
				</div>
				<div class="tile-footer">
					Prompt: {chat.chat.messages[0].content.length > 150
						? chat.chat.messages[0].content.substring(0, 150) + '...'
						: chat.chat.messages[0].content}
				</div>
				<div class="play-info">
					<button on:click={() => playPauseAudio(chat.share_id)}>
            {#if !audioStates[chat.share_id]?.isPlaying}
            <svg width="30px" height="30px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd" clip-rule="evenodd" d="M12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22ZM10.6935 15.8458L15.4137 13.059C16.1954 12.5974 16.1954 11.4026 15.4137 10.941L10.6935 8.15419C9.93371 7.70561 9 8.28947 9 9.21316V14.7868C9 15.7105 9.93371 16.2944 10.6935 15.8458Z" fill="#1C274C"/>
              </svg>
            
            {/if}
            {#if audioStates[chat.share_id]?.isPlaying}
            <svg width="30px" height="30px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect width="24" height="24" fill="white"/>
              <path fill-rule="evenodd" clip-rule="evenodd" d="M2 12C2 6.47715 6.47715 2 12 2C17.5228 2 22 6.47715 22 12C22 17.5228 17.5228 22 12 22C6.47715 22 2 17.5228 2 12ZM14 8C14.5523 8 15 8.44772 15 9L15 15C15 15.5523 14.5523 16 14 16C13.4477 16 13 15.5523 13 15L13 9C13 8.44772 13.4477 8 14 8ZM10 8C10.5523 8 11 8.44772 11 9L11 15C11 15.5523 10.5523 16 10 16C9.44771 16 9 15.5523 9 15L9 9C9 8.44772 9.44772 8 10 8Z" fill="#323232"/>
              </svg>
            
            {/if}
						
					</button>
					<!----<button on:click={() => stopAudio(chat.share_id)}>
            <svg width="30px" height="30px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd" clip-rule="evenodd" d="M12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22ZM8.58579 8.58579C8 9.17157 8 10.1144 8 12C8 13.8856 8 14.8284 8.58579 15.4142C9.17157 16 10.1144 16 12 16C13.8856 16 14.8284 16 15.4142 15.4142C16 14.8284 16 13.8856 16 12C16 10.1144 16 9.17157 15.4142 8.58579C14.8284 8 13.8856 8 12 8C10.1144 8 9.17157 8 8.58579 8.58579Z" fill="#1C274C"/>
              </svg>
          </button>-->
				</div>
			</li>
		{/each}
	</ul>
</div>

<style>
	.tile-container {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(360px, 1fr));
		gap: 1rem;
		padding: 0;
		list-style: none;
	}

	.tile {
    background-color: #f9f9f9;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    position: relative;
    overflow: hidden;
}

.wave-container {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex; 
    align-items: center;
    justify-content: center;
    pointer-events: none;  /* Prevent interaction with the wave */
    z-index: 1;
}

.wave {
    width: 8px;
    height: 30px;
    background: rgba(52, 152, 219, 0.8);
    margin: 0 2px;
    border-radius: 4px;
    animation: wave 1.2s linear infinite;
}

.wave:nth-child(2) {
    animation-delay: 0.2s;
}

.wave:nth-child(3) {
    animation-delay: 0.4s;
}

@keyframes wave {
    0%, 100% {
        height: 10px;
    }
    50% {
        height: 30px;
    }
}

	.user-info {
		display: flex;
		align-items: center;
		margin-bottom: 10px;
	}

  .play-info {
		display: flex;
		align-items: center;
    justify-content: flex-end;
    position:absolute;
    bottom:10px;
    right:10px;
		
	}

	.profile-image {
		width: 40px;
		height: 40px;
		border-radius: 50%;
		margin-right: 10px;
	}

	.user-name {
		font-weight: bold;
		color: #333;
	}

	.chat-title {
		font-size: 1.2rem;
		color: #333;
		text-decoration: none;
		margin-top: 5px;
	}

	.chat-title:hover {
		text-decoration: underline;
	}

	.tile-footer {
		margin-top: 8px;
		font-size: 0.9rem;
		color: #666;
	}

	.tile-footer button {
		background-color: #3498db;
		color: white;
		border: none;
		border-radius: 5px;
		padding: 5px 10px;
		margin-top: 5px;
		margin-right: 5px;
		cursor: pointer;
		transition: background-color 0.3s;
	}

	.tile-footer button:hover {
		background-color: #2980b9;
	}
</style>
