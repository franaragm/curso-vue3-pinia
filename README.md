## INSTRUCCIONES
1. En primer lugar, crea un nuevo proyecto Vite utilizando la plantilla Vue 3:
```js
npm init vite@latest my-contacts-app -- --template vue
cd my-contacts-app
npm install
```
---
2. Instala las dependencias necesarias:
```js
npm install vue-router@4 pinia@next axios
```
---
Crea la siguiente estructura de proyecto, que vamos a ir implmentando en los siguientes pasos:

```md
src/
 ├── components/
 │   ├── Navbar.vue
 ├── pinia/
 │   ├── store.js
 ├── views/
 │   ├── Home.vue
 │   ├── SavedContacts.vue
 ├── App.vue
 ├── main.js
 ├── index.html
 ├── router.js
 ```
---
3. Crea el archivo **main.js** en el directorio src y configura Vue, Vue Router y Pinia:

```js
import { createApp } from 'vue'
import { createRouter, createWebHashHistory } from 'vue-router'
import { createPinia } from 'pinia'
import App from './App.vue'
// Import routes
import routes from './routes'
// Create router
const router = createRouter({
  history: createWebHashHistory(),
  routes
})

// Crear Pinia store
const store = createPinia()

const app = createApp(App)
app.use(router)
app.use(store)
app.mount('#app')
```
---
4.  Crea un archivo **routes.js** en el directorio src:
```js
import Home from './views/Home.vue'
import SavedContacts from './views/SavedContacts.vue'

const routes = [
  { path: '/', component: Home },
  { path: '/saved-contacts', component: SavedContacts }
]
```
---
5. Crea un archivo **store.js** en el directorio src/pinia:
```js
import { defineStore } from 'pinia'

export const useContactsStore = defineStore({
  id: 'contacts',

  state: () => ({
    contacts: [],
    savedContacts: [],
    searchQuery: ''
  }),

  actions: {
    setContacts(contacts) {
      this.contacts = contacts
    },

    saveContact(contact) {
      this.savedContacts.push(contact)
    },

    removeContact(index) {
      this.contacts.splice(index, 1)
    },

    setSearchQuery(query) {
      this.searchQuery = query
    }
  }
})
````

---
6. Cree un componente Home.vue en el directorio src/views:
```js
<template>
  <div class="container">
    <input
      type="text"
      v-model="searchQuery"
      @input="updateSearchQuery"
      placeholder="Search contacts..."
    />
    <ul>
      <li v-for="(contact, index) in filteredContacts" :key="contact.login.uuid" :class="{ saved: isContactSaved(contact) }">
        <img :src="contact.picture.thumbnail" />
        {{ contact.name.first }} {{ contact.name.last }}
        <button @click="addContact(contact)">Save Contact</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import { useContactsStore } from '../pinia/store'
import axios from 'axios'

const store = useContactsStore()
const searchQuery = ref('')

const fetchContacts = async () => {
  const response = await axios.get('https://randomuser.me/api/?results=500')
  store.setContacts(response.data.results)
}

fetchContacts()

const filteredContacts = computed(() => {
  if (searchQuery.value === '') return store.contacts
  return store.contacts.filter((contact) =>
    (`${contact.name.first} ${contact.name.last}`)
      .toLowerCase()
      .includes(searchQuery.value.toLowerCase())
  )
})

const isContactSaved = (contact) => {
  return store.savedContacts.some(
    (savedContact) => savedContact.login.uuid === contact.login.uuid
  )
}

const isContactSavedComputed = computed(() => {
  return (contact) => isContactSaved(contact)
})

const updateSearchQuery = () => {
store.setSearchQuery(searchQuery.value)
}

const deleteContact = (index) => {
store.removeContact(index)
}

const addContact = (contact) => {
store.saveContact(contact)
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}

input {
  width: 100%;
  padding: 0.5rem;
  margin-bottom: 1rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  padding: 0.5rem 0;
  border-bottom: 1px solid #eee;
}
li.saved {
  background-color: rgba(46, 204, 113, 0.1);
}

img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  margin-right: 1rem;
}

button {
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
  margin-left: 0.5rem;
}

button:hover {
  background-color: #2980b9;
}
</style>
```
---

7. Crea un componente SavedContacts.vue en el directorio `src/views`:
```js
<template>
  <div>
    <h1>Saved Contacts</h1>
    <ul>
      <li v-for="(contact, index) in savedContacts" :key="contact.login.uuid">
        <img :src="contact.picture.thumbnail" />
        {{ contact.name.first }} {{ contact.name.last }}
        <button @click="removeSavedContact(index)">Remove</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { useContactsStore } from '../pinia/store'

const store = useContactsStore()
const savedContacts = store.savedContacts

const removeSavedContact = (index) => {
  store.savedContacts.splice(index, 1)
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}

h1 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  padding: 0.5rem 0;
  border-bottom: 1px solid #eee;
}

img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  margin-right: 1rem;
}

button {
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
  margin-left: 0.5rem;
}

button:hover {
  background-color: #c0392b;
}
</style>
```
---
8. Actualice el archivo App.vue en el directorio src:
```js
<template>
  <div id="app">
    <Navbar />
    <router-view></router-view>
  </div>
</template>

<script setup>
import Navbar from './components/Navbar.vue'

</script>
```
---

9. Crea y añade el siguiente código al archivo Navbar.vue:
```js
<template>
    <nav class="navbar">
      <router-link to="/" class="nav-link">Home</router-link>
      <router-link to="/saved-contacts" class="nav-link">Saved Contacts</router-link>
    </nav>
  </template>
  
  <style scoped>
  .navbar {
    display: flex;
    justify-content: space-around;
    align-items: center;
    background-color: #3498db;
    padding: 1rem;
  }
  
  .nav-link {
    color: white;
    text-decoration: none;
    font-size: 1.2rem;
  }
  
  .nav-link:hover {
    color: #ddd;
  }
  </style>
  ```






 
