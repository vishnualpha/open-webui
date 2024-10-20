<script lang="ts">
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	onMount(() => {
		goto('/community');
	});

	// Fetch canonical URL or fallback to the current location
	var canonicalUrl = document.querySelector("link[rel='canonical']") ? document.querySelector("link[rel='canonical']").href : window.location.href;

	// Ensure canonicalUrl is logged for debugging purposes
	console.log("Canonical URL:", canonicalUrl);

	window.DiscourseEmbed = {
		discourseUrl: 'https://community.thealpha.dev/', // Ensure trailing slash
		discourseEmbedUrl: canonicalUrl, // Automatically sets the canonical URL
		// className: 'CLASS_NAME', // Uncomment and replace with your CSS class if needed
	};

	(function() {
		var d = document.createElement('script'); 
		d.type = 'text/javascript'; 
		d.async = true;
		
		// Make sure the URL is concatenated correctly
		d.src = DiscourseEmbed.discourseUrl + 'javascripts/embed-topics.js';
		
		(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(d);
	})();
</script>

<d-topics-list discourse-url="https://community.thealpha.dev/"></d-topics-list>
<meta name='discourse-username' content='alphadude'>
