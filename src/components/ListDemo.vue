<template>
  <header>
    <div class="layout-topbar"><a href="#/" class="router-link-active layout-topbar-logo"><img alt="Logo"
                                                                                               src="images/logo-dark.svg"><span>SAKAI</span></a>
      <button class="p-link layout-menu-button layout-topbar-button"><i class="pi pi-bars"></i></button>
      <button class="p-link layout-topbar-menu-button layout-topbar-button"><i class="pi pi-ellipsis-v"></i></button>
      <ul class="layout-topbar-menu hidden lg:flex origin-top">
        <li>
          <button class="p-link layout-topbar-button"><i class="pi pi-calendar"></i><span>Events</span></button>
        </li>
        <li>
          <button class="p-link layout-topbar-button"><i class="pi pi-cog"></i><span>Settings</span></button>
        </li>
        <li>
          <button class="p-link layout-topbar-button"><i class="pi pi-user"></i><span>Profile</span></button>
        </li>
      </ul>
    </div>
  </header>
  <main>
    <div class="grid">
      <div class="col-12">
        <div class="card">
          <div class="text-wrapper">
            <h5>DataView</h5>
            <div class="form">
              <h4></h4>
              <h4></h4>
              <h4></h4>
            </div>
          </div>
          <DataView :value="dataviewValue" :layout="layout" :paginator="true" :rows="9" :sortOrder="sortOrder"
                    :sortField="sortField">
            <template #header>
              <div class="grid grid-nogutter">
                <div class="col-6 text-left">
                  <Dropdown v-model="sortKey" :options="sortOptions" optionLabel="label" placeholder="Sort By Price"
                            @change="onSortChange($event)"/>
                </div>
                <div class="col-6 text-right">
                  <Dialog header="Create" v-model:visible="display" :breakpoints="{'960px': '75vw'}" :style="{width: '30vw'}" :modal="true">
                    <div class="p-fluid">
                      <div class="field">
                        <label for="username">Name</label>
                        <InputText id="username" type="text" v-model="name"/>
                      </div>
                      <div class="field">
                        <label for="password">Description</label>
                        <InputText id="password" type="text" v-model="description"/>
                      </div>
                      <div class="field">
                        <label for="Number">Price</label>
                        <InputText id="password" type="number" v-model="price"/>
                      </div>
                      <Button label="Create" class="mt-2" @click="create"></Button>
                    </div>
                  </Dialog>
                  <Button label="Create" icon="pi pi-external-link" style="width: auto" @click="open"/>
                </div>
              </div>
              <div class="product-container">
                <div class="products" v-for="(product, index) in products" :key="index.id">
                  <div class="col-12 md:col-4">
                    <div class="card m-3 border-1 surface-border">
                      <div class="flex align-items-center justify-content-between">
                        <div class="flex align-items-center">
                          <i class="pi pi-tag mr-2"></i>
                          <span class="font-semibold">INSTOCK</span>
                        </div>
                        <div class="delete">
                          <span class="cursor pi pi-times p-button-icon p-button-icon-left" @click="splice_product(product.id)"></span>
                        </div>
                      </div>
                      <div class="text-center">
                        <img src="../assets/images/bamboo-watch.jpg" class="w-9 shadow-2 my-3 mx-0"/>
                        <div class="text-2xl font-bold">{{ product.title }}</div>
                        <div class="mb-3">{{ product.description }}</div>
                      </div>
                      <div class="flex align-items-center justify-content-between">
                        <span class="text-2xl font-semibold">${{ product.price }}</span>
                        <Button icon="pi pi-shopping-cart"></Button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </template>
          </DataView>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import ProductService from "../service/ProductService";
import axios from 'axios'

export default {
  data() {
    return {
      products: [],
        name: '',
        description: '',
        price: '',
      display: false,
      dataviewValue: null,
      layout: 'grid',
      displayConfirmation: false,
      sortKey: null,
      sortOrder: null,
      sortField: null,
      sortOptions: [
        {label: 'Price High to Low', value: '!price'},
        {label: 'Price Low to High', value: 'price'},
      ]
    }
  },
  productService: null,
  created() {
    this.productService = new ProductService();
  },
  mounted() {
    this.productService.getProducts().then(data => this.dataviewValue = data);

    axios
        .get('http://localhost:3000/products')
        .then(response => (this.products = response.data))
  },
  methods: {
    splice_product(index) {
      axios
          .delete(`http://localhost:3000/products/${index}`)
          .then(response => {
            console.log(response)
            window.location.reload()
          })
          .catch(err=> (console.log(err)))
      this.products.splice(index, 1)
    },
    onSortChange(event) {
      const value = event.value.value;
      const sortValue = event.value;

      if (value.indexOf('!') === 0) {
        this.sortOrder = -1;
        this.sortField = value.substring(1, value.length);
        this.sortKey = sortValue;
      } else {
        this.sortOrder = 1;
        this.sortField = value;
        this.sortKey = sortValue;
      }
    },
    create() {
      this.display = false
      let form = {
        title: this.name,
        description: this.description,
        price: this.price,
      }
      axios
          .post('http://localhost:3000/products', form)
          .then(() => {
            this.products.push(form)
              this.name = ''
              this.description = ''
              this.price = ''
          })

    },
    open() {
      this.display = true;
    },
    close() {
      this.display = false;
    },
  }
}
</script>

<style scoped lang="scss">
@import '../assets/demo/badges.scss';
main{
  margin: 84px 0;
}
.product-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
}
.md\:col-4 {
  flex: 0 0 auto;
  padding: 0.5rem;
  width: 455px;
}
.text-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.cursor {
  cursor: pointer;
}
</style>
