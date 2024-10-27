<script lang="ts">
    import type { NotesData } from "../types/type";
    export let noteData: NotesData;

    function routeToTicket() {
        ZOHODESK.invoke("ROUTE_TO", {
            entity: noteData.value.module,
            id: noteData.value.id,
            page: "dv",
        });
    }
</script>

<button on:click={routeToTicket}>
    <header>
        <h2>
            {noteData.value.title}
        </h2>
        <span class="module">{noteData.value.module}</span>
        {#if noteData.value.module === "ticket"}
            <span>- #{noteData.value.ticketNumber}</span>
        {/if}
    </header>
    <p>{noteData.value.note}</p>
</button>

<style>
    button {
        outline: none;
        border: none;
        background-color: var(--db-color-bg);
        color: var(--db-color-text);
        font-size: var(--db-fontsize-medium);
        font-family: var(--db-fontstack);
        line-height: 1.5;
        padding: 1rem;
        width: 100%;
        border-bottom: 1px solid var(--db-color-border);
        text-align: left;
    }

    button:hover {
        background-color: var(--db-color-bg-hover);
        cursor: pointer;
    }

    h2 {
        font-weight: var(--db-fontweight-medium);
        max-height: 21px;
        overflow: hidden;
        text-overflow: ellipsis;
    }

    header span {
        color: var(--db-color-text-tertiary);
    }

    header {
        padding-bottom: 1rem;
    }

    p {
        display: -webkit-box;
        -webkit-box-orient: vertical;
        overflow: hidden;
        text-overflow: ellipsis;
        -webkit-line-clamp: 3;
        line-height: 1.5;
        max-height: 63px;
    }

    .module {
        text-transform: capitalize;
    }
</style>
