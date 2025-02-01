<script lang="ts">
    import { flip } from "svelte/animate";
    import { fade } from "svelte/transition";
    let placedCards: number[] = [];
    let isCleared = false;

    export function addCard(cardNumber: number) {
        if (isCleared) {
            placedCards = []; // Clear old cards when starting new game
            isCleared = false;
        }
        placedCards = [...placedCards, cardNumber];
    }

    export function clear() {
        isCleared = true;
    }
</script>

<div
    class="nes-container with-title shadow-[inset_0_0_10px_#0004] bg-[#76C44233] transition-[height] duration-300"
    style="min-height: {placedCards.length ? '200px' : '100px'}"
>
    <p class="title font-bold -translate-y-2 scale-120 border-3">Placed</p>
    <div class="grid grid-cols-13 gap-2">
        {#each placedCards as cardNumber (cardNumber)}
            <img
                animate:flip={{ duration: 300 }}
                in:flip
                out:fade={{ duration: 200 }}
                draggable="false"
                class="transition-opacity duration-200 {isCleared
                    ? 'opacity-30 hover:opacity-100 hover:rotate-0 hover:scale-100'
                    : 'hover:scale-[1.1] hover:rotate-6'}"
                src="/images/{cardNumber}.webp"
                alt="card {cardNumber}"
            />
        {/each}
    </div>
</div>
