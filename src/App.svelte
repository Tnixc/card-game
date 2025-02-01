<script lang="ts">
    import Hand from "./lib/Hand.svelte";
    import { flip } from "svelte/animate";
    import { fade } from "svelte/transition";

    let cardInput = "";
    let cards: number[] = [];
    let debounceTimer: number;
    let handComponent: Hand;
    let highlightedCardIndex: number | null = null;

    function parseCardCode(code: string): number | null {
        const suit = code[0].toUpperCase();
        const value = parseInt(code.slice(1));

        if (isNaN(value) || value < 1 || value > 13) return null;

        const suitOrder: Record<string, number> = {
            D: 0, // Diamonds
            C: 1, // Clubs
            H: 2, // Hearts
            S: 3, // Spades
        };

        if (!(suit in suitOrder)) return null;

        return suitOrder[suit] * 13 + (value - 1);
    }

    function updateCards() {
        clearTimeout(debounceTimer);
        debounceTimer = setTimeout(() => {
            const codes = cardInput.trim().split(/\s+/);
            cards = codes
                .map((code) => parseCardCode(code))
                .filter((num): num is number => num !== null);
        }, 300);
    }

    function fillAllCards() {
        cardInput = Array.from({ length: 52 }, (_, i) => {
            const suit = ["D", "C", "H", "S"][Math.floor(i / 13)];
            const value = (i % 13) + 1;
            return `${suit}${value}`;
        }).join(" ");
        updateCards();
    }

    function fillOneSuite() {
        const arr = [];
        for (let i = 1; i <= 13; i++) arr.push(`S${i}`);
        cardInput = arr.join(" ");
        updateCards();
    }

    function clearCards() {
        cardInput = "";
        cards = [];
    }

    async function calculateCorrectHand(ms: number) {
        // Store original deck order
        const deck = [...cards];
        let hand: number[] = [];

        // Clear any existing cards in hand
        handComponent.setHand([]);

        // Animate the process
        for (let i = deck.length - 1; i >= 0; i--) {
            // Highlight the card we're about to take
            highlightedCardIndex = i;
            await new Promise((r) => setTimeout(r, ms));

            // If we have cards in hand, move last card to front

            if (hand.length > 0) {
                handComponent.setFlash(true);
                const lastCard = hand.pop()!;
                hand.unshift(lastCard);
                await new Promise((r) => setTimeout(r, ms));
                handComponent.setFlash(false);
                handComponent.setHand(hand);
                await new Promise((r) => setTimeout(r, ms));
            }

            // Take card from deck and add to front of hand
            hand.unshift(deck[i]);
            handComponent.setHand(hand);
            await new Promise((r) => setTimeout(r, ms));
        }

        // Clear highlight
        highlightedCardIndex = null;
    }
</script>

<main class="p-10">
    <h1 class="text-4xl mb-2">A Card Game that's not related to calculus</h1>
    <h2 class="text-lg mb-5">
        You have to put a card down, then put a card at the back of your hand.
        The objective of the game is to determine the order to have the cards in
        your hand such that it comes out in the correct order.
    </h2>

    <div class="mb-4 space-y-2">
        <textarea
            bind:value={cardInput}
            on:input={updateCards}
            placeholder="Enter card codes (e.g. D13 H1 S11)"
            class="nes-textarea resize-y"
        ></textarea>

        <div class="space-x-2">
            <button class="nes-btn" on:click={fillAllCards}>
                Place All Cards
            </button>
            <button class="nes-btn" on:click={fillOneSuite}>
                Place One Suite
            </button>
            <button class="nes-btn is-error" on:click={clearCards}>
                Clear
            </button>
        </div>
    </div>

    <div
        class="nes-container with-title shadow-[inset_0_0_10px_#0004] bg-[#1F9CED22]"
    >
        <p class="title font-bold -translate-y-2 scale-120 border-3">Table</p>
        <div class="grid grid-cols-13 gap-2">
            {#each cards as cardNumber, index (cardNumber)}
                <img
                    animate:flip={{ duration: 300 }}
                    transition:fade={{ duration: 200 }}
                    class="hover:scale-[1.1] hover:rotate-6 hover:shadow-lg duration-200
                    {index === highlightedCardIndex
                        ? 'outline-4 outline-offset-2 outline-amber-500 shadow-2xl'
                        : ''}"
                    draggable="false"
                    src="/images/{cardNumber}.webp"
                    alt="card {cardNumber}"
                    on:click={(e) => {
                        if (e.shiftKey) {
                            handComponent.addCardToBack(cardNumber);
                        } else {
                            handComponent.addCardToFront(cardNumber);
                        }
                        cards = cards.filter((_, i) => i !== index);
                    }}
                />
            {/each}
        </div>
    </div>

    <div class="flex gap-2 items-center my-10">
        <button
            class="nes-btn is-primary"
            on:click={() => calculateCorrectHand(50)}
            >Generate Solution (fast)</button
        >
        <button class="nes-btn" on:click={() => calculateCorrectHand(500)}
            >Generate Solution (slow)</button
        >
        <hr class="border-2 grow" />
    </div>
    <Hand bind:this={handComponent} />
    <div class="mt-10">
        <a href="https://github.com/Tnixc/card-game" target="_blank">
            <i class="nes-octocat animate is-small scale-50"></i>
        </a>
        <a href="https://nostalgic-css.github.io/NES.css/" target="_blank">
            <i class="nes-jp-logo scale-75 -translate-y-5"></i>
        </a>
    </div>
</main>
