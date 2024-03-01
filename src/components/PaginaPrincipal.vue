<template>
  <div>
    <!-- Carrusel -->
    <div v-show="!scrolled">
      <BookCarousel :books="books" />
    </div>
    <div class="mt-5">
    <b-row>
      <b-col cols="8">
        <b-row class="mb-3" align-v="center">
          <BookControls
            @sortByAuthor="getAllBooksByAuthor"
            @sortByDate="getAllBooksByPublicationDate"
          />
        </b-row>
        <b-row>
          <BookList :books="books" @dragStart="dragStart" />
        </b-row>
      </b-col>
      <b-col cols="4">
        <Actions @deleteBook="showDeleteModal" @edit="showEditModal" @insert="showInsertModal" />
      </b-col>
    </b-row>
      
    </div>

    <!-- Modales -->
    <!-- Modal para insertar -->
    <b-modal
      v-model="insertModalVisible"
      hide-footer
    >
      <template #modal-header="{ close }">
        <p @click="close()" class="close-tag">X</p>
      </template>
      <h3>Registar libro</h3>
      <b-form @submit="addBook" v-if="showInsertForm">
        <b-form-group
          id="input-group-title"
          label="Título de la obra"
          label-for="input-title"
        >
          <b-form-input id="input-title" v-model="form.title" required />
        </b-form-group>
        <b-form-group
          id="input-group-author"
          label="Autor de la obra"
          label-for="input-author"
        >
          <b-form-input id="input-author" v-model="form.author" required />
        </b-form-group>
        <b-form-group
          id="input-group-publicationDate"
          label="Fecha de publicación"
          label-for="input-publicationDate"
        >
          <b-form-datepicker
            id="input-publicationDate"
            v-model="form.publicationDate"
            required
          />
        </b-form-group>
        <b-button class="mt-3" type="submit" variant="success"
          >Registrar <b-icon icon="plus-square" aria-hidden="true"></b-icon></b-button
        >
      </b-form>
    </b-modal>
    <!-- Modal para editar -->
    <b-modal v-model="editModalVisible" title="Editar Libro" hide-footer>
      <template #modal-header="{ close }">
        <p @click="close()" class="close-tag">X</p>
      </template>
      <h3>Actualizar libro</h3>
      <b-form @submit="editBook">
        <b-form-group
          id="input-group-title"
          label="Título de la obra"
          label-for="input-title"
        >
          <b-form-input id="input-title" v-model="form.title" required />
        </b-form-group>
        <b-form-group
          id="input-group-author"
          label="Autor de la obra"
          label-for="input-author"
        >
          <b-form-input id="input-author" v-model="form.author" required />
        </b-form-group>
        <b-form-group
          id="input-group-publicationDate"
          label="Fecha de publicación"
          label-for="input-publicationDate"
        >
          <b-form-datepicker
            id="input-publicationDate"
            v-model="form.publicationDate"
            required
          />
        </b-form-group>
        <b-button class="mt-3" type="submit" variant="primary">Actualizar <b-icon icon="pencil-square" aria-hidden="true"></b-icon></b-button>
      </b-form>
    </b-modal>
    <!-- Modal para eliminar -->
    <b-modal v-model="deleteModalVisible" title="Eliminar Libro" hide-footer>
    <template #modal-header="{ close }">
        <p @click="close()" class="close-tag">X</p>
      </template>
      <h3>Eliminar libro</h3>
      <p>¿Estás seguro de que deseas eliminar este libro?</p>
      <div class="styleBDelete">
        <b-button variant="danger" @click="confirmDelete">Eliminar</b-button>
        <b-button variant="secondary" @click="cancelDelete">Cancelar</b-button>
      </div>
    </b-modal>
  </div>
</template>

<script>
import {
  getBooks,
  getBooksByAuthor,
  getBooksByPublicationDate,
  addNewBook,
  updateBook,
  deleteBook,
} from "../service/index";
import BookCarousel from "../components/BookCarousel.vue";
import BookControls from "../components/BookControls.vue";
import BookList from "../components/BookList.vue";
import Actions from "./Actions.vue";
export default {
  components: {
    BookCarousel,
    BookControls,
    BookList,
    Actions,
  },
  data() {
    return {
      books: [],
      slide: 0,
      sliding: null,
      scrolled: false,
      insertModalVisible: false,
      editModalVisible: false,
      deleteModalVisible: false,
      draggedBook: null,
      bookToDeleteId: null,
      form: {
        title: "",
        author: "",
        publicationDate: null,
        image:""
      },
      showInsertForm: true,
    };
  },
  methods: {
    async getAllBooks() {
      try {
        let res = await getBooks();
        this.books = res.data;
      } catch (error) {
        console.error("Error getAllBooks>>", error);
      }
    },
    async getAllBooksByAuthor() {
      try {
        let res = await getBooksByAuthor();
        this.books = res.data;
      } catch (error) {
        console.error("Error getAllBooksByAuthor>>", error);
      }
    },
    async getAllBooksByPublicationDate() {
      try {
        let res = await getBooksByPublicationDate();
        this.books = res.data;
      } catch (error) {
        console.error("Error getAllBooksByPublicationDate>>", error);
      }
    },
    async addBook(ev) {
      ev.preventDefault();
      try {
        console.log(this.form)
        let res = await addNewBook(this.form);
        console.log("res",res)
        if (res.httpStatusCode === 201) {
          res = this.getAllBooks();
          this.books = res.data;
          this.form.title = "";
          this.form.author = "";
          this.form.publicationDate = null;
          this.insertModalVisible = false;
        }else if(res.httpStatusCode===409){
          alert(res.message)
        }
      } catch (error) {
        console.error("Error registrando libro>>", error);
      }
    },
    async editBook(ev) {
      ev.preventDefault();
      try {
        let res = await updateBook(this.draggedBook.id, this.form);
        console.log(res);
        if (res.httpStatusCode === 200) {
          res = this.getAllBooks();
          this.books = res.data;
          this.form.title = "";
          this.form.author = "";
          this.form.publicationDate = null;
          this.editModalVisible = false;
        }
      } catch (error) {
        console.error("Error al actualizar libro>>", error);
      }
      this.draggedBook = null;
      // Cierra el modal de editar
      this.editModalVisible = false;
    },

    handleScroll() {
      if (window.scrollY > 0) {
        this.scrolled = true;
      } else {
        this.scrolled = false;
      }
    },
    // Mostrar modal para insertar
    showInsertModal() {
      this.insertModalVisible = true;
      this.form.title = "";
      this.form.author = "";
      this.form.publicationDate = "";
    },
    // Mostrar modal para editar
    showEditModal() {
      if (this.draggedBook) {
        this.form.title = this.draggedBook.title;
        this.form.author = this.draggedBook.author;
        this.form.publicationDate = this.draggedBook.publicationDate;
        this.editModalVisible = true;
      }
    },
    // Mostrar modal para eliminar
    showDeleteModal() {
      if (this.draggedBook) {
        this.deleteModalVisible = true;
        this.bookToDeleteId = this.draggedBook.id;
      }
    },
    // Iniciar arrastrar
    dragStart(book) {
      this.draggedBook = book;
    },
    // Confirmar eliminación
    confirmDelete() {
      console.log(this.bookToDeleteId);
      this.deleteModalVisible = false;
      if (this.bookToDeleteId) {
        deleteBook(this.bookToDeleteId)
          .then(() => {
            this.getAllBooks();
          })
          .catch((error) => {
            console.error("Error al eliminar libro:", error);
          });
        this.bookToDeleteId = null;
      }
    },
    // Cancelar eliminación
    cancelDelete() {
      this.deleteModalVisible = false;
      this.bookToDeleteId = null;
    },
    // Manejador para eliminar libro
    deleteBookHandler(ev) {
      ev.preventDefault();
      if (this.draggedBook) {
        this.bookToDeleteId = this.draggedBook.id;
        this.deleteModalVisible = true;
      }
    },
  },
  mounted() {
    this.getAllBooks();
    window.addEventListener("scroll", this.handleScroll);
  },
  destroyed() {
    window.removeEventListener("scroll", this.handleScroll);
  },
};
</script>

<style>
.styleBDelete{
  justify-content: space-around;
  display: flex;
}

.close-tag{
    cursor: pointer;
    font-size: large;
    font-weight: bold;
    color: #ed0303;
}
</style>
