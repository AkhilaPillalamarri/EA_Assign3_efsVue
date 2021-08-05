<template>
  <v-main>
  <!-- Welcome title -->
    <v-container fluid>
      <v-row align="center" justify="center">
        <v-col cols="12" align="center" justify="center">
          <blockquote>
          Welcome {{validUserName}}!
            <footer>
              <small>
                <em>&mdash;Eagle Financial Services, your Midwest Financial Services
                Partner.</em>
              </small>
            </footer>
          </blockquote>
        </v-col>
        <v-col cols="12" md="10" lg="10" align="center" justify="center">
          <v-alert v-if="showMsg === 'new'"
          dismissible
          :value="true"
          type="success"
          >
          New investment has been added.
          </v-alert>
          <v-alert v-if="showMsg === 'update'" dismissible
          :value="true"
          type="success"
          >
          Investment information has been updated.
          </v-alert>
          <v-alert v-if="showMsg === 'delete'" dismissible
          :value="true"
          type="success"
          >
          Selected Investment has been deleted.
          </v-alert>
        </v-col>
      </v-row>
    <!-- Data table -->
    <v-row align="center" justify="center">
        <v-col cols="12" md="10" v-resize="onResize">
          <v-data-table
          :headers="headers"
          :items="investments"
          class="elevation-1"
          style="max-height: 300px; overflow-y: auto"
          v-if="!isMobile"
          >
            <template v-slot:item="props">
              <tr>
                <td align="left">{{ props.item.cust_number }}</td>
                <td align="left">{{ props.item.category }}</td>
                <td nowrap="true" align="left">{{ props.item.description }}</td>
                <td nowrap="true" align="left">{{ props.item.acquired_value }}</td>
                <td nowrap="true" align="left">{{ props.item.acquired_date }}</td>
                <td nowrap="true" align="left">{{ props.item.recent_value }}</td>
                <td nowrap="true" align="left">{{ props.item.recent_date }}</td>
                <td align="center"><v-icon @click="updateInvestment(props.item)">mdi-pencil</v-icon></td>
                <td align="center"><v-icon :id="'deleteIcon-' + props.item.pk" class="small" @click.stop="popupId = props.item.pk; dialog = true;">mdi-delete</v-icon>

                  <v-dialog
                    v-model="dialog"
                    max-width="290"
                    v-if="popupId === props.item.pk"
                  >
                    <v-card>
                      <v-card-title class="headline">
                        Delete Investment?
                      </v-card-title>

                      <v-card-text>
                        Are you sure you want to delete this investment ?
                      </v-card-text>

                      <v-card-actions>
                        <v-spacer></v-spacer>

                        <v-btn
                          color="black darken-1"
                          text
                          @click="dialog = false"
                        >
                          Cancel
                        </v-btn>

                        <v-btn
                          color="red darken-1"
                          text
                          @click="deleteInvestment(popupId);"
                        >
                          Delete
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-dialog>
                </td>
              </tr>
            </template>
          </v-data-table>
        <v-data-iterator
        :items="investments"
        hide-default-footer
        v-else
        >
        <template v-slot:default="{ items, isExpanded, expand }">
        <v-row>
        <v-col
        v-for="item in items"
        :key="item.name"
        cols="12"
        >
        <v-card>
        <v-card-title class="pb-0 pt-0 pl-0">
        <v-col cols="9" class="text-left body-2 text-truncate">{{item.cust_number}} -
        {{item.category }}</v-col>
        <v-col cols="3" class="text-center">
        <v-card-actions>
        <v-icon @click="updateInvestment(item)" class="small">mdi-pencil</v-icon>
        <v-icon :id="'deleteIcon-' + item.pk" class="small" @click.stop="popupId = item.pk; dialog = true;">mdi-delete</v-icon>

          <v-dialog
            v-model="dialog"
            max-width="290"
            v-if="popupId === item.pk"
          >
            <v-card>
              <v-card-title class="headline">
                Delete Investment?
              </v-card-title>

              <v-card-text>
                Are you sure you want to delete this investment ?
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>

                <v-btn
                  color="black darken-1"
                  text
                  @click="dialog = false"
                >
                  Cancel
                </v-btn>

                <v-btn
                  color="red darken-1"
                  text
                  @click="deleteInvestment(popupId);"
                >
                  Delete
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        <v-icon @click.native="expand(item, !isExpanded(item))"
        class="small">mdi-dots-horizontal</v-icon>
        </v-card-actions>
        </v-col>
        </v-card-title>
        <v-divider></v-divider>
        <v-list v-show="isExpanded(item)" dense>
        <v-list-item>
        <v-list-item-content class="align-end">{{ item.description }}</v-list-item-content>
        </v-list-item>
        <v-list-item>
        <v-list-item-content>Category:</v-list-item-content>
        <v-list-item-content class="align-end">{{ item.category }}</v-list-item-content>
        </v-list-item>
        <v-list-item>
        <v-list-item-content>Acquired Value:</v-list-item-content>
        <v-list-item-content class="align-end">{{ item.acquired_value }} on {{item.acquired_date }}</v-list-item-content>
        </v-list-item>
        <v-list-item>
        <v-list-item-content>Recent value:</v-list-item-content>
        <v-list-item-content class="align-end">{{ item.recent_value }} on {{item.recent_date }}</v-list-item-content>
        </v-list-item>
        </v-list>
        </v-card>
        </v-col>
        </v-row>
        </template>
        </v-data-iterator>
        <v-btn class="blue mt-4 white--text" @click="addNewInvestment">Add Investment</v-btn>
        </v-col>
      </v-row>
    </v-container>
  </v-main>
</template>

<script>
import router from '../router';
import {APIService} from '../http/APIService';
const apiService = new APIService();
export default {
name: "InvestmentList",
data: () => ({
investments: [],
  dialog : false,
  popupId : '',
validUserName: "Guest",
investmentSize: 0,
showMsg: '',
isMobile: false,
headers: [
{text: 'Customer Number', sortable: false, align: 'left',},
{text: 'Category', sortable: false, align: 'left',},
{text: 'Description', sortable: false, align: 'left',},
{text: 'Acquired_Value', sortable: false, align: 'left',},
{text: 'Acquired_Date', sortable: false, align: 'left',},
{text: 'Recent_Value', sortable: false, align: 'left',},
{text: 'Recent_Date', sortable: false, align: 'left',},
{text: 'Update', sortable: false, align: 'left',},
{text: 'Delete', sortable: false, align: 'left',}
],
}),
mounted() {
  this.dialog = false;
  this.popupId = '';
this.getInvestments();
this.showMessages();
},
methods: {
onResize() {
if (window.innerWidth < 600)
this.isMobile = true;
else
this.isMobile = false;
},
showMessages(){
console.log(this.$route.params.msg)
if (this.$route.params.msg) {
this.showMsg = this.$route.params.msg;
}
},
getInvestments() {
apiService.getInvestmentList().then(response => {
this.investments = response.data.data;
this.investmentSize = this.investments.length;
if (localStorage.getItem("isAuthenticates")
&& JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
this.validUserName = JSON.parse(localStorage.getItem("log_user"));
}
}).catch(error => {
if (error.response.status === 401) {
localStorage.removeItem('isAuthenticates');
localStorage.removeItem('log_user');
localStorage.removeItem('token');
router.push("/auth");
}
});
},
addNewInvestment() {
if (localStorage.getItem("isAuthenticates")
&& JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
router.push('/investment-create');
} else {
router.push("/auth");
}
},
updateInvestment(investment) {
router.push('/investment-create/' + investment.pk);
},
deleteInvestment(investment) {
apiService.deleteInvestment(investment).then(response => {
if (response.status === 204) {
  this.dialog = false;
  this.showMsg = 'delete';
  this.getInvestments()
}
}).catch(error => {
if (error.response.status === 401) {
localStorage.removeItem('isAuthenticates');
localStorage.removeItem('log_user');
localStorage.removeItem('token');
router.push("/auth");
}
});
}
}
};
</script>
