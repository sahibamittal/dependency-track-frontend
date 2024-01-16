<template>
  <b-modal id="policyBundleInfoModal" size="lg" hide-header-close no-stacking :title="$t('message.project_details')" @show="initializeTags" @hide="resetValues()">
    <b-tabs class="body-bg-color" style="border:0;padding:0">
      <b-tab class="body-bg-color" style="border:0;padding:0" active>
        <template v-slot:title><i class="fa fa-edit"></i> {{ $t('message.general') }}</template>
        <b-card>
          <b-input-group-form-input id="bundle-url" input-group-size="mb-3" type="text" v-model="bundle.url"
                                    lazy="true" required="true" feedback="true" autofocus="false"
                                    :label="$t('message.policy_bundle_url')" :readonly="true" />
          <b-input-group-form-input id="bundle-hash" input-group-size="mb-3" type="text" v-model="bundle.hash"
                                    lazy="true" required="false" feedback="false" autofocus="false"
                                    :label="$t('message.policy_bundle_hash')" :readonly="true" />
          <b-input-group-form-select id="v-classifier-input" required="true" v-model="bundle.lastSyncDate"
                                     :label="$t('message.policy_bundle_sync_timestamp')" :readonly="true" />
        </b-card>
      </b-tab>
    </b-tabs>
    <template v-slot:modal-footer="{ cancel }">
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
    props: {},
    data() {
      return {
        addOnKeys: [9, 13, 32, ':', ';', ','], // Separators used when typing tags into the vue-tag-input
        labelIcon: {
          dataOn: '\u2713',
          dataOff: '\u2715'
        },
        bundleToken: ""
      }
    },
    beforeMount() {
      this.$root.$on('initializePolicyBundleInfoModal', async () => {
        console.log("inside initializeBundleInfoModal");
        this.bundle = (await this.axios.get(`${this.$api.BASE_URL}/${this.$api.URL_POLICY}/bundle`)).data
        this.$root.$emit("bv::show::modal", "policyBundleInfoModal")
      })
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
