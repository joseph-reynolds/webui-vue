<template>
  <b-modal id="modal-role-group" ref="modal" @ok="onOk" @hidden="resetForm">
    <template #modal-title>
      <template v-if="roleGroup">
        {{ $t('pageLdap.modal.editRoleGroup') }}
      </template>
      <template v-else>
        {{ $t('pageLdap.modal.addNewRoleGroup') }}
      </template>
    </template>
    <b-container>
      <b-row>
        <b-col sm="8">
          <b-form id="role-group" @submit.prevent="handleSubmit">
            <b-form-group
              :label="$t('pageLdap.modal.groupName')"
              label-for="role-group-name"
            >
              <b-form-input
                id="role-group-name"
                v-model="form.groupName"
                :state="getValidationState($v.form.groupName)"
                @input="$v.form.groupName.$touch()"
              />
              <b-form-invalid-feedback role="alert">
                {{ $t('global.form.fieldRequired') }}
              </b-form-invalid-feedback>
            </b-form-group>

            <b-form-group
              :label="$t('pageLdap.modal.groupPrivilege')"
              label-for="privilege"
            >
              <b-form-select
                id="privilege"
                v-model="form.groupPrivilege"
                :options="accountRoles"
                :state="getValidationState($v.form.groupPrivilege)"
                @input="$v.form.groupPrivilege.$touch()"
              >
                <template v-if="!roleGroup" #first>
                  <b-form-select-option :value="null" disabled>
                    {{ $t('global.form.selectAnOption') }}
                  </b-form-select-option>
                </template>
              </b-form-select>
              <b-form-invalid-feedback role="alert">
                {{ $t('global.form.fieldRequired') }}
              </b-form-invalid-feedback>
            </b-form-group>
          </b-form>
        </b-col>
      </b-row>
    </b-container>
    <template #modal-footer="{ cancel }">
      <b-button variant="secondary" @click="cancel()">
        {{ $t('global.action.cancel') }}
      </b-button>
      <b-button form="role-group" type="submit" variant="primary" @click="onOk">
        <template v-if="roleGroup">
          {{ $t('global.action.save') }}
        </template>
        <template v-else>
          {{ $t('global.action.add') }}
        </template>
      </b-button>
    </template>
  </b-modal>
</template>

<script>
import { required, requiredIf } from 'vuelidate/lib/validators';
import VuelidateMixin from '@/components/Mixins/VuelidateMixin.js';

export default {
  mixins: [VuelidateMixin],
  props: {
    roleGroup: {
      type: Object,
      default: null,
      validator: (prop) => {
        if (prop === null) return true;
        return (
          Object.prototype.hasOwnProperty.call(prop, 'groupName') &&
          Object.prototype.hasOwnProperty.call(prop, 'groupPrivilege')
        );
      },
    },
  },
  data() {
    return {
      form: {
        groupName: null,
        groupPrivilege: null,
      },
    };
  },
  computed: {
    accountRoles() {
      return this.$store?.getters['userManagement/filteredAccountRoles'].filter(
        (role) => role !== 'ServiceAgent' && role !== 'Operator'
      );
    },
  },
  watch: {
    roleGroup: function (value) {
      if (value === null) return;
      this.form.groupName = value.groupName;
      this.form.groupPrivilege = value.groupPrivilege;
    },
  },
  validations() {
    return {
      form: {
        groupName: {
          required: requiredIf(function () {
            return !this.roleGroup;
          }),
        },
        groupPrivilege: {
          required,
        },
      },
    };
  },
  methods: {
    handleSubmit() {
      this.$v.$touch();
      if (this.$v.$invalid) return;
      this.$emit('ok', {
        addNew: !this.roleGroup,
        groupNamePreviously: this.roleGroup?.groupName
          ? this.roleGroup?.groupName
          : null,
        groupName: this.form.groupName,
        groupPrivilege: this.form.groupPrivilege,
      });
      this.closeModal();
    },
    closeModal() {
      this.$nextTick(() => {
        this.$refs.modal.hide();
      });
    },
    resetForm() {
      this.form.groupName = null;
      this.form.groupPrivilege = null;
      this.$v.$reset();
      this.$emit('hidden');
    },
    onOk(bvModalEvt) {
      // prevent modal close
      bvModalEvt.preventDefault();
      this.handleSubmit();
    },
  },
};
</script>
