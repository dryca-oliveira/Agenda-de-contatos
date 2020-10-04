<template>
  <v-container>
    <v-row justify="space-around" align="center" class="pt-10">
      <v-col cols="12" md="4">
        <!-- incio formulário -->
        <v-form ref="form">
          <v-card class="back" elevation="15">
            <v-row justify="center" class="pl-3">
              <v-col cols="8" md="8">
                <v-text-field
                  v-model="form.name"
                  :rules="dataRules"
                  counter="35"
                  maxlength="35"
                  label="Nome"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row justify="center" class="pl-3">
              <v-col cols="8" md="8">
                <v-text-field
                  v-model="form.telephoneNumber"
                  :rules="dataRules"
                  label="Telefone"
                  v-mask="'(##)####-####'"
                  append-outer-icon="fas fa-phone-square"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row justify="center" class="pl-3">
              <v-col cols="8" md="8">
                <v-text-field
                  v-model="form.email"
                  :rules="emailRules"
                  v-on:keyup.enter="addContact()"
                  label="E-mail"
                  append-outer-icon="fas fa-at"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row justify="center" class="pb-5">
              <v-btn
                class="mr-4"
                color="grey darken-3"
                small
                dark
                rounded
                @click="clear"
              >
                limpar
              </v-btn>
              <v-btn
                color="green darken-3"
                rounded
                dark
                small
                @click="addContact"
              >
                adicionar
              </v-btn>
            </v-row>
          </v-card>
        </v-form>
        <!-- fim formulário -->
      </v-col>

      <v-col cols="12" md="8" class="pl-15 pr-0">
        <!-- incio tabela -->
        <v-row justify="center">
          <v-col md="12">
            <v-data-table
              :headers="headers"
              :items="contacts"
              :search="search"
              dark
              sort-by="calories"
              class="elevation-10 gradient"
            >
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Meus Contatos</v-toolbar-title>

                  <v-spacer></v-spacer>
                  <v-row justify="end">
                    <v-col md="5">
                      <v-text-field
                        v-model="search"
                        append-icon="fas fa-search"
                        label="Buscar"
                        single-line
                        hide-details
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-toolbar>
              </template>
              <template v-slot:item.actions="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)">
                  far fa-edit
                </v-icon>
                <v-icon small @click="deleteItem(item)"> fas fa-trash </v-icon>
              </template>
            </v-data-table>
          </v-col>
        </v-row>
        <!-- fim tabela -->
      </v-col>
    </v-row>
    <!-- incio card edição -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">Editar Contato</span>
        </v-card-title>

        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="editedItem.name"
                  label="Nome"
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="editedItem.telephoneNumber"
                  label="Telefone"
                  v-mask="'(##)####-####'"
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="editedItem.email"
                  label="E-mail"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="gray darken-1" text rounded small @click="close">
            Cancelar
          </v-btn>
          <v-btn color="green darken-1" dark rounded small @click="save">
            Salvar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- fim card edição -->

    <!-- incio card edição -->
    <v-dialog v-model="dialog2" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">Excluir Contato</span>
        </v-card-title>

        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <h3>Deseja cancelar este contato realmente?</h3>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="grey darken-1" text rounded small @click="close">
            Cancelar
          </v-btn>
          <v-btn
            color="green darken-1"
            dark
            rounded
            small
            @click="deleteItemConfirm"
          >
            Confirmar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- fim card edição -->

    <!-- incio card salvo com sucesso -->
    <v-dialog v-model="dialog3" persistent max-width="260">
      <v-card>
        <v-card-title
          class="headline white--text"
          style="background-color: #4286f4"
          primary-title
        >
          Adicionar contrato
        </v-card-title>
        <div class="pa-5">
          <h4 class="pl-5">Cadastrado com sucesso!</h4>
        </div>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue" dark @click="dialog3 = false">ok</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- fim card salvo com sucesso -->
  </v-container>
</template>
<script>
export default {
  data: () => ({
    form: {
      name: "",
      email: "",
      telephoneNumber: "",
    },
    search: "",
    dataRules: [(v) => !!v || "campo obrigatório"],
    emailRules: [
      (v) => !!v || "E-mail obrigatório",
      (v) => /.+@.+\..+/.test(v) || "E-mail não é válido",
    ],
    dialog: false,
    dialog2: false,
    dialog3: false,
    headers: [
      { text: "Cod", value: "id", class: "grey lighten-1 white--text" },
      { text: "Nome", value: "name", class: "grey lighten-1 white--text" },
      { text: "Telefone", value: "telephoneNumber", class: "grey lighten-1 white--text" },
      { text: "E-mail", value: "email", class: "grey lighten-1 white--text" },
      {
        text: "Editar/Excluir",
        value: "actions",
        sortable: false,
        class: "grey lighten-1 white--text text-center",
      },
    ],
    contacts: [],
    editedIndex: -1,
    editedItem: {
      name: "",
      telephoneNumber: 0,
      email: 0,
    },
    defaultItem: {
      name: "",
      telephoneNumber: 0,
      femail: 0,
    },
  }),

  created() {
    this.initialize();
  },

  methods: {
    async addContact() {
      console.log(this.form);
      if (this.$refs.form.validate()) {
        await this.$http
          .post("agenda", this.form)
          .then(() => {
            this.dialog3 = true;
          })
          .catch((erro) => {
            console.log(erro);
          });
        this.initialize();
        this.clear();
      }
    },

    resetValidation() {
      this.$refs.form.resetValidation();
    },

    clear() {
      this.$refs.form.reset();
    },

    submit() {
      this.$v.$touch();
    },

    async initialize() {
      await this.$http
        .get("agenda")
        .then((resp) => {
          this.contacts = resp.data;
        })
        .catch((erro) => {
          console.log(erro);
        });
    },

    editItem(item) {
      this.editedIndex = this.contacts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    async save() {
      await this.$http
        .patch("agenda", this.editedItem)
        .then(() => {          
          this.dialog = false;
        })
        .catch((erro) => {
          console.log(erro);
        });
      this.initialize();
    },

    close() {
      this.dialog = false;
      this.dialog2 = false;
    },

    deleteItem(item) {
      this.editedIndex = this.contacts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog2 = true;
    },

    async deleteItemConfirm() {
      await this.$http
        .delete("agenda/delete/" + this.editedItem.id)
        .then(() => {
          this.contacts.splice(this.editedIndex, 1);
          this.dialog2 = false;
        })
        .catch((erro) => {
          console.log(erro);
        });
    },
  },
};
</script>
<style scoped>
.back {
  background-image: url("../assets/Imagem1.png");
  background-size: 145%;
  background-position-x: 50%;
  background-position-y: 60%;
  filter: brightness(140%);
}
.gradient {
  background: linear-gradient(to right, #373b44, #4286f4);
}
</style>
