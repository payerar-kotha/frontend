<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { browser } from '$app/environment';

    import { PUBLIC_BASEURL_PROD } from '$env/static/public';
    import Modal from "../components/Modal.svelte";
    import MessageBox from "./messageBox.svelte";

    let isActiveInChannel = $state(false);
    let createChannelModalShow = $state(false);
    let joinChannelModalShow = $state(false);
    let channelName = $state('');
    let passcode = $state('');
    let name = $state('');
    let id = '';

    let socket: WebSocket;
    let messages: { message: string; sender: string; id: string; timestamp: string }[] = $state([]);
    let msg = $state('');

    const createChannelModalClosedCB = () => {
        createChannelModalShow = false;
    }

    const joinChannelModalClosedCB = () => {
        joinChannelModalShow = false;
    }

    const createChannel = () => {
        connectToTopic('create');
        createChannelModalShow = false;
        channelName = '';
        passcode = '';
        name = '';
    }

    const joinChannel = () => {
        connectToTopic('join');
        joinChannelModalShow = false;
        channelName = '';
        passcode = '';
        name = '';
    }

    const connectToTopic = (mode: 'create' | 'join') => {
        if(socket) socket.close();
        messages = [];
        socket = new WebSocket(`wss://${PUBLIC_BASEURL_PROD}/${mode}-channel/${channelName}?passcode=${passcode}&name=${name}&id=${id}`);

        socket.onmessage = (event) => {
            const parsedData = JSON.parse(event.data);
            if(parsedData.type == "INIT__TYPE") {
                const {payload} = parsedData;
                id = payload.id;
                channelName = payload.topic;
                passcode = payload.passcode;
                name = payload.name;
                localStorage.setItem('id', id);
                localStorage.setItem('topic', channelName);
                localStorage.setItem('passcode', passcode);
                localStorage.setItem('name', name);
                isActiveInChannel = true;
                return;
            }
            else if(parsedData.type == "MESSAGE__TEMP") {
                const {payload} = parsedData;
                messages = [...messages, payload];
            }
        };

        socket.onclose = (e) => {
            console.log('WebSocket connection closed', e);
            if(e.code == 1008) {
                localStorage.clear();
                isActiveInChannel = false;
            }
        };

        socket.onerror = (error) => {
            console.error('WebSocket error:', error);
        };

    }

    const publishMessage = () => {
        msg = msg.trim();
        if(socket && socket.readyState === WebSocket.OPEN) {
            socket.send(JSON.stringify({type: "MESSAGE__TEMP", payload: {
                message: msg,
                sender: name || 'Anonymous',
                id,
                timestamp: new Date().toISOString()
            }}));
            msg = '';
        }
    }

    const reconnect = () => {
        if (!socket || socket.readyState === WebSocket.CLOSED) {
            if(!channelName) channelName = localStorage.getItem('topic') || '';
            if(!passcode) passcode = localStorage.getItem('passcode') || '';
            if(!name) name = localStorage.getItem('name') || '';
            if(!id) id = localStorage.getItem('id') || '';
            connectToTopic('join');
        }
    };

    onMount(() => {
        if(!browser) return;
        if(!localStorage.getItem('topic') || !localStorage.getItem('passcode')){
            localStorage.clear();
            isActiveInChannel = false;
            return;
        }
        if(!channelName) channelName = localStorage.getItem('topic') || '';
        if(!passcode) passcode = localStorage.getItem('passcode') || '';
        if(!name) name = localStorage.getItem('name') || '';
        if(!id) id = localStorage.getItem('id') || '';
        connectToTopic('join');
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

{#if !isActiveInChannel}
    <div>
        <button disabled>Log in and go back to your saved chats.</button>
        <button onclick={() => createChannelModalShow = true}>Create New Anonymous Channel</button>
        <button onclick={() => joinChannelModalShow = true}>Join Existing Anonymous Channel</button>
    </div>

    {#if createChannelModalShow}
        <Modal onClose={() => createChannelModalClosedCB()} >
            <h2>Create New Anonymous Channel</h2>
            <input type="text" placeholder="Channel Name" bind:value={channelName} />
            <input type="text" placeholder="Passcode (optional)" bind:value={passcode} />
            <input type="text" placeholder="Your Name (optional)" bind:value={name} />
            <button onclick={() => createChannel()}>Create Channel</button>
        </Modal>
    {/if}

    {#if joinChannelModalShow}
        <Modal onClose={() => joinChannelModalClosedCB()} >
            <h2>Join Existing Anonymous Channel</h2>
            <input type="text" placeholder="Channel Name" bind:value={channelName} />
            <input type="text" placeholder="Passcode" bind:value={passcode} />
            <input type="text" placeholder="Your Name (optional)" bind:value={name} />
            <button onclick={() => joinChannel()}>Join Channel</button>
        </Modal>
    {/if}
{:else}
    <div style="background-color: lightgray; padding: 20px;">
        <div style="float: left;">typing...</div>
        <button style="float: right;" onclick={() => {
            if(socket) socket.close();
            localStorage.clear();
            isActiveInChannel = false;
        }}>Leave Channel</button>
    </div>
    <div>
        <input type="text" bind:value={msg} placeholder="Type something..." />
        <button onclick={publishMessage}>Submit</button>

        <div>
            <h4>Messages:</h4>
            <div style="width: 100%; display: flex; flex-direction: column; gap: 10px;">
                {#each messages as message}
                    <MessageBox {...message} />
                {/each}
            </div>
        </div>
    </div>
{/if}