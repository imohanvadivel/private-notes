<script lang="ts">
    import { onMount } from "svelte";
    import { Spinner } from "deskblocks";
    import { APP, DB, initApp } from "../lib/util";
    import NoteCard from "./NoteCard.svelte";

    type NotesData = {
        key: string;
        queriableValue: string;
        value: {
            id: string;
            module: string;
            ticketNumber: string;
            title: string;
            note: string;
        };
    };

    let isLoading = true;
    let agentId: string;
    let notesDatas: NotesData[];

    onMount(async () => {
        await initApp();
        refreshNotes();
        $APP?.instance.on("REFETCH", refreshNotes);
    });

    async function refreshNotes() {
        isLoading = true;

        await ZOHODESK.get("user").then((data) => {
            agentId = data.user.id;
        });

        await DB.get({ queriableValue: agentId }).then((data) => {
            notesDatas = data?.["database.get"]?.["data"];
            isLoading = false;
        });
    }
</script>

<main>
    {#if isLoading}
        <div class="spinner">
            <Spinner size="large" />
            <p>Loading Notes...</p>
        </div>
    {:else if notesDatas && notesDatas.length > 0}
        {#each notesDatas as noteData}
            <NoteCard {noteData} />
        {/each}
    {/if}
</main>

<style>
    .spinner {
        display: flex;
        justify-content: center;
        flex-direction: column;
        row-gap: 1rem;
        color: var(--db-color-text-tertiary);
        font-size: var(--db-fontsize-medium);
        font-family: var(--db-fontstack);
        align-items: center;
        height: 100vh;
        margin-top: -2rem;
    }
</style>
