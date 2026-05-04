<template>
  <div>
    <section>
      <form class="new-note-form" @submit.prevent="createNote">
        <div class="note-title-field">
          <label for="note-title">Title: </label>
          <input id="note-title" type="text" v-model="new_note.title" />
          <p v-if="formError" class="form-error" role="alert">
            {{ formError }}
          </p>
        </div>
        <div class="note-content-field">
          <label for="note-content">Content</label>
          <textarea
            id="note-content"
            placeholder="Enter your note here"
            v-model="new_note.content"
            rows="6"
            cols="50"
          ></textarea>
        </div>
        <div class="submit-button-field">
          <button type="submit">Add Note</button>
        </div>
      </form>
    </section>

    <div class="divider"></div>

    <section>
      <input
        class="notes-filter"
        type="text"
        v-model="titleFilter"
        placeholder="Search by title"
      />
      <button class="search-button" type="button" @click="filterNotes">
        Search
      </button>
      <div id="notes-list">
        <table>
          <caption>Notes</caption>
          <thead>
            <tr>
              <th scope="col">Title</th>
              <th scope="col">Content</th>
              <th scope="col">Created</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="note in notes" :key="note.id">
              <td>{{ note.title }}</td>
              <td>{{ note.content }}</td>
              <td>{{ formatDate(note.created_at) }}</td>
            </tr>
          </tbody>
        </table>

        <nav
          v-if="totalPages > 0"
          class="notes-pagination"
          aria-label="Notes pagination"
        >
          <button
            type="button"
            :disabled="currentPage <= 1"
            @click="goToPage(currentPage - 1)"
          >
            Previous
          </button>

          <span class="notes-pagination__status">
            Page {{ currentPage }} of {{ totalPages }}
          </span>

          <button
            type="button"
            :disabled="currentPage >= totalPages"
            @click="goToPage(currentPage + 1)"
          >
            Next
          </button>
        </nav>
      </div>
    </section>
  </div>
</template>

<script setup>
  import { ref, onMounted } from 'vue';
  import { format } from 'date-fns';
  import { instance } from './axios';

  const formatDate = (value) => {
    const date = new Date(value);
    return format(date, 'dd/MM/yyyy HH:mm');
  };

  const currentPage = ref(1);
  const totalPages = ref(0);
  const perPage = 10;
  const notes = ref([]);
  const titleFilter = ref('');

  const filterNotes = () => {
    instance
      .get('/notes', {
        params: {
          title: titleFilter.value
        }
      }).then((response) => {
        const data = response.data;
        notes.value = data.notes || [];
        currentPage.value = data.current_page || 1;
        totalPages.value = data.total_pages || 0;
      })
  }

  const getNotes = () => {
    instance
      .get('/notes', {
        params: {
          page: currentPage.value,
          limit: perPage,
        }
      })
      .then((response) => {
        const data = response.data;
        notes.value = data.notes || [];
        currentPage.value = data.current_page || 1;
        totalPages.value = data.total_pages || 0;
      })
  };

  const goToPage = (page) => {
    if (page >= 1 && page <= totalPages.value) {
      currentPage.value = page;
      getNotes();
    }
  };

  const new_note = ref({
    title: '',
    content: ''
  });

  const formError = ref('');

  const titleValidation = () => {
    if (new_note.value.title.length > 1 && new_note.value.title.length < 5) {
      formError.value = 'Title length must be longer than 5 characters';
    } else if (!new_note.value.title) {
      formError.value = 'Title cannot be empty'
    }
  }

  const createNote = () => {
    formError.value = '';
    instance
      .post('/notes', {
        note: {
          title: new_note.value.title,
          content: new_note.value.content
        }
      })
      .then(() => {
        currentPage.value = 1;
        getNotes();
        new_note.value.title = '';
        new_note.value.content = '';
      })
      .catch(() => {
        titleValidation()
      });
  };

  onMounted(() => {
    getNotes();
  })
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }

  #app .divider {
    width: 730px;
    border: 1px solid rgb(226, 226, 226);
    border-radius: 20px;
    margin: 40px auto 40px auto;
  }

  #app input {
    padding: 8px;
    border-radius: 4px;
    border: 1px solid grey;
  }

  #app .search-button {
    border-radius: 4px;
    border: 1px solid #64748b;
    padding: 10px 15px 10px 15px;
    background: #fff;
    color: #2c3e50;
    cursor: pointer;
  }

  #app .notes-filter {
    margin: -20px 10px 30px 0px;
  }

  #app table {
    margin: -20px auto;
    border-collapse: collapse;
    text-align: left;
  }

  #app caption {
    padding: 15px;
    font-weight: bold;
  }

  #app th,
  #app td {
    border: 1px solid #ccc;
    padding: 10px 10px;
  }

  #app #note-title {
    width: 200px;
  }

  #app #note-content {
    margin: 10px 0px 10px -100px;
  }

  #app .new-note-form {
    border: 1px solid #ccc;
    padding: 20px;
    width: 500px;
    margin: 20px auto;
  }

  #app .note-title-field {
    margin: 10px 170px 10px 10px;
  }

  #app .note-title-field label {
    margin-bottom: 5px;
    margin: 0px 5px 0px 0px;
  }

  #app .note-content-field {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    text-align: left;
    margin: 10px 0px 10px 140px;
  }

  #app .note-content-field label {
    margin: 0px 0px 0px -100px;
  }

  #app .submit-button-field button {
    border-radius: 4px;
    border: 1px solid #64748b;
    padding: 10px 15px 10px 15px;
    background: #fff;
    color: #2c3e50;
    cursor: pointer;
  }

  #app .form-error {
    color: #b91c1c;
    font-size: 14px;
    margin: 5px 50px 0px 0px;
  }

  #app .notes-pagination {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 20px;
    margin-top: 50px;
    flex-wrap: wrap;
  }

  #app .notes-pagination button {
    padding: 10px 20px 10px 20px;
    border-radius: 4px;
    border: 1px solid #64748b;
    background: #fff;
    color: #2c3e50;
    cursor: pointer;
  }

  #app .notes-pagination button:disabled {
    opacity: 0.45;
    cursor: not-allowed;
  }

  #app .notes-pagination__status {
    min-width: 20px;
  }
</style>
