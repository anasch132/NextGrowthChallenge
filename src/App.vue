<template>
  <v-app class="blue" style="background-color: beige;">
<v-container class='my-auto mx-auto' >
  <v-data-table
    :headers="headers"
    :items="users"
    sort-by="calories"
    class="elevation-1 rounded-lg pa-4"
  >
    <template v-slot:top>
      <v-toolbar
        flat
        class="rounded-lg"
      >
        <v-toolbar-title></v-toolbar-title>
        <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="70%"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="#FFAA0C"
              dark
              class="mb-2 brown--text"
              v-bind="attrs"
              v-on="on"
            >
              Ajouter Utilisateur
            </v-btn>
          </template>
          <v-card class="pa-3">
            <v-card-title class="mb-2">
              <span class="text-h4 font-weight-bold">Ajout d'utilisateur</span>
              <v-spacer></v-spacer>
              <v-icon color="red" @click="close()">
                mdi-close-circle
              </v-icon>
            </v-card-title>

            <v-card-text>
              <v-container>
        
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                    outlined
                    :error-messages="firstNameErrors"

                    required
                      v-model="editedItem.firstName"
                      label="Nom"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                    outlined
                    :error-messages="lastNameErrors"
  
                    required
                      v-model="editedItem.lastName"
                      label="Prenom"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                    outlined
                    :error-messages="userNameErrors"
    
                    required
                      v-model="editedItem.userName"
                      label="Nom d'utilisateur"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-select
                    outlined
                    :error-messages="statusErrors"
        
                    required
                    :items="status"
                    v-model="editedItem.status"
                    label="Etat"
                    ></v-select>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                    outlined
                    :error-messages="registrationNumberErrors"
                   
                    required
                      v-model="editedItem.registrationNumber"
                      label="Matricule"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-menu
        ref="menu"
        v-model="menu"
        :close-on-content-click="false"
        :return-value.sync="editedItem.createdDate"
        transition="scale-transition"
        offset-y
        min-width="auto"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-text-field
          outlined
          :error-messages="createdDateErrors"
       
          required
            v-model="editedItem.createdDate"
            label="Date de creation"
            readonly
            v-bind="attrs"
            v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker
          v-model="editedItem.createdDate"
          no-title
          scrollable
        >
          <v-spacer></v-spacer>
          <v-btn
            text
            color="primary"
            @click="$refs.menu.save(editedItem.createdDate)"
          >
            OK
          </v-btn>
        </v-date-picker>
      </v-menu>
                  </v-col>
                </v-row>

              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="#FFAA0C"
                class="brown--text"
                @click="save"
              >
                Enregistrer
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="595px">
          <v-card>
            <v-card-title class="text-h5 mx-auto">Êtes-vous sûr de vouloir supprimer cet utilisateur ?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Fermer</v-btn>
              <v-btn color="red darken-1" text @click="deleteItemConfirm">Oui</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
     <template v-slot:item.createdDate="{ item }">
      {{ formatDate(item.createdDate) }}
    </template>
    <template v-slot:item.status="{ item }">
      <v-chip
      :color="ChipColor(item.status)"
      label
      class="white--text"
      width="70%"
    >
          {{ item.status }}
    </v-chip>
      
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-trash-can-outline
      </v-icon>
    </template>
    <template v-slot:no-data>
      Aucune donnée disponible
    </template>
  </v-data-table>
</v-container>
  </v-app>
</template>

<script>
import { validationMixin } from 'vuelidate'
import { required} from 'vuelidate/lib/validators'
import moment from 'moment'

export default {
  name: 'App',
  mixins: [validationMixin],

    validations: {
      editedItem: {
        firstName: { required},
        lastName: { required},
        userName: { required},
        status: { required},
        registrationNumber: { required},
        createdDate: { required},
      }
    },
  data: () => ({
      dialog: false,
      menu: false,
      dialogDelete: false,
      headers: [
        {
          text: 'ID',
          align: 'start',
          sortable: false,
          value: 'id',
        },
        { text: 'Date de creation', value: 'createdDate', align: 'start' },
        { text: 'Etat', value: 'status' , align: 'center'},
        { text: 'Nom', value: 'firstName', align: 'start' },
        { text: 'Prenom', value: 'lastName', align: 'start' },
        { text: "Nom d'utilisateur", value: 'userName', align: 'start' },
        { text: 'Matricule', value: 'registrationNumber', align: 'start' },
        { text: 'Actions', value: 'actions', sortable: false, align: 'center' },
      ],
      users: [],
      status: ['En validation' , 'Validé' , 'Rejeté'],
      editedIndex: -1,
      editedItem: {
        id: 0,
        createdDate: '',
        status: '',
        firstName: '',
        lastName: '',
        userName: '',
        registrationNumber: '',
      },
      defaultItem: {
        id: 0,
        createdDate: '',
        status: '',
        firstName: '',
        lastName: '',
        userName: '',
        registrationNumber: '',
      },
    }),

    computed: {
      firstNameErrors () {
        const errors = []
        if (!this.$v.editedItem.firstName.$dirty) return errors
        !this.$v.editedItem.firstName.required && errors.push('Nom est obligatoire.')
        return errors
      },
      lastNameErrors () {
        const errors = []
        if (!this.$v.editedItem.lastName.$dirty) return errors
        !this.$v.editedItem.lastName.required && errors.push('Prenom est obligatoire.')
        return errors
      },
      userNameErrors () {
        const errors = []
        if (!this.$v.editedItem.userName.$dirty) return errors
        !this.$v.editedItem.userName.required && errors.push("Nom d'utilisateur est obligatoire.")
        return errors
      },
      createdDateErrors () {
        const errors = []
        if (!this.$v.editedItem.createdDate.$dirty) return errors
        !this.$v.editedItem.createdDate.required && errors.push('Date de creation est obligatoire.')
        return errors
      },
      statusErrors () {
        const errors = []
        if (!this.$v.editedItem.status.$dirty) return errors
        !this.$v.editedItem.status.required && errors.push('Etat est obligatoire.')
        return errors
      },
      registrationNumberErrors () {
        const errors = []
        if (!this.$v.editedItem.registrationNumber.$dirty) return errors
        !this.$v.editedItem.registrationNumber.required && errors.push('Matricule est obligatoire.')
        return errors
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      // format the dates values
       formatDate(value) {
      return moment(value).format("DD/MM/YYYY")
      },
      // fill the table with initial data
      initialize () {
        this.users = [
          {
          id: "123456789",
          createdDate: "2021-01-06T00:00:00.000Z",
          status: "En validation",
          firstName: "Mohamed",
          lastName: "Taha",
          userName: "mtaha",
          registrationNumber: "2584",
        },
        {
          id: "987654321",
          createdDate: "2021-07-25T00:00:00.000Z",
          status: "Validé",
          firstName: "Hamid",
          lastName: "Orrich",
          userName: "horrich",
          registrationNumber: "1594",
        },
          {
          id: "852963741",
          createdDate: "2021-09-15T00:00:00.000Z",
          status: "Rejeté",
          firstName: "Rachid",
          lastName: "Mahidi",
          userName: "rmahidi",
          registrationNumber: "3576",
        }
        ]
      },
    // Open dialog to confirm delete member
      deleteItem (item) {
        this.editedIndex = this.users.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },
      // Delete member after click on confirmation button
      deleteItemConfirm () {
        this.users.splice(this.editedIndex, 1)
        this.closeDelete()
      },

        // Close Add new memeber dialog 
      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
        this.$v.$reset()
      },

      //  Close delete dialog
      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },
      // generate ID for the new member
      makeid(length) {
                var result           = '';
                var characters       = '0123456789';
                var charactersLength = characters.length;
                for ( var i = 0; i < length; i++ ) {
                result += characters.charAt(Math.floor(Math.random() * 
            charactersLength));
            }
            return result;
            },

            // Add new member
      save () {
        this.$v.$touch()
        if (!this.$v.$invalid){
          console.log(this.editedItem)
        this.editedItem.id = this.makeid(9)
        if (this.editedIndex > -1) {
          Object.assign(this.users[this.editedIndex], this.editedItem)
        } else {
          this.users.push(this.editedItem)
        }
        this.close()
        }
      },

      ChipColor(status) {
        if (status == 'Rejeté')
          return "#FF0000"
        else if (status == 'Validé')
          return "#5BE881"

        else if (status == 'En validation')
          return "#FDB64D"
        else
          return "grey"
      }
    },
};
</script>


<style scoped lang = "scss">

.text-btn-color {
  color: #776954;
}

.v-data-table::v-deep th {
  font-size: 15px !important;
  color: black !important;
}
.v-data-table::v-deep td {
  font-size: 14px !important;
}
</style>