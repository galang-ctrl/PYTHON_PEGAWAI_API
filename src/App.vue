<template>
  <div>
    <title>Home</title>
    <h1>Karyawan SIP</h1> 
    <div>
    <input v-model="searchQuery" placeholder="Search..." />
    
    <ul>
    </ul>
  </div>
    <table>
      <thead>
        <tr>
                <th>Id</th>
                <th>Emploee Name</th>
                <th>Emploee Salary</th>
                <th>Emploee Ege</th>
                <th>Profile Image</th>
            </tr>
      </thead>
      <tbody>

        <tr v-for="post in paginatedPosts" :key="post.id" >
         
    <!-- {{ paginatedPosts(post) }} -- {{ filteredItems(post) }}-->
          <td>{{ post.id}}</td>
              <td>{{post.employee_name}}</td>
              <td>{{post.employee_salary}}</td>
              <td>{{post.employee_age}}</td>
              <td>{{post.profile_image}}</td>


      
        
        
        </tr>
       
      </tbody>
    </table>
    <div class="pagination">
      <button @click="changePage(1)" :disabled="currentPage === 1">First</button>
      <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">Previous</button>
      <button v-for="page in totalPages" :key="page" @click="changePage(page)" :class="{ active: page === currentPage }">
        {{ page }}
      </button>
      <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">Next</button>
      <button @click="changePage(totalPages)" :disabled="currentPage === totalPages">Last</button>
    </div>
    <p v-if="error">Api: {{ error.message }}</p>
  </div>
</template>
<script>
import axios from 'axios';

export default {
  
  data() {
    return {
      searchQuery: '',
      posts: [], // Array untuk menyimpan data yang diambil
      currentPage: 1, // Halaman saat ini
      itemsPerPage: 5, // Jumlah item paginatioan per halaman
      limit: 5, // limit data dalam satu halaman
      error: null // Menyimpan Pesan Kesalahan
    };
  },
  computed: {
    filteredItems() {
      const query = this.searchQuery.toLowerCase();
      return this.posts.filter(post => {
        return post.employee_name.toLowerCase().includes(query);
      });
    },
    
    limitedPosts() { // Membatasi data sesuai dengan jumlah yang diinginkan
      return this.posts.slice(0, this.limit);
    },
    
   totalPages() {
      return Math.ceil(this.posts.length / this.itemsPerPage);
      
    }, 
   paginatedPosts() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.posts.slice(start, end);
   },

  },
  mounted() {

    this.fetchPosts();// Ambil data saat komponen diinstall
 
  },
  methods: {
    async fetchPosts(retryCount = 0) {
      const maxRetries = 5;
      const delay = Math.pow(2, retryCount) * 1000; // Kemunduran Exponential 

      try {
        const response = await axios.get('https://dummy.restapiexample.com/api/v1/employees');
        this.posts = response.data.data; // Tetapkan data Array yang diposting
      } catch (error) {
        if (error.response && error.response.status === 429) {
          if (retryCount < maxRetries) {
            console.warn(`Rate limit exceeded. Retrying in ${delay / 1000} seconds...`);
            setTimeout(() => {
              this.fetchPosts(retryCount + 1);
            }, delay);
          } else {
            this.error = 'Too many requests. Please try again later.';
          }
        } else {
          this.error = error;
        }
      }
    },
    searchData() {
      this.currentPage = 1; // Reset ke halaman pertama setelah pencarian
    },
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    }
  }

};
</script>

<style>
        table {
            width: 100%;
            border-collapse: collapse;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f4f4f4;
        }
        tr:nth-child(even) {
            background-color: #f9f9f9;
        }
    </style>


