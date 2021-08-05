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
            New fund has been added.
          </v-alert>
          <v-alert v-if="showMsg === 'update'" dismissible
                   :value="true"
                   type="success"
          >
            Fund information has been updated.
          </v-alert>
          <v-alert v-if="showMsg === 'delete'" dismissible
                   :value="true"
                   type="success"
          >
            Selected Fund has been deleted.
          </v-alert>
        </v-col>
      </v-row>
      <!-- Data table -->
      <v-row align="center" justify="center">
        <v-col cols="12" md="10" v-resize="onResize">
          <v-data-table
            :headers="headers"
            :items="funds"
            class="elevation-1"
            style="max-height: 300px; overflow-y: auto"
            v-if="!isMobile"
          >
            <template v-slot:item="props">
              <tr>
                <td align="left">{{ props.item.cust_number }}</td>
                <td align="left">{{ props.item.symbol }}</td>
                <td nowrap="true" align="left">{{ props.item.description }}</td>
                <td nowrap="true" align="left">{{ props.item.quantity }}</td>
                <td nowrap="true" align="left">{{ props.item.purchase_price }}</td>
                <td nowrap="true" align="left">{{ props.item.purchase_date }}</td>
                <td align="center"><v-icon @click="updateFund(props.item)">mdi-pencil</v-icon></td>
                <td align="center"><v-icon :id="'deleteIcon-' + props.item.id" class="small" @click.stop="popupId = props.item.id; dialog = true;">mdi-delete</v-icon>

                  <v-dialog
                    v-model="dialog"
                    max-width="290"
                    v-if="popupId === props.item.id"
                  >
                    <v-card>
                      <v-card-title class="headline">
                        Delete Fund?
                      </v-card-title>

                      <v-card-text>
                        Are you sure you want to delete this fund ?
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
                          @click="deleteFund(popupId);"
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
            :items="funds"
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
                        {{item.symbol }}</v-col>
                      <v-col cols="3" class="text-center">
                        <v-card-actions>
                          <v-icon @click="updateFund(item)" class="small">mdi-pencil</v-icon>
                          <v-icon :id="'deleteIcon-' + item.id" class="small" @click.stop="popupId = item.id; dialog = true;">mdi-delete</v-icon>

                          <v-dialog
                            v-model="dialog"
                            max-width="290"
                            v-if="popupId === item.id"
                          >
                            <v-card>
                              <v-card-title class="headline">
                                Delete Fund?
                              </v-card-title>

                              <v-card-text>
                                Are you sure you want to delete this fund ?
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
                                  @click="deleteFund(popupId);"
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
                        <v-list-item-content>Symbol:</v-list-item-content>
                        <v-list-item-content class="align-end">{{ item.symbol }}</v-list-item-content>
                      </v-list-item>
                      <v-list-item>
                        <v-list-item-content>Shares:</v-list-item-content>
                        <v-list-item-content class="align-end">{{ item.quantity }}</v-list-item-content>
                      </v-list-item>
                      <v-list-item>
                        <v-list-item-content>Purchased Price:</v-list-item-content>
                        <v-list-item-content class="align-end">{{ item.purchase_price }} on {{item.purchase_date }}</v-list-item-content>
                      </v-list-item>
                    </v-list>
                  </v-card>
                </v-col>
              </v-row>
            </template>
          </v-data-iterator>
          <v-btn class="blue mt-4 white--text" @click="addNewFund">Add Fund</v-btn>
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
  name: "FundList",
  data: () => ({
    funds: [],
    dialog : false,
    popupId : '',
    validUserName: "Guest",
    fundSize: 0,
    showMsg: '',
    isMobile: false,
    headers: [
      {text: 'Customer Number', sortable: false, align: 'left',},
      {text: 'Symbol', sortable: false, align: 'left',},
      {text: 'Fund Description', sortable: false, align: 'left',},
      {text: 'Quantity', sortable: false, align: 'left',},
      {text: 'Purchase Price', sortable: false, align: 'left',},
      {text: 'Purchase Date', sortable: false, align: 'left',},
      {text: 'Update', sortable: false, align: 'left',},
      {text: 'Delete', sortable: false, align: 'left',}
    ],
  }),
  mounted() {
    this.dialog = false;
    this.popupId = '';
    this.getFunds();
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
    getFunds() {
      apiService.getFundList().then(response => {
        this.funds = response.data.data;
        this.fundSize = this.funds.length;
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
    addNewFund() {
      if (localStorage.getItem("isAuthenticates")
        && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
        router.push('/fund-create');
      } else {
        router.push("/auth");
      }
    },
    updateFund(fund) {
      router.push('/fund-create/' + fund.id);
    },
    deleteFund(fund) {
      apiService.deleteFund(fund).then(response => {
        if (response.status === 204) {
          this.dialog = false;
          this.showMsg = 'delete';
          this.getFunds()
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
