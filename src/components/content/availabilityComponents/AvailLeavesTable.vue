<template>
    <div class="availability-tile ava-tile-3">
        <div class="availability-tile-header">
            <div class="ava-tile-header-title">
                            <!-- <h2>{{ $t("header.addProject") }}</h2> -->
                <h2>{{ $t("label.availabilityOverview") }}</h2>
                <div class="availability-tile-underscore"></div>
            </div>
             <button class="profile-edit-btn" v-if="!editMode" :disabled="permissionToEditAvail" @click="edit">{{ $t("button.edit") }}</button>
             <button class="profile-edit-btn-e" v-if="editMode" @click="cancel"><span class="prof-btn-txt">{{ $t("button.finishEdit") }}</span><span class="prof-btn-icon">&#10004;</span></button>
        </div>
        <p class="ava-content-header" v-if="noAvailEntries">{{ $t("message.noEntriesForParameters") }}</p>
        <div class="availability-tile-content" v-if="!noAvailEntries">
            <div class="ava-table-s" v-for="(avail, index) in filteredUserAvail" :key="index">
                <div class="ava-thead-s">
                    <div class="ava-ths-item">{{ $t("label.entryType") }}</div>
                    <div class="ava-ths-item">{{ $t("label.from") }}</div>
                    <div class="ava-ths-item">{{ $t("label.to") }}</div>
                    <div class="ava-ths-item">{{ $t("label.status") }}</div>
                    <div class="ava-ths-item">{{ $t("label.options") }}</div>
                </div>
                <div class="ava-tbody-s">
                    <div class="ava-tbs-item">
                        <div class="ava-tbs-ititle">{{ $t("label.entryType") }}</div>
                         <select disabled v-if="!editMode || avail.StatusId === 'CO'" class="selectProfile selectDisabled" v-model="avail.TypeId">
                            <option v-for="type in availTypes" :key="type.Key" :value="type.Key">{{type.Value}}</option>
                        </select>
                        <select v-if="editMode && avail.StatusId !== 'CO'" class="selectProfile selectEdit" v-model="avail.TypeId" @change="checkFields(index, avail.EntryId)">
                            <option v-for="type in filteredAvailTypes" :key="type.Key" :value="type.Key">{{type.Value}}</option>
                        </select>
                    </div>
                    <div class="ava-tbs-item">
                        <div class="ava-tbs-ititle">{{ $t("label.from") }}</div>
                        <p class="prof-date-label" v-if="!editMode || avail.StatusId === 'CO'"> {{ formatDate(avail.DateStart) }} </p>
                        <v-date-picker v-if="editMode && avail.StatusId !== 'CO' && authType === '*' " class="prof-input-date" popoverDirection="top" @input="validateDates(index, avail.EntryId)" is-expanded mode="single" v-model="avail.DateStart">
                            <input value="avail.DateStart"/>
                        </v-date-picker>
                        <v-date-picker :min-date="new Date()" v-if="editMode && avail.StatusId !== 'CO' && authType !=='*'" class="prof-input-date" popoverDirection="top" @input="validateDates(index, avail.EntryId)" is-expanded mode="single" v-model="avail.DateStart">
                            <input value="avail.DateStart"/>
                        </v-date-picker>
                    </div>
                    <div class="ava-tbs-item">
                        <div class="ava-tbs-ititle">{{ $t("label.to") }}</div>
                        <p class="prof-date-label" v-if="!editMode || avail.StatusId === 'CO'"> {{ formatDate(avail.DateEnd) }} </p>
                        <v-date-picker v-if="editMode && avail.StatusId !== 'CO'  && authType === '*'" class="prof-input-date" popoverDirection="top" @input="validateDates(index, avail.EntryId)" is-expanded mode="single" v-model="avail.DateEnd">
                            <input value="avail.DateEnd"/>
                        </v-date-picker>
                        <v-date-picker :min-date="avail.DateStart" v-if="editMode && avail.StatusId !== 'CO' && authType !=='*'" class="prof-input-date" popoverDirection="top" @input="validateDates(index, avail.EntryId)" is-expanded mode="single" v-model="avail.DateEnd">
                            <input value="avail.DateStart"/>
                        </v-date-picker>
                    </div>
                    <div class="ava-tbs-item">
                        <div class="ava-tbs-ititle">{{ $t("label.status") }}</div>
                         <select disabled class="selectProfile selectDisabled" v-model="avail.StatusId">
                            <option v-for="status in availStatus" :key="status.Key" :value="status.Key">{{status.Value}}</option>
                        </select>
                    </div>
                    <div class="ava-tbs-item confirmButtonAvail" v-if="!editMode && authAcc && newLeave.UserId !== loginAlias && filteredTeamUsers.find(o => o.UserAlias === newLeave.UserId) || authAcc ==='*'">
                         <button v-show="!editMode && authAcc && newLeave.UserId !== loginAlias && filteredTeamUsers.find(o => o.UserAlias === newLeave.UserId) || authAcc ==='*'" :disabled="permissionToEditAvail" @click="confirm(index, avail.EntryId, avail)">{{ $t("button.confirm") }}</button>
                         <button v-show="!editMode && authAcc && newLeave.UserId !== loginAlias && filteredTeamUsers.find(o => o.UserAlias === newLeave.UserId) || authAcc ==='*'" :disabled="permissionToEditAvail" @click="remove(index, avail)">{{ $t("button.reject") }}</button>
                    </div>
                    <div class="ava-tbs-item eduButtonsAvail" v-else>
                            <button v-if="editMode" :disabled="true" @click="save(index, avail)">{{ $t("button.save") }}</button>
                            <button v-if="editMode" @click="remove(index, avail)">{{ $t("button.delete") }}</button>
                    </div>
                </div>
            </div>
        </div>
    </div>                
</template>
<script>
import {mapGetters, mapActions} from 'vuex'
import moment from 'moment'
let utils = require('../../../utils')
export default {
    props: ['selected-type', 'selected-status', 'auth-type'],
    data () {
        return {
            invalidDates: false,
            editMode: false,
            _beforeEditingCache: null,
            setFilterAllowed: true,
            loginAlias: this.$store.getters.getLoginAlias || localStorage.getItem("id")
        }
    },
    computed: Object.assign(
        mapGetters({
            userAvail: 'getUserAvail',
            availTypes: 'getAvailType',
            availStatus: 'getAvailStatus',
            loginAlias: "getLoginAlias",
            newLeave: "getNewLeaveForUser",
            authAcc: 'getAvailAcceptAuth',
            filteredTeamUsers: 'getFilteredTeamUsers',
            permissionToEditAvail: "getPermissionToEditAvail"
        }), {
        // { ...mapGetters({
        //     userAvail: 'getUserAvail',
        //     availTypes: 'getAvailType',
        //     availStatus: 'getAvailStatus',
        //     loginAlias: "getLoginAlias",
        //     newLeave: "getNewLeaveForUser",
        //     authAcc: 'getAvailAcceptAuth',
        //     filteredTeamUsers: 'getFilteredTeamUsers',
        //     permissionToEditAvail: "getPermissionToEditAvail" }),
        filteredUserAvail() {
            let aFilteredAvail = this.userAvail,
                sType = this.selectedType,
                sStatus = this.selectedStatus,
                checkFilter, 
                fnFilter;
                
// checkFilter checks if data are during editing and does not allow to filter by type
                checkFilter = aFilteredAvail.find(o => o.Filter === true);
            
           if(sType === null && sStatus === null){
               return aFilteredAvail
           } else {
               if(sType && sStatus && checkFilter){ 
                    fnFilter = function(oItem){
                        return oItem.TypeId === sType && oItem.StatusId === sStatus || oItem.Filter === true;
                    }
                }
                else if(sType && sStatus){ 
                    fnFilter = function(oItem){
                        return oItem.TypeId === sType && oItem.StatusId === sStatus;
                    }
                }
                else if(sType && checkFilter) {
                    fnFilter = function(oItem){
                        return oItem.TypeId === sType || oItem.Filter === true;
                    }
                }
                else if (sType){
                    fnFilter = function(oItem){
                        return oItem.TypeId === sType;
                    }
                }else if(sStatus){
                    fnFilter = function(oItem){
                        return oItem.StatusId === sStatus;
                    }
               }
                aFilteredAvail = aFilteredAvail.filter(fnFilter);
                
                return aFilteredAvail
           }          
        },
        noAvailEntries() {
            if (this.filteredUserAvail.length === 0) {
                return true
            } else {
                return false
            }
        },
        filteredAvailTypes() {
            let aAvailTypes = this.availTypes,
                aFilteredTypes = utils.createClone(this.availTypes)

            for(let i = 0; i < aAvailTypes.length; i++){
                if (aAvailTypes[i].Key === 'PR') {
                    aFilteredTypes.splice(i, 1)
                    return aFilteredTypes
                }
            }
        }
    }),
    watch: {
         userAvail() {
             this.editMode = false;
         }
    },
    methods: Object.assign(
        mapActions([
            "removeUserAvail", "updateUserAvail"
        ]), {
        // { ...mapActions(["removeUserAvail", "updateUserAvail"]),
         edit() {
            this.editMode = true;
            this._beforeEditingCache = utils.createClone(this.userAvail);
            // this.checkDisabled();
        },
        remove(index, data) {
            let avail = utils.createClone(data);
            this.removeUserAvail(avail);
            this.userAvail.splice(index, 1);
            this._beforeEditingCache = this.userAvail;
        },
        cancel() {
            this.$store.commit("SET_USER_AVAIL", this._beforeEditingCache);
            this.editMode = false;
        },
        validateDates(index, entryId) {
           const startDate = this.userAvail[index].DateStart,
                endDate = this.userAvail[index].DateEnd

            if (endDate && startDate) {
                const formatStartDate = moment(startDate).format("YYYY-MM-DD"),
                formatEndDate = moment(endDate).format("YYYY-MM-DD");

                this.invalidDates = formatStartDate  > formatEndDate ? true : false;
            }
            this.checkFields(index, entryId);
        },
        formatDate(date) {
            return date !== null && date !== undefined
            ? moment(date).format("DD.MM.YYYY")
            : "-";
        },
        checkFields(index, entryId) {
         let bEnd, bStart, bType, bStatus, bChanged,
         userEntryId = this.userAvail[entryId],
         beforeEdit = this._beforeEditingCache[entryId]

// check if data was changed
             bEnd = beforeEdit.DateEnd.getTime() !== userEntryId.DateEnd.getTime(),
             bStart = beforeEdit.DateStart.getTime() !== userEntryId.DateStart.getTime(),
             bType = beforeEdit.TypeId !== userEntryId.TypeId,
             bStatus = beforeEdit.StatusId !== userEntryId.StatusId;

// if data was changed set boolean variable to true
       bChanged = bEnd || bStart || bType || bStatus ? true : false

// check if data are not empty and was changed and set button to disabled or not    
// do not allow to filter by type when data are during changing 
         if( bChanged &&
             userEntryId.TypeName &&
             userEntryId.DateStart &&
             userEntryId.DateEnd &&
             userEntryId.StatusId) {
                document.getElementsByClassName("eduButtonsAvail")[index].children[0].disabled = false;
            }    else {
                document.getElementsByClassName("eduButtonsAvail")[index].children[0].disabled = true;
            }

            userEntryId.Filter = true;
        },
        save(index, data) {
             this.userAvail[data.EntryId].Filter = false;
             document.getElementsByClassName("eduButtonsAvail")[index].children[0].disabled = true;
             let avail = utils.createClone(data);
             avail.DateStartToChange = this._beforeEditingCache[index].DateStart;
             avail.DateEndToChange = this._beforeEditingCache[index].DateEnd;
             this.updateUserAvail(avail);

        },
      // set button disabled  
        // checkDisabled() {
        //     for(let i = 0; i < this.filteredUserAvail.length; i++) {
        //         if (this.filteredUserAvail[i].StatusId === 'CO') {
        //             document.getElementsByClassName("eduButtonsAvail")[i].children[0].disabled = true;
        //         }   else {
        //            document.getElementsByClassName("eduButtonsAvail")[i].children[0].disabled = false;
        //         }
        //     }
        // },
        confirm(index, entryId, data) {
          this.userAvail[entryId].StatusId = 'CO';
          document.getElementsByClassName("confirmButtonAvail")[index].children[0].disabled = true;
          let avail = utils.createClone(data);
          this.updateUserAvail(avail);
        }
    })
}
</script>

