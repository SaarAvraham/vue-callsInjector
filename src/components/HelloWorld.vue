<template>
    <div>
        <h1 class="marDown"> {{ msg }}</h1>
        <status-indicator v-if="connected" status="positive" pulse="true"/>
        <status-indicator v-else status="negative"/>
        <b-form>
            <b-form-group
                    id="input-group-1"
                    label="Number of calls:"
                    label-for="input-1"
            >
                <b-form-input
                        id="input-1"
                        v-model="startRequest.callsToInject"
                        type="number"
                        required
                        placeholder="Enter the amount of calls to inject"
                        style="width: auto; justify-content: center; display: flex; margin:auto; "
                ></b-form-input>
            </b-form-group>
        </b-form>
<!--        <month-picker-input class="centerTh1" @change="saveDate" :range="true" v-show="false"-->
<!--        ></month-picker-input>-->
        <div class="py-5">
            <div class="form-group">
                <label>Select range:  </label>
                <date-range-picker
                        ref="picker"
                        :opens="opens"
                        :locale-data="{ firstDay: 1, format: 'yyyy-mm-dd HH:MM:ss' }"
                        :minDate="minDate" :maxDate="maxDate"
                        :singleDatePicker="singleDatePicker"
                        :timePicker="timePicker"
                        :timePicker24Hour="timePicker24Hour"
                        :showWeekNumbers="showWeekNumbers"
                        :showDropdowns="showDropdowns"
                        :autoApply="autoApply"
                        v-model="dateRange"
                        :ranges="show_ranges ? undefined : false"
                        @update="updateValues"
                        @toggle="checkOpen"
                        :linkedCalendars="linkedCalendars"
                        :dateFormat="dateFormat"
                        :always-show-calendars="false"
                        :alwaysShowCalendars="alwaysShowCalendars"
                        :append-to-body="appendToBody"
                        :closeOnEsc="closeOnEsc"
                >
                    <template #input="picker" style="min-width: 350px;">
                        {{ picker.startDate | date }} - {{ picker.endDate | date }}
                    </template>
                </date-range-picker>

                <button class="btn btn-info" @click="dateRange.startDate = null, dateRange.endDate = null">
                    Clear
                </button>
            </div>
        </div>

        <b-form-checkbox v-model="isTurboMode" :disabled="isRunning">Turbo Mode</b-form-checkbox>
        <div v-show="isRunning">Calls injected: {{callsInjected}}</div>
        <div v-show="isRunning">Calls Per Second: {{callsPerSecond}}</div>
        <b-progress  :max="max" style="width: 70%; alignment: center" class="mb-3 centerTh">
            <b-progress-bar :value="injectionProgress" :label="`${injectionProgress}%`"></b-progress-bar>
        </b-progress>
        <b-button variant="primary" style="margin: 8px 8px 8px 8px" @click="sendInjectRequest()" :disabled="isRunning">Start</b-button>
        <b-button variant="danger" style="margin: 8px 8px 8px 8px" @click="sendStopRequest()" :disabled="!isRunning">Stop</b-button>
    </div>
</template>

<script>
    // import {MonthPickerInput} from 'vue-month-picker'
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
            // ProgressBar
        },
        data: function () {
            return {

                opens: 'center',
                minDate: '2019-05-02 04:00:00',
                maxDate: '2020-12-26 14:00:00',
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


                isTurboMode: false,
                isRunning: false,
                callsInjected: undefined,
                callsPerSecond: undefined,
                dateRange: {
                    startDate: undefined,
                    endDate: undefined
                },
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
                    rangeFromMonth: null,
                    rangeToMonth: null,
                    year: null,
                    isTurboMode: false
                },
                options: {
                    text: {
                        color: '#FFFFFF',
                        shadowEnable: true,
                        shadowColor: '#000000',
                        fontSize: 14,
                        fontFamily: 'Helvetica',
                        dynamicPosition: false,
                        hideText: false
                    },
                    progress: {
                        color: '#2dbd2d',
                        backgroundColor: '#333333'
                    },
                    layout: {
                        height: 35,
                        width: 640,
                        verticalTextAlign: 61,
                        horizontalTextAlign: 43,
                        zeroOffset: 0,
                        strokeWidth: 30,
                        progressPadding: 0,
                        type: 'line'
                    }
                }
                // fromDate: {
                //     from: null,
                //     to: null,
                //     month: null,
                //     year: null
                // },
                // toDate: {
                //     from: null,
                //     to: null,
                //     month: null,
                //     year: null
                // }
            }
        },
        methods: {
            updateDate: function (date) {
                this.dateRange.startDate = date.startDate
                this.dateRange.endDate = date.endDate
            },
            sendInjectRequest: function () {
                // const agent = https.Agent({
                //     rejectUnauthorized: false
                // });

                this.startRequest.isTurboMode = this.isTurboMode
                axios.post('http://localhost:9090/start', this.startRequest)
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
                console.log('Trying to connect to the server');
                const socket = new SockJS("http://localhost:9090/updates");
                this.stompClient = Stomp.over(socket);

                this.stompClient.connect(
                    {},
                    frame => {
                        try {
                            console.log(frame);
                            console.log('Connected to server, now will try to subscribe');
                            const self = this;

                            this.stompClient.subscribe("/topic/messages", message => {
                                console.log(message);
                                console.log(message.body.injectionProgress);
                                const body = JSON.parse(message.body);
                                this.injectionProgress = body.injectionProgress
                                this.callsInjected = body.callsInjected
                                this.callsPerSecond = body.callsPerSecond
                                this.remainingSeconds = body.remainingSeconds

                                self.$nextTick(function () {
                                    self.connected = true
                                })

                                // this.received_messages.push(JSON.parse(message.body).content);
                            });
                        } catch (e) {
                            console.log(e.message)
                            throw e
                        }
                    },
                    error => {
                        console.log(error);
                        this.connected = false;
                        setTimeout(stompConnectFunc, 1000);
                    }
                );
            }.bind(this)

            stompConnectFunc()
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .marDown {
        margin-bottom: 70px;
    }

    .centerTh {
        margin-top: 280px;

    }

    .centerTh1 {
        width: 50%;
        margin: 0 auto;
    }

</style>
