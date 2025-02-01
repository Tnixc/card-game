<script lang="ts">
    let placedCards: number[] = [];

    export function addCard(cardNumber: number) {
        placedCards = [...placedCards, cardNumber];
    }

    export function clear() {
        placedCards = [];
    }
</script>

<h1 class="text-xl mb-4">Placed</h1>
<div
    class="grid grid-cols-13 gap-2 p-12 border-dashed border-2 border-green-500 bg-green-100/50 shadow-[inset_0_0_10px_#0003]"
>
    {#each placedCards as cardNumber}
        <img
            draggable="false"
            src="/images/{cardNumber}.webp"
            alt="card {cardNumber}"
        />
    {/each}
</div>
