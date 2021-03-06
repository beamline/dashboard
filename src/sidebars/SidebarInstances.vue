<template>
  <div>
    <b-sidebar id="sidebar-instances" title="Instances" shadow backdrop visible>
      <template #footer="{}">
        <div class="p-3">
          <b-button block v-b-modal.new-instance>
            <font-awesome-icon icon="plus" />
            New miner instance
          </b-button>
        </div>
      </template>

      <b-list-group class="py-2" flush>
        <b-list-group-item class="bg-light" v-bind:key="i.id" v-for="i in instances" v-b-toggle="'details-id-' + i.id">
          <b-button
            size="sm"
            variant="outline-secondary"
            class="float-right ml-1"
            :to="{
              name: 'InstanceViewer',
              params: { instanceId: i.id, host: minerIdToHost(i.miner.id), protocol: minerIdToProtocol(i.miner.id) },
            }"
            v-b-tooltip.hover.bottom="'Open this instance'"
            v-on:click.stop=""
          >
            <font-awesome-icon icon="folder-open" />
          </b-button>
          <font-awesome-icon icon="sliders-h" />
          {{ i.configuration.name }}
          <b-collapse class="pt-2" :id="'details-id-' + i.id">
            <small v-b-tooltip.hover="i.configuration.stream.processName" style="white-space: nowrap; overflow: hidden; display: block" class="mb-1">
              <font-awesome-icon icon="film" /> {{ i.configuration.stream.processName }}
            </small>
            <small v-b-tooltip.hover="i.miner.name" style="white-space: nowrap; overflow: hidden; display: block" class="mb-1">
              <font-awesome-icon icon="cogs" /> {{ i.miner.name }}
            </small>
            <small style="white-space: nowrap; overflow: hidden; display: block" class="mb-1" >
              <font-awesome-icon icon="circle" :class="instancesStatus[i.id]" />&nbsp;
              <span v-if="instancesStatus[i.id] == 'mining'">Running</span>
              <span v-if="instancesStatus[i.id] == 'not_mining'">Not running</span>
              <span v-if="instancesStatus[i.id] == 'configuring'">Configuring...</span>
            </small>
            <small
              v-b-tooltip.hover="minerIdToHost(i.miner.id)"
              style="white-space: nowrap; overflow: hidden; display: block">
              <font-awesome-icon icon="server" /> <code>{{ minerIdToHost(i.miner.id).replace("http://", "").replace("https://", "") }}</code>
            </small>
          </b-collapse>
        </b-list-group-item>
      </b-list-group>
    </b-sidebar>

    <NewInstance
      :streams="streams"
      :miners="miners"
      @new-instance="newInstance"
    />
    <ConfigureInstance
      :miner="miner"
      :host="host"
      @configure-instance="configureInstance"
    />
  </div>
</template>

<script>
import NewInstance from "../modals/NewInstance";
import ConfigureInstance from "../modals/ConfigureInstance";

export default {
  name: "SidebarInstances",
  props: ["streams", "miners", "instances", "instancesStatus"],
  components: {
    NewInstance,
    ConfigureInstance,
  },
  data() {
    return {
      miner: {
        name: null,
      },
      stream: {},
      name: "",
      host: "",
    };
  },
  methods: {
    minerIdToHost(minerId) {
      for (const host in this.miners) {
        for (const miner in this.miners[host]) {
          if (this.miners[host][miner].id == minerId) {
            return host.split("://")[1];
          }
        }
      }
      return "null";
    },
    minerIdToProtocol(minerId) {
      for (const host in this.miners) {
        for (const miner in this.miners[host]) {
          if (this.miners[host][miner].id == minerId) {
            return host.split("://")[0];
          }
        }
      }
      return "null";
    },
    newInstance(event) {
      this.miner = event.miner;
      this.stream = event.stream;
      this.name = event.name;
      this.host = event.host;
      if (this.miner.configurationParameters.length > 0) {
        this.$bvModal.show("configure-instance");
      } else {
        this.configureInstance([]);
      }
    },
    configureInstance(event) {
      this.$emit("create-instance", {
        miner: this.miner,
        stream: this.stream,
        parameterValues: event,
        name: this.name,
        host: this.host,
      });
    },
  },
};
</script>

<style scoped>
.mining {
  color: rgb(0, 204, 0);
}

.not_mining {
  color: rgb(170, 20, 20);
}

.configuring {
  color: rgb(180, 141, 56);
}
</style>