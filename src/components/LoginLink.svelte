<script>
    import user from "../stores/users";
    import { link } from "svelte-routing";

    import { logoutUser } from "../stores/users";
    import globalStore from "../stores/globalStore";
</script>

{#if $user.jwt}
    <a
        href="/"
        class="logout-btn"
        on:click={() => {
            logoutUser();
            globalStore.toggleItem('sidebar', false);
        }}>logout</a>
{:else}
    <a
        href="/login"
        on:click={() => {
            globalStore.toggleItem('sidebar', false);
        }}>login</a>
{/if}
