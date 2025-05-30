<script lang="ts">
    import { t } from "$lib/i18n/translations";
    import { downloadFile } from "$lib/download";
    import { createDialog } from "$lib/state/dialogs";
    import { validateSettings } from "$lib/settings/validate";
    import { storedSettings, updateSetting, loadFromString } from "$lib/state/settings";

    import DataSettingsButton from "$components/settings/DataSettingsButton.svelte";
    import ResetSettingsButton from "$components/settings/ResetSettingsButton.svelte";

    import IconFileExport from "@tabler/icons-svelte/IconFileExport.svelte";
    import IconFileImport from "@tabler/icons-svelte/IconFileImport.svelte";

    const updateSettings = (reader: FileReader) => {
        try {
            const data = reader.result?.toString();
            if (!data) throw $t("error.import.no_data");

            const loadedSettings = loadFromString(data);
            if (!validateSettings(loadedSettings))
                throw $t("error.import.invalid");

            createDialog({
                id: "import-confirm",
                type: "small",
                icon: "warn-red",
                title: $t("dialog.safety.title"),
                bodyText: $t("dialog.import.body"),
                buttons: [
                    {
                        text: $t("button.cancel"),
                        main: false,
                        action: () => {},
                    },
                    {
                        text: $t("button.import"),
                        color: "red",
                        main: true,
                        timeout: 5000,
                        action: () => updateSetting(loadFromString(data)),
                    },
                ],
            });
        } catch (e) {
            let message = $t("error.import.no_data");

            if (e instanceof Error) {
                console.error("settings import error:", e);
                message = $t("error.import.unknown", { value: e.message });
            } else if (typeof e === "string") {
                message = e;
            }

            createDialog({
                id: "settings-import-error",
                type: "small",
                meowbalt: "error",
                bodyText: message,
                buttons: [
                    {
                        text: $t("button.gotit"),
                        main: true,
                        action: () => {},
                    },
                ],
            });
        }
    };

    const importSettings = () => {
        const pseudoinput = document.createElement("input");
        pseudoinput.type = "file";
        pseudoinput.accept = ".json";
        pseudoinput.onchange = (e: Event) => {
            const target = e.target as HTMLInputElement;
            const reader = new FileReader();

            reader.onload = () => updateSettings(reader);

            if (target.files?.length === 1) {
                reader.readAsText(target.files[0]);
            }
        };
        pseudoinput.click();
    };

    const exportSettings = async () => {
        return await downloadFile({
            file: new File(
                [JSON.stringify($storedSettings, null, 4)],
                "settings.json", { type: "application/json" }
            ),
        });
    };
</script>

<div class="button-row" id="settings-data-transfer">
    <DataSettingsButton id="import-settings" click={importSettings}>
        <IconFileImport />
        {$t("button.import")}
    </DataSettingsButton>

    {#if $storedSettings.schemaVersion}
        <DataSettingsButton id="export-settings" click={exportSettings}>
            <IconFileExport />
            {$t("button.export")}
        </DataSettingsButton>
    {/if}

    {#if $storedSettings.schemaVersion}
        <ResetSettingsButton />
    {/if}
</div>

<style>
    .button-row {
        display: flex;
        gap: var(--padding);
        flex-wrap: wrap;
    }
</style>
