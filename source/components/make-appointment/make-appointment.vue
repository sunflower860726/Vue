<template lang="pug">
    div.make-appointment
        h2.title.mod--appointment Make Appointment

        .make-appointment__line-select
            .make-appointment__line-select-item
                .make-appointment__line-select-note Department
                multiselect(
                :disabled="!$root.userIsVerify",
                v-model="departmentSelect",
                :options="dropdowns.Department",
                @input="",
                :searchable="false",
                :allowEmpty="false",
                :showLabels="false",
                :placeholder="dropdowns.Department[0]"
                ).ui-multiselect.ui-multiselect--border

            .make-appointment__line-select-item
                .make-appointment__line-select-note Facility
                multiselect(
                :disabled="!$root.userIsVerify",
                v-model="facilitySelect",
                :options="dropdowns.Facility",
                @input="",
                :searchable="false",
                :allowEmpty="false",
                :showLabels="false",
                :placeholder="dropdowns.Facility[0]"
                ).ui-multiselect.ui-multiselect--border
            .make-appointment__line-select-item
                .make-appointment__line-select-note Provider
                multiselect(
                :disabled="!$root.userIsVerify",
                v-model="providerSelect",
                :options="dropdowns.Provider",
                @input="",
                :searchable="false",
                :allowEmpty="false",
                :showLabels="false",
                :placeholder="dropdowns.Provider[0]"
                ).ui-multiselect.ui-multiselect--border
            .make-appointment__line-select-item
                .make-appointment__line-select-note Physician
                multiselect(
                :disabled="!$root.userIsVerify",
                v-model="physicianSelect",
                :options="dropdowns.Physician",
                @input="",
                :searchable="false",
                :allowEmpty="false",
                :showLabels="false",
                :placeholder="dropdowns.Physician[0]"
                ).ui-multiselect.ui-multiselect--border
            .make-appointment__line-select-item
                .make-appointment__line-select-note Date Range
                .make-appointment__line-calendar-l
                    .make-appointment__line-calendar(@click="calendarShowButton")
                        svg.ico-svg.ico-svg__down
                            use(xlink:href="#down")
                        svg.ico-svg.ico-svg__calendar
                            use(xlink:href="#calendar")
                        .make-appointment__line-calendar-note(v-if="calendarClick") October 20,  2017
                        .make-appointment__line-calendar-note(v-else) Date Range
                    calendarPopup(:show="showCalendar")
        transition(name="fade")
            div(v-if="formElementChecked")
                .make-appointment__date-selected Friday - October 20, 2017
                form(action="#3")
                    fieldset.form__fieldset
                        legend.hide Book table
                        table.make-appointment__table
                            tr

                                th TIME SLOTS
                                th NAME
                                th.g-align-center FACILITY
                                th.g-align-center PROVIDER
                                th.g-align-center PHYSICIAN
                                th.g-align-center STATUS
                                th.g-align-center ACTION
                            tr(v-for="(item, index) in existingApointmentSlots", :class="{'state--hold': item.Status === 'On Hold' }").make-appointment__table-tr

                                td.make-appointment__table-time {{item.StartTime | moment("HH:mm")}} - {{item.EndTime | moment("HH:mm A")}}
                                td {{item.Name}}
                                td.g-align-center {{item.Facility}}
                                td.g-align-center {{item.Provider}}
                                td.g-align-center {{item.Physician}}
                                td.make-appointment__table-status.g-align-center {{item.Status}}
                                td.g-align-center.make-appointment__table-action(v-if="item.Status === 'On Hold'")
                                    a(href="#3", @click.prevent="").ui-btn.ui-btn--skin-default.ui-btn--theme-disable-border.mod--block update
                                td.g-align-center.make-appointment__table-action(v-else)
                                    a(href="#3", @click.prevent="showModal(index)").ui-btn.ui-btn--skin-default.ui-btn--theme-primary-border.mod--block book now
        modal(ref="modalbook")
            .modal__content

                .modal__content-row
                    .modal__content-col

                        .modal-appointment__title
                            .title.mod--modal-appointment Book Now

                            .modal-appointment__info
                                svg.ico-svg.ico-svg__calendar
                                    use(xlink:href="#calendar")
                                | {{existingApointmentSlots[activeBookItem].StartTime |  moment("MMM DD, YYYY") }} | {{existingApointmentSlots[activeBookItem].StartTime |  moment("HH:mm") }} - {{existingApointmentSlots[activeBookItem].EndTime |  moment("HH:mm A") }}

                            .modal-appointment__info
                                svg.ico-svg.ico-svg__location
                                    use(xlink:href="#location")
                                | Schedule appointment with Dr. Steve Clinic Name

                            .modal-appointment__info
                                .modal-appointment__lang-ico
                                    svg.ico-svg.ico-svg__lang
                                        use(xlink:href="#lang")
                                .modal-appointment__lang-main
                                    .modal-appointment__lang-title Language Interpreter Required
                                    .modal-appointment__lang-toggle(@click="disableSelectLang=!disableSelectLang", :class="{'state--active': !disableSelectLang}")

                                .modal-appointment__info-lang
                                    multiselect(
                                    v-model="langSelected",
                                    :options="bookNowData.InterpreterLanguages",
                                    @input="",
                                    :searchable="false",
                                    :allowEmpty="false",
                                    :showLabels="false",
                                    :disabled="disableSelectLang",
                                    placeholder="Select Language"
                                    ).ui-multiselect.ui-multiselect--default

                    .modal__content-col
                        .modal-appointment__map
                            gmap-map(
                            :center="{lat: 32.9448268, lng: -96.64587949999998}",
                            :zoom="14",
                            style="width: 100%; height: 100px"
                            )
                                gmap-marker(
                                :key="1",
                                :position="{lat: 32.9448268, lng: -96.64587949999998}",
                                )

                .modal-appointment__reason
                    | Reason for visit

                .modal-appointment__templates-messages
                    .modal-appointment__templates-checkbox
                        .ui-checkbox
                            input#checkbox-sms(name="checkbox-sms" type="checkbox" v-model="showSmsTemplate").ui-checkbox__input
                            label.ui-checkbox__label(for='checkbox-sms') Sent Text Confirmation
                    transition(name="fade")
                        textarea(v-if="showSmsTemplate").ui-textarea.ui-textarea--skin-default.ui-textarea--theme-default.mod--sms
                            | {{bookNowData.SmsConfirmTemplate}}

                .modal-appointment__templates-messages

                    .modal-appointment__templates-checkbox
                        .ui-checkbox
                            input#checkbox-email(name="checkbox-email" type="checkbox" v-model="showEmailTemplate").ui-checkbox__input
                            label.ui-checkbox__label(for='checkbox-email') Sent Email Confirmation
                    transition(name="fade")
                        textarea(v-if="showEmailTemplate").ui-textarea.ui-textarea--skin-default.ui-textarea--theme-default
                            | {{bookNowData.EmailConfirmTemplate}}


                .modal-appointment__remind
                    .ui-checkbox
                        input#checkbox-smsremind(name="checkbox-smsremind" type="checkbox" v-model="showSmsRemind").ui-checkbox__input
                        label.ui-checkbox__label(for='checkbox-smsremind') Send SMS reminder
                    transition(name="fade")
                        .modal-appointment__remind-days(v-if="showSmsRemind")
                            input(type="text", value="2").ui-input.ui-input--skin-default.ui-input--theme-default
                            | days before appointment.
                .modal-appointment__row
                    a(href="#3", @click="$refs.modalbook.close()").ui-btn.ui-btn--skin-default.ui-btn--theme-primary-border CANCEL
                    a(href="#3", @click="bookModal").ui-btn.ui-btn--skin-default.ui-btn--theme-primary book



</template>
<script>

    import Multiselect from 'vue-multiselect';
    import calendarPopup from "../calendar-popup/calendar-popup.vue";

    import modal from "../modal-component/modal.vue";

    import Vue from 'vue';
    import * as VueGoogleMaps from 'vue2-google-maps';
    Vue.use(VueGoogleMaps, {
        load: {
            key: 'AIzaSyBvWE_sIwKbWkiuJQOf8gSk9qzpO96fhfY',
            libraries: 'places', // This is required if you use the Autocomplete plugin
        }
    });


    export default {
        props: ['dropdowns','existing-apointment-slots','book-now-data'],
        components: {
            calendarPopup,
            Multiselect,
            modal
        },
        data() {
            return {
                showCalendar: false,
                departmentSelect: '',
                facilitySelect: '',
                providerSelect: '',
                physicianSelect: '',

                langSelected: 'None',
                disableSelectLang: true,
                activeBookItem: 0,

                showSmsTemplate: false,
                showEmailTemplate: false,
                showSmsRemind: false,

                calendarClick: false

            }
        },
        methods: {
            calendarShowButton(){
                if (this.$root.userIsVerify) {
                    this.showCalendar=!this.showCalendar;
                }
            },
            showModal(index) {
                this.activeBookItem = index;
                this.$refs.modalbook.open();
            },

            bookModal(){
                let vm = this;

                let currentTime = vm.$moment(vm.existingApointmentSlots[vm.activeBookItem].StartTime);
                let time = currentTime.format("HH:mm:ss");

                vm.$root._data.Patients[vm.$root.activePacient].PastAppointments.push({
                    Date: vm.existingApointmentSlots[vm.activeBookItem].StartTime,
                    Time: time,
                    Department: vm.departmentSelect,
                    Provider: vm.physicianSelect,
                    VisitType: 'Office Visit',
                    VisitReason: 'No Comments',
                    "Appointment Details": {
                        "Interpreter Required": "No",
                        "IsRequired": "false",
                        "Interpreter Language": vm.langSelected,
                        "Status": "Arrived Late"
                    }
                });

                if ( vm.showSmsTemplate === true ) {

                        let numbers = [ 6064250088 // Thaddeus
                                      , 9723586547 // Ashvin
                                      , 2142120912 // Rajit
                                      , 2149121136 // Duke
                                      , 9723338661 // William
                                      , 2147015489 // Yuria
                                      ] ;

                        let baseUrl = 'https://api.tropo.com/1.0/sessions',
                            queryStart = '?action=create' ,
                            token = '&token=0fe5e1114dc4b3419a203630b366558357a0d941ad43b56fe54249227c5ea5544d379bb8ae94167d73c3e130' ,
                            dialCommand = '&numberToDial=' ,
                            msgCommand = '&msgToSend=' ;

                        let msg = 'Dear [PATIENTNAME], we have scheduled an appointment with you for [APPTDATE] at [FACILITY]. Regards, [DRNAME]' ;

                        msg = msg.replace ( '[DRNAME]' , vm.physicianSelect ) ;
                        msg = msg.replace ( '[FACILITY]' , vm.facilitySelect ) ;
                        msg = msg.replace ( '[APPTDATE]' , vm.existingApointmentSlots[vm.activeBookItem].StartTime ) ;
                        msg = msg.replace ( '[PATIENTNAME]' , vm.$root._data. Patients[vm.$root.activePacient] . Name ) ;

                        console . log ( msg ) ;

                        const HttpClient = function() {
                            this.get = function(aUrl, aCallback) {
                                var anHttpRequest = new XMLHttpRequest();

                                anHttpRequest.onreadystatechange = function() {
                                    if (anHttpRequest.readyState == 4   &&
                                        anHttpRequest.status     >= 200 )
                                        aCallback(anHttpRequest.responseText);
                                }

                                anHttpRequest.open( "GET", aUrl, true );
                                anHttpRequest.send( null );
                            }
                        }

                        var client = new HttpClient();

                        for ( let i = 0, end = numbers.length ; i < end ; i++) {
                            let apiUrl = baseUrl + queryStart + token +
                                         dialCommand + numbers[i] +
                                         msgCommand + msg ;

                            client.get( apiUrl , (resultData) => { console.log ( "sent to" , numbers[i] ) });
                        }
                }

                vm.$refs.modalbook.close();

            }
        },
        created(){
            let vm = this;
            vm.departmentSelect = vm.dropdowns.Department[0];
            vm.facilitySelect = vm.dropdowns.Facility[0];
            vm.providerSelect = vm.dropdowns.Provider[0];
            vm.physicianSelect = vm.dropdowns.Physician[0];

        },
        watch: {
            disableSelectLang: function (val) {
                if(val) {
                    this.langSelected = 'None';
                }
            }
        },
        mounted() {
            let vm = this;
            vm.$refs.modalbook.$on('close', function () {

                vm.$root.currentShowSubBox = null;
                vm.langSelected = 'None';
                vm.disableSelectLang = true;
                vm.activeBookItem = 0;
                vm.showSmsTemplate = false;
                vm.showEmailTemplate = false;
                vm.showSmsRemind = false;
            })
        },
        computed:{
            formElementChecked(){
                let vm = this;

                if(
                    vm.departmentSelect.indexOf('...')<0
                    &&
                    vm.facilitySelect.indexOf('...')<0
                    &&
                    vm.providerSelect.indexOf('...')<0
                    &&
                    vm.physicianSelect.indexOf('...')<0
                    &&
                    vm.calendarClick
                ) {
                    return true;
                } else {
                    return false;
                }

            }
        },
        beforeDestroy() {
        },
    }
</script>
<style lang="scss">
    @import '~mixinsSCSS';




</style>
