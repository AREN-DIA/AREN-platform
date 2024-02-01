<template>
    <modal-layout>
      <template v-slot:title>Batch user creation
    </template>
    <div class="row" style="margin-bottom: 5px;">
      <div class="col s4" style="display: inline-flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
        <toggle-action-button
                style="font-size: 12px; color: #27ae60; margin-right: 3px; text-align: center;"
                v-bind:active="plainText"
                v-on:toggle="togglePlainText"
                :off-label="'Plain text'"
                :on-label="'Form'"
                >
        </toggle-action-button>
      </div>
    <div class="col s4" style="display: inline-flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
      <label style="font-size: 12px; color: #3498db; margin-right: 3px; text-align: left;">Patron username</label>
      <select 
        class="browser-default" 
        v-model="usernameOption" 
        style="border: 1px solid #3498db; padding: 4px; border-radius: 2px; font-size: 10px;">
          <option value="full">nom.prénom</option>
          <option value="initials">n.prénom</option>
          <option value="reverse">p.nom</option>
      </select>
  </div>
  <div class="col s4" style="display: inline-flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
    <label style="font-size: 12px; color: #e74c3c; margin-right: 3px; text-align: left;">Nbr d'utilis.</label>
    <input
      type="text"
      v-model.number="userCount"
      placeholder="Nombre d'utilisateurs"
      class="small-input"
      style="border: 1px solid #e74c3c; padding: 4px; border-radius: 2px; font-size: 12px;"
      :disabled="plainText"
    />
  </div>
</div>
<div v-if="plainText" class="row" style="border-bottom: 1px solid #ccc;">
        <label style="font-size: 12px; color: #27ae60; margin-right: 3px; width: 70px; text-align: left;">Entrez les noms et prénoms des utilisateurs séparés par un saut de ligne.</label>
        <textarea 
          v-model="plainTextField" 
          @input="parsePlainText" 
          rows="10" 
          style="border: 1px solid #27ae60; padding: 4px; border-radius: 2px; font-size: 15px;">
      </textarea>
</div>

<div v-if="!loading">
  <div v-for="(user, index) in users" :key="index" class="row" style="align-items: center; margin-bottom: 5px; background-color: #ecf0f1; padding: 8px; border-radius: 3px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); font-size: 10px;">
    <button 
          @click="removeUser(index)" 
          class="waves-effect waves-light btn-small red" 
          style="margin-right: 2px; margin-bottom: 2px;">-
  </button>
    <text-input 
          type="text" 
          class="col s1" 
          v-model="index" 
          :disabled="true" 
          style="font-size: 9px;margin-bottom: 0;">
  </text-input>
  <text-input 
          type="text" 
          class="col s3" 
          v-model="user.firstName" 
          v-bind:label="$t('first_name')" 
          style="font-size: 9px;margin-bottom: 0;">
  </text-input>
  <text-input 
          type="text" 
          class="col s3" 
          v-model="user.lastName" 
          v-bind:label="$t('last_name')" 
          style="font-size: 9px;margin-bottom: 0;">
  </text-input>
  <text-input 
          type="text" 
          class="col s3"
          v-model="replacements[index]"
          v-bind:error-helper="existsAll.get(user) ? $t('helper.username_exists'):'Pseudos identiques'"
          v-bind:label="$t('username') + ' ' +{ isUsernameValid } " 
          v-bind:validate="isUsernameValid" 
          :disabled="!existsAll.get(user)" 
          style="font-size: 9px;margin-bottom: 0;">
    </text-input>
  </div>
  <button 
          @click="addUser"
          class="waves-effect waves-light btn-small"
          style="margin-bottom: 5px; background-color: #3498db; color: #FFF; border: none; border-radius: 2px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); font-size: 10px;">+
  </button>
</div>





      <div v-for="(username) in Array.from(existingUsernames)" :key="username" class="row">
          <text-input v-if=" username !=='.' " type="text" class="col s4" v-model="username" v-bind:label="'existants'" v-bind:validate="!testUserLocalExistance(username)" :disabled="true">
          </text-input>
        </div>
      <template v-slot:footer>
        <button @click="close()" v-bind:disabled="loading" class="waves-effect waves-green btn-flat">{{ $t('cancel') }}</button>
        <button @click="signup()" v-bind:disabled="loading || !validateUsers() || !isUsernameValid" class="waves-effect waves-green btn-flat">{{ $t('validate') }}</button>
      </template>
    </modal-layout>
  </template>
  
  <script>
    module.exports = {
      mixins: [VueModal],
      data: function () {
        return {
          user : new User ( ),
          users: Array.from({ length: 10 }, () => ({ 
            firstName: '', 
            lastName: '',
            username: '',
          })),
          usernameOption: 'full',
          exists: {username: false},
          existsAll: new Map(),
          replacements: {},
          checkExistsTimeout: -1,
          existingUsernames: new Map(),
          loading: false,
          userCount: 10,
          plainText: false,
          plainTextField: ''
        };
      },
      computed:{
       
          isUsernameValid() {
            // Customize this logic based on your validation requirements
            
              return !this.testUserLocalExistance(this.user.username) && !this.existsAll.get(this.user);
          },
      },
      watch: {
        users: {
          handler: 'updatePlainTextField',
          deep: true,
        },
        userCount: function (newCount, oldCount) {
          // Handle array size change
          if (newCount !== oldCount) {
            const tmpArray = this.users ;
            this.users = Array.from({ length: newCount }, () => ({
              firstName: '',
              lastName: '',
              username: '',
            }));
            minLgth = (tmpArray.length > this.users.length) ?  this.users.length : tmpArray.length
            for (let i = 0; i < minLgth; i++) {
               this.users[i] = (tmpArray[i]);
            }
            this.users.concat(tmpArray);
          }
        },
      },
      methods: {
        updatePlainTextField() {
          this.plainTextField = this.users.map(user => `${user.firstName} ${user.lastName}`).join('\n');
          this.assignUser(this.users[this.users.length - 1]);
          this.testUsersEx();
          console.log(this.existsAll);
          
        },
        assignUser(user){
          this.user.firstName = user.firstName;
          this.user.lastName = user.lastName; 
          this.user.username = this.replacements[this.users.length - 1];
          this.user.password = this.generateUsername(user);
        },
        async testUserExistance(field, user) {
  console.log("USER : " + user.username);
  clearTimeout(this.checkExistsTimeout);
  await new Promise(resolve => {
    this.checkExistsTimeout = setTimeout(() => {
      ArenService.Users.exists({
        data: user[field],
        onSuccess: (doExists) => {
          this.existsAll.set(user, doExists);
          resolve();
        }
      });
    }, 500);
  });
},
        async testUsersEx() {
  let field = 'username';
  if (typeof this.user !== 'undefined' && this.user !== false) {
    await this.testUserExistance(field, this.user);
    console.log(this.existsAll);
  }
},
        testUserLocalExistance(field) {
            return this.existingUsernames.has(field) && this.existingUsernames.get(field) > 1;
        },
        addUser() {
          this.users.push({ 
            firstName: '', 
            lastName: '',
            username: ''
          });
          this.userCount = this.users.length;
          this.updatePlainTextField();
        },
        removeUser(index) {
          this.users.splice(index, 1);
          this.userCount = this.users.length;
        },
        generateUsername(user) {
          const firstName = user.firstName.trim().toLowerCase();
          const lastName = user.lastName.trim().toLowerCase();
  
          switch (this.usernameOption) {
            case 'full':
              return `${lastName}.${firstName}`;
            case 'initials':
              return `${lastName.charAt(0)}.${firstName}`;
            case 'reverse':
              return `${firstName.charAt(0)}.${lastName}`;
            default:
              return ''; // Handle default case or return an error
          }
        },
        validateUsername(username) {
            this.testUserExistance(username);
            return !this.exists[username];
        },
        replaceUsernames() {
          for (const key in this.replacements) {
            if (Object.hasOwnProperty.call(this.users, key)) {
              this.users[key].username = this.replacements[key];
            }
          }
          return true;
        },
        afterOpen( ) {

                  this.loading = false;
                  this.user = false; 
                  let user = false;
                  Vue.nextTick(( ) => {
                      this.user = new User( );
                      user =  this.users[this.users.length - 1]
                      this.user.firstName = user.firstName;
                      this.user.lastName = user.lastName; 
                      this.user.username = this.replacements[this.users.length - 1];
                      this.user.authority = Authority.USER;
                      this.user.password = this.generateUsername(user);
                      this.user.email = 'aren@aren.fr';
                  });
                
        },
        validateUsers() {
          this.existingUsernames.clear();
          let myUserId = 0;
          let myUsername ='';
          for (const [index, user] of this.users.entries()) {
            user.username = this.generateUsername(user);
            myUserId = index;
            myUsername =user.username;
            let attempt = 1;
            //console.log(this.exists['username']);
            //console.log(this.user);
            while (user.username!= "." && (this.existingUsernames.has(myUsername))) {
              // If username is not valid, add a number and retry
              myUsername =  user.username + attempt;
              attempt++;
              this.user.username = myUsername;
              /*console.log(attempt);
              console.log(user.username);
              console.log(myUsername);
              console.log(this.existingUsernames.has(user.username));
              console.log(this.existingUsernames.has(myUsername) && user.username!= ".");*/
            }
            this.existingUsernames.set(myUsername, (this.existingUsernames.get(myUsername) || 0) + 1);
            this.replacements[index]=myUsername;
          } 
          //this.replaceUsernames(myUserId,myUsername);
          return this.existingUsernames.size == this.userCount; // You may add additional validation logic here if needed
        },
        togglePlainText() {    
          this.plainText = !this.plainText;
        },

        parsePlainText() {
          // Implement the parsing logic here to fill the 'users' array from the plain text field
          // Example parsing logic (customize as needed):
          const lines = this.plainTextField.split('\n');
          this.users = lines.map(line => {
            const [firstName, lastName] = line.split(' ');
            return { firstName, lastName, username: this.generateUsername({ firstName, lastName }) };
          });
          this.userCount = this.users.length;
        },
        signup() {
          this.loading = true;
          // Add logic to handle user creation for each user in the 'users' array
          // You can use a loop to go through the array and call the user creation logic
          // For example:
          // for (const user of this.users) {
          //   // Call user creation logic here
          // }
        },
      },
    };
  </script>
  