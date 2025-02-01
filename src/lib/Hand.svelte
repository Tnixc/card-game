<script lang="ts">
    import autoAnimate from "@formkit/auto-animate";

    let handInput = "";
    let handCards: number[] = [];
    let debounceTimer: number;
    function addCardToBack(cardNumber: number) {
        handCards = [...handCards, cardNumber];
    }

    function addCardToFront(cardNumber: number) {
        handCards = [cardNumber, ...handCards];
    }

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

    function updateHand() {
        clearTimeout(debounceTimer);
        debounceTimer = setTimeout(() => {
            const codes = handInput.trim().split(/\s+/);
            handCards = codes
                .map((code) => parseCardCode(code))
                .filter((num): num is number => num !== null);
        }, 300);
    }

    function clearHand() {
        handInput = "";
        handCards = [];
    }
</script>

<div class="mb-4 space-y-2">
    <input
        type="text"
        bind:value={handInput}
        on:input={updateHand}
        placeholder="Enter cards in your hand (e.g. D13 H1 S11)"
        class="w-full p-2 border border-stone-300 rounded"
    />

    <div class="space-x-2">
        <button on:click={clearHand}> Clear Hand </button>
    </div>
</div>

<h1 class="text-xl mb-4">Your Hand</h1>
<div
    class="grid grid-cols-13 gap-2 p-12 border-dashed border-2 border-stone-400 bg-green-100/50 relative"
    use:autoAnimate
>
    {#each handCards as cardNumber, index}
        <img
            src="/images/{cardNumber}.webp"
            alt="card {cardNumber}"
            draggable="false"
            class={`
                ${index === 0 ? "shadow-2xl outline-4 outline-amber-500" : ""}
                cursor-move transition-transform duration-200
            `}
        />
    {/each}
</div>
