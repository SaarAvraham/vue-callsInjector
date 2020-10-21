<template>
    <div class="hello">
        <h1 class="marDown"> {{ msg }}</h1>
        <!--        <div class="centerTh">-->
        <!--            <progress-bar-->
        <!--                    :options="options"-->
        <!--                    :value="injectionProgress"-->
        <!--            />-->
        <!--        </div>-->
        <b-form>
            <b-form-group
                    id="input-group-1"
                    label="Number of calls:"
                    label-for="input-1"
            >
                <b-form-input
                        id="input-1"
                        v-model="searchRequest.callsToInject"
                        type="number"
                        required
                        placeholder="Enter the amount of calls to inject"
                        style="width: auto; justify-content: center; display: flex; margin:auto; "
                ></b-form-input>
            </b-form-group>
        </b-form>
        <month-picker-input class="centerTh1" @change="saveDate" :range="true" v-show="true"
        ></month-picker-input>
        <date-range-picker
                v-show="false"
                :opens="true"
                v-model="dateRange"
                @update="updateDate"
        >
            <template v-slot:input="picker" style="min-width: 350px;">
                {{ picker.startDate}} - {{ picker.endDate}}
            </template>
            <div>sdknsakdsankdsan</div>
        </date-range-picker>
        <div v-show="isRunning">Calls injected: {{callsInjected}}</div>
        <div v-show="isRunning">Calls Per Second: {{callsPerSecond}}</div>
        <b-progress :max="max" class="mb-3 centerTh">
            <b-progress-bar :value="injectionProgress" :label="`${injectionProgress}%`"></b-progress-bar>
        </b-progress>

        <!--        <month-picker-input @change="saveDate" :range="true"></month-picker-input>-->
        <!--    <month-picker-input @change="saveToDate"></month-picker-input>-->
        <b-button variant="primary" style="margin: 8px 8px 8px 8px" @click="sendInjectRequest()">Start</b-button>
        <b-button variant="danger" style="margin: 8px 8px 8px 8px" @click="sendStopRequest()">Stop</b-button>
    </div>
</template>

<script>
    import {MonthPickerInput} from 'vue-month-picker'
    import axios from 'axios'
    // import ProgressBar from 'vuejs-progress-bar'
    // import * as https from "https";
    import DateRangePicker from 'vue2-daterange-picker'
    import SockJS from "sockjs-client"
    import Stomp from "webstomp-client"
    import 'bootstrap/dist/css/bootstrap.css'
    import 'bootstrap-vue/dist/bootstrap-vue.css'


    export default {
        name: 'HelloWorld',
        props: {
            msg: String
        },
        components: {
            MonthPickerInput, DateRangePicker
            // ProgressBar
        },
        data: function () {
            return {
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
                searchRequest: {
                    callsToInject: undefined,
                    rangeFromMonth: null,
                    rangeToMonth: null,
                    year: null
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

                axios.post('http://localhost:9090/start', this.searchRequest)
                    .then(response => {
                        console.log(response.data)
                        if(response.status === 200){
                            this.isRunning = true
                        }
                    })
            },
            sendStopRequest: function () {
                axios.delete('http://localhost:9090/stop')
                    .then(response => {
                        console.log(response.data)
                        if(response.status === 200){
                            this.isRunning = false
                        }
                    })
            },
            saveDate: function (date) {
                this.searchRequest.rangeFromMonth = date.rangeFrom
                this.searchRequest.rangeToMonth = date.rangeTo
                this.searchRequest.year = date.year
                console.log('Current date is:')
                console.log(this.date)
            }
            // saveFromDate: function (date) {
            //   console.log(date)
            //   this.fromDate = date
            // },
            // saveToDate: function (date) {
            //   console.log(date)
            //   this.toDate = date
            // }
        },
        created: function () {
            // console.log("Starting connection to WebSocket Server")
            // this.connection = new WebSocket('ws://localhost:9090/chat')
            //
            // this.connection.onmessage = function (event) {
            //     console.log(event);
            // }
            //
            // this.connection.onopen = function (event) {
            //     console.log(event)
            //     console.log("Successfully connected to the echo websocket server...")
            // }
            const socket = new SockJS("http://localhost:9090/updates");
            this.stompClient = Stomp.over(socket);
            this.stompClient.connect(
                {},
                frame => {
                    console.log(frame);
                    this.connected = true;
                    this.stompClient.subscribe("/topic/messages", message => {
                        console.log(message);
                        console.log(message.body.injectionProgress);
                        const body = JSON.parse(message.body);
                        this.injectionProgress = body.injectionProgress
                        this.callsInjected = body.callsInjected
                        this.callsPerSecond = body.callsPerSecond
                        this.remainingSeconds = body.remainingSeconds
                        // this.received_messages.push(JSON.parse(message.body).content);
                    });
                },
                error => {
                    console.log(error);
                    this.connected = false;
                }
            );
        }
        // disconnect() {
        //     if (this.stompClient) {
        //         this.stompClient.disconnect();
        //     }
        //     this.connected = false;
        // },
        // tickleConnection() {
        //     this.connected ? this.disconnect() : this.connect();
        // }
        // }
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

    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
