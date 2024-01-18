<template>
  <b-modal id="policyBundleInfoModal" size="lg" hide-header-close no-stacking :title="$t('message.policy_bundle_info')">
    <b-tabs class="body-bg-color" style="border:0;padding:0">
      <b-tab class="body-bg-color" style="border:0;padding:0" active>
        <template v-slot:title><i class="fa fa-edit"></i> {{ $t('message.general') }}</template>
        <b-card>
          <b-input-group-form-input id="bundle-url" input-group-size="mb-3" type="text" v-model="bundleInfo.url"
                                    lazy="true" feedback="false" autofocus="true"
                                    :label="$t('message.policy_bundle_url')" disabled="true" />
          <b-input-group-form-input id="bundle-hash" input-group-size="mb-3" type="text" v-model="bundleInfo.hash"
                                    lazy="true" feedback="false" autofocus="false"
                                    :label="$t('message.policy_bundle_hash')" disabled="true" />
          <b-input-group-form-input id="bundle-created" input-group-size="mb-3" type="text" :value="bundleCreated" 
                                    lazy="true" feedback="false" autofocus="false"
                                    :label="$t('message.created')" disabled="true" />
          <b-input-group-form-input id="bundle-last-sync" :value="bundleLastSynced" feedback="false"
                                    :label="$t('message.policy_bundle_sync_timestamp')" disabled="true" />
        </b-card>
      </b-tab>
    </b-tabs>
    <template v-slot:modal-footer="{ cancel }">
      <b-button id="sync-button" size="md" variant="outline-primary"
                @click="sync()"
                v-permission:or="[PERMISSIONS.POLICY_MANAGEMENT]">
        <span class="fa fa-refresh"></span> {{ $t('message.policy_bundle_sync') }}
      </b-button>
      <b-button size="md" variant="secondary" @click="cancel()">{{ $t('message.close') }}</b-button>
    </template>
  </b-modal>
</template>

<script>
  import BInputGroupFormInput from "../../forms/BInputGroupFormInput";
  import BInputGroupFormSelect from "../../forms/BInputGroupFormSelect";
  import { Switch as cSwitch } from '@coreui/vue';
  import permissionsMixin from "../../mixins/permissionsMixin";
  import Multiselect from "vue-multiselect"
  import VueTagsInput from '@johmun/vue-tags-input';
  import common from "../../shared/common";

  export default {
    name: "PolicyBundleInfoModal",
    mixins: [permissionsMixin],
    components: {
      BInputGroupFormInput,
      VueTagsInput,
      BInputGroupFormSelect,
      cSwitch,
      Multiselect
    },
    props: {
      bundleInfo: Object,
    },
    data() {
      return {
        syncToken: {}
      }
    },
    computed: {
      bundleCreated: {
        get() {
          return (typeof this.bundleInfo.created !== 'undefined' && this.bundleInfo.created != null) ? common.formatTimestamp(this.bundleInfo.created) : "-";
        }
      },
      bundleLastSynced: {
        get() {
          return (typeof this.bundleInfo.lastSuccessfulSync !== 'undefined' && this.bundleInfo.lastSuccessfulSync != null) ? common.formatTimestamp(this.bundleInfo.lastSuccessfulSync) : "-";
        }
      }
    },
    beforeMount() {},
    methods: {
      sync: function () {
        let syncUrl = `${this.$api.BASE_URL}/${this.$api.URL_VULNERABILITY_POLICY}/bundle/sync`
        this.axios.post(syncUrl).then((response) => {
          this.$toastr.s(this.$t('message.policy_bundle_sync_requested'));
        })
        .then(response => {
          this.syncToken = response.token;
          // TODO : do something with token
        })
        .catch(error => {
          this.$toastr.w(error.response.data.message);
        });
      }
    }
  }
</script>

<style lang="scss">
  @import "../../assets/scss/vendors/vue-tags-input/vue-tags-input";
</style>

<style scoped>
.tab-content .tab-pane{
  padding: 0 !important;
}
.tab-content {
  border: 0 !important;
}
.card {
  border:0;
  padding:0;
  margin-bottom:0;
}
</style>
