<script lang="ts">
    import Placed from "./Placed.svelte";
    let handInput = "";
    let handCards: number[] = [];
    let debounceTimer: number;
    let placedComponent: Placed;
    let playTimer: number;
    let counter = 0;
    export function addCardToBack(cardNumber: number) {
        handCards = [...handCards, cardNumber];
        updateInputFromCards();
    }

    export function addCardToFront(cardNumber: number) {
        handCards = [cardNumber, ...handCards];
        updateInputFromCards();
    }

    function updateInputFromCards() {
        handInput = handCards
            .map((num) => {
                const suit = ["D", "C", "H", "S"][Math.floor(num / 13)];
                const value = (num % 13) + 1;
                return `${suit}${value}`;
            })
            .join(" ");
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
        if (placedComponent) placedComponent.clear();
    }

    function playOne() {
        if (handCards.length === 0) return;
        const [first, ...rest] = handCards;
        if (handCards.length <= 1) counter = 0;
        if (counter % 2 === 0) {
            placedComponent.addCard(first);
            // Place the card
            handCards = rest;
        } else {
            // Move to back
            handCards = [...rest, first];
        }
        counter++;
        updateInputFromCards();
    }

    function playAll() {
        if (handCards.length === 0) return;
        playOne();
        playTimer = setTimeout(playAll, 100);
    }
</script>

<div class="mb-4 space-y-2">
    <textarea
        bind:value={handInput}
        on:input={updateHand}
        placeholder="Enter cards in your hand (e.g. D13 H1 S11)"
        class="w-full p-2 border border-stone-300 rounded h-[42px] min-h-[42px] resize-y"
    />
    <div class="flex gap-3 items-center">
        <button on:click={clearHand}>Clear All</button>
        <button on:click={playOne}>Play One</button>
        <button on:click={playAll}>Play All</button>
        <p>
            Click on cards on the table to add to your hand. Hold *shift* to add
            the the back of your hand
        </p>
    </div>
</div>

<Placed bind:this={placedComponent} />
<h1 class="text-xl mb-4 mt-5">Hand</h1>
<div
    class="grid grid-cols-13 gap-2 p-12 border-dashed border-2 border-red-500 bg-red-100/50 shadow-[inset_0_0_10px_#0003]"
>
    {#each handCards as cardNumber, index}
        <img
            draggable="false"
            class={index == 0
                ? counter % 2 === 0
                    ? "outline-amber-500 outline-4 outline-offset-2"
                    : "outline-lime-500 outline-4 outline-offset-2"
                : ""}
            src="/images/{cardNumber}.webp"
            alt="card {cardNumber}"
        />
    {/each}
</div>
