<script>
    let msg = $state('Hello world');
</script>

<div style="background-color: lightgray; padding: 20px;">
    typing...
</div>
<div>
    <input type="text" bind:value={msg} placeholder="Type something..." />
</div>