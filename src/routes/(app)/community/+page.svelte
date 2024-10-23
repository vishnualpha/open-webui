<!-- SharedChats.svelte -->

<script>
    import { onMount } from 'svelte';
    import { getAllSharedChats } from '$lib/apis/chats';

    let sharedChats = [];
    let page = 0;
    let loading = false;
    const limit = 10;

    const loadMoreChats = async () => {
        if (loading) return;
        loading = true;
        try {
            const newChats = await getAllSharedChats(localStorage.token ,page, limit);
            sharedChats = [...sharedChats, ...newChats];
            page += 1;
        } catch (error) {
            console.error('Failed to load more chats:', error);
        } finally {
            loading = false;
        }
    };

    onMount(() => loadMoreChats());
</script>

<div class="shared-chats">
    <ul class="tile-container">
        {#each sharedChats as chat}
            <li class="tile">
                <a href={"/s/" + chat.share_id}>{chat.title}</a>
                <div class="tile-footer">Model: {chat.chat.models}</div>
                <div class="tile-footer">Prompt: {chat.chat.messages[0].content}</div>
                <div class="tile-footer">by {chat.user_id}</div>
            </li>
        {/each}
    </ul>
    {#if loading}
        <p>Loading...</p>
    {/if}
    <button on:click={loadMoreChats} disabled={loading}>
        Load More
    </button>
</div>

<style>
    .tile-container {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
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
        
    }

    .tile a {
        font-size: 1.2rem;
        color: #333;
        text-decoration: none;
    }

    .tile a:hover {
        text-decoration: underline;
    }

    .tile-footer {
        margin-top: 8px;
        font-size: 0.9rem;
        color: #666;
    }
</style>
