<script>
    import loginUser from "../strapi/loginUser";
    import registerUser from "../strapi/registerUser";
    import { navigate } from "svelte-routing";
    import globalStore from "../stores/globalStore";

    let email = "";
    let password = "";
    let username = "default username";
    let isMember = true;
    //add alert
    $: isEmpty = !email || !password || !username || $globalStore.alert;

    //toggle member
    function toggleMember() {
        isMember = !isMember;
        if (!isMember) {
            username = "";
        } else {
            username = "default username";
        }
    }

    //handle submit
    async function handleSubmit() {
        //add alert
        globalStore.toggleItem("alert", true, "logging in...");
        let user;
        if (isMember) {
            user = await loginUser({ email, password });
        } else {
            user = await registerUser({ email, password, username });
        }

        //successful log in / register or not
        if (user) {
            navigate("/");
            globalStore.toggleItem(
                "alert",
                true,
                "Welcome back! Enjoy your shopping."
            );
            //add alert
            return;
        }
        //add alert
        globalStore.toggleItem(
            "alert",
            true,
            "Login failed! Please check the fields and try again.",
            true
        );
    }
</script>

<section class="form">
    <h2 class="section-title">
        {#if isMember}sign in{:else}register{/if}
    </h2>
    <form class="login-form" on:submit|preventDefault={handleSubmit}>
        <!-- single input-->
        <div class="form-control">
            <label for="email">email</label>
            <input type="email" id="email" bind:value={email} />
        </div>
        <!-- end of single input-->

        <!-- single input-->
        <div class="form-control">
            <label for="password">password</label>
            <input type="password" id="password" bind:value={password} />
        </div>
        <!-- end of single input-->

        <!-- single input-->
        {#if !isMember}
            <div class="form-control">
                <label for="username">username</label>
                <input type="text" id="username" bind:value={username} />
            </div>
        {/if}
        <!-- end of single input-->
        {#if isEmpty}
            <p class="form-empty">please fill out all form fields</p>
        {/if}

        <button
            type="submit"
            class="btn btn-block btn-primary"
            disabled={isEmpty}
            class:disabled={isEmpty}>
            submit
        </button>

        {#if isMember}
            <p class="register-link">
                Need to register?
                <button type="button" on:click={toggleMember}>click here</button>
            </p>
        {:else}
            <p class="register-link">
                Already a member?
                <button type="button" on:click={toggleMember}>click here</button>
            </p>
        {/if}
    </form>
</section>
