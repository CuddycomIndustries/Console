<template>
  <section>
    <div class="columns">
      <div class="column is-one-fifth is-paddingless agent-list-column">
        <app-agent-list :agents="agents"></app-agent-list>
      </div>
      <div v-if="agent" class="column is-paddingless console-column">
        <div class="console-wrapper">
          <div ref="console" class="console">
            <h2 class="subtitle console-title">Console Output for {{ agent.Name }}</h2>
            <app-console-message-list ref="messageList" :messages="messages"></app-console-message-list>
          </div>
          <div class="console-input">
            <div class="field">
              <div class="control has-icons-left">
                <input type="text" class="input" v-model="content" v-on:keyup.enter="submitMessage">
                <span class="icon is-left">
                  <i class="mdi mdi-chevron-right mdi-24px"></i>
                </span>
                <a @click="submitMessage" class="button is-dark is-radiusless">Send</a>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="columns" v-else>
        <div class="column has-text-centered">Loading Agents</div>
      </div>
    </div>
  </section>
</template>

<script>
import consoleMessageList from './ConsoleMessageList'
import agentList from './AgentList'
import { mapState, mapActions } from 'vuex'

export default {
  name: 'console',
  data () {
    return {
      content: null,
      showAgentDetails: false,
      getAgentFired: null,
      query: null
    }
  },
  components: {
    appConsoleMessageList: consoleMessageList,
    appAgentList: agentList
  },
  computed: {
    ...mapState({
      agent: state => state.agents.currentAgent,
      agents: state => state.agents.list,
      messages: state => state.consoleMessages.list
    }),
    osIcon () {
      if (this.agent.OperatingSystem.includes('indows')) {
        return 'windows'
      } else if (this.agent.OperatingSystem.includes('acOS')) {
        return 'apple'
      } else if (this.agent.OperatingSystem.includes('nix')) {
        return 'linux'
      } else {
        return 'laptop'
      }
    }
  },
  created () {
    console.log('page created: ' + this.$route.params.AgentId)
    this.updateCurrentAgentId(this.$route.params.AgentId)
  },
  beforeRouteLeave (to, from, next) {
    console.log('leaving route, clearing stuff')
    this.clearCurrentAgent()
    this.clearMessages()
    next()
  },
  methods: {
    ...mapActions([
      'updateCurrentAgentId',
      'clearCurrentAgent',
      'clearMessages'
    ]),
    toggleDetails () {
      this.showAgentDetails = !this.showAgentDetails
    },
    submitMessage (message) {
      console.log('Sumitting Message')
      this.$socket.emit('newMessage', {
        AgentId: this.agent.Id,
        Content: this.content
      })
      this.content = ''
    }
  },
  watch: {
    $route () {
      console.log('route update route id: ' + this.$route.params.AgentId)
      console.log('page loaded. agents updated: ' + this.agents.updated)
      this.updateCurrentAgentId(this.$route.params.AgentId)
      this.clearMessages()
    },
    agent () {
      if (this.agent != null) {
        console.log('joining room ' + this.agent.Id)
        this.$socket.emit('joinAgent', { AgentId: this.agent.Id })
      }
    },
    agents () {
      console.log('agents updated, setting current agents')
      console.log(this.agents)
      this.updateCurrentAgentId(this.$route.params.AgentId)
    },
    messages () {
      this.$nextTick(function () {
        console.log('Console: Messages updated')
        var messageList = this.$refs.console
        if (messageList.scrollHeight !== undefined) {
          messageList.scrollTop = messageList.scrollHeight
        }
      })
    }
  }
}
</script>

<style scoped>
.agent-list-column {
  min-width: 350px;
}
.console-column {
  background: #012121;
}
.console-wrapper {
  height: 100vh;
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  top: 40px;
  background: #012121;
  z-index: -1;
}

.console-title,
.console-input > input {
  font-family: "Source Code Pro", "Lucida Console", Monaco, monospace;
}

.console {
  position: fixed;
  top: 52px;
  left: 20%;
  right: 0;
  bottom: 40px;
  min-height: 400px;
  padding-top: 10px;
  padding-left: 10px;
  overflow-y: auto;
}

.console-title {
  color: #eff7f7;
}

.console-input {
  border-top: solid;
  border-top-color: #034748;
  border-top-width: 1px;
  position: fixed;
  right: 0;
  left: 20%;
  bottom: 0;
  height: 40px;
}

.console-input .control {
  width: 100%;
  height: 40px;
}

.console-input .input {
  background: #012121;
  height: 39px;
  color: #eff7f7;
  font-family: "Source Code Pro", "Lucida Console", Monaco, monospace;
  border: none;
  position: fixed;
  right: 0;
  left: 20%;
  bottom: 0;
  border-radius: 0;
}

.console-input input:focus {
  box-shadow: 0;
}

.console-input .button {
  position: fixed;
  right: 0;
  height: 40px;
}

@media only screen and (max-width: 1023px) {
  .agent-list-column {
    display: none;
  }
  .console {
    left: 0;
  }

  .console-input,
  .console-input .input {
    left: 0;
  }
}

@media only screen and (min-width: 1024px) and (max-width: 1407px) {
  .console {
    left: 30%;
  }

  .console-input,
  .console-input .input {
    left: 30%;
  }
}

@media only screen and (min-width: 1408px) and (max-width: 1524px) {
  .console {
    left: 25%;
  }

  .console-input,
  .console-input .input {
    left: 25%;
  }
}
</style>
