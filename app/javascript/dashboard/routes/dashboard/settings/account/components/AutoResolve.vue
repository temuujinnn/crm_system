<script setup>
import { ref, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { useAccount } from 'dashboard/composables/useAccount';
import { useAlert } from 'dashboard/composables';
import SectionLayout from './SectionLayout.vue';
import WithLabel from 'v3/components/Form/WithLabel.vue';
import DurationInput from 'next/input/DurationInput.vue';
import TextArea from 'next/textarea/TextArea.vue';
import Switch from 'next/switch/Switch.vue';
import NextButton from 'dashboard/components-next/button/Button.vue';

const { t } = useI18n();
const duration = ref(0);
const message = ref('');
const isEnabled = ref(false);

const { currentAccount, updateAccount } = useAccount();

watch(
  currentAccount,
  () => {
    const { auto_resolve_after, auto_resolve_message } =
      currentAccount.value?.settings || {};

    duration.value = auto_resolve_after;
    message.value = auto_resolve_message;

    if (duration.value) {
      isEnabled.value = true;
    }
  },
  { deep: true, immediate: true }
);

const updateAccountSettings = async settings => {
  try {
    await updateAccount(settings);
    useAlert(t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.API.SUCCESS'));
  } catch (error) {
    useAlert(t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.API.ERROR'));
  }
};

const handleSubmit = async () => {
  return updateAccountSettings({
    auto_resolve_after: duration.value,
    auto_resolve_message: message.value,
  });
};

const handleDisable = async () => {
  duration.value = null;
  message.value = '';

  return updateAccountSettings({
    auto_resolve_after: null,
    auto_resolve_message: '',
  });
};

const toggleAutoResolve = async () => {
  if (!isEnabled.value) handleDisable();
};
</script>

<template>
  <SectionLayout
    :title="t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE.TITLE')"
    :description="t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE.NOTE')"
    with-border
  >
    <template #headerActions>
      <div class="flex justify-end">
        <Switch v-model="isEnabled" @change="toggleAutoResolve" />
      </div>
    </template>

    <form class="grid gap-4" @submit.prevent="handleSubmit">
      <WithLabel
        :label="t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.LABEL')"
        :help-message="t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.HELP')"
      >
        <div class="gap-2 w-full grid grid-cols-[3fr_1fr]">
          <!-- allow 10 mins to 999 days -->
          <DurationInput
            v-model="duration"
            min="10"
            max="1439856"
            class="w-full"
          />
        </div>
      </WithLabel>
      <WithLabel
        :label="t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.MESSAGE_LABEL')"
        :help-message="
          t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.MESSAGE_HELP')
        "
      >
        <TextArea
          v-model="message"
          class="w-full"
          :placeholder="
            t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.MESSAGE_PLACEHOLDER')
          "
        />
      </WithLabel>
      <div class="flex gap-2">
        <NextButton
          blue
          type="submit"
          :label="
            t('GENERAL_SETTINGS.FORM.AUTO_RESOLVE_DURATION.UPDATE_BUTTON')
          "
        />
      </div>
    </form>
  </SectionLayout>
</template>
