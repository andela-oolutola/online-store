<template>
  <div class="row">
    <div class="col-sm-6 col-sm-push-3 col-xs-12">
      <div class="panel">
        <div class="panel-body">
          <form>
            <div class="form-group">
              <label for="firstname">FirstName:</label>
              <input type="text" class="form-control" id="firstname" placeholder="First Name" v-model="firstname">
            </div>
            <div class="form-group">
              <label for="lastname">LastName:</label>
              <input type="text" class="form-control" id="lastname" placeholder="Last Name" v-model="lastname">
            </div>
            <div class="form-group">
              <label for="email">Email address</label>
              <input type="email" class="form-control" id="email_field" placeholder="Email Address" v-model="email_address">
            </div>
            <div class="form-group">
              <label for="password">Password</label>
              <input type="password" class="form-control" id="password_field" placeholder="Password" v-model="password">
            </div>
            <div class="form-group">
              <label for="password">Confirm Password</label>
              <input type="password" class="form-control" id="confirm_password_field" placeholder="Confirm Password" v-model="confirm_password">
            </div>
            <p class="text-center text-danger"><strong>{{error_message}}</strong></p>
            <button type="submit" class="btn btn-primary" v-on:click.stop.stop.prevent="register">Submit</button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import firebaseApp from '../firebase';
import VueRouter from '../router/index';

const router = VueRouter.router;

export default{
  data() {
    return {
      email_address: '',
      password: '',
      firstname: '',
      lastname: '',
      confirm_password: '',
      authenticated: false,
      error_message: '',
    };
  },
  methods: {
    register() {
      // console.log(this.email_address);
      if (this.firstname.length > 0 &&
          this.lastname.length &&
          this.password.length > 0 &&
          this.confirm_password > 0 && this.email_address.length > 0) {
        if (this.password === this.confirm_password) {
          const payload = { name: this.fullname, email: this.email_address };
          firebaseApp.
          firebaseAuth.
          createUserWithEmailAndPassword(this.email_address, this.password).catch((error) => {
            // Handle Errors here.
            this.errorMessage(error.message);
          }).then((user) => {
            console.log(user.uid);
            if (user !== null && typeof user !== 'undefined') {
              this.addUser(payload, user.uid);
            }
          });
        } else {
          this.error_message = 'Please enter the same password';
        }
      } else {
        this.error_message = 'You have an empty field';
      }
    },
    addUser(payload, uid) {
      firebaseApp.firebaseDB.ref(`/users/${uid}`).set(payload);
      this.authenticated = true;
      window.localStorage.setItem('online_store_uid', uid);
      window.localStorage.setItem('online_store_user_authenticated', true);
      window.localStorage.setItem('online_store_user_name', payload.name.split(' ')[0]);
      window.localStorage.setItem('online_store_user_email', payload.email);
      location.reload();
      router.go({ path: '/shops' });
    },
    errorMessage(errorMessage) {
      this.error_message = errorMessage;
    },
    checkAuth() {
      const authCondition = localStorage.getItem('online_store_user_authenticated');

      if (authCondition !== null && authCondition) {
        router.go({ path: '/shops' });
      }
    },
  },
  ready() {
    this.checkAuth();
  },
  computed: {
    fullname() {
      return `${this.firstname} ${this.lastname}`;
    },
  },
};
</script>
