<template>
    <div class="availability-tile ava-tile-2">
        <div class="availability-tile-header">
            <div class="ava-tile-header-title">
                <i18n path="message.addEntryforUser" tag="h2">
                    <span place="user"> {{formattedUsername}} </span>
                 </i18n>
                <div class="availability-tile-underscore"></div>
            </div>
        <!-- <button class="ava-button ava-button-add" @click="showAddProjectDialog = true"> Dodaj projekt </button> -->
        </div>
        <div class="availability-tile-content">
            <div id="add-project-dialog">
                <div class="ava-add">
                    <p class="ava-content-header" v-if="newLeave.TypeId === null">{{ $t("message.selectTypeToAddEntry") }}</p>
                    <p class="ava-content-header" v-if="newLeave.TypeId !== null">{{ $t("label.entryType") }}: 
                        <span class="ava-tile-entry">&nbsp;{{formattedType}}</span>
                    </p>
                    <div class="ava-div-select-cool">
                        <v-date-picker v-if="authType === '*'" required class="ava-input-range-wide" popoverDirection="top" is-expanded mode="range" v-model="selectedDates" @change="checkFields">
                            <input class="ava-input-range-wide" value="selectedDates"/>
                        </v-date-picker>
                        <v-date-picker  v-if="authType !== '*'" :min-date="new Date()" required class="ava-input-range-wide" popoverDirection="top" is-expanded mode="range" v-model="selectedDates" @change="checkFields">
                            <input class="ava-input-range-wide" value="selectedDates"/>
                        </v-date-picker>
                        <label class="ava-input-label-cool">{{ $t("label.dates") }}</label>
                    </div>
                    <div class="ava-div-select-cool" v-if="authType === '*' || authType === 'TEAM' && selectedUser!==loginAlias && filteredTeamUsers.find(o => o.UserAlias === newLeave.UserId)">
                        <select required class="ava-select-cool" v-model="newLeave.StatusId" @change="checkFields">
                            <option v-for="status in availStatusList" :key="status.Key" :value="status.Key">{{ status.Value }}</option>
                        </select>
                        <label class="ava-select-label-cool">{{ $t("label.status") }}</label>
                    </div>
                </div>
                <div class="ava-add">
                    <div class="ava-div-input-cool" v-if="newLeave.TypeId == 'OT'">
                        <textarea class="ava-textarea" required maxlength="50" @change="checkFields"/>
                    <label class="ava-select-label-cool">{{ $t("label.remarks") }}</label>
                    </div>
                    <div class="ava-div-buttons">
                        <button class="ava-button" >{{ $t("button.cancel") }}</button>
                        <button :disabled="disableAddNew" class="ava-button ava-button-edit" @click="addNewLeave" >{{ $t("button.addNewEntry") }}</button>
                    </div>
                </div>
            </div>
        </div>
    </div>    
</template>
<script>
import { mapGetters, mapActions } from 'vuex';
let utils = require('../../../utils')
export default {
    props: ['selected-type', 'selected-user', 'auth-type'],
    data() {
        return {
            selectedDates: null,
            disableAddNew: true,
            loginAlias: this.$store.getters.getLoginAlias || localStorage.getItem("id")
        }
    },
    computed: Object.assign(
        mapGetters({
            newLeave: 'getNewLeaveForUser',
            availStatusList: 'getAvailStatus',
            usersList: 'usersList',
            availTypesList: 'getAvailType',
            filteredTeamUsers: 'getFilteredTeamUsers',
            permissionToEditAvail: "getPermissionToEditAvail"
        }), {
            // { ...mapGetters({
            // newLeave: 'getNewLeaveForUser',
            // availStatusList: 'getAvailStatus',
            // usersList: 'usersList',
            // availTypesList: 'getAvailType',
            // filteredTeamUsers: 'getFilteredTeamUsers',
            // permissionToEditAvail: "getPermissionToEditAvail" }),
        formattedUsername() {
            const userId = this.selectedUser
            for (let i = 0; i < this.usersList.length; i++){
                if (this.usersList[i].UserAlias === userId) {
                     return this.usersList[i].Fullname
                }
            }
        },
        formattedType() {
            const typeId = this.newLeave.TypeId
            for (let i = 0; i < this.availTypesList.length; i++){
                if (this.availTypesList[i].Key === typeId) {
                     return this.availTypesList[i].Value;
                }
            }
        }
    }),
    watch: {
        selectedDates(value){
            this.newLeave.DateStart = utils.formatDateForBackend(value.start);
            this.newLeave.DateEnd = utils.formatDateForBackend(value.end);
            this.checkFields();
        },
        selectedType(value){
            if (value){
                this.checkFields();
            } else {
                this.disableAddNew = true;
            }
        }
    },
    methods: Object.assign(
        mapActions({
            addNewLeave: 'addUserLeave'
        }),{
        // { ...mapActions({addNewLeave: 'addUserLeave'}),
        checkFields() {
            let obj = this.newLeave;
            if(this.permissionToEditAvail===false) {
            if(this.selectedUser === this.loginAlias && this.authType !== '*' && obj.DateStart && obj.DateEnd) {
                this.disableAddNew = false;
            } else {
                for (let key in obj) {
                if (!obj[key]){
                    this.disableAddNew = true;
                    return;
                } else {
                    this.disableAddNew = false;
                }
            }
            }
            }
        },
        
    })
};
</script>