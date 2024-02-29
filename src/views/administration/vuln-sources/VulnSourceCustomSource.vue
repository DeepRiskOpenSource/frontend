<template>
  <b-card no-body :header="header">
    <b-card-body>
      <img alt="GitHub logo" src="@/assets/img/github-logo.svg" width="65"/>
      <hr/>
      <b-validated-input-group-form-input
        id="github-advisories-apitoken"
        :label="$t('admin.personal_access_token')"
        input-group-size="mb-3"
        rules="required"
        type="password"
        v-model="apitoken"
        lazy="true"
      />
      <hr/>
      Its custom GitHub vulnerability source
    </b-card-body>
    <b-card-footer>
      <b-button
        :disabled="!this.apitoken"
        @click="saveChanges"
        class="px-4"
        variant="outline-primary">
          {{ $t('message.update') }}
      </b-button>
    </b-card-footer>
  </b-card>
</template>

<script>
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
import common from "../../../shared/common";
import configPropertyMixin from "../mixins/configPropertyMixin";

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String
  },
  components: {
    BValidatedInputGroupFormInput
  },
  data() {
    return {
      apitoken: '',
    }
  },
  methods: {
    saveChanges: function() {
      this.updateConfigProperties([
        {groupName: 'vuln-source', propertyName: 'customsource.access.token', propertyValue: this.apitoken}
      ]);
    }
  },
  created () {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) { return item.groupName === "vuln-source" });
      for (let i=0; i<configItems.length; i++) {
        let item = configItems[i];
        if (item.propertyName === "customsource.access.token") {
          this.apitoken = item.propertyValue; 
        }
      }
    });
  }
}
</script>