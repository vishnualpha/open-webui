<!-- SharedChats.svelte -->
<script>
    import { onMount, onDestroy } from 'svelte';
    import { getAllSharedChats } from '$lib/apis/chats';
    import { getUserById } from '$lib/apis/users';
  
    let sharedChats = [];
    let page = 0;
    let loading = false;
    const limit = 10;
    let interval;
  
    const truncateText = (text, length)=> {
        return text.length > length ? text.substring(0, length) + '...' : text;
    }
    const loadMoreChats = async () => {
      if (loading) return;
      loading = true;
      try {
        const newChats = await getAllSharedChats(localStorage.token, page, limit);
        await Promise.all(newChats.map(async (chat) => {
          try {
            const response = await getUserById(localStorage.token, chat.user_id);
            chat.user = response;
          } catch (error) {
            chat.user = { name: 'Unknown User', profile_image_url: 'default-profile.png' };
          }
        }));
        sharedChats = [...sharedChats, ...newChats];
        page += 1;
      } catch (error) {
        console.error('Failed to load more chats:', error);
      } finally {
        loading = false;
      }
    };
  
    const onScroll = () => {
      const windowHeight = window.innerHeight || document.documentElement.offsetHeight;
      const docHeight = Math.max(document.body.scrollHeight, document.body.offsetHeight, document.documentElement.clientHeight, document.documentElement.scrollHeight, document.documentElement.offsetHeight);
      const windowBottom = windowHeight + window.pageYOffset;
      if (windowBottom >= docHeight - 100 && !loading) {
        loadMoreChats();
      }
    };
  
    const fetchPeriodically = () => {
      interval = setInterval(() => {
        if (!loading) {
          loadMoreChats();
        }
      }, 30000);
    };
  
    onMount(() => {
      loadMoreChats();
      fetchPeriodically();
      window.addEventListener('scroll', onScroll);
    });
  
    onDestroy(() => {
      clearInterval(interval);
      window.removeEventListener('scroll', onScroll);
    });
  </script>
  
  <div class="shared-chats">
    <ul class="tile-container">
      {#each sharedChats as chat}
        <li class="tile">
          <div class="user-info">
            <img class="profile-image" src={chat.user?.profile_image_url} alt="{chat.user?.name}'s profile image" />
            <span class="user-name">{chat.user?.name}</span>
          </div>
          <a href={"/s/" + chat.share_id} target="_blank" class="chat-title">{chat.title}</a>
          <div class="tile-footer">Model: {chat.chat.models}</div>
          <div class="tile-footer">Prompt: {chat.chat.messages[0].content.length>150?chat.chat.messages[0].content.substring(0,150)+"...":chat.chat.messages[0].content}
            {#if chat.chat.messages[0].content.length>150}
                <a href={"/s/" + chat.share_id} target="_blank">Show More</a>
            {/if}
          </div>
          <div class="tile-footer">Response: {chat.chat.messages[1]?.content.length>150?chat.chat.messages[1]?.content.substring(0,150)+"...":chat.chat.messages[1]?.content}
            {#if chat.chat.messages[1]?.content.length>150}
                <a href={"/s/" + chat.share_id} target="_blank">Show More</a>
            {/if}
          </div>
        </li>
      {/each}
    </ul>
    <!--{#if loading}
      <p>Loading...</p>
    {/if}-->
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
    }
  
    .user-info {
      display: flex;
      align-items: center;
      margin-bottom: 10px;
    }
  
    .profile-image {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      margin-right: 10px;
    }
  
    .user-name {
      font-weight: bold;
      color:#333;
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
  </style>
  