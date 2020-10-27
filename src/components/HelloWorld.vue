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
            </b-form-group>
        </div>

        <b-form-checkbox v-model="isTurboMode" >Turbo Mode</b-form-checkbox>
        <b-button variant="primary" style="margin: 8px 8px 8px 8px" @click="sendInjectRequest()"
                  :disabled="isRunning || !connected">Start
        </b-button>
        <b-button variant="danger" style="margin: 8px 8px 8px 8px" @click="sendStopRequest()"
                  :disabled="!isRunning || !connected">Stop
        </b-button>
        <div v-show="isRunning">Calls injected: {{callsInjected}}</div>
        <div v-show="isRunning">Calls Per Second: {{callsPerSecond}}</div>
        <div v-show="isRunning">
            All calls Will Be Queryable From Egress After (Approx.): {{queryableInEgressAfterDate}}</div>
        <div style="width: 70%; alignment: center; margin-left: auto; margin-right: auto">
            <b-progress :max="max"
            class="mb-3 centerTh">
                <b-progress-bar :value="injectionProgress" :label="`${injectionProgress}%`"></b-progress-bar>
            </b-progress>
        </div>

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
            callsToInject: function (newValue) {
                const result = newValue.replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");

                this.$nextTick(() => {
                    this.callsToInject = result
                })
            }
        },
        data: function () {
            return {
                retryTimeout: undefined,

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

                queryableInEgressAfterDate: undefined,

                isTurboMode: false,
                isRunning: false,
                callsInjected: undefined,
                callsPerSecond: undefined,
                // dateRange: {
                //     startDate: undefined,
                //     endDate: undefined
                // },
                form: {
                    callsToInject: undefined
                },
                types: [
                    'number'
                ],

                injectionProgress: 0,
                max: 100,
                stompClient: undefined,
                connected: false,
                startRequest: {
                    callsToInject: undefined,
                    dateRange: {
                        endDate: undefined,
                        startDate: undefined
                    },
                    isTurboMode: false
                }
            }
        },
        methods: {
            updateDate: function (date) {
                this.dateRange.startDate = date.startDate
                this.dateRange.endDate = date.endDate
            },
            sendInjectRequest: function () {
                this.startRequest.isTurboMode = this.isTurboMode
                this.startRequest.callsToInject = this.callsToInject
                let requestClone = JSON.parse(JSON.stringify(this.startRequest));
                requestClone.callsToInject = requestClone.callsToInject.toString().replace(/,/g, '')
                console.log(requestClone)
                axios.post('http://localhost:9090/start', requestClone)
                    .then(response => {
                        console.log(response.data)
                        if (response.status === 200) {
                            this.isRunning = true
                        }
                    })
            },
            sendStopRequest: function () {
                axios.delete('http://localhost:9090/stop')
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
                const socket = new SockJS("http://localhost:9090/updates");
                this.stompClient = Stomp.over(socket);

                this.stompClient.connect(
                    {},
                    frame => {
                        console.log(frame);
                        console.log('Connected to server, now will try to subscribe');
                        const self = this;

                        this.stompClient.subscribe("/topic/messages", message => {

                            console.log(message);
                            const body = JSON.parse(message.body);
                            this.injectionProgress = body.injectionProgress
                            this.callsInjected = body.callsInjected.toString().replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");
                            this.callsPerSecond = body.callsPerSecond.toString().replace(/\D/g, "").replace(/\B(?=(\d{3})+(?!\d))/g, ",");
                            this.remainingSeconds = body.remainingSeconds
                            if(body.queryableInEgressAfterDate !== null && body.queryableInEgressAfterDate !== undefined){
                                this.queryableInEgressAfterDate = body.queryableInEgressAfterDate.replace("T", " ").substring(0, body.queryableInEgressAfterDate.indexOf("."))
                            }
                            this.startRequest.callsToInject = body.totalCallsToInject === 0 ? undefined : body.totalCallsToInject
                            const totalCalls = body.totalCallsToInject === 0 ? undefined:body.totalCallsToInject.toString().replace(",", "")

                            self.$nextTick(function () {
                                self.connected = true
                                self.isRunning = body.running
                                self.isTurboMode = body.turboMode
                                if(self.isRunning){
                                    this.callsToInject = totalCalls
                                }
                            })

                            // this.received_messages.push(JSON.parse(message.body).content);
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

<style scoped>
    .marDown {
        margin-bottom: 70px;
    }

    .centerTh {
        margin-top: 180px;

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
