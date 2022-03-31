<template>
  <div id="app">
    <div>
      <TabView>
        <TabPanel header="detail">
          <DataTable
            ref="dt"
            :value="items"
            class="p-datatable-sm"
            edit-mode="row"
            data-key="id"
            :editing-rows.sync="editingRows"
            @row-edit-init="onRowEditInit"
            @row-edit-cancel="onRowEditCancel"
          >
            <template #empty>
              <p class="p-field p-align-center">
                No items Added
              </p>
            </template>
            <Column :rowReorder="true" headerStyle="width: 3rem" :reorderableColumn="false" />
            <Column field="item" header="Item">
              <template #body="slotProps">
                {{  slotProps.data.name || slotProps.data[slotProps.column.field].item  }}
              </template>
              <template #editor="slotProps">
                <AutoComplete
                  v-model="slotProps.data[slotProps.column.field]"
                  class="p-col-12 p-field"
                  :suggestions="filteredServices"
                  :dropdown="true"
                  field="item"
                  placeholder="Service"                        
                  :minQueryLength="0"
                  @item-select="updateDescription($event, slotProps.index)"
                  @complete="searchService($event)"
                />
              </template>
            </Column>
            <Column field="description" header="Description">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field] }}
              </template>
              <template #editor="slotProps">
                <TextArea
                  v-model="slotProps.data[slotProps.column.field]"
                  class="p-col-12 p-field"
                  :auto-resize="true"
                  placeholder="Description"
                />
              </template>
            </Column>
            <Column field="quantity" header="Qt.">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field] }}
              </template>
              <template #editor="slotProps">
                <InputText
                  v-model="slotProps.data[slotProps.column.field]"
                  type="number"
                  class="p-col-12 p-field"
                />
              </template>
            </Column>
            <Column field="price" header="Price">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field] | currency }}
              </template>
              <template #editor="slotProps">
                <InputText
                  v-model="slotProps.data[slotProps.column.field]"
                  type="number"
                  class="p-col-12 p-field"
                />
              </template>
            </Column>
            <Column
              :row-editor="true"
              header-style="width:50px"
              body-style="text-align:center"
            />
            <Column header-style="width:50px">
              <template #body="slotProps">
                <Button
                  icon="pi pi-trash"
                  class="p-button-rounded p-button-secondary p-button-text"
                  @click="onDeleteRow(slotProps.index)"
                />
              </template>
            </Column>
          </DataTable>
          <div class="p-formgrid p-grid" style="margin-top: 16px">
            <div class="p-field p-col">
              <Button
                label="Add Item"
                icon="pi pi-plus"
                icon-pos="right"
                @click="addItem"
              />
            </div>
          </div>
        </TabPanel>
      </TabView>
      <TabView>
        <TabPanel header="Items array binding">
          <DataTable
            :value="items"
            class="p-datatable-sm"
          >
            <template #empty>
              <p class="p-field p-align-center">
                Empty Array
              </p>
            </template>
            <Column field="item" header="name">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field].item }}
              </template>
            </Column>
            <Column field="description" header="Description">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field] }}
              </template>
            </Column>
            <Column field="quantity" header="Quantity">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field] }}
              </template>
            </Column>
            <Column field="price" header="Price">
              <template #body="slotProps">
                {{ slotProps.data[slotProps.column.field] }}
              </template>
            </Column>
          </DataTable>
        </TabPanel>
      </TabView>
    </div>
  </div>
</template>

<script>
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import TabView from 'primevue/tabview'
import TabPanel from 'primevue/tabpanel'
import Button from 'primevue/button'
import InputText from 'primevue/inputtext'
import TextArea from 'primevue/textarea'
import AutoComplete from 'primevue/autocomplete'
const sortableJs = require('sortablejs')

export default {
  name: 'App',
  components: {
    DataTable,
    Column,
    TabView,
    TabPanel,
    Button,
    InputText,
    TextArea,
    AutoComplete
  },
  data () {
    return {
      editingRows: [],
      originalRows: [],
      allServices: [
        {
          item: '1st Service',
          description: 'description 1'
        },
        {
          item: '2nd Service',
          description: 'description 2'
        },
        {
          item: '3th Service',
          description: 'description 3'
        }
      ],
      filteredServices: [],
      items: []
    }
  },
  created() {
    this.originalRows = {}
  },
  mounted() {
    const sortableTableBody = window.document.querySelector('.p-datatable-tbody')
    if (sortableTableBody) {
      sortableJs.Sortable.create(sortableTableBody, {
        group: 'items',
        sort: true,
        pull: true,
        animation: 100,
        onUpdate: (event) => {
          const movedItem = this.items[event.oldIndex]
          const itemToSwap = this.items[event.newIndex]
          this.$set(this.items, event.newIndex, movedItem)
          this.$set(this.items, event.oldIndex, itemToSwap)
          console.log('updated: ', event)
        }
      })
    }
  },
  filters: {
    currency: (value) => {
      const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
      })
      return formatter.format(value)
    },
  },
  methods: {
    onRowEditCancel (event) {
      this.$set(this.items, event.index, this.originalRows)
    },
    updateDescription (event, index) {
      this.items[index].description = event.value.description
    },
    async searchService (event) {
      const services = this.allServices.filter((service) => service.item.includes(event.query.trim()) )
      this.filteredServices = services.sort(( a, b ) => {
        if ( a.item < b.item ){
          return -1;
        }
        if ( a.item > b.item ){
          return 1;
        }
        return 0;
      });
    },
    onRowEditInit (event) {
      this.originalRows = { ...this.items[event.index] }
    },
    onRowReorder(event) {
      this.items = event.value
    },
    onDeleteRow (index) {
      this.items.splice(index, 1)
    },
    addItem() {
      this.items.push({
        name: '',
        id: this.items.length + 1,
        item: {
          item: ''
        },
        description: '',
        quantity: 1,
        price: 0
      })
      this.enableRows(this.items.length - 1)
    },
    enableRows (index) {
      setTimeout(() => {
        const table = this.$refs.dt
        const rowElements = table.$el.querySelectorAll('.p-datatable-tbody > tr')
        if (rowElements.length === 0) {
          return
        }
        const editButton = rowElements[index].querySelector('td .p-row-editor-init')
        if (editButton) {
          editButton.click()
        }
      }, 0)
    }
  }
}
</script>

<style src="primevue/resources/primevue.min.css"></style>
<style src='primevue/resources/themes/saga-blue/theme.css'></style>
<style src='primeflex/primeflex.min.css'></style>
<style src='primeicons/primeicons.css'></style>
<style src='primeicons/primeicons.css'></style>
