<script lang="ts">
    import { onMount, setContext } from "svelte";
    import { Button, Spinner, IconButton, Icon } from "deskblocks";
    import { IconRefresh } from "deskblocks/icons";
    import { APP, DB, initApp } from "../lib/util";
    import type { NotesData } from "../types/type";
    import NoteCard from "./NoteCard.svelte";

    let App;
    setContext("App", App);

    let isLoading = true;
    let agentId: string;
    let notesDatas: NotesData[];
    let searchStr: string;

    onMount(async () => {
        App = await initApp();

        await ZOHODESK.get("user").then((data) => {
            agentId = data.user.id;
        });

        await DB.get({ queriableValue: agentId }).then((data) => {
            notesDatas = data?.["database.get"]?.["data"];
            isLoading = false;
        });

        $APP?.instance.on("REFETCH", refreshNotes);
    });

    function refreshNotes() {
        console.log("refreshing notes...");

        isLoading = true;
        DB.get({ queriableValue: agentId }).then((data) => {
            notesDatas = data?.["database.get"]?.["data"];
            isLoading = false;
        });
    }

    function handleFilter(event: Event) {
        let searchString = (event.target as HTMLInputElement).value;
        console.log({ searchString });

        if (!notesDatas || !searchString) return;
        if (searchString.length < 2) return;

        console.log("string length", searchString.length);
        

        if (searchString.length === 0) {
            refreshNotes();
            return;
        }
        isLoading = true;
        let filteredNotes = notesDatas.filter((noteData) => {
            return noteData.value.note.toLowerCase().includes(searchString.toLowerCase());
        });
        notesDatas = filteredNotes;

        isLoading = false;
    }
</script>

<main>
    {#if isLoading}
        <div class="spinner">
            <Spinner size="large" />
            <p>Loading Notes...</p>
        </div>
    {:else if notesDatas && notesDatas.length > 0}
        <header>
            <input on:input={handleFilter} type="search" placeholder="Search Notes" />

            <IconButton variant="tertiary" on:click={refreshNotes}>
                <Icon icon={IconRefresh} />
            </IconButton>
        </header>
        {#each notesDatas as noteData}
            <NoteCard {noteData} />
        {/each}
    {/if}
</main>

<style>
    header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 1rem;
        background-color: var(--db-color-bg-secondary);
        column-gap: 0.5rem;
        border-bottom: 1px solid var(--db-color-border);
    }

    input {
        width: 100%;
        height: 1.75rem;
        border: 1px solid var(--db-color-border);
        border-radius: var(--db-border-radius-medium);
        font-size: var(--db-fontsize-medium);
        font-family: var(--db-fontstack);
        padding: 0.5rem;
        outline: none;
        background-color: var(--db-color-bg);
    }

    input:focus {
        border: 1px solid var(--db-color-border-brand-strong);
    }

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
