<script lang="ts">
    import type { Snippet } from "svelte";

    let { onClose, children }: { onClose: () => void, children: Snippet } = $props();

    const close = () => {
        onClose()
    };
</script>

<div class="backdrop">
    <div class="modal">
        <button class="close-button" onclick={() => close()}>×</button>
        {@render children()}
    </div>
</div>

<style>
    .backdrop {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .modal {
        background-color: white;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        position: relative;
    }

    .close-button {
        position: absolute;
        top: 10px;
        right: 10px;
        padding: 10px;
        box-sizing: border-box;
        background: none;
        border: none;
        font-size: 20px;
        cursor: pointer;
    }
</style>
