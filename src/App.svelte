<script lang="ts">
    import autoAnimate from "@formkit/auto-animate";
    import Hand from "./lib/Hand.svelte";

    let cardInput = "";
    let cards: number[] = [];
    let debounceTimer: number;
    let handComponent: Hand;

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
</script>

<main class="p-10">
    <h1 class="text-4xl mb-2">A Card Game that's not related to calculus</h1>
    <h2 class="text-lg mb-5">
        You have to put a card down, then put a card at the back of your hand.
        The objective of the game is to determine the order to have the cards in
        your hand such that it comes out in the correct order.
    </h2>

    <div class="mb-4 space-y-2">
        <input
            type="text"
            bind:value={cardInput}
            on:input={updateCards}
            placeholder="Enter card codes (e.g. D13 H1 S11)"
            class="w-full p-2 border border-stone-300 rounded"
        />

        <div class="space-x-2">
            <button on:click={fillAllCards}> Place All Cards </button>
            <button on:click={fillOneSuite}> Place One Suite </button>
            <button on:click={clearCards}> Clear </button>
        </div>
    </div>

    <h1 class="text-xl mb-4">Table</h1>
    <div
        class="grid grid-cols-13 gap-2 p-12 border-dashed border-2 border-stone-400 bg-blue-100/50"
        use:autoAnimate
    >
        {#each cards as cardNumber, index}
            <img
                draggable="false"
                src="/images/{cardNumber}.webp"
                alt="card {cardNumber}"
                on:click={(e) => {
                    if (e.shiftKey) {
                        handComponent.addCardToFront(cardNumber);
                    } else {
                        handComponent.addCardToBack(cardNumber);
                    }
                    cards = cards.filter((_, i) => i !== index);
                }}
            />
        {/each}
    </div>
    <hr class="border-dashed my-10" />
    <Hand bind:this={handComponent} />
</main>
