<template>
    <modal-layout>
      <template v-slot:title>{{ $t('user_creation') }}</template>
      <div v-if="!loading">
        <div v-for="(user, index) in users" :key="index" class="row">
          <text-input type="text" class="col s6" v-model="user.firstName" v-bind:label="$t('first_name')">
          </text-input>
          <text-input type="text" class="col s6" v-model="user.lastName" v-bind:label="$t('last_name')">
          </text-input>
        </div>
        <button @click="addUser" class="waves-effect waves-light btn-small">+</button>
      </div>
      <div v-else class="row">
        {{ $t("helper.creating_user") }}
      </div>
      <template v-slot:footer>
        <button @click="close()" v-bind:disabled="loading" class="waves-effect waves-green btn-flat">{{ $t('cancel') }}</button>
        <button @click="signup()" v-bind:disabled="loading || !validateUsers()" class="waves-effect waves-green btn-flat">{{ $t('validate') }}</button>
      </template>
    </modal-layout>
  </template>
  
  <script>
    module.exports = {
      mixins: [VueModal],
      data: function () {
        return {
          users: Array.from({ length: 10 }, () => ({ firstName: '', lastName: '' })),
          loading: false,
        };
      },
      methods: {
        addUser() {
          this.users.push({ firstName: '', lastName: '' });
        },
        validateUsers() {
          // Add validation logic here if needed
          return true;
        },
        signup() {
          this.loading = true;
          // Add logic to handle user creation for each user in the 'users' array
          // You can use a loop to go through the array and call the user creation logic
          // For example:
           for (const user of this.users) {
          //   // Call user creation logic here
           }
        },
      },
    };
  </script>
  