<script lang="ts">
    import Placed from "./Placed.svelte";
    let handInput = "";
    let handCards: number[] = [];
    let savedHandState: number[] = [];
    let hasStoredState = false;
    let debounceTimer: number;
    let placedComponent: Placed;
    let playTimer: number;
    let counter = 0;
    export let isSwapping = false;

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

    function storeHandState() {
        savedHandState = [...handCards];
        hasStoredState = true;
    }

    function revertHand() {
        handCards = [...savedHandState];
        updateInputFromCards();
        hasStoredState = false;
        counter = 0;
        if (placedComponent) placedComponent.clear();
    }

    function playOne() {
        if (handCards.length === 0) return;
        if (!hasStoredState) storeHandState();
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
        if (!hasStoredState) storeHandState();
        playOne();
        playTimer = setTimeout(playAll, 100);
    }

    function clearHand() {
        handInput = "";
        handCards = [];
        counter = 0;
        hasStoredState = false;
        updateInputFromCards();
    }

    function clearPlaced() {
        if (placedComponent) placedComponent.clear();
        counter = 0;
    }

    export function setHand(newHand: number[]) {
        handCards = newHand;
        updateInputFromCards();
        counter = 0;
        hasStoredState = false;
        if (placedComponent) placedComponent.clear();
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
        <button class="" on:click={clearHand}>Clear Hand</button>
        <button class="" on:click={revertHand} disabled={!hasStoredState}
            >Revert</button
        >
        <button class="" on:click={playOne}>Play One</button>
        <button class="" on:click={playAll}>Play All</button>
        <p>
            Click on cards on the table to add to your hand. Hold *shift* to add
            the the back of your hand
        </p>
    </div>
</div>

<Placed bind:this={placedComponent} />
<h1 class="text-xl mb-4 mt-5">Hand</h1>
<div
    class="grid grid-cols-13 gap-2 p-4 border-dashed border-2 border-purple-500 bg-purple-100/50 shadow-[inset_0_0_10px_#0003]"
>
    {#each handCards as cardNumber, index}
        <img
            draggable="false"
            class="{index === 0 && isSwapping
                ? 'outline-amber-500 outline-4 outline-offset-2 shadow-2xl'
                : index === handCards.length - 1 && isSwapping
                  ? 'outline-lime-500 outline-4 outline-offset-2 shadow-2xl'
                  : index === 0
                    ? counter % 2 === 0
                        ? 'outline-amber-500 outline-4 outline-offset-2 shadow-2xl'
                        : 'outline-lime-500 outline-4 outline-offset-2 shadow-2xl'
                    : ''} hover:scale-[1.1] hover:rotate-6 hover:shadow-lg duration-200"
            src="/images/{cardNumber}.webp"
            alt="card {cardNumber}"
        />
    {/each}
</div>
