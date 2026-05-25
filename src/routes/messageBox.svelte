<script>
    let { message, sender, id, timestamp } = $props();
    let userid = localStorage.getItem('id');

    const isOwnMessage = userid === id;
</script>

<div class={isOwnMessage ? "right" : "left"}>
    {#if !isOwnMessage}
        <div><strong>{sender}:</strong></div>
    {/if}
    <div>{message}</div>
    <div style="font-size: 0.8em; color: gray;">{new Date(timestamp).toLocaleTimeString()}</div>
</div>

<style>
    .left, .right {
        background-color: orange;
        padding: 5px;
        box-sizing: border-box;
        color: black;
    }
    
    .left {
        align-self: flex-start;
    }

    .right {
        align-self: flex-end;
    }
</style>