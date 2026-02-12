<template>
  <v-dialog v-model="communityModal" max-width="850px">
    <v-card>
      <v-card-title>
        <v-icon left>mdi-account-group</v-icon>
        Community
      </v-card-title>

      <v-card-text>
        <v-text-field
          v-model="searchQuery"
          label="Search Sounds"
          prepend-inner-icon="mdi-magnify"
          @keyup.enter="search"
          :loading="loading"
          clearable
        ></v-text-field>

        <v-list two-line>
            <v-list-item v-for="sound in sounds" :key="sound.slug">
                <v-list-item-avatar>
                    <v-img :src="sound.image || require('@/assets/icon.svg')"></v-img>
                </v-list-item-avatar>
                <v-list-item-content>
                    <v-list-item-title>{{ sound.name }}</v-list-item-title>
                    <v-list-item-subtitle>{{ sound.description }}</v-list-item-subtitle>
                </v-list-item-content>
                <v-list-item-action>
                    <v-btn icon @click="preview(sound.sound)">
                        <v-icon>{{ currentPreview === sound.sound ? 'mdi-stop' : 'mdi-play' }}</v-icon>
                    </v-btn>
                </v-list-item-action>
                <v-list-item-action>
                    <v-btn icon @click="download(sound.sound, sound.name)">
                        <v-icon>mdi-download</v-icon>
                    </v-btn>
                </v-list-item-action>
            </v-list-item>
        </v-list>
        
        <div v-if="!loading && sounds.length === 0 && searched" class="text-center">
            No results found.
        </div>

      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text color="primary" @click="close">Close</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
  name: 'CommunityModal',
  data() {
    return {
      searchQuery: '',
      sounds: [] as any[],
      loading: false,
      searched: false,
      currentPreview: null as string | null,
      audioPlayer: new Audio(),
    };
  },
  computed: {
      communityModal: {
          get(): boolean {
              return this.$store.getters.communityModal;
          },
          set(value: boolean) {
              this.$store.commit('setCommunityModal', value);
          }
      }
  },
  methods: {
      async search() {
          if (!this.searchQuery) return;
          this.loading = true;
          this.searched = true;
          this.sounds = [];
          try {
              const res = await window.getCommunitySounds(this.searchQuery);
              const data = JSON.parse(res);
              this.sounds = data.results || [];
          } catch (e) {
              console.error(e);
          } finally {
              this.loading = false;
          }
      },
      preview(url: string) {
          if (this.currentPreview === url) {
              this.audioPlayer.pause();
              this.currentPreview = null;
              return;
          }
          
          this.audioPlayer.src = url;
          this.audioPlayer.play();
          this.currentPreview = url;
          this.audioPlayer.onended = () => {
              this.currentPreview = null;
          };
      },
      async download(url: string, name: string) {
          try {
              const success = await window.downloadCommunitySound(url, name);
              if (success) {
                  this.$toast.success(`Downloaded ${name}`);
                  this.$store.dispatch('refreshTab');
              } else {
                  this.$toast.error('Failed to download');
              }
          } catch (e) {
              console.error(e);
              this.$toast.error('Error downloading');
          }
      },
      close() {
          this.$store.commit('setCommunityModal', false);
          this.audioPlayer.pause();
          this.currentPreview = null;
      }
  }
});
</script>
