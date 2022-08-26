<template>
  <b-card no-body :header="header">
    <b-card-body>
      <hr/>
      <c-switch
        color="primary"
        id="vulnsourceEnabled"
        label
        v-bind="labelIcon"
        v-model="vulnsourceEnabled"
      />
      {{$t('admin.vulnsource_osv_advisories_enable')}}
      <hr/>
      {{ $t('admin.vulnsource_osv_advisories_desc') }}
    </b-card-body>
    <b-card-footer>
      <b-button
        @click="saveChanges"
        class="px-4"
        variant="outline-primary">
          {{ $t('message.update') }}
      </b-button>
    </b-card-footer>
  </b-card>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import common from "../../../shared/common";
import EcosystemModal from "./EcosystemModal";
import configPropertyMixin from "../mixins/configPropertyMixin";

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String
  },
  components: {
    cSwitch
  },
  data() {
    return {
      vulnsourceEnabled: false,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715'
      },
    }
  },
  options: {
    detailFormatter: (index, row) => {
            return this.vueFormatter({
              i18n,
              template: `
                <b-row class="expanded-row">
                  <b-col sm="6">
                    <b-input-group-form-input id="ecosystem-table" :label="$t('admin.team_name')" input-group-size="mb-3"
                                              required="true" type="text" v-model="name" lazy="true" autofocus="true"
                                              v-debounce:750ms="updateTeam" :debounce-events="'keyup'" />
                    <b-form-group :label="this.$t('admin.permissions')">
                      <div class="list-group">
                        <span v-for="ecosystem in ecosystems">
                          <actionable-list-group-item :value="ecosystem" :delete-icon="true" v-on:actionClicked="removePermission(permission)"/>
                        </span>
                        <actionable-list-group-item :add-icon="true" v-on:actionClicked="$root.$emit('bv::show::modal', 'selectPermissionModal')"/>
                      </div>
                    </b-form-group>
                  </b-col>
                  <select-permission-modal v-on:selection="updatePermissionSelection" />
                </b-row>
              `,
              mixins: [],
              components: {
                cSwitch,
                ActionableListGroupItem,
                SelectPermissionModal,
                BInputGroupFormInput
              },
              data() {
                return {
                  team: row,
                  name: row.name,
                  permissions: row.permissions,
                  labelIcon: {
                    dataOn: '\u2713',
                    dataOff: '\u2715'
                  }
                }
              },
              methods: {
                
                updatePermissionSelection: function(selections) {
                  this.$root.$emit('bv::hide::modal', 'selectPermissionModal');
                  for (let i=0; i<selections.length; i++) {
                    let selection = selections[i];
                    let url = `${this.$api.BASE_URL}/${this.$api.URL_PERMISSION}/${selection.name}/team/${this.team.uuid}`;
                    this.axios.post(url
                    ).then((response) => {
                      this.syncVariables(response.data);
                      this.$toastr.s(this.$t('message.updated'));
                    }).catch((error) => {
                      if (error.response.status === 304) {
                        //this.$toastr.w(this.$t('condition.unsuccessful_action'));
                      } else {
                        this.$toastr.w(this.$t('condition.unsuccessful_action'));
                      }
                    });
                  }
                },
                removePermission: function(permission) {
                  let url = `${this.$api.BASE_URL}/${this.$api.URL_PERMISSION}/${permission.name}/team/${this.team.uuid}`;
                  this.axios.delete(url).then((response) => {
                    this.syncVariables(response.data);
                    this.$toastr.s(this.$t('message.updated'));
                  }).catch((error) => {
                    this.$toastr.w(this.$t('condition.unsuccessful_action'));
                  });
                },
                syncVariables: function(team) {
                  this.permissions = team.permissions;
                }
              }
            })
          },
  },
  methods: {
    saveChanges: function() {
      this.updateConfigProperties([
        {groupName: 'vuln-source', propertyName: 'google.osv.enabled', propertyValue: this.vulnsourceEnabled}
      ]);
    }
  },
  created () {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) { return item.groupName === "vuln-source" });
      for (let i=0; i<configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case "google.osv.enabled":
            this.vulnsourceEnabled = common.toBoolean(item.propertyValue); break;
        }
      }
    });
  }
}
</script>
