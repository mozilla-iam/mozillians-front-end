<template>
  <EditMutationWrapper
    :editVariables="{
      uris,
    }"
    :formName="fluent('profile_accounts', 'edit')"
  >
    <header class="profile__section-header" ref="header" tabindex="-1">
      <div class="profile__section-header__title-info">
        <h2>{{ fluent('profile_accounts') }}</h2>
        <Tooltip
          :buttonText="fluent('profile_accounts', 'tooltip-open')"
          :alternateButtonText="fluent('profile_accounts', 'tooltip-close')"
          >{{ fluent('profile_accounts', 'tooltip') }}</Tooltip
        >
      </div>
      <PrivacySetting
        class="privacy-select--large"
        :label="fluent('profile_accounts', 'privacy')"
        id="section-accounts-privacy"
        profileFieldName="uris"
        :profileFieldObject="uris"
        :collapsedShowLabel="true"
      />
    </header>
    <div
      v-for="({ k, v }, index) in uris.values"
      v-if="isAccountKey(k)"
      :key="index"
      class="edit-contact__item"
    >
      <Button
        class="button--icon-only button--x"
        v-on:click="() => deleteUri(index)"
      >
        <Icon id="x" :width="17" :height="17"></Icon>
        <span class="visually-hidden">{{
          fluent('profile_accounts', 'remove')
        }}</span>
      </Button>
      <Select
        class="options--chevron"
        :label="`Account ${index} type`"
        :id="`field-account-${index}-type`"
        :options="[
          { label: EXTERNAL_ACCOUNTS[destructUriKey(k).name].text, value: k },
          ...remainingAccounts,
        ]"
        v-model="uris.values[index].k"
      />
      <label :for="`field-account-${index}-username`" class="visually-hidden">{{
        fluent('profile_accounts_username-on', {
          service: EXTERNAL_ACCOUNTS[destructUriKey(k).name].text,
        })
      }}</label>
      <input
        :id="`field-account-${index}-username`"
        type="text"
        v-model="uris.values[index].v"
        :placeholder="
          EXTERNAL_ACCOUNTS[destructUriKey(k).name].placeholder ||
            `Your username on ${EXTERNAL_ACCOUNTS[destructUriKey(k).name].text}`
        "
      />
      <Checkbox
        @input="newValue => toggleUriContactMe(newValue, index)"
        :checked="destructUriKey(k).contact"
        :label="fluent('profile_contact-me', 'show-in')"
        class="edit-contact__set-as-contact"
      />
      <hr role="presentation" />
    </div>
    <Button
      class="edit-accounts__add-more button--secondary button button--action"
      type="button"
      :disabled="noAccountsLeft"
      v-on:click="addUri"
      ><Icon id="plus" :width="16" :height="16" />{{
        fluent('profile_accounts', 'add')
      }}</Button
    >
  </EditMutationWrapper>
</template>

<script>
import AccountsMixin from '@/components/_mixins/AccountsMixin.vue';
import Button from '@/components/ui/Button.vue';
import Checkbox from '@/components/ui/Checkbox.vue';
import Icon from '@/components/ui/Icon.vue';
import PrivacySetting from '@/components/profile/PrivacySetting.vue';
import Select from '@/components/ui/Select.vue';
import { DISPLAY_LEVELS } from '@/assets/js/display-levels';
import EditMutationWrapper from './EditMutationWrapper.vue';
import Tooltip from '@/components/ui/Tooltip.vue';

export default {
  name: 'EditAccounts',
  props: {
    initialUris: {
      type: Object,
      default: () => {
        return {};
      },
    },
    editVariables: Object,
  },
  mixins: [AccountsMixin],
  components: {
    Button,
    Checkbox,
    EditMutationWrapper,
    Icon,
    PrivacySetting,
    Select,
    Tooltip,
  },
  methods: {
    formatAsKeyValues([k, v]) {
      return {
        k,
        v,
      };
    },
    addUri() {
      if (this.remainingAccounts.length > 0) {
        const k = this.remainingAccounts[0].value;
        this.uris.values.push({
          k,
          v: '',
        });
      }
    },
    deleteUri(index) {
      if (this.uris.values.length > index) {
        this.uris.values.splice(index, 1);
      }
    },
    toggleUriContactMe(checked, index) {
      const account = this.destructUriKey(this.uris.values[index].k, index);
      account.contact = checked;
      this.uris.values[index].k = this.constructUriKey(account);
    },
  },
  mounted() {
    this.$refs.header.focus();
  },
  computed: {
    remainingAccounts() {
      const selectedUris = this.uris.values.map(
        ({ k }) => this.destructUriKey(k).name
      );
      return this.availableAccounts
        .filter(account => !selectedUris.includes(account))
        .map(account => {
          const label = this.EXTERNAL_ACCOUNTS[account].text;
          return { label, value: this.constructUriKey({ name: account }) };
        });
    },
    noAccountsLeft() {
      return this.remainingAccounts.length <= 0;
    },
  },
  data() {
    const {
      values: initialUris,
      metadata: { display = DISPLAY_LEVELS.private.value },
    } = this.initialUris;
    return {
      uris: {
        values: Object.entries(initialUris || {}).map(([k, v]) => {
          return { k, v };
        }),
        display,
      },
    };
  },
};
</script>

<style>
.profile__section-header__title-info {
  display: flex;
  align-items: center;
}

.profile__section-header__title-info .tooltip {
  margin-left: 1em;
}

.edit-accounts__add-more {
  margin-left: auto;
}
</style>
