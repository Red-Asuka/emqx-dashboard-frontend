<template>
  <div class="websocket-view">
    <div class="page-title">{{ $t('leftbar.websocket') }}</div>
    <el-card class="el-card--self" @keyup.enter.native="mqttConnect">
      <div slot="header">
        <span>{{ $t('websocket.connect') }}</span>
      </div>
      <el-form size="medium">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-form-item :label="$t('websocket.host')">
              <el-input v-model="host"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item :label="$t('websocket.port')">
              <el-input v-model.number="port"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="Path">
              <el-input v-model="path"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item :label="$t('websocket.clientID')">
              <el-input v-model="clientId"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item :label="$t('websocket.username')">
              <el-input v-model="username"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item :label="$t('websocket.password')">
              <el-input v-model="password"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item :label="$t('websocket.keepAlive')">
              <el-input v-model.number="keepalive"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="24">
            <el-form-item class="check-area">
              <el-checkbox v-model="clean">
                {{ $t('websocket.cleanSession') }}
              </el-checkbox>
              <el-checkbox
                style="margin-left: 50px"
                v-model="isSSL"
                @change="handleSSL">
                SSL
              </el-checkbox>
              <span style="margin: 5px 0 20px 50px;color: #42d885">{{ connectURL }}</span>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div class="operation-area">
        <el-button
          type="success"
          class="confirm-btn"
          :disabled="loading || client.connected"
          @keyup.enter.native="mqttConnect"
          @click="mqttConnect">
          {{ $t('websocket.connect') }}
        </el-button>

        <el-button
          type="danger"
          :loading="loading && client.connected"
          :disabled="!loading && !client.connected"
          style="margin-left: 20px"
          @keyup.enter.native="disconnectSwitch"
          @click="disconnectSwitch">
          {{ $t('websocket.disconnect') }}
        </el-button>

        <div class="connect-state">
          {{ $t('websocket.currentState') }}:
          <span :style="client.connected ? 'color: #42d885' : 'color: #ff6d6d'">
          {{ getStatus }}
          </span>
        </div>
      </div>
    </el-card>

    <el-card class="el-card--self" style="max-height: 450px">
      <div slot="header">
        <span>{{ $t('websocket.subscribe') }}</span>
      </div>
      <el-form size="medium">
        <el-row :gutter="20" @keyup.enter.native="mqttSubscribe">
          <el-col :span="12">
            <el-form-item :label="$t('websocket.topic')">
              <el-input v-model="subTopic"></el-input>
            </el-form-item>
            <el-form-item :label="$t('websocket.qoS')">
              <el-select v-model.number="subQos">
                <el-option :value="0"></el-option>
                <el-option :value="1"></el-option>
                <el-option :value="2"></el-option>
              </el-select>
            </el-form-item>
            <div class="operation-area">
              <el-button
                class="confirm-btn"
                type="success"
                @keyup.enter.native="mqttSubscribe"
                @click="mqttSubscribe">
                {{ $t('websocket.subscribe') }}
              </el-button>
            </div>
          </el-col>
          <el-col :span="12">
            <el-form-item :label="$t('websocket.subscribe')">
              <el-table :data="subscriptions" :max-height="320">
                <el-table-column prop="topic" min-width="150" :label="$t('websocket.topic')">
                </el-table-column>
                <el-table-column prop="qos" min-width="120" :label="$t('websocket.qoS')">
                </el-table-column>
                <el-table-column prop="time" min-width="180" :label="$t('websocket.time')">
                </el-table-column>
                <el-table-column width="90" :label="$t('websocket.oper')">
                  <template slot-scope="props">
                    <i
                      title="Unsubscribe"
                      class="unsubscribe el-icon-close"
                      @click="mqttCacheScuscribe(props.row.topic)">
                    </i>
                  </template>
                </el-table-column>
              </el-table>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>

    <el-card class="el-card--self" style="max-height: 800px;padding-bottom: 20px">
      <div slot="header">
        <span>{{ $t('websocket.messages') }}</span>
      </div>
      <el-form size="medium">
        <el-row :gutter="20" @keyup.enter.native="mqttPublish">
          <el-col :span="6">
            <el-form-item :label="$t('websocket.topic')">
              <el-input v-model="publishTopic"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item :label="$t('websocket.messages')">
              <el-input v-model="publishMessage"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item :label="$t('websocket.qoS')">
              <el-select v-model.number="publishQos">
                <el-option :value="0"></el-option>
                <el-option :value="1"></el-option>
                <el-option :value="2"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col style="margin-top: 33px" :span="6">
            <el-form-item>
              <el-checkbox v-model="publishRetain">
                {{ $t('websocket.retained') }}
              </el-checkbox>
              <el-button
                class="confirm-btn"
                type="success"
                style="float: right;margin-top: 4px"
                @click="mqttPublish"
                @keyup.enter.native="mqttPublish">
                {{ $t('websocket.send') }}
              </el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <el-form size="medium" style="margin-top: 20px;">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item :label="$t('websocket.messagesAlreadySent')">
              <i
                title="clear message"
                class="fa fa-refresh refresh-btn"
                @click="clearMessage(false)">
              </i>
              <el-table border :data="publishedMessages" :max-height="600">
                <el-table-column prop="message" min-width="100" :label="$t('websocket.messages')">
                </el-table-column>
                <el-table-column prop="topic" :label="$t('websocket.topic')">
                </el-table-column>
                <el-table-column prop="qos" min-width="120" :label="$t('websocket.qoS')">
                </el-table-column>
                <el-table-column prop="time" min-width="180" :label="$t('websocket.time')">
                </el-table-column>
              </el-table>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item :label="$t('websocket.messagesReceived')">
              <i
                title="clear message"
                class="fa fa-refresh refresh-btn"
                @click="clearMessage">
              </i>
              <el-table border :data="receivedMessages" :max-height="600">
                <el-table-column prop="message" min-width="100" :label="$t('websocket.messages')">
                </el-table-column>
                <el-table-column prop="topic" :label="$t('websocket.topic')">
                </el-table-column>
                <el-table-column prop="qos" min-width="120" :label="$t('websocket.qoS')">
                </el-table-column>
                <el-table-column prop="time" width="180" :label="$t('websocket.time')">
                </el-table-column>
              </el-table>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>
  </div>
</template>


<script>
import NProgress from 'nprogress'
import mqtt from 'mqtt'
import dateformat from 'dateformat'
import {
  Input, Checkbox, Select, Option, Button, Form, FormItem,
  Table, TableColumn, Row, Col, Card,
} from 'element-ui'

import MQTTConnect from '../common/MQTTConnect'

export default {
  name: 'websocket-view',
  components: {
    'el-input': Input,
    'el-checkbox': Checkbox,
    'el-select': Select,
    'el-option': Option,
    'el-button': Button,
    'el-table': Table,
    'el-table-column': TableColumn,
    'el-row': Row,
    'el-col': Col,
    'el-card': Card,
    'el-form': Form,
    'el-form-item': FormItem,
  },
  data() {
    return {
      retryTimes: 0,
      loading: false,
      sending: false,
      host: window.location.hostname,
      port: 8083,
      path: '/mqtt',
      username: '',
      isSSL: false,
      password: '',
      keepalive: 60,
      clean: true,
      clientId: `mqttjs_${Math.random().toString(16).substr(2, 10)}`,
      subQos: 0,
      publishQos: 0,
      publishMessage: '{ "msg": "Hello, World!" }',
      subTopic: 'testtopic/#',
      publishTopic: 'testtopic',
      publishRetain: false,
      receivedMessages: [],
      publishedMessages: [],
      subscriptions: [],
      client: {},
    }
  },
  computed: {
    getStatus() {
      if (this.client.connected) {
        return this.$t('websocket.connected')
      }
      if (this.loading) {
        return this.$t('websocket.connecting')
      }
      return this.$t('websocket.disconnected')
    },
    connectURL() {
      const path = this.path && this.path.startsWith('/') ? this.path : `/${this.path}`
      return `${this.isSSL ? 'wss' : 'ws'}://${this.host}:${this.port}${path}`
    },
  },
  methods: {
    handleSSL() {
      this.port = this.isSSL ? 8084 : 8083
    },
    now() {
      return dateformat(new Date(), 'yyyy-mm-dd HH:MM:ss')
    },
    disconnectSwitch() {
      // connecting
      if (this.loading && !this.client.connected) {
        this.loading = false
        NProgress.done()
        this.client.end()
        this.client = {}
      } else {
        this.mqttDisconnect()
      }
    },
    mqttConnect() {
      if (!window.WebSocket) {
        this.$message.error(this.$t('websocket.notSupport'))
        return
      }
      // prevent the connect from keyboard event
      if (this.client.connected || this.loading) {
        return
      }
      NProgress.start()
      this.loading = true
      this.retryTimes = 0
      const options = {
        keepalive: this.keepalive,
        username: this.username,
        password: this.password,
        clientId: this.clientId,
        clean: this.clean,
        connectTimeout: 10 * 1000,
      }
      try {
        this.client = mqtt.connect(this.connectURL, options)
        this.client.on('connect', () => {
          this.loading = false
          NProgress.done()
        })
        this.client.on('reconnect', this.handleReconnect)
        this.client.on('error', (error) => {
          this.$message.error(error.toString() || this.$t('error.networkError'))
          // to prevent reconnect
          this.retryTimes = 0
          this.client.end()
          this.sending = false
          this.loading = false
          NProgress.done()
        })
        this.client.on('message', (topic, message, packet) => {
          this.receivedMessages.unshift({
            topic,
            message: message.toString(),
            qos: packet.qos,
            time: this.now(),
          })
        })
      } catch (error) {
        this.loading = false
        NProgress.done()
        this.$message.error(error.toString())
      }
    },
    mqttDisconnect() {
      if (this.client.connected) {
        NProgress.start()
        this.client.end()
        this.client.on('close', () => {
          this.loading = false
          this.reset()
          this.client = {}
          NProgress.done()
        })
      } else {
        this.$message.error(this.$t('websocket.connectLeave'))
      }
    },
    mqttSubscribe() {
      if (this.client.connected) {
        this.subscriptions.forEach((x, index) => {
          if (x.topic === this.subTopic) {
            this.subscriptions.splice(index, index + 1)
          }
        })
        NProgress.start()
        this.client.subscribe(this.subTopic, { qos: this.subQos }, (error, granted) => {
          if (error || (
            granted[0] && ![0, 1, 2].includes(granted[0].qos)
          )) {
            NProgress.done()
            this.$message.error(error ? error.message : this.$t('websocket.subscribeFailure'))
          } else {
            this.subscriptions.unshift({
              topic: this.subTopic,
              qos: this.subQos,
              time: this.now(),
            })
            this.$message.success(this.$t('websocket.subscribeSuccess'))
            NProgress.done()
          }
        })
      } else {
        this.$message.error(this.$t('websocket.connectLeave'))
      }
    },
    mqttPublish() {
      if (this.client.connected) {
        NProgress.start()
        const options = {
          qos: this.publishQos,
          retain: this.publishRetain,
        }
        // to mark which trigger the reconnect
        this.sending = true
        this.client.publish(this.publishTopic,
          this.publishMessage, options, (error) => {
            if (error) {
              NProgress.done()
              this.$message.error(error.toString())
            } else {
              this.publishedMessages.unshift({
                message: this.publishMessage,
                topic: this.publishTopic,
                qos: this.publishQos,
                time: this.now(),
              })
              this.$message.success(this.$t('websocket.messageSendOut'))
              NProgress.done()
            }
          })
      } else {
        this.$message.error(this.$t('websocket.connectLeave'))
      }
    },
    mqttCacheScuscribe(topic) {
      if (!this.client.connected) {
        this.$message.error(this.$t('websocket.connectLeave'))
        return
      }
      this.client.unsubscribe(topic, (error) => {
        if (error) {
          this.$message.error(`${this.$t('websocket.unsubscribeFailure')} ${error.toString()}!`)
          return
        }
        this.subscriptions.forEach((element, index) => {
          if (element.topic === topic) {
            this.subscriptions.splice(index, 1)
            // clear message which in this topic
          }
        })
      })
    },
    handleReconnect() {
      if (this.retryTimes > 1) {
        try {
          if (this.sending) {
            this.$message.error(this.$t('websocket.connectError'))
          } else {
            this.$message.error(`${this.$t('websocket.connectFailure')} ${this.host}:${this.port}`)
          }
        } catch (e) {
          this.retryTimes = 0
          this.sending = false
          this.loading = false
          NProgress.done()
          this.client.end()
          this.client = {}
        }
        this.retryTimes = 0
        this.sending = false
        this.loading = false
        NProgress.done()
        this.client.end()
        this.client = {}
        return
      }
      // trigger by sending illegal topic
      if (this.sending) {
        this.$message.error(this.$t('websocket.connectError'))
      }
      this.retryTimes += 1
    },
    reset() {
      this.subscriptions = []
      this.receivedMessages = []
      this.publishedMessages = []
      this.publishMessage = '{ "msg": "Hello, World!" }'
      this.subTopic = 'testtopic/#'
      this.publishTopic = 'testtopic'
    },
    clearMessage(received = true) {
      if (received) {
        this.receivedMessages = []
      } else {
        this.publishedMessages = []
      }
    },
    loadConnect() {
      if (MQTTConnect.client && MQTTConnect.client.connected) {
        this.client = MQTTConnect.client
        Object.keys(MQTTConnect.options).forEach((item) => {
          this[item] = MQTTConnect.options[item]
        })
      }
    },
    stashConnect() {
      MQTTConnect.client = this.client
      Object.keys(MQTTConnect.options).forEach((item) => {
        MQTTConnect.options[item] = this[item]
      })
    },
    setSSL() {
      if (window.location.protocol === 'https:') {
        this.isSSL = true
        this.port = 8084
      }
    },
  },
  created() {
    this.setSSL()
    this.loadConnect()
  },
  beforeRouteLeave(to, from, next) {
    if (this.client.connected) {
      this.stashConnect()
    }
    next()
  },
}
</script>


<style lang="scss">
.websocket-view {
  .el-form-item--small.el-form-item {
    margin-bottom: 2px;
  }
  .check-area {
    .el-form-item__content {
      line-height: 20px !important;
    }
  }
  .operation-area {
    margin-top: 10px !important;
  }
  .el-input .el-input--medium {
    margin-bottom: 10px !important;
  }
  .el-select {
    width: 100%;
  }
  .refresh-btn {
    font-size: 12px;
    margin-left: 8px;
    cursor: pointer;
  }
  .connect-state {
    display: inline-block;
    margin-left: 20px;
    font-size: 14px;
    color: #a7a7a7;
    span {
      margin-left: 4px;
    }
  }
  .el-table {
    margin-top: 5px;
    /* 增加内边距 */
    border-width: 0 !important;
  }
  .el-card {
    margin-top: 24px;
  }
  .el-input,
  .el-checkbox {
    margin: 5px 0 20px;
  }
}
</style>
