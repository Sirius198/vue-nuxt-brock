<template>
  <PageContentWrapper>
    <ValidationObserver ref="form">
      <LoadingBar v-if="$apollo.loading" />
      <CustomTable :w-table="520">
        <template #header>
          <div class="table-row">
            <span>Vendor Number</span>

            <span>Vendor Name</span>

            <span>Terms</span>
          </div>
        </template>

        <template v-if="vendors" #content>
          <CustomTableRow
            v-for="vendor in vendors"
            :key="vendor.id"
            class="table-row"
          >
            <CustomInput
              v-if="isEdit === vendor.id"
              v-model="vendorEdit.code"
              rules="required"
              do-not-show-error-message
            />
            <span v-else>{{ vendor.code }}</span>

            <CustomInput
              v-if="isEdit === vendor.id"
              v-model="vendorEdit.name"
              rules="required"
              do-not-show-error-message
            />
            <span v-else>{{ vendor.name }}</span>

            <CustomSelect
              v-if="isEdit === vendor.id"
              :options="terms"
              :selected-items="vendor.terms"
              do-not-preselect
              select-by="name"
              multi-select
              @input="selectTerms($event, vendor)"
            />
            <span v-else-if="vendor.terms">{{
              vendor.terms.map((vendor) => vendor.name).join(', ')
            }}</span>

            <CustomTableIconsColumn
              :is-edit-active="isEdit === vendor.id"
              :is-delete-active="isDelete === vendor.id"
              @edit="editVendor(vendor)"
              @delete="deleteItem(vendor.id)"
              @cancel="cancelVendorEdit"
              @cancel-delete="cancelDelete"
              @confirm-edit="confirmEdit(vendor)"
              @confirm-delete="confirmDelete(vendor.id)"
            />
          </CustomTableRow>

          <CustomTableRow v-if="isAdd" class="table-row">
            <CustomInput
              v-model="vendorNew.code"
              rules="required"
              do-not-show-error-message
            />

            <CustomInput
              v-model="vendorNew.name"
              rules="required"
              do-not-show-error-message
            />

            <CustomSelect
              :options="terms"
              select-by="name"
              multi-select
              do-not-preselect
              @input="selectTerm"
            />
          </CustomTableRow>

          <CustomTableRow class="table-row add-row">
            <CustomTableAddIcon :is-hide="isHide" @add-row="addRow" />
          </CustomTableRow>
        </template>
      </CustomTable>
    </ValidationObserver>

    <div v-if="isAdd" class="buttons-area">
      <DefaultButton @event="addVendor">+ Add Vendor </DefaultButton>

      <DefaultButton @event="cancelAdd"> Cancel </DefaultButton>
    </div>
  </PageContentWrapper>
</template>

<script>
import { ValidationObserver } from 'vee-validate'
import Vendors from '../graphql/queries/vendors.gql'
import Terms from '../graphql/queries/terms.gql'
import CreateVendor from '../graphql/mutations/vendor/createVendor.gql'
import UpdateVendor from '../graphql/mutations/vendor/updateVendor.gql'
import DeleteVendor from '../graphql/mutations/vendor/deleteVendor.gql'
import PageContentWrapper from './PageContentWrapper.vue'
import CustomTable from './CustomTable.vue'
import CustomInput from './CustomInput.vue'
import CustomTableRow from './CustomTableRow.vue'
import CustomTableAddIcon from './CustomTableAddIcon.vue'
import { tableActionsMixin } from '~/mixins/tableActionsMixin'
import { mutationMixin } from '~/mixins/mutationMixin'
export default {
  name: 'HQVendorContent',
  components: {
    ValidationObserver,
    PageContentWrapper,
    CustomTable,
    CustomInput,
    CustomTableRow,
    CustomTableAddIcon,
  },
  mixins: [mutationMixin, tableActionsMixin],
  apollo: {
    vendors: {
      query: Vendors,
    },
    terms: {
      query: Terms,
    },
  },
  data() {
    return {
      vendorNew: {
        code: '',
        name: '',
        terms: [],
      },
      vendorTerms: '',
      vendorEdit: {
        terms: [],
      },
    }
  },
  methods: {
    editVendor(vendor) {
      this.vendorEdit = Object.assign(
        {},
        {
          code: vendor.code,
          name: vendor.name,
          terms: [...vendor.terms],
        }
      )
      this.edit(vendor.id)
    },
    selectTerm(option) {
      if (this.vendorNew.terms.find((vendor) => vendor.id === option.id)) {
        this.vendorNew.terms = this.vendorNew.terms.filter(
          (item) => item.id !== option.id
        )
      } else {
        this.vendorNew.terms.push(option)
      }
    },
    selectTerms(option, vendor) {
      if (this.vendorEdit.terms.find((itm) => itm.id === option.id)) {
        this.vendorEdit.terms = [
          ...this.vendorEdit.terms.filter((item) => item.id !== option.id),
        ]
      } else {
        this.vendorEdit.terms.push(option)
      }
    },
    addVendor() {
      this.mutationAction(
        CreateVendor,
        {
          vendorInput: {
            code: this.vendorNew.code,
            name: this.vendorNew.name,
            terms: {
              sync: this.vendorNew.terms.map((term) => term.id),
            },
          },
        },
        Vendors,
        'Add vendor success',
        'Add vendor error'
      )
    },
    confirmEdit(vendor) {
      const editedVendor = {
        id: vendor.id,
        code: this.vendorEdit.code,
        name: this.vendorEdit.name,
        terms: {
          sync: this.vendorEdit.terms.map((term) => term.id),
        },
      }

      this.mutationAction(
        UpdateVendor,
        { vendorInput: editedVendor },
        Vendors,
        'Add vendor success',
        'Add vendor error'
      )
    },
    confirmDelete(id) {
      this.mutationAction(
        DeleteVendor,
        { id },
        Vendors,
        'Add vendor success',
        'Add vendor error'
      )
    },
    cancelVendorEdit() {
      this.cancelEdit()
    },
  },
}
</script>
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style lang="scss">
.input {
  &-row--offset {
    display: flex;
    margin: 0 -15px;
  }

  &-col {
    min-width: 240px;
    padding: 0 15px;
  }
}

.mb-20 {
  margin-bottom: 20px;
}

.multiselect__tags {
  border: 1px solid gainsboro;
  border-radius: 3px;
}

.multiselect__option {
  padding: 10px 16px 10px 8px;
  color: #000;
  font-size: 14px;
  background: #fff;
}

.multiselect__option--highlight,
.multiselect__option--highlight:after {
  color: #fff;
  background-color: #b01d22;
}
.multiselect__option--selected.multiselect__option--highlight {
  background: rgba(#b01d22, 0.6);
}

.multiselect__tag {
  background-color: #b01d22;
}

.multiselect__tag-icon:focus,
.multiselect__tag-icon:hover {
  background-color: #b01d22;
}

.multiselect__tag-icon:after {
  color: #fff;
}

.multiselect__select:before {
  border: none;
  width: 24px;
  height: 24px;
  background: url(assets/images/icons/chevron-down.svg);
  display: block;
  top: 0;
}
</style>
<style lang="scss" scoped>
.table-row {
  display: grid;
  align-items: center;
  column-gap: 30px;
  padding: 12px 0;
  @media screen and (min-width: $md) {
    grid-template-columns: 100px repeat(2, 200px) auto;
  }
  @media screen and (max-width: $md) {
    grid-template-columns: 100px repeat(2, 120px) auto;
  }
}

.row {
  padding: 16.5px 10px;
}

.icon {
  cursor: pointer;

  &--add {
    grid-column: 4;
    justify-self: end;
  }
}

.buttons-area {
  margin-top: 25px;
}
</style>
