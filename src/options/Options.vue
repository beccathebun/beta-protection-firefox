<template>
  <n-config-provider :theme-overrides="themeOverrides" :theme="theme">
    <n-message-provider>
      <n-dialog-provider>
        <n-notification-provider>
          <n-page-header style="padding-bottom: 2rem;">
            <template #title>Beta Protection</template>
            <template #avatar>
              <n-avatar :src="iconSrc" />
            </template>
            <template #extra>
              <n-space item-style="display: flex;" justify="end">
                <n-popover trigger="hover" placement="bottom">
                  <template #trigger>
                    <n-button @click="openCensoring">
                      <n-icon size="30" :component="Images" />
                    </n-button>
                  </template>
                  Open Censoring
                </n-popover>
                <n-popover trigger="hover" placement="bottom">
                  <template #trigger>
                    <n-button @click="openOverrides">
                      <n-icon size="30" :component="LockOpen" />
                    </n-button>
                  </template>
                  Open Overrides
                </n-popover>
                <n-popover trigger="hover" placement="bottom">
                  <template #trigger>
                    <n-button @click="openStatistics">
                      <n-icon size="30" :component="StatsChart" />
                    </n-button>
                  </template>
                  Open Statistics
                </n-popover>
                <n-popover trigger="click" placement="bottom-end">
                  <template #trigger>
                    <n-button>
                      <n-icon size="30" :component="InformationCircleOutline" />
                    </n-button>
                  </template>
                  <extension-info />
                </n-popover>
              </n-space>
            </template>
            <template #footer>To change the censoring mode, use the popup from your extension toolbar!</template>
          </n-page-header>
          <connection-status :style="{ marginBottom: '2em' }" :host-config="getHost" />
          <n-collapse :style="{ marginTop: '1em', marginBottom: '1em', padding: '0.5em' }" style="width: unset;">
            <n-collapse-item title="Backend Host" name="backend-host">
              <suspense>
                <backend-host class="control-group" />
              </suspense>
              <privacy-options :preferences="prefs" class="control-group" />
              <template #header-extra>Set where your backend is running</template>
            </n-collapse-item>
            <n-collapse-item title="Censoring Options" name="censoring-options" v-if="prefs">
              <overridable-option :option="currentOverride?.preferences?.covered" title="Censoring Preferences">
                <censoring-preferences :preferences="prefs" class="control-group" />
              </overridable-option>
              <overridable-option title="Error Display Preferences" :option="currentOverride?.preferences?.errorMode">
                <error-options :preferences="prefs" class="control-group" />
              </overridable-option>
              <overridable-option title="Loading Filter Preferences" :option="currentOverride?.preferences?.loadingFilter">
                <loading-filter-options :preferences="prefs" class="control-group" />
              </overridable-option>
              <template #header-extra>Fine tune the censoring</template>
            </n-collapse-item>
            <n-collapse-item title="Video Options" name="video-options" v-if="prefs">
              <overridable-option :option="currentOverride?.preferences?.videoCensorMode">
                <video-options :preferences="prefs" class="control-group" />
              </overridable-option>
              <template #header-extra>Choose video behaviour</template>
            </n-collapse-item>
            <n-collapse-item title="Placeholders and Stickers" name="placeholder-options" v-if="prefs">
              <n-alert title="About Placeholders" type="default" closable>
                <template #icon>
                  <n-icon :component="InformationCircle" />
                </template>
                While censoring takes place, images will be replaced by a placeholder image randomly selected from the
                images in any enabled categories. Here is where you choose which categories you want to see images from.
              </n-alert>
              <Suspense>
                <placeholder-options :preferences="prefs" v-if="prefs" class="control-group" />
              </Suspense>
              <Suspense>
                <sticker-options :preferences="prefs" v-if="prefs" class="control-group" />
              </Suspense>
              <template #header-extra>Choose your placeholders and stickers</template>
            </n-collapse-item>
            <n-collapse-item title="Placeholder Management" name="placeholder-store" v-if="prefs">
              <n-alert title="About Placeholders" type="default" closable>
                <template #icon>
                  <n-icon :component="InformationCircle" />
                </template>
                While censoring takes place, images will be replaced by a placeholder image randomly selected from the
                images in any enabled categories. Here is where you add additional placeholders.
              </n-alert>
              <n-collapse :style="{ marginTop: '1em', marginBottom: '1em', padding: '0.5em' }">
                <n-collapse-item title="Placeholder Store">
                  <open-store />
                  <template #header-extra>Manage your installed placeholders</template>
                </n-collapse-item>
                <n-collapse-item title="Upload and Import">
                  <placeholder-upload :preferences="prefs" class="control-group" />
                  <beta-safety-import :preferences="prefs" class="control-group" />
                  <template #header-extra>Import new placeholders</template>
                </n-collapse-item>
              </n-collapse>

              <template #header-extra>Add and remove placeholders</template>
            </n-collapse-item>
            <n-collapse-item title="Domain Lists" name="domain-options" v-if="prefs">
              <overridable-option :option="currentOverride?.preferences?.allowList">
                <domain-list-options :allow-list="prefs.allowList" :force-list="prefs.forceList" v-if="prefs"
                  class="control-group" />
              </overridable-option>
              <template #header-extra>Edit your allowed and forced sites</template>
            </n-collapse-item>
            <n-collapse-item title="Danger Zone" name="danger=zone">
              <subliminal-options :preferences="prefs" v-if="prefs.subliminal" class="control-group" />
              <settings-reset class="control-group" />
              <hardcore-options />
              <template #header-extra>Be careful in here!</template>
            </n-collapse-item>
          </n-collapse>
          <import-export @imported="handleImport" :preferences="prefs" />
        </n-notification-provider>
      </n-dialog-provider>
    </n-message-provider>
    <n-global-style />
  </n-config-provider>
</template>

<script setup lang="ts">
import BackendHost from '@/components/BackendHost.vue';
import DomainListOptions from "@/components/DomainListOptions.vue";
import ExtensionInfo from "@/components/ExtensionInfo.vue";
import HardcoreOptions from "@/components/HardcoreOptions.vue";
import LoadingFilterOptions from "@/components/LoadingFilterOptions.vue";
import PrivacyOptions from "@/components/PrivacyOptions.vue";
import SettingsReset from "@/components/SettingsReset.vue";
import StickerOptions from "@/components/StickerOptions.vue";
import SubliminalOptions from "@/components/SubliminalOptions.vue";
import VideoOptions from "@/components/VideoOptions.vue";
import { BetaSafetyImport, PlaceholderOptions, PlaceholderUpload } from "@/components/placeholders";
import OpenStore from "@/components/placeholders/OpenStore.vue";
import { IExtensionPreferences, IPreferences } from '@/preferences';
import { loadPreferencesStore, useUserOptionsStore } from "@/stores";
import { dbgLog, themeOverrides } from "@/util";
import { updateUserPrefs } from "@silveredgold/beta-shared-components";
import { Images, InformationCircle, InformationCircleOutline, LockOpen, StatsChart } from "@vicons/ionicons5";
import { NAlert, NAvatar, NButton, NCollapse, NCollapseItem, NConfigProvider, NDialogProvider, NGlobalStyle, NIcon, NMessageProvider, NNotificationProvider, NPageHeader, NPopover, NSpace, darkTheme, useOsTheme } from "naive-ui";
import { Suspense, computed, onBeforeMount, provide } from 'vue';
// import CensoringPreferences from "@/components/CensoringPreferences.vue";
import ImportExport from '@/components/ImportExport.vue';
import { OverridableOption } from "@/components/overrides";
import { webExtensionNavigation } from "@/components/util";
import { useOverrideStore } from "@/stores/overrides";
import type { HostConfigurator } from '@silveredgold/beta-shared-components';
import { CensoringPreferences, ConnectionStatus, ErrorOptions } from "@silveredgold/beta-shared-components";
import browser from 'webextension-polyfill';
const { openOverrides, openStatistics, openCensoring } = webExtensionNavigation;

const getHost: HostConfigurator = {
  getBackendHost: async (): Promise<string> => {
    const configHost = await browser.storage.local.get('backendHost');
    const host = configHost['backendHost'];
    return host;
  }
}

const osTheme = useOsTheme()
const theme = computed(() => (osTheme.value === 'dark' ? darkTheme : null));

const iconSrc = browser.runtime.getURL('/images/icon.png');

const store = await loadPreferencesStore(300);
const options = useUserOptionsStore();
const overrides = useOverrideStore(undefined, true);
await overrides.ready;
// overrides.$subscribe(() => {
//   console.log('got subscription fired from overrides store');
// });

const prefs = computed(() => store.currentPreferences);
const currentOverride = computed(() => overrides.currentOverride);

const updatePrefs = async (preferences?: IExtensionPreferences) => {
  dbgLog('saving preferences in Options', preferences);
  await store.save(preferences || prefs.value);
  return true;
}

const handleImport = (prefs: IPreferences) => {
  updatePrefs(prefs as IExtensionPreferences).then(() => {
    console.log('imported new preferences!', prefs);
  });
}

onBeforeMount(async () => {
  // await store.load();
  await options.load();
});

browser.runtime.onMessage.addListener((request, sender) => {
  if (request['msg'] === 'reloadPreferences') {
    setTimeout(() => {
      dbgLog('reloading preferences for options view. SKIPPING!', store.currentPreferences);
      // browser.tabs.getCurrent().then(tab => {
      //   if (!!tab) {
      //     browser.tabs.reload(tab.id);
      //   }
      // });
      // store.load();
      // getCurrentPrefs().then(prefs => {
      //   store.preferences = prefs;
      // });
    }, 1000);
  }
});

provide(updateUserPrefs, updatePrefs);


</script>
<style>

html {
  max-width: 800px;
  height: 800px;
  padding: 1.5rem;
  margin-left: auto;
  margin-right: auto;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
}

.control-group {
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
}
</style>
