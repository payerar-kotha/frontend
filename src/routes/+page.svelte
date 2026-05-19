<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { browser } from '$app/environment';
    import { PUBLIC_BASEURL_PROD } from '$env/static/public';

    let socket: WebSocket;
    let messages: string[] = $state([]);
    let msg = $state('');

    const connectToTopic = () => {
        if(socket) socket.close();
        messages = [];
        socket = new WebSocket(`wss://${PUBLIC_BASEURL_PROD}/ws/technology`);

        socket.onmessage = (event) => {
            messages = [...messages, event.data];
        };

        socket.onclose = () => {
            console.log('WebSocket connection closed');
        };

        socket.onerror = (error) => {
            console.error('WebSocket error:', error);
        };

    }

    const publishMessage = () => {
        msg = msg.trim();
        if(socket && socket.readyState === WebSocket.OPEN) {
            socket.send(msg);
            msg = '';
        }
    }

    const reconnect = () => {
    if (!socket || socket.readyState === WebSocket.CLOSED) {
        connectToTopic();
    }
};

    onMount(() => {
        if(!browser) return;
        connectToTopic();
        messages = [];
        window.addEventListener('online', reconnect);
        window.addEventListener('focus', reconnect);
    });

    onDestroy(() => {
        if(!browser) return;
        if(socket) socket.close();
        window.removeEventListener('online', reconnect);
        window.removeEventListener('focus', reconnect);
    });
</script>

<div style="background-color: lightgray; padding: 20px;">
    typing...
</div>
<div>
    <input type="text" bind:value={msg} placeholder="Type something..." />
    <button onclick={publishMessage}>Submit</button>

    <div>
        <h4>Messages:</h4>
        <ul>
            {#each messages as message}
                <li>{message}</li>
            {/each}
        </ul>
    </div>
</div>