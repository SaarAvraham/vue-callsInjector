<template>
    <div>
        <h1 class="marDown"> {{ msg }}</h1>

        <status-indicator v-if="connected" status="positive" pulse="true" style="pointer-events: none"/>
        <status-indicator v-else status="negative" style="pointer-events: none"/>
        <label v-if="connected" style="padding-left: 10px; padding-right: 1px; padding-bottom: 5px">Connected</label>
        <label v-else style="padding-left: 10px; padding-right: 1px; padding-bottom: 5px;"> Disconnected</label>

        <div>
            <b-form-group
                    label-cols-lg="3"
                    label-size="lg"
                    label-class="font-weight-bold pt-0"
                    class="mb-0"
            >
                <b-form-group
                        label-cols-sm="3"
                        label="Number of Calls:"
                        label-align-sm="right"
                        label-for="nested-street"
                >
                    <b-form-input id="nested-street" style="width: 332px; text-align: left"
                                  v-model="callsToInject"
                                  required
                    ></b-form-input>
                </b-form-group>

                <b-form-group
                        label-cols-sm="3"
                        label="Select Date Range:"
                        label-align-sm="right"
                        label-for="nested-country"
                >
                    <b-row
                            style="padding-left: 15px">
                        <date-range-picker
                                class="b-form-tag"
                                style="width: 332px; text-align: left"
                                ref="picker"
                                :opens="opens"
                                :locale-data="{ firstDay: 1, format: 'yyyy-mm-dd' }"
                                :minDate="minDate" :maxDate="maxDate"
                                :singleDatePicker="singleDatePicker"
                                :timePicker="timePicker"
                                :timePicker24Hour="timePicker24Hour"
                                :showWeekNumbers="showWeekNumbers"
                                :showDropdowns="showDropdowns"
                                :autoApply="autoApply"
                                v-model="startRequest.dateRange"
                                :ranges="show_ranges ? undefined : false"
                                :linkedCalendars="linkedCalendars"
                                :always-show-calendars="false"
                                :alwaysShowCalendars="alwaysShowCalendars"
                                :append-to-body="appendToBody"
                                :closeOnEsc="closeOnEsc"
                                :disabled="isRunning"
                        >
                            <template #input="picker" style="min-width: 350px">
                                {{ picker.startDate | date }} - {{ picker.endDate | date }}
                            </template>
                        </date-range-picker>
                    </b-row>
                    <!--                    </b-form-input>-->
                </b-form-group>
                    <b-form-group
                            label-cols-sm="3"
                            label="Calls Per Second:"
                            label-align-sm="right"
                            label-for="nested-street32"
                    >
                    <b-row style="padding-left: 15px">
                        <b-form-input 
                        style="width: 332px; alignment: left; margin-top: 8px; margin-left: 0px; margin-right: 8px; margin-bottom: 0px;" 
                    id="range-1" v-model="cpsUserValue" type="range" min="100" max="5000" step="100" ></b-form-input> 
                                            <div class="mt-2" style="alignment: left; margin-top: 0px; margin-left: 0px; margin-right: auto"> {{ cpsUserView }}</div>
           
                    </b-row>
                    </b-form-group>

                    <b-form-group
                            label-cols-sm="3"
                            label="Business Data Definition (Optional):"
                            label-align-sm="right"
                            label-for="nested-street32"
                    >
                    <b-row style="padding-left: 15px">
        <b-form-file :placeholder="dsadsds" v-model="file" style="margin-left: 0px; margin-bottom: 5px; padding-top: 5px" plain ></b-form-file>

           
                    </b-row>
                    </b-form-group>

                                        <b-form-group
                            label-cols-sm="3"
                            label=""
                            label-align-sm="right"
                            label-for="nested-street32"
                    >
                    <b-row style="padding-left: 15px">
        <b-form-checkbox v-model="isDLOnly" style="margin-bottom: 25px">Data Lake Only</b-form-checkbox>

           
                    </b-row>
                    </b-form-group>
            </b-form-group>

            
            
        </div>


        <!-- <b-form-checkbox v-model="isDLOnly" style="margin-left: 25px; margin-right: 80px;margin-bottom: 25px">Data Lake Only</b-form-checkbox> -->

<!-- :placeholder="Optional - Upload a Busniness Data Definition File (JSON)" -->
<!--         <b-form-checkbox v-model="isTurboMode" style="margin-bottom: 5px">Turbo Mode</b-form-checkbox> -->

        <b-button variant="primary" style="margin: 8px 8px 8px 8px" @click="sendInjectRequest()"
                  :disabled="isRunning || !connected">Start
        </b-button>
        <b-button variant="danger" style="margin: 8px 8px 8px 8px" @click="sendStopRequest()"
                  :disabled="!isRunning || !connected">Stop
        </b-button>
        
        <h5 v-show="isRunning || injectionProgress===100" style="margin-top: 80px; text-decoration: underline;">Stats</h5>
        <div style="margin-left: 30px" v-show="isRunning || injectionProgress===100">
                <b-form-group label-cols-sm="6" label="Calls Injected:" label-align-sm="right" label-for="nested-street11" label-class="pt-0" class="mb-0" >
                    <b-form-row id="nested-street11" style="width: 332px; text-align: left">{{callsInjected}}</b-form-row>
                </b-form-group>

                <b-form-group label-cols-sm="6" label="Calls Per Second:" label-align-sm="right" label-for="nested-street11" label-class="pt-0" class="mb-0">
                    <b-form-row style="width: 332px; text-align: left" >{{callsPerSecond}}</b-form-row>
                </b-form-group>
                
                <b-form-group label-cols-sm="6" label="Injection Will Be Over At : " label-align-sm="right" label-for="nested-street11" label-class="pt-0" class="mb-0">
                    <b-form-row style="width: 332px; text-align: left" >{{injectionIsDoneAfterDate}}</b-form-row>
                </b-form-group>

                  <!-- b-form-group label-cols-sm="6" label="All Calls Will Be Queryable From Egress After (Approx.): " label-align-sm="right" label-for="nested-street11" label-class="pt-0" class="mb-0">
                     <b-form-row style="width: 332px; text-align: left" >{{queryableInEgressAfterDate}}</b-form-row>
                 </b-form-group-->
        </div>

        <div v-show="isRunning || injectionProgress === 100" style="width: 70%; alignment: center; margin-top: 20px; margin-left: auto; margin-right: auto">
            <b-progress :max="max"
            class="mb-3 centerTh">
                <b-progress-bar :value="injectionProgress" :label="`${injectionProgress}%`"></b-progress-bar>
            </b-progress>
        </div>
<!-- 
        <div v-show="isRunning || injectionProgress===100">Calls Injected: {{callsInjected}}</div>
        <div v-show="isRunning || injectionProgress===100">Calls per Second: {{callsPerSecond}}</div>
        <div v-show="isRunning || injectionProgress===100">Injection Will Be Over at (Approx.): {{injectionIsDoneAfterDate}}</div>
        <div v-show="isRunning || injectionProgress===100">All Calls Will Be Queryable From Egress After (Approx.): {{queryableInEgressAfterDate}}</div> -->

    </div>
</template>

<script>
    import axios from 'axios'
    import {StatusIndicator} from 'vue-status-indicator';
    import DateRangePicker from 'vue2-daterange-picker'
    import SockJS from "sockjs-client"
    import Stomp from "webstomp-client"
    import 'bootstrap/dist/css/bootstrap.css'
    // import 'bootstrap-vue/dist/bootstrap-vue.css'
    import 'vue2-daterange-picker/dist/vue2-daterange-picker.css'


    export default {
        name: 'HelloWorld',
        props: {
            msg: String
        },
        components: {
            DateRangePicker, StatusIndicator
        },
        filters: {
            date(value) {
                if (!value)
                    return ''
                let options = {year: 'numeric', month: 'long', day: 'numeric'};
                return Intl.DateTimeFormat('en-US', options).format(value)
            }
        },
        watch: {
            file: function name(newFile) {
                console.log('New file upload request')
                this.file = newFile;
            
                var reader = new FileReader();
                reader.onload = (event) => {
                    // isValid = true;
                            console.log(event.target.result);
                            try {
                                var obj = JSON.parse(event.target.result);
                                let businessDataArray = obj.businessdata;
                                businessDataArray.forEach((json) => { 
                                    if(json.Key === undefined) {
                                        // isValid = false
                                        throw new Error({'hehe':'haha'});
                                    }
                                    if(json.DataType === undefined || (json.DataType !== 'STRING' && json.DataType !== 'NUMBER')) {
                                        // isValid = false
                                        throw new Error({'hehe':'haha'});
                                    
                                    }
                                });
                                    // if(isValid){
                                // alert("File seems to be valid...")
                                    // }
                            
                            console.log(obj)
                            this.startRequest.businessData = obj
                            } catch (error) {
                                alert("File is not a valid!");
                                this.file = null;
                                this.startRequest.businessData = null
                            }
                };
                reader.readAsText(this.file);
                // var businessData = JSON.parse(this.file)
                // alert(businessData)
                // console.log("new file uploaded!")
                // console.log(businessData)
            },
            callsToInject: function (newValue) {
                if(newValue !== undefined && newValue !== null){
                const result = newValue.replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");

                this.$nextTick(() => { this.callsToInject = result})
                }
            },
            cpsUserValue: function (newVal) {
                if(newVal !== undefined && newVal !== null){
                    let result;

                    if(newVal == 5000){
                        result = "Max"
                    }
                    else{
                        result = newVal.toString().replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");
                    }

                    this.$nextTick(() => {this.cpsUserView = result})
                }
            }
        },
        data: function () {
            return {
                cpsUserView: "Max",
                cpsUserValue: 5000,
                retryTimeout: undefined,

                baseUrl: window.location.host.toString().substring(0, window.location.host.toString().indexOf(':')),

                opens: 'center',
                minDate: '2016-05-02 04:00:00',
                maxDate: '2030-12-26 14:00:00',
                single_range_picker: false,
                show_ranges: true,
                singleDatePicker: false,
                timePicker: false,
                timePicker24Hour: false,
                showDropdowns: true,
                autoApply: false,
                showWeekNumbers: false,
                linkedCalendars: false,
                alwaysShowCalendars: true,
                appendToBody: false,
                closeOnEsc: true,
                callsToInject: undefined,

                injectionIsDoneAfterDate: undefined,
                queryableInEgressAfterDate: undefined,

                isDLOnly: true,
                isWithBD: true,
              /*   isTurboMode: false, */
                isRunning: false,
                callsInjected: undefined,
                callsPerSecond: undefined,
                // dateRange: {
                //     startDate: undefined,
                //     endDate: undefined
                // },
                types: [
                    'number'
                ],

                injectionProgress: 0,
                max: 100,
                stompClient: undefined,
                connected: false,
                startRequest: {
                    callsToInject: undefined,
                    cpsUserValue: undefined,
                    dateRange: {
                        endDate: undefined,
                        startDate: undefined
                    },
                    /* isTurboMode: false, */
                    isDLOnly: true,
                    isWithBD: true,
                    businessData: null
                },

                file: null
            }
        },
        methods: {
            updateDate: function (date) {
                this.dateRange.startDate = date.startDate
                this.dateRange.endDate = date.endDate
            },
            sendInjectRequest: function () {
                /* this.startRequest.isTurboMode = true */
                this.startRequest.callsPerSecond = this.cpsUserValue
                this.startRequest.isDLOnly = this.isDLOnly
                this.startRequest.isWithBD = this.isWithBD

                this.startRequest.callsToInject = this.callsToInject
                let requestClone = JSON.parse(JSON.stringify(this.startRequest));
                requestClone.callsToInject = requestClone.callsToInject.toString().replace(/,/g, '')
                console.log('Sending start request')
                console.log(requestClone)
                axios.post("http://"+this.baseUrl+":9090/start", requestClone)
                    .then(response => {
                        this.injectionProgress = 0
                        console.log(response.data)
                        if (response.status === 200) {
                            this.isRunning = true
                        }
                    })
            },
            sendStopRequest: function () {
                console.log('Sending stop request')
                axios.delete("http://"+this.baseUrl+":9090/stop")
                    .then(response => {
                        console.log(response.data)
                        if (response.status === 200) {
                            this.isRunning = false
                        }
                    })
            },
            // saveDate: function (date) {
            //     this.startRequest.rangeFromMonth = date.rangeFrom
            //     this.startRequest.rangeToMonth = date.rangeTo
            //     this.startRequest.year = date.year
            //     console.log('Current date is:')
            //     console.log(this.date)
            // }
        },
        created: function () {
            const stompConnectFunc = function stompConnect() {

                if (this.stompClient !== undefined && this.stompClient.isConnected) {
                    this.stompClient.disconnect()
                }

                console.log('Trying to connect to the server');
                const socket = new SockJS("http://"+this.baseUrl+":9090/updates");
                this.stompClient = Stomp.over(socket);

                this.stompClient.connect(
                    {},
                    frame => {
                        console.log(frame);
                        console.log('Connected to server, now will try to subscribe');
                        const self = this;

                        this.stompClient.subscribe("/topic/messages", message => {
                            console.log('Writing WS message to log')
                            console.log(message);
                            const body = JSON.parse(message.body);
                            this.injectionProgress = body.injectionProgress
                            this.callsInjected = body.callsInjected.toString().replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");
                            this.callsPerSecond = body.callsPerSecond.toString().replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");
                            this.remainingSeconds = body.remainingSeconds

                            if(body.queryableInEgressAfterDate !== null && body.queryableInEgressAfterDate !== undefined){
                                this.queryableInEgressAfterDate = body.queryableInEgressAfterDate.replace("T", " ").substring(0, body.queryableInEgressAfterDate.indexOf("."))
                            }

                            if(body.injectionIsDoneAfterDate !== null && body.injectionIsDoneAfterDate !== undefined){
                                this.injectionIsDoneAfterDate = body.injectionIsDoneAfterDate.replace("T", " ").substring(0, body.injectionIsDoneAfterDate.indexOf("."))
                            }
                            
                            this.startRequest.callsToInject = body.totalCallsToInject === null ? this.totalCallsToInject : body.totalCallsToInject
                            const totalCalls = body.totalCallsToInject === null ? this.callsToInject:body.totalCallsToInject.toString().replace(",", "")

                            self.$nextTick(function () {

                            if(body.requestedCallsPerSecond !== null && body.requestedCallsPerSecond !== undefined) {
                                self.cpsUserValue = body.requestedCallsPerSecond
                                self.cpsUserValue
                            }  

                                if(body.startDate !== null && body.endDate){
                                  self.startRequest.dateRange = {
                                     startDate: body.startDate,
                                        endDate: body.endDate
                                 }
                                }
                                

                                self.callsToInject = totalCalls
                                    
                                self.injectionProgress = body.injectionProgress
                                self.connected = true
                                self.isRunning = body.running
   /*                              if(body.turboMode !== null && body.turboMode !== undefined) {
                                    self.isTurboMode = body.turboMode
                                }   */
                                if(body.isDLOnly !== null && body.isDLOnly !== undefined) {
                                    self.isDLOnly = body.isDLOnly
                                }  

                                if(body.isWithBD !== null && body.isWithBD !== undefined) {
                                    self.isWithBD = body.isWithBD
                                }  
                            })

                        });

                        while (this.retryTimeout--) {
                            window.clearTimeout(this.retryTimeout); // will do nothing if no timeout with id is present
                        }
                    },
                    error => {
                        console.log(error);
                        this.connected = false;
                        console.log('Setting timeout');
                        this.retryTimeout = setTimeout(stompConnectFunc, 1000);
                    }
                );
            }.bind(this)

            stompConnectFunc()
        }
    }
</script>

<style>
    :root {
        --status-indicator-size: 15px;
        --status-indicator-animation-duration: 2s;

        --status-indicator-color: rgb(216, 226, 233);
        --status-indicator-color-semi: rgba(216, 226, 233, .5);
        --status-indicator-color-transparent: rgba(216, 226, 233, 0);

        --status-indicator-color-active: rgb(0, 149, 255);
        --status-indicator-color-active-semi: rgba(0, 149, 255, .5);
        --status-indicator-color-active-transparent: rgba(0, 149, 255, 0);

        --status-indicator-color-positive: rgb(75, 210, 143);
        --status-indicator-color-positive-semi: rgba(75, 210, 143, .5);
        --status-indicator-color-positive-transparent: rgba(75, 210, 143, 0);

        --status-indicator-color-intermediary: rgb(255, 170, 0);
        --status-indicator-color-intermediary-semi: rgba(255, 170, 0, .5);
        --status-indicator-color-intermediary-transparent: rgba(255, 170, 0, 0);

        --status-indicator-color-negative: rgb(255, 77, 77);
        --status-indicator-color-negative-semi: rgba(255, 77, 77, .5);
        --status-indicator-color-negative-transparent: rgba(255, 77, 77, 0);
    }

    .marDown {
        margin-bottom: 70px;
    }

    .centerTh1 {
        width: 50%;
        margin: 0 auto;
    }

    .aParent div {
        float: left;
        clear: none;
    }

</style>
