<script lang="ts">
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

<h1 class="text-xl mb-4">Placed</h1>
<div
    class="grid grid-cols-13 gap-2 p-4 border-emerald-500 border-2 border-dashed bg-emerald-100/50 shadow-[inset_0_0_10px_#0003]"
>
    {#each placedCards as cardNumber}
        <img
            draggable="false"
            class="transition-opacity duration-200 {isCleared
                ? 'opacity-30 hover:opacity-100 hover:rotate-0 hover:scale-100'
                : 'hover:scale-[1.1] hover:rotate-6'}"
            src="/images/{cardNumber}.webp"
            alt="card {cardNumber}"
        />
    {/each}
</div>
