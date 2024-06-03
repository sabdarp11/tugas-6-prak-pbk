<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <label for="title">Title:</label>
      <input type="text" v-model="form.title" id="title" /><br />
      <label for="content">Content:</label>
      <textarea v-model="form.content" id="content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="deleteArticle(article.id)">Delete</button>
        <button @click="edit(article)">Edit</button>
      </li>
    </ul>
    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    })

    const articles = ref([])

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles')
        articles.value = response.data
      } catch (error) {
        console.log('Error loading articles:', error)
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles'
        const method = form.id ? 'put' : 'post'

        const contentId = Math.random().toString(36).substr(2, 9)

        const response = await axios[method](url, { ...form, id: contentId })

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          )
        } else {
          articles.value.push(response.data)
        }

        form.id = null
        form.title = ''
        form.content = ''
      } catch (error) {
        console.log('Error saving article:', error)
      }
    }

    function edit(article) {
      form.id = article.id
      form.title = article.title
      form.content = article.content
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`)
        articles.value = articles.value.filter((article) => article.id !== id)
      } catch (error) {
        console.log('Error deleting article:', error)
      }
    }

    onMounted(load)

    return { form, articles, save, edit, deleteArticle, load }
  }
}
</script>

<style>
body {
  background-image: url('../src/assets/1.jpg');
  background-size: cover;
  color: black;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.form {
  background: rgba(249, 249, 249, 0.8);
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.form label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

.form input,
.form textarea {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form button {
  background: #4caf50;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form button:hover {
  background: #45a049;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.article-item h3 {
  margin: 0 0 10px;
}

.article-item p {
  margin: 0 0 10px;
}

.article-item button {
  background: #ff4d4d;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
}

.article-item button:hover {
  background: #ff3333;
}

.article-item button:last-child {
  background: #008cba;
}

.article-item button:last-child:hover {
  background: #007bb5;
}

button {
  background: #008cba;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #007bb5;
}
</style>
