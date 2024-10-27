<script lang="ts">
    import { onMount, setContext, afterUpdate } from "svelte";
    import { Button, Spinner } from "deskblocks";
    import { APP, DB, initApp } from "../lib/util";

    let isLoading = true;
    let note: string;
    let isEditing = false;
    let agentId: string;
    let subject: string;
    let ticketId: string;
    let ticketNumber: string;
    let textarea: HTMLTextAreaElement;

    onMount(async () => {
        await initApp(); // Initialize the app
        getNote(); // Get the notes for the current ticket
        $APP?.instance.on("ticket_Shift", getNote); // When ticket switched, get the notes for the new ticket
    });

    // Get the notes for the current ticket form extension DB
    async function getNote() {
        let ticketData = await ZOHODESK.get("ticket");
        subject = ticketData.ticket.subject;
        ticketId = ticketData.ticket.id;
        ticketNumber = ticketData.ticket.number;

        let userData = await ZOHODESK.get("user");
        agentId = userData.user.id;

        let key = `${ticketId}_${agentId}`;
        await DB.get({ key }).then((data) => {
            let notesData = data?.["database.get"]?.["data"];
            if (notesData && notesData.length > 0) {
                note = notesData[0].value.note;
            } else {
                note = "";
            }
        });
        if (textarea) textarea.blur();
        isEditing = false;
        isLoading = false;
    }

    // Save the notes to the extension DB
    function saveNotes() {
        let key = `${ticketId}_${agentId}`;
        let queriableValue = agentId;
        let value = { note, title: subject, id: ticketId, module: "ticket", ticketNumber };

        DB.set({ key, value, queriableValue })
            .then(() => {
                isEditing = false;
                ZOHODESK.notify({ icon: "success", title: "Notes saved", content: "The notes have been saved successfully." });
                if (textarea) textarea.blur();
                refreshBottomBand();
            })
            .catch((error) => {
                ZOHODESK.notify({ icon: "failure", title: "Error", content: error.message });
            });
    }

    // Refresh the bottom band widget to update the notes count
    function refreshBottomBand() {
        $APP?.instance.getWidgets().then((widgets) => {
            let bottomBandWidget = widgets.find((widget) => widget.location === "desk.bottomband");
            if (!bottomBandWidget) return;

            let bottomWidgetInstance = $APP.instance.getWidgetInstance(bottomBandWidget.widgetID);
            bottomWidgetInstance.emit("REFETCH", {});
        });
    }

    // Delete the notes from the extension DB
    function deleteNote() {
        DB.delete({ key: `${ticketId}_${agentId}` })
            .then(() => {
                note = "";
                ZOHODESK.notify({ icon: "success", title: "Note deleted", content: "The note has been deleted successfully." });
            })
            .catch((error) => {
                ZOHODESK.notify({ icon: "failure", title: "Error", content: error.message });
            });
    }
</script>

<main>
    {#if isLoading}
        <div class="spinner">
            <Spinner size="large" />
            <p>Loading Notes...</p>
        </div>
    {:else}
        <textarea
            bind:value={note}
            bind:this={textarea}
            spellcheck="false"
            on:input={() => (isEditing = true)}
            class="notes"
            placeholder="Enter the notes here"
        />
    {/if}

    {#if note && note.length > 0}
        <footer>
            {#if isEditing}
                <Button on:click={saveNotes}>Save</Button>
                <Button on:click={getNote} variant="tertiary">Cancel</Button>
            {:else}
                <Button on:click={deleteNote} variant="danger-secondary">Delete</Button>
            {/if}
        </footer>
    {/if}
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        height: 100vh;
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
        height: 100%;
        margin-top: -2rem;
    }

    .notes {
        padding: 1rem;
        height: 100%;
        border: none;
        outline: none;
        resize: none;
        font-size: var(--db-fontsize-medium);
        font-family: var(--db-fontstack);
        background-color: var(--db-color-bg);
        color: var(--db-color-text);
        line-height: 1.5;
    }
    .notes:focus {
        background-color: var(--db-color-bg-secondary);
    }

    .notes::placeholder {
        color: var(--db-color-text-placeholder);
    }

    footer {
        display: flex;
        flex-direction: row;
        column-gap: 0.5rem;
        border-top: 1px solid var(--db-color-border);
        padding: 1rem;
    }
</style>
