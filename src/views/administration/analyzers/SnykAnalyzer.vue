<template>
  <b-card no-body :header="header">
    <b-card-body>
      <c-switch id="scannerEnabled" color="primary" v-model="scannerEnabled" label v-bind="labelIcon" />{{$t('admin.analyzer_snyk_enable')}}
      <b-validated-input-group-form-input
        id="snyk-apitoken"
        :label="$t('admin.api_token')"
        input-group-size="mb-3"
        rules="required"
        type="password"
        v-model="apitoken"
        lazy="true"
      />
      <hr/>
      {{ $t('admin.analyzer_snyk_desc') }}
    </b-card-body>
    <b-card-footer>
      <b-button variant="outline-primary" class="px-4" @click="saveChanges">{{ $t('message.update') }}</b-button>
    </b-card-footer>
  </b-card>
</template>

<script>
  import { Switch as cSwitch } from '@coreui/vue';
  import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
  import common from "../../../shared/common";
  import configPropertyMixin from "../mixins/configPropertyMixin";

  export default {
    mixins: [configPropertyMixin],
    props: {
      header: String
    },
    components: {
      cSwitch,
      BValidatedInputGroupFormInput
    },
    data() {
      return {
        scannerEnabled: false,
        apitoken: '',
        labelIcon: {
          dataOn: '\u2713',
          dataOff: '\u2715'
        },
      }
    },
    methods: {
      saveChanges: function() {
        this.updateConfigProperties([
          {groupName: 'scanner', propertyName: 'snyk.enabled', propertyValue: this.scannerEnabled},
          {groupName: 'scanner', propertyName: 'snyk.api.token', propertyValue: this.apitoken}
        ]);
      }
    },
    created () {
      this.axios.get(this.configUrl).then((response) => {
        let configItems = response.data.filter(function (item) { return item.groupName === "scanner" });
        for (let i=0; i<configItems.length; i++) {
          let item = configItems[i];
          switch (item.propertyName) {
            case "snyk.enabled":
              this.scannerEnabled = common.toBoolean(item.propertyValue); break;
            case "snyk.api.token":
              this.apitoken = item.propertyValue; break;
          }
        }
      });
    }
  }
</script>
