<template>
  <q-layout view="lHh Lpr lFf">
    <q-header flat>
      <q-toolbar>
        <q-toolbar-title>Fadhlur Rohman</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page padding class="flex flex-center q-pa-md">
        <div class="q-pa-md q-mx-auto q-ma-md q-gutter-md">
          <h6 class="q-my-xs text-center">Input Data</h6>
          <q-form
            @submit.prevent="save"
            class="q-gutter-md q-pa-md q-display-flex q-justify-center"
          >
            <div
              class="q-gutter-md q-display-flex q-flex-wrap q-items-center q-justify-center"
            >
              <q-input
                filled
                v-model="form.title"
                label="Title"
                class="q-w-1/2 q-pr-sm"
              />
              <q-input
                filled
                v-model="form.content"
                label="Content"
                type="textarea"
                class="q-w-1/2 q-pl-sm"
              />
              <q-btn
                type="submit"
                label="Simpan"
                color="primary"
                class="q-ml-md q-mt-md"
              />
            </div>
          </q-form>
          <h6 class="q-my-s text-center">Isi Content</h6>
          <q-list
            bordered
            class="q-mt-md q-pa-md q-elevate-2 q-mx-auto q-w-full"
            style="max-width: 600px"
          >
            <q-item
              v-for="article in articles"
              :key="article.id"
              clickable
              v-ripple
              class="q-pa-md q-ma-md"
            >
              <q-item-section>
                <q-item-label class="text-h6">{{ article.title }}</q-item-label>
                <q-item-label caption>{{ article.content }}</q-item-label>
              </q-item-section>
              <q-item-section side class="q-gutter-md">
                <q-btn
                  dense
                  round
                  icon="edit"
                  color="primary"
                  @click="edit(article)"
                />
                <q-btn
                  dense
                  round
                  icon="delete"
                  color="negative"
                  @click="remove(article.id)"
                />
              </q-item-section>
            </q-item>
          </q-list>

          <q-btn
            label="Muat"
            @click="load"
            color="secondary"
            class="q-mt-md q-mx-auto"
          />
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

const form = reactive({
  id: null,
  title: "",
  content: "",
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get("http://localhost:3000/articles");
    articles.value = response.data;
  } catch (error) {
    console.error("Error loading articles: ", error);
  }
}

async function save() {
  try {
    let url, method;
    if (form.id === null) {
      const lastArticle = articles.value[articles.value.length - 1];
      const newId = lastArticle ? parseInt(lastArticle.id) + 1 : 1;
      form.id = newId.toString();
      url = "http://localhost:3000/articles/";
      method = "post";
    } else {
      url = "http://localhost:3000/articles/" + form.id;
      method = "put";
    }
    const response = await axios[method](url, form);
    if (method === "post") {
      articles.value.push(response.data);
    } else {
      articles.value = articles.value.map((article) =>
        article.id === response.data.id ? response.data : article
      );
    }
    form.id = null;
    form.title = "";
    form.content = "";
  } catch (error) {
    console.log("Error saving article: ", error);
  }
}

async function remove(id) {
  try {
    await axios.delete(`http://localhost:3000/articles/${id}`);
    articles.value = articles.value.filter((article) => article.id !== id);
  } catch (error) {
    console.error("Error deleting article: ", error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

onMounted(load);
</script>

<style scoped>
.q-display-flex {
  display: flex;
}

.q-flex-wrap {
  flex-wrap: wrap;
}

.q-items-center {
  align-items: center;
}

.q-justify-center {
  justify-content: center;
}

.q-pa-md {
  padding: 16px;
}

.q-mx-auto {
  margin-left: auto;
  margin-right: auto;
}

.q-ma-md {
  margin: 16px;
}

.q-elevate-2 {
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.text-center {
  text-align: center;
}
</style>
