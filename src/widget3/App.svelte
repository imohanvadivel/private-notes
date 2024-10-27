<script lang="ts">
    import { onMount, setContext, afterUpdate } from "svelte";
    import { Button, Spinner } from "deskblocks";
    import { APP, DB, initApp } from "../lib/util";

    let App: APP | {} = {};
    setContext("App", App);

    let isLoading = true;
    let note: string;
    let isEditing = false;
    let firstName: string;
    let lastName: string;
    let agentId: string;
    let contactId: string;
    let module = "contact";
    let textarea: HTMLTextAreaElement;

    onMount(async () => {
        App = await initApp();
        getNote();
    });

    async function getNote() {
        let contactData = await ZOHODESK.get("contact");
        firstName = contactData.contact.firstName;
        lastName = contactData.contact.lastName;
        contactId = contactData.contact.id;

        let userData = await ZOHODESK.get("user");
        agentId = userData.user.id;

        let key = `${contactId}_${agentId}`;
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

    function saveNotes() {
        let key = `${contactId}_${agentId}`;
        let queriableValue = agentId;
        let fullName = `${firstName ? firstName : ""} ${lastName ? lastName : ""}`.trim();
        let value = { note, title: fullName, id: contactId, module };

        console.log({ value });

        DB.set({ key, value, queriableValue })
            .then(() => {
                isEditing = false;
                ZOHODESK.notify({ icon: "success", title: "Notes saved", content: "The notes have been saved successfully." });
                if (textarea) textarea.blur();
            })
            .catch((error) => {
                ZOHODESK.notify({ icon: "failure", title: "Error", content: error.message });
            });

        $APP?.instance.getWidgets().then((widgets) => {
            let bottomBandWidget = widgets.find((widget) => widget.location === "desk.bottomband");
            if (!bottomBandWidget) return;

            let bottomWidgetInstance = $APP.instance.getWidgetInstance(bottomBandWidget.id);
            bottomWidgetInstance.emit("REFETCH", {});
        });
    }

    function deleteNote() {
        DB.delete({ key: `${contactId}_${agentId}` })
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
