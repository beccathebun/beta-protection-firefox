<template>
    <n-card size="small">
        <n-thing>
            <template #header>Subliminal Messages</template>
            <template
                #description
            >Enable displaying "subliminal" messages while censoring is enabled.</template>
            This will enable a "subliminal message" overlay that will flash messages across the page very briefly at semi-regular intervals.
            These messages will noticeably flash and may cause irritation for some users, as well as being (deliberately) intrusive.
        </n-thing>
        <template #footer>
            <n-space item-style="display: flex;" justify="space-around" align="center" v-if="loaded && prefs?.subliminal" >
                <n-input-group>
                    <n-input-group-label>Delay</n-input-group-label>
                    <n-input-number
                        v-model:value="prefs.subliminal.delay"
                        :style="{ width: '75%' }"
                        :step="100"
                    />
                    <n-input-group-label>ms</n-input-group-label>
                </n-input-group>
                <n-input-group>
                    <n-input-group-label>Duration</n-input-group-label>
                    <n-input-number
                        v-model:value="prefs.subliminal.duration"
                        :style="{ width: '75%' }"
                        :step="100"
                    />
                    <n-input-group-label>ms</n-input-group-label>
                </n-input-group>
            </n-space>
        </template>
        <template #action>
            <n-space item-style="display: flex;" justify="space-between" v-if="loaded && prefs?.subliminal">
                <n-space vertical>
                    {{messageCount}} messages loaded.
                    <n-tooltip trigger="hover">
                        <template #trigger>
                            <n-button @click="openMessageFile">Import custom messages...</n-button>
                        </template>
                        Import a text file with one message per line to use as custom messages.
                    </n-tooltip>
                </n-space>
                <n-space>
                    <!-- TODO: refactor to use HelpTooltip -->
                    <n-space>
                        <help-tooltip style="max-width: 40rem;" :size="20" :placement="'bottom-start'">When this is enabled, subliminals will be shown based on the extension mode, and will ignore censoring state. With this enabled, subliminals will be shown on all pages when in On Demand or Enabled mode, regardless of whether the current page is being censored (or whitelisted).</help-tooltip>
                        <n-checkbox size="large" v-model:checked="prefs.subliminal.ignoreCensorState">Ignore Censoring State</n-checkbox>
                    </n-space>
                    <n-space vertical>
                        <n-checkbox size="large" v-model:checked="prefs.subliminal.enabled">Enable Subliminal Messaging</n-checkbox>
                        <div>Only enable if you're ready for it!</div>
                    </n-space>
                </n-space>
            </n-space>
        </template>
    </n-card>
</template>
<script setup lang="ts">
import type { IExtensionPreferences } from '@/preferences';
import { FileSystemClient } from '@/services';
import { SubliminalService } from '@/services/subliminal-service';
import { dbg } from '@/util';
import { HelpTooltip, updateUserPrefs, watchForChanges } from '@silveredgold/beta-shared-components';
import { NButton, NCard, NCheckbox, NInputGroup, NInputGroupLabel, NInputNumber, NSpace, NThing, NTooltip, useNotification } from "naive-ui";
import { Ref, computed, inject, onBeforeMount, ref, toRefs, watch } from 'vue';

interface Props {
    preferences?: IExtensionPreferences,
    compact?: boolean
}

const props = withDefaults(defineProps<Props>(), {
    compact: false
});

const notif = useNotification();
const { preferences } = toRefs(props);
const prefs = preferences;
const updatePrefs = inject(updateUserPrefs);
const messages: Ref<string[]> = ref([]);

const loaded = computed(() => preferences?.value !== undefined);
const messageCount = computed(() => messages.value.length);

const svc = new SubliminalService();

//@ts-expect-error
watch(prefs!, watchForChanges(true, updatePrefs), {deep: true});

onBeforeMount(() => {
    loadMessages().then(msgs => {
        messages.value = msgs;
    });
});

const loadMessages = async () => {
    const msgs = await svc.getMessages();
    return [...new Set(msgs)];
}

const openMessageFile = async () => {
    const file = await openFile();
    const contents = await file.file.text();
    dbg('got contents', contents);
    const records = SubliminalService.loadFromText(contents);
    dbg('got records', records);
    await svc.loadMessages(records);
    const msgs = await loadMessages();
    dbg('got messages', msgs);
    messages.value = msgs;
    const n = notif?.create({
          content: `Imported ${msgs.length} messages`,
          duration: 2500,
          closable: true
        });
}

const openFile = async () => {
    const fs = new FileSystemClient();
    const result = await fs.getFile(fs.textFiles);
    dbg('loaded files', result);
    return result;
}

</script>
