<template>
<v-app id="nyaa">
  <v-card dark="dark"
          class="nyaa-header"
          style="background: url('./assets/nyaabg.jpg') 50% -437px no-repeat;">
    <template v-if="initialUpdatePercents"
              fixed="fixed"
              bottom="bottom"
              id="hideandseek">
      <v-progress-linear background-color="transparent"
                         color="accent"
                         height="1"
                         v-model="initialUpdatePercents"></v-progress-linear>
    </template>

    <v-layout row="row"
              wrap="wrap"
              text-xs-center="text-xs-center">
      <v-flex xs12
              class="mt-5 mb-3">
        <v-btn flat="flat"
               color="blue"
               class="display-1"
               style="text-transform: none;"
               @click="updateAnimeInitial('forced')">
          <v-tooltip bottom="bottom">
            <span slot="activator">
              <b>NyaaOngoMan</b>
              =( ^^,)=
            </span>
            <span>{{this.qCycleInWorkWith || 'Forced-update-desu!!'}}</span>
          </v-tooltip>
        </v-btn>
      </v-flex>
      <v-flex xs10="xs10"
              offset-xs1="offset-xs1">
        <v-tabs color="transparent"
                v-model="currentTab"
                centered="centered">
          <v-tabs-slider color="yellow"></v-tabs-slider>
          <v-tab v-for="tab in tabs"
                 :key="tab.title"
                 @click="switchTabs(tab)">
            {{ tab.title }}
          </v-tab>
        </v-tabs>
      </v-flex>
    </v-layout>

    <transition name="slide-y-transition">
      <v-card v-show="isTabActive('search')"
              class="search-block"
              flat
              color="transparent">
        <v-layout row="row"
                  wrap="wrap"
                  text-xs-center="text-xs-center">
          <v-flex xs10="xs10"
                  offset-xs1="offset-xs1"
                  sm6="sm6"
                  offset-sm1="offset-sm1">

            <v-tooltip bottom="bottom">
              <v-text-field slot="activator"
                            clearable="clearable"
                            hide-details="hide-details"
                            name="add-new"
                            prepend-icon="video_library"
                            label="Add a new anime"
                            @keyup.enter="searchAnime"
                            v-model="q"></v-text-field>
              <span>Press Enter to search</span>
            </v-tooltip>
          </v-flex>
          <v-flex xs10="xs10"
                  offset-xs1="offset-xs1"
                  sm2="sm2"
                  offset-sm0="offset-sm0">
            <v-select box="box"
                      multiple="multiple"
                      label="Quality"
                      :items="availableQuality"
                      v-model="searchQuality"
                      @input="onChangeQuality"
                      append-icon="switch_video"
                      hide-details="hide-details"></v-select>
          </v-flex>
          <v-flex xs10="xs10"
                  offset-xs1="offset-xs1"
                  sm2="sm2"
                  offset-sm0="offset-sm0">
            <v-btn @click="addAnime"
                   color="primary"
                   class="btn--search">
              <v-icon>playlist_add</v-icon>ADD
            </v-btn>
          </v-flex>
        </v-layout>
      </v-card>
    </transition>
  </v-card>

  <v-tabs-items v-model="currentTab">
    <v-tab-item v-for="tab in tabs"
                :key="tab.title">

      <v-data-table v-if="tab.title === 'all'"
                    :headers="getAnimeTableHeaders('queries')"
                    :items="getAnime('queries')"
                    :pagination.sync="paginationQueries"
                    :rows-per-page-items="paginationQueries.rppi"
                    :rows-per-page-text="'Items per page: '"
                    class="elevation-1">
        <template slot="items"
                  slot-scope="props">
          <td>{{ props.item.q }}</td>
          <td class="justify-center layout px-0">
            <v-btn icon="icon"
                   flat="flat"
                   class="mx-0"
                   color="red"
                   @click="removeAnime(props.item)">
              <v-icon>delete_forever</v-icon>
            </v-btn>
            <v-btn icon="icon"
                   flat="flat"
                   class="mx-0"
                   @click="markAll(props.item)">
              <v-icon color="teal">collections_bookmark</v-icon>
            </v-btn>
          </td>
        </template>
        <template slot="no-data">
          No results
        </template>
      </v-data-table>

      <v-layout v-if="tab.title === 'all' && dev()"
                row="row"
                wrap="wrap"
                text-xs-center="text-xs-center">
        <v-flex xs10="xs10"
                offset-xs1="offset-xs1">
          <v-btn color="error"
                 @click="resetAnimeDB">
            <v-icon dark="dark"
                    left="left">not_interested</v-icon>Reset Anime DB
          </v-btn>
        </v-flex>
      </v-layout>

      <v-data-table :headers="getAnimeTableHeaders(tab.title)"
                    :items="getAnime(tab.title)"
                    :pagination.sync="paginationAllAnime"
                    :rows-per-page-items="paginationAllAnime.rppi"
                    :rows-per-page-text="'Items per page: '"
                    class="elevation-10">
        <template slot="items"
                  slot-scope="props"
                  class="meow">
          <td :class="{ downloaded: props.item.downloaded && props.item.new }">{{ props.item.title | stripOutSubsTeamFromTitle }}</td>
          <td :class="{ downloaded: props.item.downloaded && props.item.new }"
              class="text-xs-left">{{ props.item.size }}</td>
          <td :class="{ downloaded: props.item.downloaded && props.item.new }"
              class="text-xs-left">
            <v-tooltip left="left">
              <a :href="props.item.link.replace('.torrent', '').replace('download', 'view')"
                 slot="activator"
                 target="_blank"
                 flat="flat"
                 class="mx-0"
                 color="blue">
                {{ props.item.time | humanizeTime }}
              </a>
              <span>{{ props.item.timesince }}</span>
            </v-tooltip>
          </td>
          <td :class="{ downloaded: props.item.downloaded && props.item.new }"
              v-if="!isTabActive('search')"
              class="justify-center layout px-0">
            <v-tooltip left="left">
              <v-btn slot="activator"
                     icon="icon"
                     flat="flat"
                     class="mx-0"
                     color="blue"
                     @click="downloadAndOpenTorrent(props.item)">
                <v-icon>file_download</v-icon>
              </v-btn>
              <span>Mark as downloaded, and open .torrent</span>
            </v-tooltip>
            <v-tooltip bottom="bottom">
              <v-btn slot="activator"
                     icon="icon"
                     flat="flat"
                     class="mx-0"
                     color="blue"
                     @click="markDownloaded(props.item, false)">
                <v-icon>undo</v-icon>
              </v-btn>
              <span>Unmark as downloaded</span>
            </v-tooltip>
            <v-tooltip bottom="bottom">
              <v-btn slot="activator"
                     icon="icon"
                     flat="flat"
                     class="mx-0"
                     @click="toggleMark(props.item)">
                <v-icon v-if="props.item.new"
                        color="teal">done</v-icon>
                <v-icon v-else="v-else"
                        color="teal">done_all</v-icon>
              </v-btn>
              <span>(un)Mark as watched</span>
            </v-tooltip>
          </td>
        </template>
      </v-data-table>

    </v-tab-item>
  </v-tabs-items>
</v-app>
</template>
<script src="./App.js"></script>
<style src="./App.css"></style>
