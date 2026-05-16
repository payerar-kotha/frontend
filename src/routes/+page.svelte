<script lang="ts">
    import { onDestroy } from 'svelte';
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
    }

    const publishMessage = () => {
        msg = msg.trim();
        if(socket && socket.readyState === WebSocket.OPEN) {
            socket.send(msg);
            msg = '';
        }
    }

    connectToTopic();

    onDestroy(() => {
        if(socket) socket.close();
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