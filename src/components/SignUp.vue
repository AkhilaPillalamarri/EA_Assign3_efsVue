<template>
  <v-container grid-list-md>
    <v-row align="center" justify="center">
      <v-col class="align-center">
        <v-row class="align-center" justify="center">
          <v-col cols="12" sm="10" md="10" lg="6" class="align-center">
            <v-alert v-if="showMsg === 'error'"
                     dismissible
                     :value="true"
                     type="error"
            >
              Please verify your information.
            </v-alert>
          </v-col>
        </v-row>
        <v-row align="center" justify="center">
          <v-col cols="12" sm="10" md="10" lg="6" class="align-center">
            <v-card class="login-card">
              <v-card-title>
                <span class="headline">{{pageTitle}}</span>
              </v-card-title>
              <v-spacer/>
              <v-card-text>
                <v-form ref="form" lazy-validation>
                  <v-container>
                    <v-text-field
                      v-model="user.first_name"
                      :rules="[rules.required]"
                      label="First Name"
                      required
                    />
                    <v-text-field
                      v-model="user.last_name"
                      :rules="[rules.required]"
                      label="Last Name"
                      required
                    />
                    <v-text-field
                      v-model="user.email"
                      :rules="[rules.required, rules.email]"
                      label="Email"
                      required
                    />
                    <v-text-field
                      v-model="user.username"
                      :rules="[rules.required, rules.username]"
                      label="Username"
                      required
                    />
                    <v-text-field
                      v-model="user.password"
                      label="Password"
                      :rules="[rules.required, rules.min]"
                      required
                      :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="showPassword ? 'text' : 'password'"
                      @click:append="showPassword = !showPassword"
                      hint="At least 8 characters"
                      counter
                    />
                    <v-text-field
                      v-model="passwordConfirm"
                      label="Confirm Password"
                      :rules="[rules.required, rules.min, rules.passwordMatch]"
                      required
                      :append-icon="showConfirmPassword ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="showConfirmPassword ? 'text' : 'password'"
                      @click:append="showConfirmPassword = !showConfirmPassword"
                      hint="At least 8 characters"
                      counter
                    ></v-text-field>
                    <v-btn class="blue white--text" @click="registerUser">Register</v-btn>
                    <v-btn class="blue white--text"  @click="reset">Reset</v-btn>
                  </v-container>
                </v-form>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import router from "../router";
import {APIService} from '../http/APIService';
const apiService = new APIService();
export default {
name: "SignUp",
  data() {
    return {
      showError: false,
      showMsg: '',
      showConfirmPassword : false,
      showPassword : false,
      pageTitle : 'User Registration',
      user: {

      },
      rules: {
        required: value => !!value || 'Required.',
        min: v => v.length >= 8 || 'Min 8 characters',
        passwordMatch : value => value == this.$data.user.password || 'Password did not match',
        username : v => /^[a-z0-9_]+$/.test(v) || "A username can only contain letters and digits",
        email : v =>  /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(v) || "Enter a valid email address",
      },
      passwordConfirm: '',
    }
    },
  methods : {
      registerUser() {
        apiService.registerUser(this.user).then(response => {
          if (response.status === 201) {
            this.user = response.data;
            this.showMsg = "";
            router.push('/auth');
          }else{
            this.showMsg = "error";
          }
        }).catch(error => {
          if (error.response.status === 401) {
            router.push("/auth");
          }else if (error.response.status === 400) {
            this.showMsg = "error";
          }
        });
      },
    reset(){
      this.$refs.form.reset()
    }
  },
}
</script>

<style scoped>

</style>
