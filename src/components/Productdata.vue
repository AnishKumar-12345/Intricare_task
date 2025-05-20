<template>
    <div>
      <h2>Product Dashboard</h2>
  
      <!-- Search and Category Filter -->
      <div style="margin-bottom: 10px;">
        <input v-model="search" placeholder="Title Search" />
        <select v-model="selectCategory" @change="filterByCategory">
          <option value="">All Categories</option>
          <option v-for="(cat,index) in categories" :key="index" :value="cat">{{ cat }}</option>
        </select>
      </div>
  
      <!-- Add / Edit Form -->
      <form @submit.prevent="submitForm" style="margin-bottom: 15px;">
        <input v-model="form.title" placeholder="Title" required />
        <input v-model="form.price" type="number" placeholder="Price" required />
        <input v-model="form.category" placeholder="Category" required /> &nbsp;
        <button type="submit" style="background-color:blue; color: white;">{{ editing ? 'Update' : 'Add' }}</button>
      </form>
  
      <!-- Products Table -->
      <table border="1" cellpadding="8" cellspacing="0">
        <thead>
          <tr>
            <th>Title</th>
            <th>Price</th>
            <th>Category</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product,index) in filtereProducts" :key="index">
            <td>{{ product.title }}</td>
            <td>{{ product.price }}</td>
            <td>{{ product.category }}</td>
            <td>
              <button @click="editProduct(product)" style="background-color:blue; color: white;">Edit</button> &nbsp;
              <button @click="deleteProduct(product.id)" style="background-color:red;color: white;">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted } from 'vue'
  import axios from 'axios'
  
  // State
  const products = ref([])
  const categories = ref([])
  const selectCategory = ref('')
  const search = ref('')
  const editing = ref(false)
  const editId = ref(null)
  
  const form = ref({
    title: '',
    price: 0,
    category: ''
  })
  
 
  const getProducts = () => {
    axios.get('https://fakestoreapi.com/products')
      .then(res => {
        products.value = res.data
      })
      .catch(err => console.error('Error fetching products:', err))
  }
  

  const getCategories = () => {
    axios.get('https://fakestoreapi.com/products/categories')
      .then(res => {
        categories.value = res.data
      })
      .catch(err => console.error('Error fetching categories:', err))
  }
  
//   Add and edit the form
  const submitForm = () => {
  if (editing.value) {
    updateProduct()
  } else {
    addProduct()
  }
}

const addProduct = () => {
  axios.post('https://fakestoreapi.com/products', form.value)
    .then(res => {
        console.log('see',res);
      products.value.push({ ...form.value, id: res.data.id })
      form.value = { title: '', price: 0, category: '' }
    })
    .catch(err => console.error('Error adding product:', err))
}

const updateProduct = () => {
  axios.put(`https://fakestoreapi.com/products/${editId.value}`, form.value)
    .then(() => {
     
      products.value.map((product, i) => {
        if (product.id === editId.value) {
          products.value[i] = { ...form.value, id: editId.value }
        }
      })

      editing.value = false
      form.value = { title: '', price: 0, category: '' }
    })
    .catch(err => console.log('Error:', err))
}

  
  // Delete Product
  const deleteProduct = (id) => {
    axios.delete(`https://fakestoreapi.com/products/${id}`)
      .then(() => {
        products.value = products.value.filter(p => p.id !== id)
      })
      .catch(err => console.error('Error deleting product:', err))
  }
  
  // Edit Product
  const editProduct = (product) => {
    form.value = {
      title: product.title,
      price: product.price,
      category: product.category
    }
    editing.value = true
    editId.value = product.id
  }
  
  // Filter by Category
  const filterByCategory = () => {
    if (selectCategory.value) {
      axios.get(`https://fakestoreapi.com/products/category/${selectCategory.value}`)
        .then(res => {
          products.value = res.data
        })
        .catch(err => console.error('Error filtering by category:', err))
    } else {
        getProducts()
    }
  }
  
  
  const filtereProducts = computed(() => {
    return products.value.filter(p =>
      p.title.toLowerCase().includes(search.value.toLowerCase())
    )
  })
  

  onMounted(() => {
    getProducts()
    getCategories()
  })
  </script>
  