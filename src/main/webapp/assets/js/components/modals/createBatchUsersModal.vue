<template>
    <modal-layout>
      <template v-slot:title>Création d’un nouveau groupe et des identifiants de ses utilisateurs 
    </template>
    <div class="row" style="margin-bottom: 5px;">
      <div class="col s1" style="display: inline-flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
        <toggle-action-button
                style="font-size: 12px; color: #27ae60; margin-right: 3px; text-align: center;"
                v-model="plainText"
                :off-label="'Form'"
                :on-label="'PlainText'"
        >
        </toggle-action-button>
      </div>
      
    <div class="col s6" style="display: inline-flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
      <label style="font-size: 12px; color: #3498db; margin-right: 3px; text-align: left;">Patron username</label>
      <select 
        class="browser-default" 
        v-model="usernameOption" 
        style="border: 1px solid #3498db; padding: 4px; border-radius: 2px; font-size: 10px;">
          <option value="full">nom.prénom</option>
          <option value="initials">n.prénom</option>
          <option value="reverse">p.nom</option>
          <option value="dash">prénom-nom</option>
          <option value="initialDash">p-nom</option>
          <option value="under">prénom_nom</option>
          <option value="initialUnder">p_nom</option>
      </select>
  </div>
  <div class="col s6" style="display: inline-flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
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

<div>
<text-input type="text"
      v-model="team.name"
      v-bind:label="'Nom du groupe à créer'"
      v-bind:validate="!existsTeams[team.name]"
      v-bind:error-helper="existsTeams[team.name] ? 'helper.team_exists' : ''"
      v-bind:placeholder="$t('helper.title_placeholder')">
</text-input>
</div>     
<div v-if="plainText" class="row" style="border-bottom: 1px solid #ccc;">
        <label style="font-size: 12px; color: #27ae60; margin-right: 3px; text-align: left;">Collez en bloc les noms et prénoms des utilisateurs en séparant les utilisateurs par le (un des) <span style="color: #db9e9e;">séparateur(s) </span> choisi(s) plus bas.</label>
        <textarea 
          v-model="plainTextField" 
          @input="parsePlainText" 
          rows="30" 
          style="border: 1px solid #27ae60; padding: 4px; border-radius: 2px; font-size: 12px;">
      </textarea>
</div>
<!-- Ajout des checkbox pour le séparateur -->
 <div v-if="plainText" class="row" style="display: flex; align-items: center; border-bottom: 1px solid #db9e9e; padding-bottom: 3px; margin-bottom: 5px;">
  <label style="font-size: 12px; color: #db9e9e; margin-right: 3px; text-align: left;">Choisissez le séparateur entre le nom et le prénom.</label>
 </div>
<div v-if="plainText" class="row" style="display: flex; align-items: center; border-bottom: 1px solid #db9e9e; padding-bottom: 3px; margin-bottom: 5px;">
        <label style="font-size: 12px; color: #db9e9e; margin-right: 8px;"></label>
        <label style="font-size: 11px; margin-right: 5px;">
          <input type="checkbox" v-model="separators.space" :disabled="separators.all" />
          <span>espace                </span>
        </label>
        <label style="font-size: 11px; margin-right: 5px;">
          <input type="checkbox" v-model="separators.semicolon" :disabled="separators.all" />
          <span>Point-virgule(;)                </span>
        </label>
        <label style="font-size: 11px; margin-right: 5px;">
          <input type="checkbox" v-model="separators.colon" :disabled="separators.all" />
          <span>deux-points(:)                </span>
        </label>
        <label style="font-size: 11px;">
          <input type="checkbox" v-model="separators.all" />
          <span>tous</span>
        </label>
      </div>       
      <div style="height: 10px;"></div>
      <div  class="row" style="display: flex; align-items: center; border-bottom: 1px solid #ccc; padding-bottom: 3px; margin-bottom: 5px;">
        <label style="font-size: 12px; color: #3498db; margin-right: 3px; text-align: left;">Formulaires utilisateurs :</label>
      </div>
      <div v-if="!loading">
  <div v-for="(user, index) in users" :key="index" class="row" style="align-items: center; margin-bottom: 5px; background-color: #ecf0f1; padding: 8px; border-radius: 3px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); font-size: 10px;">
    <!-- Toggle rôle -->
    <toggle-action-button
      v-model="user.isModo"
      :off-label="'U'"
      :on-label="'M'"
      style="font-size: 10px; color: #8e44ad; margin-right: 4px; text-align: center;"
    ></toggle-action-button>
    <text-input 
          type="text" 
          class="col s1"
          v-model="index" 
          :disabled="true" 
          style="font-size: 9px; margin-bottom: 0;">
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
          v-model="user.username"
          v-bind:error-helper="exists[user.username] ? $t('helper.username_exists') : (usernameMap.get(user.username) > 1 ? 'Username dupliqué' : '')"
          v-bind:label="$t('username')" 
          v-bind:validate="!exists[user.username] && usernameMap.get(user.username) <= 1"
          :value="generateUsername(user)"
          :disabled="true"
          style="font-size: 9px;margin-bottom: 0;">
    </text-input>
    <button 
          @click="removeUser(index)" 
          class="waves-effect waves-light btn-small red" 
          style="margin-right: 2px; margin-bottom: 2px;">-
    </button>
  </div>
  <button 
          @click="addUser"
          class="waves-effect waves-light btn-small"
          style="margin-bottom: 5px; background-color: #3498db; color: #FFF; border: none; border-radius: 2px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); font-size: 10px;">+
  </button>
</div>




<!--
      <div v-for="(username) in Array.from(existingUsernames)" :key="username" class="row">
          <text-input v-if=" username !=='.' " type="text" class="col s4" v-model="username" v-bind:label="'existants'" v-bind:validate="!testUserLocalExistance(username)" :disabled="true">
          </text-input>
        </div>-->
      <template v-slot:footer>
        <div style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
          <label style="font-size: 12px; margin-right: 3px; text-align: left;">
            <b>Form</b> : Création à l'aide des formulaires uniquement (bouton <b>+</b>/<b>-</b> pour l'ajout/suppression d'un formulaire)<br> <b>PlainText</b> : Création à l'aide d'un champs de texte <br>
            <b>U</b> : utilisateur / <b>M</b> : modérateur  
          </label>
          <div>
            <button @click="close()" v-bind:disabled="loading" class="waves-effect waves-green btn-flat">{{ $t('cancel') }}</button>
            <button @click="signup()" v-bind:disabled="loading || !validateUsers() || !isUsernameValid" class="waves-effect waves-green btn-flat">{{ $t('validate') }}</button>
          </div>
        </div>
      </template>
    </modal-layout>
  </template>
  
  <script>
    module.exports = {
      mixins: [VueModal],
      data: function () {
        return {
          team: new Team( ),
          user : new User ( ),
          users: Array.from({ length: 0 }, () => ({ 
            firstName: '', 
            lastName: '',
            username: '',
            isModo: false // Ajout du champ rôle
          })),
          usernameOption: 'full',
          existsTeams: {}, // team.name: bool (existe en base)
          exists: {}, // username: bool (existe en base)
          existsLocal: {},
          usernameMap: new Map(), // username: count (doublons locaux)
          checkExistsTimeout: -1,
          loading: false,
          userCount: 0,
          plainText: true,
          plainTextField: '',
          separators: {
            space: false,
            semicolon: true,
            colon: true,
            all: false
          },
        };
      },
      created() {
        this.team.name = `Nouvelle classe de ${this.$root.user.username}`;
      },
      computed: {
        isUsernameValid() {
          return this.users.every(
            user =>
              user.username &&
              !this.exists[user.username] &&
              this.usernameMap.get(user.username) === 1
          );
        }
      },
      watch:{
        users: {
          handler(newUsers) {
            // Recalcule la map des usernames locaux et vérifie l'existence en base
            const map = new Map();
            newUsers.forEach(user => {
              user.username = this.generateUsername(user);
              if (user.username) {
                map.set(user.username, (map.get(user.username) || 0) + 1);
                this.testUserExistance(user.username);
              }
            });
            this.usernameMap = map;
            this.updatePlainTextField();
          },
          deep: true
        },
        usernameOption: {
          handler() {
            this.users.forEach(user => {
              user.username = this.generateUsername(user);
              this.testUserExistance(user.username);
            });
            // Re-déclenche la map
            const map = new Map();
            this.users.forEach(user => {
              if (user.username) map.set(user.username, (map.get(user.username) || 0) + 1);
            });
            this.usernameMap = map;
          }
        },
        userCount: function(newCount, oldCount) {
          if (newCount !== oldCount) {
            const tmpArray = this.users;
            this.users = Array.from({ length: newCount }, (_, i) =>
              tmpArray[i] || { firstName: '', lastName: '', username: '', isModo: false }
            );
          }
        },
        team: {
          handler(newTeam) {
            if (newTeam && newTeam.name) {
              this.testTeamExistance(newTeam);
            }
          },
          deep: true
        },
        plainTextField: function(newPlainTextField) {
          if (this.plainText) {
            this.parsePlainText(newPlainTextField);
          }
        },
        separators: {
          handler() {
            // Recalcule le champ texte et les lignes utilisateurs selon les séparateurs cochés
            if (this.plainText) {
              // Si on est en mode texte, on recalcule les users à partir du texte
              this.parsePlainText();
            } else {
              // Sinon, on recalcule le champ texte à partir des users
              this.updatePlainTextField();
            }
          },
          deep: true
        },
        plainText(newVal, oldVal) {
          // Si besoin, tu peux forcer une action ici, mais ce n'est pas obligatoire pour l'affichage
          // Par exemple, tu peux forcer une mise à jour du champ texte ou des users
          // this.updatePlainTextField();
        },
      },
      methods: {
        testUserLocalExistance(username) {
          if (this.usernameMap.get(username) > 1) {    
                  this.existsLocal[username] = true;
              }
        },
        updatePlainTextField() {
          if (this.users.length > 0) {
            const lastUser = this.users[this.users.length - 1];
            if (lastUser && typeof lastUser === 'object' && lastUser.username) {
              // Détermine le séparateur à utiliser
              let sep = ' ';
              if (this.separators.all) {
                sep = ' '; // Par défaut espace si "tous" coché
              } else if (this.separators.semicolon && !this.separators.space && !this.separators.colon) {
                sep = ';';
              } else if (this.separators.colon && !this.separators.space && !this.separators.semicolon) {
                sep = ':';
              } else if (this.separators.semicolon && this.separators.space && !this.separators.colon) {
                sep = ';'; // Priorité à ; si plusieurs cochés
              } else if (this.separators.colon && this.separators.space && !this.separators.semicolon) {
                sep = ':'; // Priorité à : si plusieurs cochés
              } else if (this.separators.semicolon && this.separators.colon && !this.separators.space) {
                sep = ';'; // Priorité à ; si plusieurs cochés
              }
              // Si plusieurs séparateurs cochés, on prend le premier trouvé dans l'ordre espace, ;, :
              else if (this.separators.space) {
                sep = ' ';
              } else if (this.separators.semicolon) {
                sep = ';';
              } else if (this.separators.colon) {
                sep = ':';
              }
              this.plainTextField = this.users.map(user => `${user.firstName}${sep}${user.lastName}`).join('\n');
              this.assignUser(lastUser);
            }
          }
        },
        assignUser(user) {
          if (!user || typeof user !== 'object' || !this.user || typeof this.user !== 'object') return; // Vérifie que user et this.user sont valides
      
          this.user.firstName = user.firstName || '';
          this.user.lastName = user.lastName || '';
          const username = this.generateUsername(user);
          Vue.set(user, 'username', username);
          Vue.set(this.user, 'username', username);
        },
        testTeamExistance(team) {
          /*if (!team.name) return;
          // On ne clear pas le timeout globalement, sinon on annule les requêtes pour chaque champ
          // On stocke un timeout par username pour éviter d'écraser les autres
          if (!this._checkExistsTimeouts) this._checkExistsTimeouts = {};
          clearTimeout(this._checkExistsTimeouts[team.name]);
          this._checkExistsTimeouts[team.name] = setTimeout(() => {
            ArenService.Teams.exists({
              data: team.name,
              onSuccess: existsTeams => {
                // Utiliser Vue.set pour la réactivité
                this.$set(this.existsTeams, team.name, !!existsTeams);
              }
            });
          }, 300);*/
        },
        testUserExistance(username) {
          if (!username) return;
          // On ne clear pas le timeout globalement, sinon on annule les requêtes pour chaque champ
          // On stocke un timeout par username pour éviter d'écraser les autres
          if (!this._checkExistsTimeouts) this._checkExistsTimeouts = {};
          clearTimeout(this._checkExistsTimeouts[username]);
          this._checkExistsTimeouts[username] = setTimeout(() => {
            ArenService.Users.exists({
              data: username,
              onSuccess: exists => {
                // Utiliser Vue.set pour la réactivité
                this.$set(this.exists, username, !!exists);
              }
            });
          }, 300);
        },
        addUser() {
          const newUser = { 
            firstName: '', 
            lastName: '',
            username: '',
            isModo: false // Ajout du champ rôle
          };
          this.users.push(newUser);
          this.userCount = this.users.length;
          this.updatePlainTextField();
        },
        removeUser(index) {
          this.users.splice(index, 1);
          this.userCount = this.users.length;
        },
        generateUsername(user) {
          if (!user || !user.firstName || !user.lastName) return '';
          const firstName = user.firstName.trim().toLowerCase();
          const lastName = user.lastName.trim().toLowerCase();
          if (!firstName || !lastName) return '';
          switch (this.usernameOption) {
            case 'full': return `${lastName}.${firstName}`;
            case 'initials': return `${lastName.charAt(0)}.${firstName}`;
            case 'reverse': return `${firstName.charAt(0)}.${lastName}`;
            case 'dash': return `${lastName}-${firstName}`;
            case 'under': return `${lastName}_${firstName}`;
            case 'initialDash': return `${lastName.charAt(0)}-${firstName}`;
            case 'initialUnder': return `${lastName.charAt(0)}_${firstName}`;

            default: return '';
          }
        },
        afterOpen( ) {

                  this.loading = false;
                  this.user = false; 
                  let user = false;
                  Vue.nextTick(( ) => {
                      if (this.users.length>0) { 
                      this.user = new User( );
                      user =  this.users[this.users.length - 1]
                      this.user.firstName = user.firstName;
                      this.user.lastName = user.lastName; 
                      this.user.username= user.username;
                      this.user.authority = Authority.USER;
                      this.user.password = this.generateUsername(user);
                      this.user.email = 'aren@aren.fr';
                      }
                      
                  });
                
        },
        validateUsers() {
          return this.users.every(
            user =>
              user.firstName.trim() &&
              user.lastName.trim() &&
              user.username &&
              !this.exists[user.username] &&
              this.usernameMap.get(user.username) === 1
          ) && this.team.name && !this.existsTeams[this.team.name];
        },

        parsePlainText() {
          // Détermine les séparateurs à utiliser
          let regex;
          if (this.separators.all) {
            regex = /[\s;:]+/;
          } else {
            let parts = [];
            if (this.separators.space) parts.push('\\s');
            if (this.separators.semicolon) parts.push(';');
            if (this.separators.colon) parts.push(':');
            if (parts.length === 0) parts.push('\\s'); // fallback
            regex = new RegExp(`[${parts.join('')}]`, 'g');
          }
          const lines = this.plainTextField.split('\n');
          this.users = lines.map(line => {
            const [firstName, lastName] = line.split(regex);
            const user = { firstName: firstName || '', lastName: lastName || '', username: '', isModo: false };
            return user;
          });
          this.users.forEach((user) => {
            this.generateUsername(user);
          });
          this.userCount = this.users.length;
        },
        addUserToTeam(user) {
                ArenService.Teams.addUser({
                    id: this.team.id,
                    data: user
                });
            },
        createTeam(team) {
                return ArenService.Teams.createOrUpdate({
                            data: team,
                            onSuccess: (newTeam) => {this.team.id=newTeam.id}
                        });
        },
        signup() {
          if (!this.validateUsers()) {
            return;
          }
          this.loading = true; 
          this.createTeam(this.team);
          const createdUsers = [];
          const promises = this.users.map(user => {
            const newUser = new User();
            newUser.firstName = user.firstName;
            newUser.lastName = user.lastName;
            newUser.active = true;
            newUser.username = this.generateUsername(user);
            newUser.password = this.generateUsername(user);
            newUser.authority = user.isModo ? Authority.MODO : Authority.USER;
            newUser.email = 'aren@aren.fr';
        
            // On stocke les infos pour l'affichage
            createdUsers.push({
              username: newUser.username,
              password: newUser.password
            });
           
            return ArenService.Users.create({
              data: newUser,
              onSuccess: (newUser) => {this.addUserToTeam(newUser);}
            }); 
          });
          
          Promise.all(promises)
            .then(() => {
              this.close();
              // Génère le texte à afficher et à copier
              const userList = this.users.map((u, i) => {
                const base = `${createdUsers[i].username} / ${createdUsers[i].password}`;
                return u.isModo ? `${base} <------- Modérateur` : base;
              }).join('\n');
              const htmlList = `Les utilisateurs suivants ont été créés et ajoutés au nouveau groupe <b> ${this.team.name} </b> <br> (<small><b>Nom d'utilisateur</b> / <span style='font-family:monospace;'>Mot de passe</span></small>): <ul style="font-size:13px;padding-left:20px;">${
                this.users.map((u, i) => {
                  const base = `<b>  ${createdUsers[i].username}</b> / <span style='font-family:monospace;'>${createdUsers[i].password}</span>`;
                  return `<li>${base}${u.isModo ? " <span style='color:#e74c3c;font-weight:bold;'>&lt;----- Modérateur</span>" : ""}</li>`;
                }).join('')
              }</ul>`;
              // Ajoute un bouton copier
              const copyBtn = `<button style="margin:10px 0;padding:4px 10px;font-size:12px;cursor:pointer;" onclick="navigator.clipboard.writeText('${userList.replace(/'/g,"\\'").replace(/\n/g,'\\n')}')">📋 Copier</button>`;
              this.$confirm({
                title: this.$t('user_created'),
                message: `<div><br>${htmlList}${copyBtn}</div>`,
                isInfo: true,
                html: true
              });
            })
            .catch(error => {
              console.error(error);
            })
            .finally(() => {
              this.loading = false;
            });
        },
        
      },
    };
  </script>

<style scoped>
/* Conteneur principal */
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  background-color: #f9f9f9;
}



/* Alignement des champs */
.row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-bottom: 15px;
}

.col {
  flex: 1;
  min-width: 150px;
  margin: 5px;
}


</style>
