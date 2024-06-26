<template>
  <div class="container">
    <form class="form" @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong><br />
        {{ article.content }}<br />
        <button @click="edit(article)" class="button">Edit</button>
        <button @click="deleteArticle(article.id)" class="button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios'; 

export default {
  setup() {
    const form = reactive({
      id: '',
      title: '',
      content: '', 
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id 
          ? `http://localhost:3000/articles/${form.id}` 
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, { title: form.title, content: form.content });

        if (form.id) {
          // Update existing article
          articles.value = articles.value.map((article) => 
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Add new article
          articles.value.push(response.data);
        }

        // Reset form
        resetForm();
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function resetForm() {
      form.id = '';
      form.title = '';
      form.content = '';
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle };
  }
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Helvetica Neue', Arial, sans-serif;
  background-color: #f0f4c3;  /* Background color of the container */
  border-radius: 12px;         /* More rounded corners */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 20px;
  background-color: #ffffff;  /* Background color of the form */
  padding: 20px;
  border-radius: 12px;        /* More rounded corners */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.input, .textarea {
  width: 100%;
  padding: 12px;
  margin-bottom: 12px;
  border: 1px solid #cddc39;  /* Lighter border color */
  border-radius: 6px;
  font-size: 16px;
  background-color: #ffffff;
}

.button {
  padding: 12px 24px;
  background-color: #8bc34a;  /* Primary button color */
  color: #ffffff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

.button:hover {
  background-color: #558b2f;  /* Darker hover color */
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #ffffff;  /* Background color of the article item */
  padding: 15px;
  border: 1px solid #cddc39;  /* Lighter border color */
  border-radius: 12px;        /* More rounded corners */
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.article-item strong {
  display: block;
  font-size: 18px;
  margin-bottom: 5px;
  color: #8bc34a;  /* Primary text color for title */
}

.load-button {
  display: block;
  width: 100%;
  margin-top: 20px;
  background-color: #8bc34a;  /* Primary button color */
}

.load-button:hover {
  background-color: #558b2f;  /* Darker hover color */
}
</style>