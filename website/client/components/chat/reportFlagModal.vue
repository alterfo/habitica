<template lang="pug">
  b-modal#report-flag(:title='$t("abuseFlagModalHeading")', size='lg', :hide-footer='true')
    .modal-header
      h4(v-html="$t('abuseFlagModalHeading', reportData)")
    .modal-body
      blockquote
        // @TODO: markdown(text='abuseObject.text')
      p(v-html="$t('abuseFlagModalBody', abuseFlagModalBody)")
    .modal-footer
      button.pull-left.btn.btn-danger(@click='clearFlagCount()', v-if='user.contributor.admin && abuseObject.flagCount > 0')
        | Reset Flag Count
      button.btn.btn-primary(@click='close()') {{ $t('cancel') }}
      button.btn.btn-danger(@click='reportAbuse()') {{ $t('abuseFlagModalButton') }}
</template>

<script>
import bModal from 'bootstrap-vue/lib/components/modal';
import { mapState } from 'client/libs/store';
import notifications from 'client/mixins/notifications';

export default {
  mixins: [notifications],
  components: {
    bModal,
  },
  computed: {
    ...mapState({user: 'user.data'}),
    reportData () {
      let reportMessage = this.abuseObject.user;
      let isSystemMessage = this.abuseObject.uuid === 'system';
      if (isSystemMessage) reportMessage = this.$t('systemMessage');
      return {
        name: `<span class='text-danger'>${reportMessage}</span>`,
      };
    },
    abuseObject () {
      return this.$store.state.flagChatOptions.message;
    },
    groupId () {
      return this.$store.state.flagChatOptions.groupId;
    },
  },
  data () {
    let abuseFlagModalBody = {
      firstLinkStart: '<a href="/static/community-guidelines" target="_blank">',
      secondLinkStart: '<a href="/static/terms" target="_blank">',
      linkEnd: '</a>',
    };

    return {
      abuseFlagModalBody,
    };
  },
  methods: {
    close () {
      this.$root.$emit('hide::modal', 'report-flag');
    },
    async reportAbuse () {
      await this.$store.dispatch('chat:flag', {
        groupId: this.groupId,
        chatId: this.abuseObject.id,
      });
      this.notify('Thank you for reporting this violation. The moderators have been notified.');
    },
    async clearFlagCount () {
      await this.$store.dispatch('chat:clearFlagCount', {
        groupId: this.groupId,
        chatId: this.abuseObject.id,
      });
    },
  },
};
</script>
