<script>
    import { onMount } from "svelte";
    import { navigate, link } from "svelte-routing";
    import user from "../stores/users";
    import cart, { cartTotal } from "../stores/cart";
    import submitOrder from "../strapi/submitOrder.js";
    import globalStore from "../stores/globalStore";

    let name = "";
    $: isEmpty = !name || $globalStore.alert;

    //stripe variables
    let cardElement;
    let cardErrors;
    let card;
    let stripe;
    let elements;

    onMount(() => {
        if (!$user.jwt) {
            navigate("/");
            return;
        }
        if ($cartTotal > 0) {
            stripe = Stripe(
                "pk_test_51I7TdKGSgTK68w8hVcdrCQ0za2zeiwcnZqOJDlF1wkHfrMGhKERrM9EL7wSTL7W0E6LHPuuEKcfaTxoSw8JCbmhu00pKzWR9aj"
            );
            elements = stripe.elements();
            card = elements.create("card");
            card.mount(cardElement);
            card.addEventListener("change", function (event) {
                if (event.error) {
                    cardErrors.textContent = event.error.message;
                } else {
                    cardErrors.textContent = "";
                }
            });
        }
    });

    async function handleSubmit() {
        globalStore.toggleItem("alert", true, "Submitting order...");
        let response = await stripe
            .createToken(card)
            .catch((error) => console.log(error));
        const { token } = response;
        if (token) {
            const { id } = token;
            let order = await submitOrder({
                name,
                total: $cartTotal,
                items: $cart,
                stripeTokenId: id,
                userToken: $user.jwt,
            });

            //if order went through w/out errors
            if (order) {
                globalStore.toggleItem("alert", true, "order complete!");
                cart.set([]); //reset cart
                localStorage.setItem("cart", JSON.stringify([]));
                navigate("/"); //go to home page
            } else {
                globalStore.toggleItem(
                    "alert",
                    true,
                    "an error occured with the purchase. Please try again",
                    true
                );
            }
            //token.id
            //submit order
        } else {
        }
    }
</script>

{#if $cartTotal > 0}
    <section class="form">
        <h2 class="section-title">Checkout</h2>
        <form class="checkout-form" on:submit|preventDefault={handleSubmit}>
            <h3>order total: ${$cartTotal}</h3>
            <!-- single input -->
            <div class="form-control">
                <label for="name">your name </label>
                <input type="text" id="name" bind:value={name} />
            </div>
            <!-- end of single input-->
            <!-- stripe stuff -->

            <div class="stripe-input">
                <!-- info -->
                <label for="card-element">Credit / Debit card</label>
                <p class="stripe-info">
                    Test using this card#:
                    <span>4242 4242 4242 4242</span>
                    <br />
                    enter any 5 digits for the zip code
                    <br />
                    enter any 3 digits for CVC
                </p>
                <div id="card-element" bind:this={cardElement}>
                    <!--stripe-->
                </div>
                <div id="card-errors" bind:this={cardErrors} role="alert">
                    <!-- stripe -->
                </div>
            </div>

            <!-- end of stripe stuff -->
            <!-- error message -->
            {#if isEmpty}
                <p class="form-empty">Please fill out name field</p>
            {/if}
            <!-- submit button -->
            <button
                type="submit"
                class="btn btn-block btn-primary"
                disabled={isEmpty}
                class:disabled={isEmpty}>submit</button>
        </form>
    </section>
{:else}
    <div class="checkout-empty">
        <h2>your cart is empty</h2>
        <a href="/products" use:link class="btn btn-primary">Please add at least
            one product to the cart</a>
    </div>
{/if}
