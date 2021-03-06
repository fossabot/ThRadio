<template>
  <div class="StationPage">
    <v-row>
      <v-col cols="12" md="7">
        <v-row no-gutters>
          <v-col class="mb-3" cols="12">
            <v-card>
              <v-app-bar flat color="rgba(0, 0, 0, 0)">
                <v-toolbar-title class="title white--text pl-0">
                  {{ station.name }}
                </v-toolbar-title>

                <v-spacer></v-spacer>

                <v-menu auto bottom light min-width="150">
                  <template #activator="{ on, attrs }">
                    <v-btn dark icon v-bind="attrs" v-on="on">
                      <v-icon>mdi-dots-vertical</v-icon>
                    </v-btn>
                  </template>

                  <v-list>
                    <v-list-item
                      :to="localePath(`/stations/edit/${station._id}`)"
                    >
                      <v-list-item-icon>
                        <v-icon dense> mdi-pencil </v-icon>
                      </v-list-item-icon>
                      <v-list-item-title>{{ $t('edit') }}</v-list-item-title>
                    </v-list-item>
                    <v-list-item v-if="$auth.hasScope('admin')" @click="remove">
                      <v-list-item-icon>
                        <v-icon dense> mdi-delete </v-icon>
                      </v-list-item-icon>
                      <v-list-item-title>{{ $t('remove') }}</v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>
              </v-app-bar>

              <v-list two-line class="pt-0">
                <v-list-item v-if="station.state == 20">
                  <v-list-item-content>
                    <client-only>
                      <vue-plyr :options="options">
                        <audio controls crossorigin playsinline>
                          <source
                            :src="`${$store.getters.config.url_base}/radio/${station.icecast_port}/radio.mp3`"
                            type="audio/mp3"
                          />
                        </audio>
                      </vue-plyr>
                    </client-only>
                  </v-list-item-content>
                </v-list-item>

                <v-divider v-if="station.state == 20"></v-divider>

                <v-list-item v-if="station.description">
                  <v-list-item-icon>
                    <v-icon color="primary"> mdi-card-text </v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>{{
                      $t('description')
                    }}</v-list-item-title>
                    <v-list-item-subtitle>{{
                      station.description
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>

                <v-divider v-if="station.description"></v-divider>

                <v-list-item v-if="station.genre">
                  <v-list-item-icon>
                    <v-icon color="primary">
                      mdi-format-list-bulleted-type
                    </v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('genre') }}</v-list-item-title>
                    <v-list-item-subtitle>{{
                      station.genre
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>

                <v-divider v-if="station.genre"></v-divider>

                <v-list-item>
                  <v-list-item-icon>
                    <v-icon color="primary"> mdi-radio </v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>{{
                      $t('maximum_listeners')
                    }}</v-list-item-title>
                    <v-list-item-subtitle>{{
                      station.listeners
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>

          <v-col cols="12">
            <v-card>
              <v-card-title primary-title>
                {{ $t('statistics') }}
              </v-card-title>

              <v-list two-line>
                <v-list-item>
                  <v-list-item-icon>
                    <v-icon color="primary"> mdi-radio </v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('listeners') }}</v-list-item-title>
                    <v-list-item-subtitle>{{
                      statistics.listeners
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>

                <v-divider></v-divider>

                <v-list-item>
                  <v-list-item-icon>
                    <v-icon color="primary"> mdi-trending-up </v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>{{
                      $t('listeners_peak')
                    }}</v-list-item-title>
                    <v-list-item-subtitle>{{
                      statistics.listeners_peak
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>
        </v-row>
      </v-col>

      <v-col cols="12" md="5">
        <v-card>
          <v-app-bar flat color="rgba(0, 0, 0, 0)">
            <v-toolbar-title class="title white--text pl-0">
              Icecast connection ({{
                station.state == 20 ? $t('running') : $t('stopped')
              }})
            </v-toolbar-title>

            <v-spacer></v-spacer>

            <v-menu auto bottom light min-width="150">
              <template #activator="{ on, attrs }">
                <v-btn dark icon v-bind="attrs" v-on="on">
                  <v-icon>mdi-dots-vertical</v-icon>
                </v-btn>
              </template>

              <v-list>
                <v-list-item v-if="station.state == 0" @click="start">
                  <v-list-item-icon>
                    <v-icon dense> mdi-play-circle-outline </v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>{{ $t('start') }}</v-list-item-title>
                </v-list-item>
                <v-list-item v-if="station.state == 20" @click="restart">
                  <v-list-item-icon>
                    <v-icon dense> mdi-restart </v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>{{ $t('restart') }}</v-list-item-title>
                </v-list-item>
                <v-list-item v-if="station.state == 20" @click="stop">
                  <v-list-item-icon>
                    <v-icon dense> mdi-stop-circle-outline </v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>{{ $t('stop') }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
          </v-app-bar>
          <v-list two-line>
            <v-list-item>
              <v-list-item-icon>
                <v-icon color="primary"> mdi-link-box-variant </v-icon>
              </v-list-item-icon>

              <v-list-item-content>
                <v-list-item-title>Host URL</v-list-item-title>
                <v-list-item-subtitle
                  >{{ $store.getters.config.url_base }}:{{
                    station.icecast_port
                  }}</v-list-item-subtitle
                >
              </v-list-item-content>

              <v-list-item-icon
                @click="
                  copy(
                    `${$store.getters.config.url_base}:${station.icecast_port}`
                  )
                "
              >
                <v-icon>mdi-clipboard-text-multiple</v-icon>
              </v-list-item-icon>
            </v-list-item>

            <v-divider></v-divider>

            <v-list-item>
              <v-list-item-icon>
                <v-icon color="primary"> mdi-open-in-new </v-icon>
              </v-list-item-icon>

              <v-list-item-content>
                <v-list-item-title>{{
                  $t('administration_url')
                }}</v-list-item-title>
                <v-list-item-subtitle
                  >{{ $store.getters.config.url_base }}:{{
                    station.icecast_port
                  }}/admin.html</v-list-item-subtitle
                >
              </v-list-item-content>

              <v-list-item-icon
                @click="
                  copy(
                    `${$store.getters.config.url_base}:${station.icecast_port}/admin.html`
                  )
                "
              >
                <v-icon>mdi-clipboard-text-multiple</v-icon>
              </v-list-item-icon>
            </v-list-item>

            <v-divider></v-divider>

            <v-list-item>
              <v-list-item-icon>
                <v-icon color="primary"> mdi-lock-outline </v-icon>
              </v-list-item-icon>

              <v-list-item-content>
                <v-list-item-title>{{ $t('password') }}</v-list-item-title>
                <v-list-item-subtitle>{{
                  station.icecast_password
                }}</v-list-item-subtitle>
              </v-list-item-content>

              <v-list-item-icon @click="copy(station.icecast_password)">
                <v-icon>mdi-clipboard-text-multiple</v-icon>
              </v-list-item-icon>
            </v-list-item>

            <v-divider></v-divider>

            <v-list-item>
              <v-list-item-icon>
                <v-icon color="primary"> mdi-airplane </v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>{{ $t('port') }}</v-list-item-title>
                <v-list-item-subtitle>{{
                  station.icecast_port
                }}</v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-icon @click="copy(station.icecast_port)">
                <v-icon>mdi-clipboard-text-multiple</v-icon>
              </v-list-item-icon>
            </v-list-item>

            <v-divider></v-divider>

            <v-list-item>
              <v-list-item-icon>
                <v-icon color="primary"> mdi-radio-tower </v-icon>
              </v-list-item-icon>

              <v-list-item-content>
                <v-list-item-title>{{ $t('mount_point') }}</v-list-item-title>
                <v-list-item-subtitle>/radio.mp3</v-list-item-subtitle>
              </v-list-item-content>

              <v-list-item-icon @click="copy('/radio.mp3')">
                <v-icon>mdi-clipboard-text-multiple</v-icon>
              </v-list-item-icon>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>
    </v-row>

    <v-overlay :value="loading">
      <v-progress-circular indeterminate size="64"></v-progress-circular>
    </v-overlay>

    <v-snackbar v-model="snackbar.show" :timeout="3000">
      {{ snackbar.text }}

      <template #action="{ attrs }">
        <v-btn color="pink" text v-bind="attrs" @click="snackbar.show = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

@Component({
  head(this: StationPage): object {
    return {
      title: this.station.name,
    }
  },
})
export default class StationPage extends Vue {
  interval: any
  station: any
  statistics: any
  loading = false
  $copyText: any
  snackbar = {
    show: false,
    text: '',
  }

  options = {
    controls: ['play', 'mute', 'volume'],
  }

  async asyncData({ $axios, params }: any) {
    const station = await $axios.$get(`/api/stations/${params.id}`)
    const statistics = await $axios.$get(
      `/api/stations/statistics/${params.id}`
    )
    return { station, statistics }
  }

  mounted() {
    this.interval = window.setInterval(async () => {
      const statistics = await this.$axios.$get(
        `/api/stations/statistics/${this.station._id}`,
        { progress: false }
      )
      this.statistics = statistics
    }, 10000)
  }

  destroyed() {
    window.clearInterval(this.interval)
  }

  async remove() {
    this.loading = true
    await this.$axios.$delete(`/api/stations/${this.station._id}`)
    this.$router.push('/')
  }

  async copy(text: string) {
    await this.$copyText(text)
    this.snackbar.show = true
    this.snackbar.text = 'Successfully copied'
  }

  async start() {
    this.loading = true
    await this.$axios.$get(`/api/stations/start/${this.station._id}`)
    await this.$nuxt.refresh()
    this.loading = false
  }

  async restart() {
    this.loading = true
    await this.$axios.$get(`/api/stations/restart/${this.station._id}`)
    await this.$nuxt.refresh()
    this.loading = false
  }

  async stop() {
    this.loading = true
    await this.$axios.$get(`/api/stations/stop/${this.station._id}`)
    await this.$nuxt.refresh()
    this.loading = false
  }
}
</script>

<style>
.plyr {
  border-radius: 10px;
}
</style>
