<template>
  <v-card variant="outlined" :loading="loading">
    <v-card-title
      class="d-flex align-center"
      @click="toggleExpanded"
      style="cursor: pointer"
      v-ripple
    >
      <v-icon start>mdi-chat</v-icon>
      Chatwoot
      <v-spacer></v-spacer>
      <v-btn
        size="small"
        icon
        :disabled="loading"
        variant="tonal"
        @click.stop="toggleExpanded"
        :style="{ transform: expanded ? 'rotate(180deg)' : '' }"
      >
        <v-icon>mdi-chevron-down</v-icon>
      </v-btn>
    </v-card-title>
    <v-card-text v-if="expanded">
      <v-alert v-if="error" type="error" class="mb-3">
        {{ error }}
      </v-alert>

      <v-form v-model="valid">
        <v-text-field
          v-model="chatwootData.url"
          label="URL"
          :disabled="loading"
          outlined
          dense
          hide-details="auto"
          class="mb-3"
          :rules="[
            (url) => {
              if (!url) return 'URL é obrigatório';
              if (!url.startsWith('http'))
                return 'URL deve começar com http ou https';
              return true;
            },
          ]"
        />

        <div class="d-flex gap-4 flex-wrap">
          <div class="flex-grow-1">
            <v-text-field
              v-model="chatwootData.account_id"
              label="ID da conta"
              :disabled="loading"
              outlined
              dense
              hide-details="auto"
              class="mb-3"
              :rules="[
                (account_id) => {
                  if (!account_id) return 'ID da conta é obrigatório';
                  return true;
                },
              ]"
            />
          </div>
          <div class="flex-grow-1">
            <v-text-field
              v-model="chatwootData.token"
              label="Token da conta"
              :disabled="loading"
              outlined
              dense
              hide-details="auto"
              class="mb-3"
              :rules="[
                (token) => {
                  if (!token) return 'Token é obrigatório';
                  return true;
                },
              ]"
            />
          </div>
        </div>
        <div class="d-flex gap-x-4 flex-wrap">
          <div>
            <v-checkbox
              v-model="chatwootData.sign_msg"
              label="Assinar mensagens"
              :disabled="loading"
              hide-details
              class="mb-3"
              density="compact"
            />
          </div>
          <div>
            <v-checkbox
              v-model="chatwootData.reopen_conversation"
              label="Reabrir conversa"
              :disabled="loading"
              hide-details
              class="mb-3"
              density="compact"
            />
          </div>

          <div>
            <v-checkbox
              v-model="chatwootData.conversation_pending"
              label="Conversa pendente"
              :disabled="loading"
              hide-details
              class="mb-3"
              density="compact"
            />
          </div>
        </div>
      </v-form>
    </v-card-text>
    <v-card-actions v-if="expanded">
      <v-switch
        v-model="chatwootData.enabled"
        label="Habilitado"
        color="primary"
        :disabled="loading"
        hide-details
      ></v-switch>
      <v-btn variant="text" @click="chatwootConfig">
        Como configurar o chatwoot?
      </v-btn>
      <v-spacer></v-spacer>
      <v-btn
        :disabled="
          !valid ||
          JSON.stringify(chatwootData) === JSON.stringify(defaultChatwootData)
        "
        :loading="loading"
        color="primary"
        @click="saveChatwoot"
        variant="tonal"
      >
        Salvar
      </v-btn>
    </v-card-actions>
  </v-card>
  <chatwoot-config :instance="instance" ref="chatwootConfig" />
</template>

<script>
import ChatwootConfig from "@/components/modal/ChatwootConfig.vue";
import instanceController from "@/services/instanceController";

export default {
  name: "InstanceChatwoot",
  props: {
    instance: {
      type: Object,
      required: true,
    },
  },
  data: () => ({
    expanded: false,
    loading: false,
    error: false,
    valid: false,
    chatwootData: {
      enabled: false,
      url: "",
      account_id: "",
      token: "",
      sign_msg: true,
      reopen_conversation: true,
      conversation_pending: false,
    },
    defaultChatwootData: {
      enabled: false,
      url: "",
      account_id: "",
      token: "",
      sign_msg: true,
      reopen_conversation: true,
      conversation_pending: false,
    },
  }),
  methods: {
    toggleExpanded() {
      if (this.loading) return;
      this.expanded = !this.expanded;
    },
    chatwootConfig() {
      this.$refs.chatwootConfig.open();
    },
    async saveChatwoot() {
      try {
        this.loading = true;
        this.error = false;
        await instanceController.chatwoot.set(
          this.instance.instance.instanceName,
          {
            ...this.chatwootData,
            url: this.chatwootData.url.trim().replace(/\/$/, ""),
          }
        );
        this.defaultChatwootData = Object.assign({}, this.chatwootData);
      } catch (e) {
        this.error = e.message?.message || e.message || e;
      } finally {
        this.loading = false;
      }
    },
    async loadChatwoot() {
      try {
        this.loading = true;
        this.error = false;
        const chatwootData = await instanceController.chatwoot.get(
          this.instance.instance.instanceName
        );
        this.chatwootData = Object.assign({}, chatwootData || {});
        this.defaultChatwootData = Object.assign({}, chatwootData || {});
      } catch (e) {
        this.error = e.message?.message || e.message || e;
      } finally {
        this.loading = false;
      }
    },
  },
  watch: {
    expanded(expanded) {
      if (expanded) this.loadChatwoot();
    },
  },
  components: { ChatwootConfig },
};
</script>

<style></style>
