<template>
  <v-data-table
    dense
    class="hide-footer pointer"
    :options.sync="options"
    :server-items-length="totalMatches"
    :headers="headers"
    :items="matches"
    item-key="id"
    @click:row="gotToMatchDetailPage"
    no-data-text="no matches found"
    :footer-props="{ showFirstLastPage: true }"
  >
    <template v-slot:item.map="{ item }">
      <span>{{ $t("mapNames." + item.map) }}</span>
    </template>
    <template v-slot:item.startTime="{ item }">
      <v-tooltip top>
        <template v-slot:activator="{ on }">
          <span v-on="on" class="number-text">
            {{ item.startTime | moment("HH:mm") }}
          </span>
        </template>
        <div class="number-text">
          {{ item.startTime | moment("HH:mm MM.DD.YYYY") }}
        </div>
      </v-tooltip>
    </template>
    <template class="number-text" v-slot:item.duration="{ item }">
      <span class="number-text">{{ getDuration(item) }}</span>
    </template>
    <template v-slot:item.players="{ item }">
      <v-row>
        <v-col cols="5.5">
          <team-match-info
            :not-clickable="true"
            :team="alwaysLeftName ? getPlayerTeam(item) : getWinner(item)"
            left="true"
          ></team-match-info>
        </v-col>
        <v-col cols="1">VS</v-col>
        <v-col cols="5.5">
          <team-match-info
            :not-clickable="true"
            :team="alwaysLeftName ? getOpponentTeam(item) : getLoser(item)"
          ></team-match-info>
        </v-col>
      </v-row>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop, Watch } from "vue-property-decorator";
import { Match, DataTableOptions, Team, PlayerInTeam } from "@/store/typings";
import PlayerMatchInfo from "./PlayerMatchInfo.vue";
import moment from "moment";
import TeamMatchInfo from "@/components/matches/TeamMatchInfo.vue";

@Component({
  components: {
    TeamMatchInfo,
    PlayerMatchInfo
  }
})
export default class MatchesGrid extends Vue {
  @Prop() public value!: Match[];
  @Prop() public totalMatches!: number;
  @Prop() public itemsPerPage!: number;
  @Prop() public alwaysLeftName!: string;

  get matches(): Match[] {
    return this.value;
  }

  public options = {
    itemsPerPage: 100
  } as DataTableOptions;

  @Watch("options", { deep: true })
  public onOptionsChanged() {
    this.$emit("pageChanged", this.options.page);
  }

  public getWinner(match: Match) {
    return match.teams[0];
  }

  public gotToMatchDetailPage(match: Match) {
    this.$router.push({
      path: `/match/${match.id}`
    });
  }

  public getLoser(match: Match) {
    return match.teams[1];
  }

  public getPlayerTeam(match: Match) {
    return match.teams.find((team: Team) =>
      team.players.some(
        (player: PlayerInTeam) => player.battleTag === this.alwaysLeftName
      )
    );
  }

  public getOpponentTeam(match: Match) {
    return match.teams.find(
      (team: Team) =>
        !team.players.some(
          (player: PlayerInTeam) => player.battleTag === this.alwaysLeftName
        )
    );
  }

  public getDuration(match: Match) {
    if (
      !Object.prototype.hasOwnProperty.call(match, "endTime") ||
      !match.endTime
    ) {
      return "ongoing";
    }

    return moment
      .utc(moment.duration(match.durationInSeconds, "seconds").asMilliseconds())
      .format("mm:ss");
  }

  mounted() {
    this.options.itemsPerPage = this.itemsPerPage;
  }

  public headers = [
    {
      text: "Players",
      align: "center",
      sortable: false,
      value: "players",
      width: "600px"
    },
    {
      text: "Map",
      align: "start",
      sortable: false,
      value: "map"
    },
    {
      text: "Start Time",
      align: "end",
      sortable: false,
      value: "startTime",
      width: "150px"
    },
    {
      text: "Duration",
      align: "center",
      sortable: false,
      value: "duration",
      width: "150px"
    }
  ];
}
</script>

<style lang="scss">
.playerCol {
  max-width: 500px;
}
</style>