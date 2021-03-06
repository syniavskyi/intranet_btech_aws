<template>
    <div class="availability-tile" >
        <div class="availability-tile-header">
            <div class="ava-tile-header-title">
                <h2>{{ $t("label.projectsOverview") }}</h2>
                <div class="availability-tile-underscore"></div>
            </div>
             <button class="profile-edit-btn" v-if="!editMode" :disabled="permissionToEditProject" @click="edit">{{ $t("button.edit") }}</button>
             <label class="avail-error" v-if="permissionToEditProject">{{ $t("label.editProject") }}</label>
             <button class="profile-edit-btn-e" v-if="editMode" @click="cancel"><span class="prof-btn-txt">{{ $t("button.finishEdit") }}</span><span class="prof-btn-icon">&#10004;</span></button>
        </div>
        <p class="ava-content-header" v-if="noAvailEntries">{{ $t("message.noEntriesForParameters") }}</p>
        <div v-if="!noAvailEntries" class="availability-tile-content">
            <div class="ava-proj-table">
                <div class="ava-proj-thead">
                    <div class="ava-thproj-item">{{ $t("label.entryType") }}</div>
                    <div class="ava-thproj-item">{{ $t("label.projectName") }}</div>
                    <div class="ava-thproj-item">{{ $t("label.engag") }}</div>
                    <div class="ava-thproj-item">{{ $t("label.from") }}</div>
                    <div class="ava-thproj-item">{{ $t("label.to") }}</div>
                    <div class="ava-thproj-item">{{ $t("label.status") }}</div>
                    <div class="ava-thproj-item">{{ $t("label.remarks") }}</div>
                </div>
                <div class="ava-proj-tbody" v-for="(project, index) in filteredUserProjects" :key="index">
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.entryType") }}</div>
                        <p>{{formattedProjectType()}}</p>
                    </div>
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.projectName") }}</div>
                        <select disabled v-if="!editMode" class="selectProfile selectDisabled" v-model="project.ProjectId" >
                            <option v-for="proj in allProjects" :key="proj.ProjectId" :value="proj.ProjectId">{{proj.ProjectName}}</option>
                        </select>
                        <select v-if="editMode && authType !== 'OWN'" class="selectProfile selectEdit" v-model="project.ProjectId" @change="checkFields(index)">
                            <option v-for="proj in allProjects" :key="proj.ProjectId" :value="proj.ProjectId">{{proj.ProjectName}}</option>
                        </select>
                    </div>
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.engag") }}</div>
                        <p v-if="!editMode">{{project.Engag}}</p>
                        <div v-if="editMode && authType !== 'OWN'">
                            <input required class="ava-input-range-perc" v-model="project.Engag" @input="validateNewEngag(index)" type="number" min="0" max="100"/><span class="ava-perc-span">%</span>
                            <span class="ava-div-bar"></span>
                        </div>
                    </div>
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.from") }}</div>
                        <p class="prof-date-label" v-if="!editMode"> {{ formatDate(project.StartDate) }} </p>
                        <v-date-picker v-if="editMode && authType !== 'OWN'" class="prof-input-date" popoverDirection="top" @input="validateDates(index)" is-expanded mode="single" v-model="project.StartDate">
                            <input value="project.StartDate" />
                        </v-date-picker>
                    </div>
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.to") }}</div>
                        <p class="prof-date-label" v-if="!editMode"> {{ formatDate(project.EndDate) }} </p>
                        <v-date-picker v-if="editMode && authType !== 'OWN'" class="prof-input-date" popoverDirection="top" @input="validateDates(index)" is-expanded mode="single" v-model="project.EndDate">
                            <input value="project.EndDate" />
                        </v-date-picker>
                    </div>
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.status") }}</div>
                        <select v-if="editMode && authType !== 'OWN'" class="selectProfile selectEdit" v-model="project.StatusId" @change="checkFields(index)">
                            <option v-for="status in availStatus" :key="status.Key" :value="status.Key">{{status.Value}}</option>
                        </select>
                         <select disabled v-if="!editMode" class="selectProfile selectDisabled" v-model="project.StatusId" >
                            <option v-for="status in availStatus" :key="status.Key" :value="status.Key">{{status.Value}}</option>
                        </select>
                    </div>
                    <div class="ava-tbproj-item">
                        <div class="ava-tbproj-ititle">{{ $t("label.remarks") }}</div>
                        <div class="ava-tbproj-itxt">
                            <textarea :disabled="!editMode" v-model="project.Description"  @input="checkFields(index)"></textarea>
                        </div>
                    </div>
                    <div class="ava-tbs-item eduButtonsProj">
                        <div class="ava-tbs-ititle">{{ $t("label.options") }}</div>
                         <button v-if="editMode && authType !== 'OWN'" @click="update(index)" :disabled="true">{{ $t("button.save") }}</button>
                         <button v-if="editMode && authType !== 'OWN'" @click="remove(project)">{{ $t("button.delete") }}</button>
                    </div>
                </div>
            </div>
        </div>
    </div>                
</template>
<script>
import {mapGetters, mapActions} from 'vuex';
import moment from 'moment';
let utils = require('../../../utils');
export default {
    props: ['selected-type', 'selected-status', 'auth-type'],
    data () {
        return {
            invalidDates: false,
            editMode: false,
            _beforeEditingCache: null     
        }
    },
    computed: Object.assign(
        mapGetters({
            userProjects: 'userProjectsList',
            availTypes: 'getAvailType',
            availStatus: 'getAvailStatus',
            allProjects: 'projectsList',
            permissionToEditProject: "getPermissionToEditProject"
        }), {
            // { ...mapGetters({
            // userProjects: 'userProjectsList',
            // availTypes: 'getAvailType',
            // availStatus: 'getAvailStatus',
            // allProjects: 'projectsList',
            // permissionToEditProject: "getPermissionToEditProject" }),
        noAvailEntries() {
            if (this.filteredUserProjects.length === 0) {
                return true
            } else {
                return false
            }
        },
        filteredUserProjects() {
            let aFilteredProjets = this.userProjects,
                sStatus = this.selectedStatus;            
           
           if(sStatus === null){
               return aFilteredProjets
           } else {
               aFilteredProjets = aFilteredProjets.filter(function(oItem){
                  return oItem.StatusId === sStatus;
              })
            return aFilteredProjets
            }
            
        }
    }),
    methods: Object.assign(
        mapActions([
            "updateUserProject", "removeUserProject"
        ]), {
        // { ...mapActions(["updateUserProject", "removeUserProject"]),
        edit() {
            this.editMode = true;
            this._beforeEditingCache = utils.createClone(this.userProjects);
        },
        remove(project) {
            // this._beforeEditingCache.splice(index, 1);
            this.removeUserProject(project);
            this._beforeEditingCache = utils.createClone(this.userProjects);
        },
        update(index) {
            const dataToChange = this._beforeEditingCache[index],
                newData = utils.createClone(this.userProjects[index]);
                newData.Action ='U';
            
                newData.StartDateToChange = dataToChange.StartDate;
                newData.EndDateToChange = dataToChange.EndDate;
                newData.ProjectIdToChange = dataToChange.ProjectId;

            this.updateUserProject(newData);
            this._beforeEditingCache = utils.createClone(this.userProjects);
            document.getElementsByClassName("eduButtonsProj")[index].children[1].disabled = true;
        },
        cancel() {
            this.$store.commit("SET_USER_AVAIL_PROJECTS", this._beforeEditingCache);
            this.editMode = false;
        },
        validateDates(index) {
           const startDate = this.userProjects[index].StartDate,
                endDate = this.userProjects[index].EndDate

            if (endDate && startDate) {
                const formatStartDate = moment(startDate).format("YYYY-MM-DD"),
                formatEndDate = moment(endDate).format("YYYY-MM-DD");

                this.invalidDates = formatStartDate  > formatEndDate ? true : false;
            }
            this.checkFields(index);
        },
        formatDate(date) {
            return date !== null && date !== undefined
            ? moment(date).format("DD.MM.YYYY")
            : "-";
        },
        validateNewEngag(index){
            let project = this.userProjects[index]
            if (project.Engag < 0) project.Engag = null;
            if (project.Engag > 100) project.Engag = 100;
            this.checkFields(index);
        },
        formattedProjectType() {
            for(let i = 0; i < this.availTypes.length; i++) {
                if (this.availTypes[i].Key === "PR") {
                    return this.availTypes[i].Value
                }
            }
        },
        checkFields(index) {
            let bEnd, bStart, bType, bStatus, bEngag, bChanged, bDesc,
            beforeEdit = this._beforeEditingCache[index];
// check if data was changed
             bEnd = beforeEdit.EndDate.getTime() !== this.userProjects[index].EndDate.getTime(),
             bStart = beforeEdit.StartDate.getTime() !== this.userProjects[index].StartDate.getTime(),
             bType = beforeEdit.ProjectId !== this.userProjects[index].ProjectId,
             bStatus = beforeEdit.StatusId !== this.userProjects[index].StatusId,
             bEngag = beforeEdit.Engag !== this.userProjects[index].Engag,
             bDesc = beforeEdit.Description !== this.userProjects[index].Description;

// if data was changed set boolean variable to true
        bChanged = bEnd || bStart || bType || bStatus || bEngag || bDesc ? true : false;
        
// check if data are not empty and was changed and set button to disabled or not
            if(this.filteredUserProjects.length > 0) {
               if( bChanged &&
                 this.filteredUserProjects[index].ProjectId &&
                 this.filteredUserProjects[index].StartDate &&
                 this.filteredUserProjects[index].EndDate &&
                 this.filteredUserProjects[index].Engag) {
                     document.getElementsByClassName("eduButtonsProj")[index].children[1].disabled = false;
               }    else {
                    document.getElementsByClassName("eduButtonsProj")[index].children[1].disabled = true;
               }
            }    
        },

    })
}
</script>


